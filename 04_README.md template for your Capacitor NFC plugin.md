Here’s a complete `README.md` template for your **Capacitor NFC plugin** — written in a clean, developer-friendly format suitable for publishing on npm and GitHub:

---

````markdown
# @your-org/capacitor-nfc-plugin

A Capacitor plugin for reading and writing NFC tags on **iOS** and **Android**.

Supports scanning, writing, and erasing common NFC tag types (e.g. NDEF-formatted tags).

---

## ✅ Features

- 📲 Works on **iOS** (Core NFC) and **Android** (NfcAdapter)
- 📡 Read NFC tags (NDEF)
- ✏️ Write custom text to tags
- ❌ Erase tags
- 🔔 Event-based tag discovery

---

## 📦 Installation

```bash
npm install @your-org/capacitor-nfc-plugin
npx cap sync
````

---

## 🛠 Platform Requirements

| Platform | Minimum Version | Notes                               |
| -------- | --------------- | ----------------------------------- |
| Android  | 6.0+ (API 23)   | Requires NFC hardware               |
| iOS      | 13+             | Requires real device, not simulator |

> ⚠️ You must enable NFC capabilities manually (see below).

---

## 🧩 Usage

### Import the Plugin

```ts
import { NFC } from "@your-org/capacitor-nfc-plugin";
```

---

### Check if NFC is available

```ts
const { available } = await NFC.isAvailable();
console.log("NFC available?", available);
```

---

### Start Scanning

```ts
await NFC.startScanning();
```

---

### Stop Scanning

```ts
await NFC.stopScanning();
```

---

### Write a Tag

```ts
await NFC.writeTag({ data: "Hello NFC!" });
```

---

### Erase a Tag

```ts
await NFC.eraseTag();
```

---

### Listen for Events

```ts
NFC.addListener("onTagDiscovered", (tag) => {
  console.log("Tag data:", tag);
});

NFC.addListener("onError", (error) => {
  console.error("NFC error:", error);
});
```

---

## 🧪 Example App

We provide a sample React + Capacitor app to test the plugin:

```bash
git clone https://github.com/your-org/nfc-example-app.git
cd nfc-example-app
npm install
npx cap sync
```

Then run via:

```bash
npx cap open ios   # or
npx cap open android
```

---

## 📱 Native Configuration

### Android

In `android/app/src/main/AndroidManifest.xml`:

```xml
<uses-permission android:name="android.permission.NFC" />
<uses-feature android:name="android.hardware.nfc" android:required="true" />
```

### iOS

* Open your project in Xcode: `npx cap open ios`
* Go to *Signing & Capabilities* ➜ add **NFC Tag Reading**
* In `Info.plist`, add:

```xml
<key>NFCReaderUsageDescription</key>
<string>This app uses NFC to scan and write tags.</string>
```

---

## 🤝 Contributing

Contributions welcome! Please submit issues and pull requests to improve the plugin.

---

## 📄 License

```
MIT © \[Your Name or Org]
```

---

## ✅ Optional: Rename `@your-org` & URLs

If you're using:

- A GitHub repo: change `https://github.com/your-org/nfc-example-app`
- A different npm package name: change `@your-org/capacitor-nfc-plugin`

Would you like help generating the GitHub repo with Actions, badges, or release workflow too?

