# November 4 authentication and receipt case studies

Observation date: November 4, 2026 (Europe/Paris), including comparison of receipts acquired earlier. Documentation added November 10, 2026.

These legacy Windows game tests extend the [October case studies](october-auth-and-achievements.md). They are separate from both the original F1/MSIXVC investigation and the [November 9–10 Windows 8 follow-up](windows-8-follow-up.md). They produced no F1 content key or new F1 gameplay result.

## Evidence and scope

**Observed** means a retained trace, file comparison, cryptographic check, or recorded Windows crash event. **Tester-reported** means visible behavior described during the experiment. Conclusions below apply to the examined builds and configurations; unresolved causes are identified explicitly.

The source records comprise the November 4 handoff, Crash Course sign-in and receipt-test summaries, a four-game receipt comparison, and Rayman build verification. Only summarized findings are published: no raw receipts, tokens, account/device identifiers, personal paths, captures, binaries, or saves.

## Crash Course Go sign-in did not establish full service functionality

Crash Course Go is the Windows 8 title with package name `Microsoft.CrashCourse2`, not the Xbox 360 Doritos Crash Course 2. The tests preceded the local trial-patched build: the active-license-patched branch was in use, with trial behavior retained.

**Observed:** A temporary runtime change set `newAuth=false`, verified at sign-in. The legacy XSTS exchange was followed by successful profile and achievement reads. **Tester-reported:** the game signed in.

| Service boundary | Observed result |
| --- | --- |
| Profile identity/settings reads | HTTP 200 |
| Title-achievement and achievement-progress reads | HTTP 200 |
| Presence recent endpoint | HTTP 400, `ArgumentError` |
| Title history | HTTP 400, code 6, invalid request body |
| Public title-storage configuration read | HTTP 200 |
| Separate user title-storage request | HTTP 400, code 2013, no associated TitleGroupId configuration |

These results establish mixed endpoint outcomes after sign-in, not a general Xbox service restoration. An achievement read is not an achievement write. The errors do not by themselves isolate token permissions, request construction, or title configuration as the cause. In particular, not every title-storage request failed.

The auth change was temporary; it was not incorporated into the subsequently installed trial-patched package. The later receipt experiment below verified `newAuth=true` independently. Neither experiment establishes the current installed state after later sessions.

## TY receipt reached Crash Course authentication but was rejected

The separate receipt test used Crash Course Go local build `1.1.37.3`, with trial behavior unchanged and `newAuth=true` verified at authentication. A byte-identical copy of TY the Tasmanian Tiger's 1,302-byte cached receipt was placed in Crash Course's previously absent receipt-cache location. The TY source was left unchanged.

**Observed:** The SDK read 1,302 receipt characters and passed 1,302 characters into authentication. XSTS returned HTTP 200 with a 459-byte response, after which the SDK reported `0x8015DA87`. No successful sign-in was established.

This differed from the no-receipt baseline's `0x82BC000C`: a nonempty receipt reached authentication and was rejected. The trace did not capture human-readable fault text establishing the precise reason for `0x8015DA87`. The game process was absent afterward, but the trace did not isolate why it exited.

**Diagnostic lesson:** HTTP 200 did not establish successful Xbox authentication. The protocol result and subsequent SDK behavior mattered. A valid receipt from another title did not authorize this Crash Course run.

## Four-game receipt comparison

The examined `pop.bin` files contained signed Store receipt XML, distinct from SAML assertions, JWTs, and the eventual XBL2.0 authorization value. Their lengths are sample measurements, not a fixed-size license region inside every Xbox token.

| Receipt sample | File bytes | LicenseType | Original signature verified | Signature valid after changing AppId to Crash Course |
| --- | --- | --- | --- | --- |
| Adera | 1,288 | Full | Yes | No |
| Hexic | 1,288 | Full | Yes | No |
| Halo Spartan Assault | 1,301 | Full | Yes | No |
| TY the Tasmanian Tiger | 1,302 | Full | Yes | No |

**Observed:** All four samples used RSA-SHA256 with 256-byte signatures and matched the same examined Store Licensing certificate. Changing AppId invalidated each original signature.

This demonstrates integrity protection over the examined signed content: editing another title's AppId did not produce a valid receipt for Crash Course. Verification against the inspected public certificate is distinct from independent certificate-chain validation and from service acceptance. No accepted manufactured receipt was demonstrated.

The sample comparison does not establish a universal receipt lifetime, account portability, or device-binding rule. Receipt dates, signed license assertions, and Xbox token lifetimes must not be treated as interchangeable.

## TY cached-receipt removal and restoration control

In TY's tested receipt-backed configuration, the cached `pop.bin` was renamed out of its normal location for a login attempt, then restored for another launch. **Tester-reported:** login failed without it and worked again after restoration. The tester also reported working presence with the supplied receipt.

This supports dependence on that cached receipt for the tested login path. It does not establish that any receipt works for any title, account, or device; the Crash Course substitution above failed. Nor does login or reported presence establish a server-accepted achievement unlock. The experiment does not isolate all conditions that made the supplied receipt usable in TY.

## Rayman startup verification did not establish gameplay stability

Rayman Fiesta Run local build `1.1.0.16` retained the original Xbox DLL, added the previously used local license-helper and trial-getter changes, and configured `newAuth=false`, `useSamlToken=true`. Installed critical files matched staging, all 13 backed-up data files were unchanged immediately after installation, and the process initially launched and responded.

**Tester-reported:** entering level select crashed the game. **Observed:** a Windows Application event recorded an access violation, `0xC0000005`, in `RFR_WinRT.exe` at RVA `0xADB23`. No retained crash dump was available at the subsequent check. A recursive search found no `pop.bin` in the package's installation or app-data folders.

The absence of a cached receipt was consistent with the tested legacy auth configuration, but did not establish the access violation's cause. Neither the local patches nor missing receipt was isolated as the culprit. Fresh sign-in, presence, and achievement success were not verified for this build. The crash remained unresolved in the November 4 record; this is not a claim about later repairs.

**Diagnostic lesson:** package integrity, save preservation, successful startup, level-select stability, authentication, and server-accepted achievements require separate validation.
