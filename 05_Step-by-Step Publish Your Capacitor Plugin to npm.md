Publishing your Capacitor NFC plugin on npm involves a few clear steps. Here's a full guide so you can:

✅ Make your plugin reusable
✅ Share it publicly or privately
✅ Install it via `npm install @your-org/capacitor-nfc-plugin`

---

# ✅ Step-by-Step: Publish Your Capacitor Plugin to npm

---

## 1. 📁 Set up your plugin folder structure

Your plugin should look like this:

```
capacitor-nfc-plugin/
├── android/
├── ios/
├── src/
│   ├── definitions.ts
│   └── index.ts
├── package.json
├── tsconfig.json
├── capacitor.config.ts
├── README.md
├── LICENSE
```

Ensure you’ve added:

* `main` → compiled output (usually `dist/index.js`)
* `types` → `dist/definitions.d.ts`
* `files` → whitelist for published files

---

## 2. 📝 Create or update `package.json`

Here’s a minimal example for a public plugin:

```json
{
  "name": "@your-org/capacitor-nfc-plugin",
  "version": "1.0.0",
  "description": "A Capacitor plugin for reading and writing NFC tags on Android and iOS",
  "main": "dist/index.js",
  "types": "dist/definitions.d.ts",
  "files": [
    "android",
    "ios",
    "dist",
    "src"
  ],
  "scripts": {
    "build": "tsc"
  },
  "author": "Your Name",
  "license": "MIT",
  "keywords": ["capacitor", "nfc", "plugin", "ios", "android"],
  "capacitor": {
    "platforms": ["android", "ios"]
  },
  "devDependencies": {
    "typescript": "^5.4.0"
  },
  "peerDependencies": {
    "@capacitor/core": "^5.0.0"
  }
}
```

> Replace `@your-org` with your actual npm org or username (see below).

---

## 3. 🏷️ Optional: Create a scoped npm org

If you want your plugin to be published under an org like `@myorg`, make sure you have access or create one here:

[https://www.npmjs.com/org/create](https://www.npmjs.com/org/create)

Otherwise, just use a name like `capacitor-nfc-plugin` (no scope).

---

## 4. 🔐 Log into npm

```bash
npm login
```

> You need an npm account: [https://www.npmjs.com/signup](https://www.npmjs.com/signup)

---

## 5. 🛠️ Build your plugin

Compile the TypeScript:

```bash
npm run build
```

This generates `dist/index.js` and `dist/definitions.d.ts`.

---

## 6. 🚀 Publish it

### For scoped packages (e.g. `@your-org/capacitor-nfc-plugin`):

```bash
npm publish --access public
```

### For unscoped packages (e.g. `capacitor-nfc-plugin`):

```bash
npm publish
```

You should now see your plugin on [https://www.npmjs.com/package/@your-org/capacitor-nfc-plugin](https://www.npmjs.com/package/@your-org/capacitor-nfc-plugin) (or without the scope).

---

## ✅ Done!

Now others (or you) can install it with:

```bash
npm install @your-org/capacitor-nfc-plugin
npx cap sync
```

---

## 📦 Optional: Add `prepare` script for GitHub + npm hybrid

If you want to publish from GitHub too (as a plugin that can be installed via `npm install github:your-org/plugin`), add:

```json
"scripts": {
  "build": "tsc",
  "prepare": "npm run build"
}
```

---

