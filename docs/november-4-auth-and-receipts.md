# November 4 authentication and receipt case studies

Observation date: November 4, 2026 (Europe/Paris), including comparison of receipts acquired earlier. Documentation added November 10, 2026.

These legacy Windows game tests extend the [October case studies](october-auth-and-achievements.md). They are separate from both the original F1/MSIXVC investigation and the [November 9–10 Windows 8 follow-up](windows-8-follow-up.md). They produced no F1 content key or new F1 gameplay result.

## Evidence and scope

**Observed** means a retained trace, file comparison, cryptographic check, or recorded Windows crash event. **Tester-reported** means visible behavior described during the experiment. Conclusions below apply to the examined builds and configurations; unresolved causes are identified explicitly.

The source records comprise the November 4 handoff, Crash Course sign-in and receipt-test summaries, a four-game receipt comparison, Rayman build verification and later repair/authentication summaries, and Taptiles achievement-submission results. Only summarized findings are published: no raw receipts, tokens, account/device identifiers, personal paths, captures, binaries, or saves.

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

The absence of a cached receipt was consistent with the tested legacy auth configuration, but did not establish the access violation's cause. Neither the local patches nor missing receipt was isolated as the culprit. Fresh sign-in, presence, and achievement success were not verified for this build. The initial record stopped with this crash unresolved; the later repair below supersedes that status for build `1.1.0.17`.

**Diagnostic lesson:** package integrity, save preservation, successful startup, level-select stability, authentication, and server-accepted achievements require separate validation.

## Rayman level-select repair and its limits

**Observed:** Build `1.1.0.17` corrected twelve URI-escaped resource filenames to their decoded names, including spaces, before repacking. The executable and original Xbox SDK were unchanged from the preceding patched build. Installed-file verification covered the manifest, configuration, executable, SDK and all twelve corrected assets; the 60 checked save-data files had no differences. The settings remained `newAuth=false`, `useSamlToken=true`.

**Tester-reported:** level select opened and a level started successfully. This establishes a working repair in that test, not comprehensive gameplay stability or successful Xbox service authorization. Service sign-in was not tested as part of that verification. The repair did not require removing the existing local license/trial changes.

## Rayman's own signed receipt still failed authentication

A later supplied Rayman `pop.bin` was 1,561 bytes and contained a Full AppReceipt whose AppId matched the Rayman package family, plus one ProductReceipt. Its XML signature verified against the public certificate retrieved from Microsoft's Store licensing certificate endpoint; an independent canonicalization/SHA256 check also matched the reference digest. These checks establish signed-content integrity against that certificate, not independent chain validation, account entitlement or Xbox acceptance. See Microsoft's [receipt-verification procedure](https://learn.microsoft.com/en-us/windows/uwp/monetize/use-receipts-to-verify-product-purchases).

**Observed:** With the original x86 SDK `2.0.131.0`, both `newAuth=true, useSamlToken=false` and `newAuth=true, useSamlToken=true` read the receipt and failed authentication. Tracing confirmed the matching receipt AppId and Rayman title ID at the authentication payload input. XSTS returned HTTP 200 with a 459-byte SOAP `s:Sender` fault and the generic reason that its creator did not specify a reason. No accepted token was established in those attempts.

In the ordinary `false/true` path, the receipt was not read; SAML authentication and profile/achievement-list reads succeeded, while presence and title history failed as described below. Separate temporary experiments forced receipt reading and inclusion while retaining `newAuth=false`, using either token selection. They read/included the receipt but again received the SOAP fault. Downstream presence/history requests were not reached in those experiments; this was not a demonstrated repair of their HTTP 400 failures.

These are build-specific observations, not universal SDK flag semantics. The receipt's additional ProductReceipt is a difference from TY's sample, not an established cause of rejection.

## TY comparison and unsuccessful SDK substitution

TY's working x86 SDK was `1.2.5.0`, compared with Rayman's `2.0.131.0`. The comparison confirmed each game's own title and receipt inputs. The summarized outer SOAP structure and requested JWT token type matched, but the encrypted payloads were not decrypted or shown to be equivalent. TY signed in and returned a successful profile read; Rayman returned the SOAP fault.

A separate Rayman build `1.1.0.23` substituted TY's SDK DLL and matching metadata while preserving Rayman's executable, package/title identity, receipt and saves. With `newAuth=true, useSamlToken=false`, it read the correct Rayman receipt and produced the same `s:Sender` fault. The original SDK was restored afterward.

The working TY SDK did not fix Rayman by itself. This narrows a simple SDK-version explanation; it does not establish the precise backend rejection cause or general cross-title SDK compatibility.

## Rayman presence/history failures were not fixed by timestamp formatting

The ordinary `newAuth=false, useSamlToken=true` run captured these complete small error responses:

| Request | Result |
| --- | --- |
| Presence recent PUT, numeric Rayman titleId, contract version 1 | HTTP 400, `ArgumentError` |
| Title-history POST, titleId plus lastPlayed, contract version 1 | HTTP 400, Progress code 6, invalid request body |

**Observed:** Changing the outgoing history timestamp to include an explicit UTC `Z` still returned HTTP 400/code 6. A retained successful Adera control used the same body structures and contract version, including a timestamp without `Z`: presence returned HTTP 202 and history HTTP 201. The reference capture was inspected without replaying its credentials.

The error wording therefore does not establish that these schemas or the absent `Z` caused the failure. Title-specific validation and authorization context remained unresolved. The temporary timestamp experiment was removed; no service repair was established.

## Taptiles sign-in and local achievement state did not establish write acceptance

Taptiles build `2.6.288.6` used `newAuth=false, useSamlToken=true`; the tester confirmed sign-in. A screenshot and local SDK achievement read agreed that Milestone (ID 3) and Back-to-Back (ID 4) were locally earned. Their explicitly requested resubmission reached the Xbox Progress service.

**Observed:** Both requests returned service code 11, describing an invalid platform type, and asynchronous completion error `0x800C000C`. Initial API invocation had returned success, which did not establish successful asynchronous completion. Response bodies were captured, but numeric HTTP status was not captured for these submissions and must not be inferred from another game's trace.

No successful Xbox-profile update was confirmed. This is a separate title's example of sign-in/local earned state coexisting with rejected achievement writes; it does not isolate the literal platform field as the root cause.
