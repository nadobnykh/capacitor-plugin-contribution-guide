# Step-by-Step Example App to Test Your NFC Plugin

---

## 1. Create a new React + Capacitor app

If you don’t have one yet, create it:

```bash
npm create vite@latest nfc-example -- --template react-ts
cd nfc-example
npm install
```

Initialize Capacitor:

```bash
npm install @capacitor/core @capacitor/cli
npx cap init nfc-example com.example.nfc
```

Add iOS and Android platforms:

```bash
npx cap add ios
npx cap add android
```

---

## 2. Install your custom NFC plugin

Assuming your plugin is local (e.g. in `../my-nfc-plugin`), install it via npm link or direct path:

```bash
npm install ../my-nfc-plugin
```

---

## 3. Use the plugin in your React app

Edit `src/App.tsx` with this minimal UI and logic:

```tsx
import React, { useEffect, useState } from "react";
import { NFC } from "my-nfc-plugin";

function App() {
  const [available, setAvailable] = useState(false);
  const [logs, setLogs] = useState<string[]>([]);
  const [scanning, setScanning] = useState(false);

  useEffect(() => {
    NFC.isAvailable().then(({ available }) => setAvailable(available));

    const tagListener = NFC.addListener("onTagDiscovered", (tag) => {
      setLogs((logs) => [...logs, "Tag: " + JSON.stringify(tag)]);
    });

    const errorListener = NFC.addListener("onError", (error) => {
      setLogs((logs) => [...logs, "Error: " + JSON.stringify(error)]);
    });

    return () => {
      tagListener.then(l => l.remove());
      errorListener.then(l => l.remove());
    };
  }, []);

  async function startScan() {
    try {
      await NFC.startScanning();
      setScanning(true);
      setLogs((logs) => [...logs, "Started scanning..."]);
    } catch (e) {
      setLogs((logs) => [...logs, "Error starting scan: " + e]);
    }
  }

  async function stopScan() {
    try {
      await NFC.stopScanning();
      setScanning(false);
      setLogs((logs) => [...logs, "Stopped scanning."]);
    } catch (e) {
      setLogs((logs) => [...logs, "Error stopping scan: " + e]);
    }
  }

  async function writeTag() {
    try {
      await NFC.writeTag({ data: "Hello from Capacitor NFC!" });
      setLogs((logs) => [...logs, "Tag written successfully"]);
    } catch (e) {
      setLogs((logs) => [...logs, "Write error: " + e]);
    }
  }

  async function eraseTag() {
    try {
      await NFC.eraseTag();
      setLogs((logs) => [...logs, "Tag erased successfully"]);
    } catch (e) {
      setLogs((logs) => [...logs, "Erase error: " + e]);
    }
  }

  return (
    <div style={{ padding: 20 }}>
      <h1>Capacitor NFC Plugin Demo</h1>
      <p>NFC Available: {available ? "Yes" : "No"}</p>
      <button onClick={startScan} disabled={!available || scanning}>
        Start Scanning
      </button>
      <button onClick={stopScan} disabled={!scanning}>
        Stop Scanning
      </button>
      <button onClick={writeTag} disabled={!scanning}>
        Write Tag
      </button>
      <button onClick={eraseTag} disabled={!scanning}>
        Erase Tag
      </button>

      <h2>Logs:</h2>
      <pre style={{ whiteSpace: "pre-wrap" }}>
        {logs.length > 0 ? logs.join("\n") : "No logs yet."}
      </pre>
    </div>
  );
}

export default App;
```

---

## 4. Sync native projects and run


### ✅ Add the NFC Plugin to Vite Project

Update `src/App.tsx` and create `src/definitions.ts` and `src/index.ts` as explained earlier. Then, wire them up to use your plugin.

---

### ✅ Sync Capacitor Plugins and Code

```bash
npx cap sync
```

> This copies native plugin code into `ios` and `android` folders.

---

### ✅ Run on a Real Device

Build and run on your device (not simulator) with NFC support.

### For iOS:

```bash
npx cap open ios
```

* Open the Xcode project.
* Enable "NFC Tag Reading" capability.
* Build & run on a real iPhone.

### For Android:

```bash
npx cap open android
```

* Open Android Studio.
* Grant `NFC` permission in `AndroidManifest.xml`.
* Build & run on a real Android device.

---

## 5. Testing workflow

* Click **Start Scanning**, bring your device near NFC tags.
* When a tag is read, the details appear in the logs.
* Use **Write Tag** and **Erase Tag** buttons to modify the detected tag.
* Use **Stop Scanning** to end the session.

---


Here are the **mandatory changes** you need to make to both **AndroidManifest.xml** and your **Xcode project settings** to support NFC properly on real devices.

---

## ✅ Android Setup: `AndroidManifest.xml`

Edit the file:

```
android/app/src/main/AndroidManifest.xml
```

### 🔧 Add NFC Permissions & Features

Add these **inside the `<manifest>` tag**, but **outside `<application>`**:

```xml
<uses-permission android:name="android.permission.NFC" />
<uses-feature android:name="android.hardware.nfc" android:required="true" />
```

Your `AndroidManifest.xml` should look like this:

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.nfc">

    <uses-permission android:name="android.permission.NFC" />
    <uses-feature android:name="android.hardware.nfc" android:required="true" />

    <application
        android:name=".MainApplication"
        android:label="@string/app_name"
        android:icon="@mipmap/ic_launcher"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:theme="@style/AppTheme">
        
        <!-- Your activity and other config -->
        
    </application>
</manifest>
```

✅ That's all you need for Android — as long as your plugin is properly initializing the `NfcAdapter` and using foreground dispatch.

---

## ✅ iOS Setup: Xcode Project (in `nfc-example/ios/App`)

### Step 1: Enable NFC Capability

1. Open Xcode:

   ```bash
   npx cap open ios
   ```

2. Select the root project in the **Project Navigator**.

3. Click on your **App Target** (e.g. `App`), then go to the **"Signing & Capabilities"** tab.

4. Click **+ Capability**, then choose:

   ✅ **NFC Tag Reading**

> This automatically adds the correct entitlement.

---

### Step 2: Info.plist — NFC Usage Description

Still in Xcode:

1. Select `Info.plist` under the App target.
2. Add a new entry:

```xml
<key>NFCReaderUsageDescription</key>
<string>We need NFC access to scan and write tags for your app experience.</string>
```

✅ This ensures your app passes App Store review and shows a proper permission dialog.

---

## 🧪 Now You're Ready to Test!

After these changes:

```bash
npx cap sync
```

Then launch via:

* `npx cap open android` ➜ Build + run in Android Studio
* `npx cap open ios` ➜ Build + run in Xcode on real iPhone

---
