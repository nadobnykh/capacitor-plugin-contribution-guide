# JS Wrapper Layer Implementation — Step-by-Step

---

## 1. Create Plugin Interface (TypeScript)

Create a new file `src/definitions.ts` inside your plugin project:

```typescript
export interface NFCPlugin {
  isAvailable(): Promise<{ available: boolean }>;
  startScanning(): Promise<void>;
  stopScanning(): Promise<void>;
  writeTag(options: { data: string }): Promise<void>;
  eraseTag(): Promise<void>;

  addListener(
    eventName: 'onTagDiscovered' | 'onError',
    listenerFunc: (data: any) => void
  ): Promise<PluginListenerHandle>;
}
```

---

## 2. Implement Plugin Class

Create a file `src/index.ts`:

```typescript
import { registerPlugin } from '@capacitor/core';
import type { NFCPlugin } from './definitions';

const NFC = registerPlugin<NFCPlugin>('NFCPlugin');

export * from './definitions';
export { NFC };
```

---

## 3. Usage in App Code

Here’s a minimal example of using the plugin in a Capacitor/React or plain TS/JS app:

```typescript
import { NFC } from 'your-plugin-name';

async function scanNFC() {
  const availability = await NFC.isAvailable();
  if (!availability.available) {
    console.log("NFC not available");
    return;
  }

  await NFC.startScanning();

  const listener = await NFC.addListener('onTagDiscovered', (tag) => {
    console.log('Tag discovered:', tag);
    // Process tag info here
  });

  const errorListener = await NFC.addListener('onError', (error) => {
    console.error('NFC error:', error);
  });

  // Later, stop scanning and remove listeners if needed
  // await NFC.stopScanning();
  // listener.remove();
  // errorListener.remove();
}
```

---

## 4. Notes on Event Listeners

* The native code sends events `"onTagDiscovered"` when a tag is detected.
* Errors are sent with `"onError"`.
* Your JS app can subscribe to these events to update UI or handle data.

---

## 5. Build and Test Plugin

* Build the plugin and copy it into your app.
* Test on real Android and iOS devices with NFC hardware.
* Verify scanning, writing, and erasing works as expected.

---
