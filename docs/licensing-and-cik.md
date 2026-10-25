# Content licensing and CIK files

## What Xodus did

The tested Xodus flow authenticated the user and device, exchanged Microsoft tokens, and requested content-license data from `https://licensing.mp.microsoft.com/v7.0/licenses/content`.

On success, its implementation parses the returned license, derives the device key from the device license, unwraps the supplied content keys, and writes one `<key-id>.cik` file per key. The device key and the game's content key have different roles: the device key unwraps license material; the content key decrypts the matching content.

For the pinned Xodus implementation, a CIK file contains a 16-byte little-endian key GUID followed by a 32-byte content key: 48 bytes total. Other tools or package generations must be checked for format compatibility rather than assuming every `.cik` file has the same layout.

An entitlement refusal contains no usable replacement content key. Changing an error to success, editing a key ID, or using another game's key does not reconstruct the missing key.

## Controlled comparison

All three tests used the same Xodus account/device session. Acquisition types for the two successful controls were supplied by the tester.

| Test | License situation | Result |
| --- | --- | --- |
| Minecraft for Windows | Purchased outright | Successful content-license request and key unpacking; one 48-byte CIK file |
| Minecraft Dungeons for Windows | Available through the tester's active Game Pass access | Successful content-license request and key unpacking; one 48-byte CIK file |
| F1 2018 PC GP | No valid license for this test case | Entitlement refusal; no F1 key returned |

The F1 response was: `Users do not possess any satisfying entitlements for the content id in question.` It was reproduced after the successful controls. Requests using the US market and the neutral default both failed.

An earlier request encountered the Store device limit. After unused device associations were removed, that obstacle cleared, but F1 still received the entitlement refusal. Device-limit exhaustion and lack of a satisfying content entitlement were separate observations.

## What the controls establish

The tested authentication, device provisioning, token exchange, license parsing, and key-unpacking chain could succeed. The content-license endpoint was not universally nonfunctional in this environment. The F1 denial was consistent with an unlicensed negative control.

The controls do not establish that every title, license type, device identity, or newer licensing flow is supported. They also do not establish that Xodus's exact request matches the native Windows request.

## No game license was fabricated

No F1 content key was obtained, fabricated, or decrypted. No license blob, Microsoft service response, or Gaming Services executable was modified. Successful Xbox sign-in, package registration, or an in-game presence update was not treated as proof of a valid game license.

The study did not produce a working F1 license bypass. Earlier discussion of a hypothetical redeemed Windows copy was exploratory; it was not evidence that this account had purchased or redeemed the tested product.
