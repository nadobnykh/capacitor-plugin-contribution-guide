# Android Native Code Implementation — Step-by-Step

---

## 1. Setup and Permissions

### 1.1 Open Android folder in Android Studio

* Navigate to your plugin's `android` folder.
* Open it in Android Studio for native development.

### 1.2 Add NFC permissions and features in `AndroidManifest.xml`

```xml
<uses-permission android:name="android.permission.NFC" />

<uses-feature android:name="android.hardware.nfc" android:required="true" />
```

* This ensures your app declares NFC usage and requires NFC hardware.

---

## 2. Plugin Class Setup

Create or update the main plugin class (e.g., `NFCPlugin.kt`) under:

`android/src/main/java/com/yourdomain/nfcplugin/NFCPlugin.kt`

This will extend `Plugin` from Capacitor and implement NFC handling.

---

## 3. Initialize NFC Adapter and Manager

```kotlin
import android.content.Intent
import android.nfc.NfcAdapter
import android.nfc.Tag
import android.nfc.tech.Ndef
import android.nfc.tech.MifareClassic
import android.nfc.tech.MifareUltralight
import android.os.Bundle
import com.getcapacitor.Plugin
import com.getcapacitor.PluginCall
import com.getcapacitor.PluginMethod
import com.getcapacitor.annotation.CapacitorPlugin
import com.getcapacitor.JSObject

@CapacitorPlugin(name = "NFCPlugin")
class NFCPlugin : Plugin() {

    private var nfcAdapter: NfcAdapter? = null
    private var currentCall: PluginCall? = null

    override fun load() {
        nfcAdapter = NfcAdapter.getDefaultAdapter(context)
    }
}
```

---

## 4. Check NFC Availability

Add a method to check NFC support and if enabled:

```kotlin
@PluginMethod
fun isAvailable(call: PluginCall) {
    val available = nfcAdapter != null && nfcAdapter!!.isEnabled
    val ret = JSObject()
    ret.put("available", available)
    call.resolve(ret)
}
```

---

## 5. Setup Foreground Dispatch for Scanning

### 5.1 Start scanning by enabling foreground dispatch:

```kotlin
@PluginMethod
fun startScanning(call: PluginCall) {
    if (nfcAdapter == null) {
        call.reject("NFC not supported on this device")
        return
    }

    currentCall = call

    val intent = Intent(context, this.activity.javaClass).apply {
        addFlags(Intent.FLAG_ACTIVITY_SINGLE_TOP)
    }
    val pendingIntent = PendingIntent.getActivity(
        context,
        0,
        intent,
        PendingIntent.FLAG_MUTABLE or PendingIntent.FLAG_UPDATE_CURRENT
    )
    val filters = arrayOfNulls<IntentFilter>(1)
    filters[0] = IntentFilter(NfcAdapter.ACTION_TAG_DISCOVERED)

    val techList = arrayOf(
        arrayOf(Ndef::class.java.name),
        arrayOf(MifareClassic::class.java.name),
        arrayOf(MifareUltralight::class.java.name)
        // Add more techs as needed
    )

    nfcAdapter?.enableForegroundDispatch(activity, pendingIntent, filters, techList)
    call.resolve()
}
```

### 5.2 Stop scanning by disabling foreground dispatch:

```kotlin
@PluginMethod
fun stopScanning(call: PluginCall) {
    nfcAdapter?.disableForegroundDispatch(activity)
    currentCall = null
    call.resolve()
}
```

---

## 6. Handle Incoming NFC Intents in `onNewIntent`

Override the `handleOnNewIntent` method to process NFC tags when discovered:

```kotlin
override fun handleOnNewIntent(intent: Intent) {
    super.handleOnNewIntent(intent)

    if (intent.action == NfcAdapter.ACTION_TAG_DISCOVERED ||
        intent.action == NfcAdapter.ACTION_NDEF_DISCOVERED ||
        intent.action == NfcAdapter.ACTION_TECH_DISCOVERED) {

        val tag: Tag? = intent.getParcelableExtra(NfcAdapter.EXTRA_TAG)
        if (tag != null) {
            processTag(tag)
        }
    }
}
```

---

## 7. Process NFC Tag and Read NDEF Data

Implement `processTag` function to parse the tag and send data back to JS:

```kotlin
private fun processTag(tag: Tag) {
    val ndef = Ndef.get(tag)

    if (ndef != null) {
        try {
            ndef.connect()
            val ndefMessage = ndef.ndefMessage
            val records = ndefMessage?.records
            val messages = records?.map { record ->
                // Convert record payload to readable text
                String(record.payload)
            } ?: listOf()

            val ret = JSObject()
            ret.put("tagId", tag.id.toHexString())
            ret.put("messages", messages)

            notifyListeners("onTagDiscovered", ret)
            currentCall?.resolve(ret)

            ndef.close()
        } catch (e: Exception) {
            val ret = JSObject()
            ret.put("error", e.message)
            notifyListeners("onError", ret)
            currentCall?.reject(e.message)
        }
    } else {
        // Handle tags without NDEF support
        val ret = JSObject()
        ret.put("tagId", tag.id.toHexString())
        ret.put("message", "Non-NDEF tag detected")
        notifyListeners("onTagDiscovered", ret)
        currentCall?.resolve(ret)
    }
}
```

Helper extension function for tag ID hex:

```kotlin
fun ByteArray.toHexString() = joinToString(separator = "") { String.format("%02X", it) }
```

---

## 8. Write to NFC Tags

Add a `writeTag` method to write NDEF data to the tag:

```kotlin
@PluginMethod
fun writeTag(call: PluginCall) {
    val data = call.getString("data") ?: run {
        call.reject("Data is required")
        return
    }

    val tag = currentTag // You need to store the last detected tag
    if (tag == null) {
        call.reject("No tag detected")
        return
    }

    val ndef = Ndef.get(tag)

    if (ndef != null) {
        try {
            ndef.connect()

            val record = NdefRecord.createTextRecord("en", data)
            val message = NdefMessage(arrayOf(record))

            ndef.writeNdefMessage(message)
            ndef.close()

            call.resolve()
        } catch (e: Exception) {
            call.reject("Failed to write tag: ${e.message}")
        }
    } else {
        call.reject("Tag is not NDEF compatible")
    }
}
```

Note: You’ll need to track the current tag detected during scanning (`currentTag`) to enable writing.

---

## 9. Erase NFC Tag

You can erase the tag by writing an empty NDEF message:

```kotlin
@PluginMethod
fun eraseTag(call: PluginCall) {
    val tag = currentTag
    if (tag == null) {
        call.reject("No tag detected")
        return
    }

    val ndef = Ndef.get(tag)
    if (ndef != null) {
        try {
            ndef.connect()
            val emptyMessage = NdefMessage(arrayOf())
            ndef.writeNdefMessage(emptyMessage)
            ndef.close()
            call.resolve()
        } catch (e: Exception) {
            call.reject("Failed to erase tag: ${e.message}")
        }
    } else {
        call.reject("Tag is not NDEF compatible")
    }
}
```

---

## 10. Final Notes and Next Steps

* Make sure to handle lifecycle events to properly enable and disable foreground dispatch.
* Consider thread safety and error handling thoroughly.
* Expand support for other tag types (MifareClassic, Ultralight, ISO-DEP) as needed.
* Integrate Capacitor events for JS listeners to react in real-time.
* Test on real Android devices with a variety of NFC tags.

---
