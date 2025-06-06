## Step-by-step guide to creating a Capacitor plugin

### 1. Understand Capacitor Plugins

Capacitor plugins are native code modules (Java, Swift, Objective-C) that expose native device functionality to your JavaScript/TypeScript app. Plugins can be used in Capacitor apps with any framework (React, Angular, Vue, etc.).

**Official docs:**
[https://capacitorjs.com/docs/plugins](https://capacitorjs.com/docs/plugins)

---

### 2. Set up your development environment

Make sure you have the latest Node.js and npm/yarn installed.

Install Capacitor CLI globally (if not already):

```bash
npm install -g @capacitor/cli
```

---

### 3. Create a new plugin project using the Capacitor CLI

Capacitor provides a plugin starter template you can generate:

```bash
npm init @capacitor/plugin@latest
```

This will prompt you for plugin name, id, description, and create a folder with the starter plugin scaffold.

---

### 4. Understand the generated plugin structure

Inside your new plugin folder you will see:

* `src/` — contains the TypeScript code for your plugin interface
* `android/` — Android native implementation (Java or Kotlin)
* `ios/` — iOS native implementation (Swift or Objective-C)
* `example/` — a demo app to test your plugin
* `package.json` — npm package info
* `README.md` — documentation

---

### 5. Define your plugin API in TypeScript

In `src/definitions.ts` define your plugin interface:

```ts
export interface MyPlugin {
  echo(options: { value: string }): Promise<{ value: string }>;
}
```

In `src/index.ts` implement the JS interface that calls the native code:

```ts
import { registerPlugin } from '@capacitor/core';
import type { MyPlugin } from './definitions';

const MyPlugin = registerPlugin<MyPlugin>('MyPlugin');

export * from './definitions';
export { MyPlugin };
```

---

### 6. Implement native code for Android

In the `android/src/main/java/...` folder:

* Modify the main plugin class that extends `Plugin`
* Override methods and use the `@PluginMethod` annotation for methods exposed to JS

Example (Kotlin):

```kotlin
@PluginMethod
fun echo(call: PluginCall) {
  val value = call.getString("value") ?: ""
  val ret = JSObject()
  ret.put("value", value)
  call.resolve(ret)
}
```

---

### 7. Implement native code for iOS

In the `ios/Plugin.swift` file:

* Create methods annotated with `@objc` and make sure they call the completion handler

Example (Swift):

```swift
@objc func echo(_ call: CAPPluginCall) {
  let value = call.getString("value") ?? ""
  call.resolve([
    "value": value
  ])
}
```

---

### 8. Build the native projects

For Android:

* Open `android/` in Android Studio and build or run tests

For iOS:

* Open `ios/Plugin.xcworkspace` in Xcode and build the plugin

---

### 9. Test your plugin in the example app

The scaffold includes an example app under `/example`. Run it:

```bash
cd example
npm install
npx cap sync
npx cap open ios
# or
npx cap open android
```

Run on device/emulator and test the plugin methods from JavaScript.

---

### 10. Publish your plugin (optional)

Publish to npm so it can be installed via npm/yarn.

Make sure to update `package.json` with repository, keywords, version, and test thoroughly.

---

## Useful official resources

* Official plugin authoring docs:
  [https://capacitorjs.com/docs/plugins/creating-plugins](https://capacitorjs.com/docs/plugins/creating-plugins)

* Plugin API docs (JS and native):
  [https://capacitorjs.com/docs/plugins/api](https://capacitorjs.com/docs/plugins/api)

* Plugin starter template GitHub repo:
  [https://github.com/ionic-team/create-capacitor-plugin](https://github.com/ionic-team/create-capacitor-plugin)

* Example plugin repo by Ionic:
  [https://github.com/ionic-team/capacitor-plugin-template](https://github.com/ionic-team/capacitor-plugin-template)


