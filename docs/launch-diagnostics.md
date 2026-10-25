# Native launch diagnostics

## Captured launch sequence

A manual Process Monitor capture showed `GameLaunchHelper.exe` starting `GamingServicesUI.exe`, then exiting normally with status zero. No `F1_2018.exe` process was identified in that captured sequence or the subsequent process checks.

The Gaming Services UI log recorded:

`SigningIn -> CheckingForUpdates -> PreparingUpdates -> Updating -> Error`

`QueryUpdateAsync` returned `2`; the update attempt then failed with `0x803F8001`. The numeric query result is recorded without assigning an undocumented enum meaning to it.

Microsoft documents `0x803F8001` as `LM_E_CONTENT_NOT_OWNED` in its package-license acquisition API reference. Here it is the observed native failure result, not evidence that the game itself crashed after startup.

An earlier resource-service initialization returned `0x80070015` (device not ready), but the state machine continued through sign-in and into the update path. The experiments did not independently establish the cause or significance of that earlier error.

Fresh launches reproduced the final `0x803F8001` failure. A Store cache refresh did not change the result. Gaming Services, GamingServicesNet, ClipSVC, LicenseManager, and XblAuthManager were running; the installed package reported status `Ok`.

## Additional diagnostic attempts

Regular Store, licensing Admin, and AppModel logs yielded no new matching event that explained the fresh F1 failure. Unrelated licensing errors elsewhere in the Store log were not attributed to F1 merely because they shared an HRESULT.

Two short elevated ETW sessions were attempted, each approximately 35 seconds:

- Gaming Services and Client License Flexible Platform providers.
- Store and WinHTTP providers.

The sessions started and stopped, but their ETL files contained only trace headers. They did not reveal the request path, response body, device claims, or a more specific failure. A quiet trace does not prove that no request was sent.

No HTTPS trust changes or interception were used. No request replay or service-binary patch was performed.

## Interpretation and limits

The native error and Xodus's explicit entitlement denial are consistent with the unlicensed test case. The traces did not expose enough native protocol detail to prove that both clients made identical requests or failed at precisely the same server-side step.

The reviewed external research discussed `DeviceNameNotHexBinary` (3043) and `DeviceEkbRequired` (3023 / `0x87e10bcf`). Neither was observed in this F1 test. Applying a device-provisioning explanation to this result would therefore be speculation.

Xbox presence was observed even though the main game process was not found. Presence is not evidence that the executable loaded, that gameplay started, or that achievement submission works.

Because no main game process was identified, the proposed memory-inspection route had no running F1 executable to inspect. No game memory dump was captured.

See [Microsoft's error-code reference](https://learn.microsoft.com/en-us/xbox/gdk/docs/reference/system/xstore/functions/xstoreacquirelicenseforpackageresult).
