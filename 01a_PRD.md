# NFC Capacitor Plugin — Product Requirements Document (PRD)

## 1. **Project Overview**

Build a Capacitor plugin that allows reading, writing, and erasing NFC tags on both **iOS** and **Android**, supporting the most important NFC tag types and covering typical NFC use cases in apps.

---

## 2. **Objectives**

* Provide a unified Capacitor plugin API for NFC functionality.
* Support common NFC tag types on iOS and Android.
* Enable starting/stopping scanning, reading tag content, writing data, and erasing tags.
* Handle platform-specific NFC APIs transparently.
* Provide fallback for unsupported platforms (e.g., web).
* Ensure security and user permissions management.

---

## 3. **Supported Platforms**

* Android (API level 19+)
* iOS (iOS 13+)
* Web (stub implementation that signals unsupported)

---

## 4. **Supported NFC Tag Types**

### Android

* NDEF (NFC Data Exchange Format)
* MIFARE Classic (where supported)
* MIFARE Ultralight
* ISO-DEP (ISO 14443-4)
* NFC-F (FeliCa)
* NFC-V (ISO 15693)

### iOS

* NDEF (read/write)
* ISO7816 (ISO-DEP)
* FeliCa (NFC-F)
* MIFARE (Read-only support)

---

## 5. **Core Features**

### 5.1 Plugin API

| Method            | Description                           | Parameters         | Returns                  |
| ----------------- | ------------------------------------- | ------------------ | ------------------------ |
| `isAvailable()`   | Check if NFC is available and enabled | None               | `{ available: boolean }` |
| `startScanning()` | Start scanning for NFC tags           | None               | `void`                   |
| `stopScanning()`  | Stop scanning for NFC tags            | None               | `void`                   |
| `writeTag(data)`  | Write data to a detected NFC tag      | `{ data: string }` | `void`                   |
| `eraseTag()`      | Erase the detected tag (if supported) | None               | `void`                   |

### 5.2 Event Callbacks / Listeners

* `onTagDiscovered` — Provides tag info and payload when a tag is scanned.
* `onError` — Reports errors during scanning or writing.
* Optional: `onTagLost` (when tag is removed)

---

## 6. **Platform Implementation Details**

### 6.1 Android

* Use `NfcAdapter`, `NfcManager`, and handle intents via `onNewIntent`.
* Handle foreground dispatch for scanning.
* Support reading NDEF messages and other tag tech.
* Writing to NDEF tags.
* Erasing tags if supported.
* Permissions: `NFC` in manifest, runtime checks.

### 6.2 iOS

* Use `NFCNDEFReaderSession` and related Core NFC classes.
* Start session for scanning.
* Read and write NDEF tags.
* Erasing tags if possible.
* User permissions are prompted on session start.

### 6.3 Web

* No NFC support.
* Provide stub implementation that rejects calls.

---

## 7. **Error Handling & Edge Cases**

* Detect and report when NFC is not supported or disabled.
* Handle user cancellation on iOS.
* Handle unsupported tag types gracefully.
* Timeouts on scanning.
* Conflicts when multiple tags are in range.

---

## 8. **Security & Privacy**

* Follow platform guidelines for user permissions.
* Do not store or transmit tag data outside the device unless explicitly handled by the app.
* Ensure all sensitive operations require explicit user action.

---

## 9. **Testing**

* Unit tests for JS API surface.
* Manual testing on real devices (Android and iOS).
* Edge cases:

  * Multiple tags.
  * Unsupported tag formats.
  * Permission denial.

---

## 10. **Documentation**

* README with setup and usage instructions.
* API reference.
* Sample app integration code snippets.

---

## 11. **Milestones**

| Milestone              | Description                                          | Estimated Time |
| ---------------------- | ---------------------------------------------------- | -------------- |
| Plugin scaffold setup  | Create basic Capacitor plugin scaffold               | 1 day          |
| Android read/write NFC | Implement scanning, reading, writing on Android      | 3-4 days       |
| iOS read/write NFC     | Implement scanning, reading, writing on iOS          | 3-4 days       |
| API event handling     | Add event listeners for tag discovery/errors         | 1-2 days       |
| Erasing tag support    | Implement tag erasing if supported on both platforms | 2 days         |
| Web fallback           | Stub implementation for web platform                 | 0.5 day        |
| Testing and bug fixing | Full manual and unit testing                         | 2-3 days       |
| Documentation          | Write documentation and sample code                  | 1-2 days       |

---

## 12. **Future Enhancements (Post MVP)**

* Support for advanced tag types or vendor-specific tags.
* Batch reading/writing multiple records.
* NFC peer-to-peer communication.
* Integration with background NFC scanning (if supported).
* UI components for scanning status and errors.

---

