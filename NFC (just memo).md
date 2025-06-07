



Got it — you’re using **react-native-nfc-manager** and when reading a tag, you only get the **ID**, but you expect way more info stored on the NFC chip.

This is a common situation because:

* The **ID** (also called **UID**) is a unique hardware identifier, but often the actual data is stored in NDEF records on the tag.
* To get the data payload, you need to **parse the NDEF message** from the tag.
* Sometimes tags are **not NDEF formatted** or contain proprietary data formats.

---

### What to check and do:

1. **Make sure you read the NDEF message**, not just the tag ID.
   In react-native-nfc-manager, the NDEF data is usually in `tag.ndefMessage`.

2. **Example snippet to read NDEF records:**

```js
import NfcManager, {Ndef} from 'react-native-nfc-manager';

async function readNdefTag() {
  try {
    await NfcManager.start();
    const tag = await NfcManager.requestTechnology(NfcTech.Ndef);
    if (tag.ndefMessage) {
      const ndefRecords = tag.ndefMessage;
      const text = ndefRecords
        .map(record => Ndef.text.decodePayload(record.payload))
        .join(', ');
      console.log('NDEF content:', text);
    } else {
      console.log('No NDEF message found');
    }
    await NfcManager.cancelTechnologyRequest();
  } catch (ex) {
    console.warn('Error reading NFC:', ex);
  }
}
```

3. **If `tag.ndefMessage` is empty or missing:**

   * The tag might not have any NDEF data.
   * The tag could be formatted differently or protected.
   * Try reading other technologies (e.g., MifareClassic) if supported.

4. **Check the type of NFC tag you have** (e.g., NTag213, NTAG215, etc.) and see if it stores data in a standard NDEF format.

---

### TL;DR

* The tag ID is just a unique identifier.
* The actual stored data is in `tag.ndefMessage`.
* Parse `tag.ndefMessage` with the helper from `Ndef` to get real content.

---

If you want, I can help you debug your current reading code or create a sample to properly parse the NFC tag data! Would you like that?






Thanks for specifying — **ISO 15693** tags are a bit different from the typical NFC Forum NDEF tags (like Type 2 or Type 4). They often don’t store data in standard NDEF format by default, and their reading is more low-level.

---

### What’s special about ISO 15693 tags?

* They support **block-level reading/writing** — you can read and write raw memory blocks.
* They might **not have an NDEF message**, so `tag.ndefMessage` can be empty.
* You usually need to **manually read memory blocks** to get data beyond the tag UID.

---

### How to read data from an ISO 15693 tag with `react-native-nfc-manager`

`react-native-nfc-manager` supports ISO 15693 commands via the `NfcTech.Iso15693` technology.

You can:

1. Request the ISO15693 technology.
2. Use low-level commands like `readSingleBlock` or `readMultipleBlocks` to read data blocks.
3. Parse the data you get from these blocks.

---

### Example snippet:

```js
import NfcManager, {NfcTech} from 'react-native-nfc-manager';

async function readIso15693Tag() {
  try {
    await NfcManager.start();
    // Request ISO15693 tech
    await NfcManager.requestTechnology(NfcTech.Iso15693);
    
    // Read the first block (block number 0)
    const block0 = await NfcManager.iso15693ReadSingleBlock(0);
    console.log('Block 0:', block0); // usually returns hex string or byte array

    // Read multiple blocks if needed (e.g., blocks 0-3)
    const blocks = await NfcManager.iso15693ReadMultipleBlocks(0, 4);
    console.log('Blocks 0-3:', blocks);

    await NfcManager.cancelTechnologyRequest();

  } catch (ex) {
    console.warn('Error reading ISO 15693 tag:', ex);
  }
}
```

---

### Notes

* The data returned is raw bytes (often hex strings or Uint8Array) — you’ll need to parse it according to your tag’s memory layout.
* If you know the data structure stored on the tag, you can decode the bytes into meaningful info.
* Reading NDEF is usually for NFC Forum standard tags; ISO15693 can have proprietary or custom data.

---

If you want, I can help you write a parser for your tag data or assist in working out how to interpret the raw blocks you read. Just share some raw data if you have it!





Yes, it is possible to determine the **type of an NFC tag** before fully reading its content, but how you do it depends on the NFC technology and what info the system/API exposes initially.

---

### How to detect tag type with **react-native-nfc-manager**

When you scan a tag, the library returns a **tag object** that contains some info about the tag technology and properties. Here’s what you can check:

1. **Tag tech list**
   The `tag.techTypes` or similar field (depending on platform) usually lists all NFC techs the tag supports, e.g.:

   ```js
   tag.techTypes = ['NfcA', 'Ndef', 'Iso15693', ...]
   ```

   This tells you the protocols/standards the tag supports.

2. **Tag type or standard**
   The tag object may contain a `type` or `standard` field, like:

   * `Type 1`, `Type 2`, `Type 3`, `Type 4` (NFC Forum Types)
   * `ISO 15693`
   * `MifareClassic`, `MifareUltralight`
   * `Felica`

3. **Platform differences**

   * On Android, you get more raw access and multiple techs listed.
   * On iOS, the CoreNFC framework exposes slightly different info.

---

### Example: check tag type after scanning

```js
import NfcManager from 'react-native-nfc-manager';

async function detectTagType() {
  try {
    await NfcManager.start();
    const tag = await NfcManager.requestTechnology(); // just to get the tag

    console.log('Tag tech types:', tag.techTypes);
    console.log('Tag type:', tag.type || tag.standard);

    await NfcManager.cancelTechnologyRequest();

    return tag;
  } catch (ex) {
    console.warn('Failed to detect tag type:', ex);
  }
}
```

---

### Summary

* The tag tech list tells you what protocols the tag supports (e.g., `Iso15693` means ISO 15693 tag).
* The tag object may have a type/standard field.
* Use this info to choose how to read the tag (NDEF, Iso15693 commands, etc.)

---

If you want, I can help you write a detection routine that automatically figures out the tag type and reads it accordingly!
