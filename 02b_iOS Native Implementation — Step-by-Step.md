# iOS Native Implementation — Step-by-Step

---

## 1. Setup and Permissions

### 1.1 Open iOS folder in Xcode

* Open the `ios` directory of your Capacitor plugin project in Xcode.

### 1.2 Enable NFC capability

* Select your project target.
* Go to the **Signing & Capabilities** tab.
* Click **+ Capability** and add **Near Field Communication Tag Reading**.

### 1.3 Add usage description in `Info.plist`

Add a key to request user permission for NFC access:

```xml
<key>NFCReaderUsageDescription</key>
<string>We use NFC to scan and write NFC tags.</string>
```

---

## 2. Import Core NFC Framework

Add this import in your main plugin Swift file (`NFCPlugin.swift`):

```swift
import CoreNFC
import Capacitor
```

---

## 3. Define Plugin Class and Protocols

Create the main class that extends `CAPPlugin` and adopts the required Core NFC delegate protocols:

```swift
@objc(NFCPlugin)
public class NFCPlugin: CAPPlugin, NFCNDEFReaderSessionDelegate {

    var nfcSession: NFCNDEFReaderSession?
    var currentCall: CAPPluginCall?

    // Store the detected tag for writing
    var detectedTag: NFCNDEFTag?
}
```

---

## 4. Check NFC Availability

Add a method to check if NFC reading is available on the device:

```swift
@objc func isAvailable(_ call: CAPPluginCall) {
    let available = NFCNDEFReaderSession.readingAvailable
    call.resolve([
        "available": available
    ])
}
```

---

## 5. Start Scanning

Implement `startScanning` to begin an NFC reading session:

```swift
@objc func startScanning(_ call: CAPPluginCall) {
    guard NFCNDEFReaderSession.readingAvailable else {
        call.reject("NFC is not available on this device")
        return
    }
    
    currentCall = call
    
    nfcSession = NFCNDEFReaderSession(delegate: self, queue: nil, invalidateAfterFirstRead: false)
    nfcSession?.alertMessage = "Hold your iPhone near an NFC tag to scan."
    nfcSession?.begin()
}
```

---

## 6. Stop Scanning

Add `stopScanning` to invalidate the current session:

```swift
@objc func stopScanning(_ call: CAPPluginCall) {
    nfcSession?.invalidate()
    nfcSession = nil
    currentCall?.resolve()
}
```

---

## 7. Delegate Methods for Tag Detection and Session Handling

Implement the required delegate methods:

### 7.1 Tag Detection

```swift
public func readerSession(_ session: NFCNDEFReaderSession, didDetectNDEFs messages: [NFCNDEFMessage]) {
    // Not used here; we handle individual tags in didDetect tags method below
}
```

### 7.2 Did Detect Tags (iOS 13+)

```swift
@available(iOS 13.0, *)
public func readerSession(_ session: NFCNDEFReaderSession, didDetect tags: [NFCNDEFTag]) {
    guard let tag = tags.first else { return }
    
    // Connect to the tag
    session.connect(to: tag) { (error: Error?) in
        if let error = error {
            session.invalidate(errorMessage: "Connection failed: \(error.localizedDescription)")
            self.notifyError(error.localizedDescription)
            return
        }
        
        // Query tag NDEF status
        tag.queryNDEFStatus { (status, capacity, error) in
            if let error = error {
                session.invalidate(errorMessage: "Failed to query tag: \(error.localizedDescription)")
                self.notifyError(error.localizedDescription)
                return
            }
            
            if status == .notSupported {
                session.invalidate(errorMessage: "Tag is not NDEF compliant")
                self.notifyError("Tag is not NDEF compliant")
                return
            }
            
            // Read NDEF message
            tag.readNDEF { (ndefMessage, error) in
                if let error = error {
                    session.invalidate(errorMessage: "Failed to read tag: \(error.localizedDescription)")
                    self.notifyError(error.localizedDescription)
                    return
                }
                
                var recordsArray: [[String: Any]] = []
                
                if let ndefMessage = ndefMessage {
                    for record in ndefMessage.records {
                        let payloadText = String(data: record.payload, encoding: .utf8) ?? ""
                        let recordDict: [String: Any] = [
                            "type": record.typeNameFormat.rawValue,
                            "identifier": record.identifier,
                            "payload": payloadText
                        ]
                        recordsArray.append(recordDict)
                    }
                }
                
                // Save tag for write/erase
                self.detectedTag = tag
                
                // Notify JS layer
                let ret: [String: Any] = [
                    "messages": recordsArray
                ]
                self.notifyListeners("onTagDiscovered", data: ret)
                self.currentCall?.resolve(ret)
            }
        }
    }
}
```

---

## 8. Session Invalidated Delegate

Handle session invalidation and errors:

```swift
public func readerSession(_ session: NFCNDEFReaderSession, didInvalidateWithError error: Error) {
    let nsError = error as NSError
    
    if nsError.code != NFCReaderError.readerSessionInvalidationErrorFirstNDEFTagRead.rawValue {
        notifyError(error.localizedDescription)
    }
    
    currentCall = nil
    nfcSession = nil
}
```

---

## 9. Write to Tag

Implement the write method:

```swift
@objc func writeTag(_ call: CAPPluginCall) {
    guard let tag = detectedTag else {
        call.reject("No NFC tag detected")
        return
    }
    
    guard let data = call.getString("data") else {
        call.reject("Data is required")
        return
    }
    
    let payload = NFCNDEFPayload.wellKnownTypeTextPayload(string: data, locale: Locale(identifier: "en"))!
    let message = NFCNDEFMessage(records: [payload])
    
    nfcSession?.connect(to: tag) { error in
        if let error = error {
            call.reject("Failed to connect to tag: \(error.localizedDescription)")
            return
        }
        
        tag.writeNDEF(message) { error in
            if let error = error {
                call.reject("Failed to write tag: \(error.localizedDescription)")
                return
            }
            
            call.resolve()
        }
    }
}
```

---

## 10. Erase Tag

Erase the tag by writing an empty NDEF message:

```swift
@objc func eraseTag(_ call: CAPPluginCall) {
    guard let tag = detectedTag else {
        call.reject("No NFC tag detected")
        return
    }
    
    let emptyMessage = NFCNDEFMessage(records: [])
    
    nfcSession?.connect(to: tag) { error in
        if let error = error {
            call.reject("Failed to connect to tag: \(error.localizedDescription)")
            return
        }
        
        tag.writeNDEF(emptyMessage) { error in
            if let error = error {
                call.reject("Failed to erase tag: \(error.localizedDescription)")
                return
            }
            
            call.resolve()
        }
    }
}
```

---

## 11. Helper: Notify Errors to JS

```swift
func notifyError(_ message: String) {
    let ret: [String: Any] = ["error": message]
    notifyListeners("onError", data: ret)
    currentCall?.reject(message)
}
```

---

## 12. Final Notes

* NFC reading requires a real device (not a simulator).
* The session invalidates after errors or when stopped.
* Make sure to manage the session lifecycle correctly.
* Test on iOS devices with various NFC tag types.

---

