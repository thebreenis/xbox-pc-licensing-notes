# November 4–6 local launch and bridge follow-up

Observation period: November 4–6, 2026 (Europe/Paris). Documentation added November 10, 2026.

These title-specific results supplement the [November 4 authentication and receipt cases](november-4-auth-and-receipts.md). They do not change the F1/MSIXVC negative-control result or demonstrate a missing content key being recovered. Sources are retained local verification reports, bridge-result summaries, asset-validation records, and monitor status records. No raw credentials, receipts, account identifiers, personal paths, or binaries are published.

## Rayman: profile access with the later bridge did not restore presence

The November 4 experiment used [xct-win8bridge at revision `4ae771aa225464bd4ad637f8d3d2217d9f583b24`](https://github.com/freshdex/xct-win8bridge/tree/4ae771aa225464bd4ad637f8d3d2217d9f583b24), scoped to the Rayman process, with the original game executable and Xbox SDK. Effective settings were `newAuth=true`, `useSamlToken=false`.

**Observed:** The upstream legacy XSTS exchange returned HTTP 200 with a 459-byte SOAP fault and recorded error `0x8015DA83`. The bridge's replacement SOAP result was accepted by the game, but subsequent endpoints had different outcomes:

| Boundary | Recorded result |
| --- | --- |
| Profile read | HTTP 200 |
| Presence | HTTP 401; `WWW-Authenticate: Token realm='xboxlive.com', error='token_required'` |
| Title history | HTTP 400, code 6, invalid request body |

Two properly initialized captures reproduced the result. An intervening addon hot reload had no token available and passed the upstream fault through; that run was not treated as equivalent to the initialized bridge tests. The confirming capture followed a proxy restart.

Request bodies and downstream responses were not changed. The experiment submitted no achievement unlock. Temporary bridge components were stopped afterward.

This establishes profile access in the tested bridge context, not restored presence, history, or achievement-write authority. It does not establish that malformed JSON caused the history error, nor that an `XBL3.0` scheme label implies universal token compatibility.

### The proposed native-versus-bridge comparison was not executed

A subsequent comparison intended to use the native and bridge credentials on the same presence request was blocked by automatic approval review before execution. No new tokens were obtained and no presence requests were replayed in that comparison. It produced no controlled comparison result.

Consequently, differences between errors observed in separate native and bridge runs do not isolate the token as their cause. The original request, credential context, and other conditions were not held constant in an executed paired test.

## Dragon's Lair: package activation and local trial state were separate changes

The original x64 package was `1.0.1.6`. The activation-only local package `1.0.1.7` addressed a pre-execution Windows Store activation failure reported as `0x803F8001`.

**Observed by file comparison:** All 637 payload files were unchanged. The manifest identity version changed, and the Store signature was replaced by locally trusted package signing with regenerated signing metadata. The executable, trial code, and Xbox SDK were unchanged; no configuration file was added and saves were not reset.

The later `1.0.1.9` package changed only the executable among those 637 payload files, at 16 documented trial-result locations: a shared wrapper and 15 inlined comparisons. Identity version and signing metadata also changed. The intermediate wrapper-only change was not the final verified result.

**Observed at runtime for `1.0.1.9`:** The game launched, the Buy Full Game button was absent, and the Extras menu displayed. A full playthrough was not tested, and Xbox authentication or achievement-write success was not established by this verification.

This is a local package/behavior result. It does not demonstrate issuance of a Store entitlement, a trusted Store receipt, or an MSIXVC content key. The same numeric launch error in another package generation does not establish that the same remedy applies.

## Minion Rush: intact payload and restored assets, unresolved loading outcome

The original x86 package was `4.1.4.1`; the locally signed package was `4.1.4.2`. Comparison covered 165 game files and found no game-file changes. Only manifest version and package signing changed.

No game-wide trial-to-full check was identified or patched. The licensing callers examined concerned in-app products. This is an inspection result for this build, not proof that every possible licensing path was exhaustively excluded.

The game played its opening movie and then remained on an animated Quick Tip loading screen, as reported by the tester. A subsequent asset backup passed ZIP CRC validation, and 49 asset/configuration files were installed. Other-user saves, account data, Xbox receipts, telemetry, settings, and app packages were excluded from that import.

Getting past the loading screen after restoration was **not confirmed**. Archive integrity and asset installation are not evidence of a completed gameplay repair, and the earlier animated loading screen was not established as a licensing failure.

## Imperia: account-switch changes and a bounded negative capture

For the temporary account-switch variant, comparison covered 5,964 files; only the manifest and `www/app/libraries/xbox_manager.js` changed. The modification attempted an app-scoped Xbox sign-out before the normal sign-in flow. Shared credentials were not cleared and saved data was not reset by that modification. Successful reappearance of the Microsoft account picker was not established.

The original `6.1.8.8` package was subsequently reinstalled. Verification of 5,961 payload files found no modified or missing files and recorded Store signing. These are separate stages; the temporary account-switch package was not the final original-package verification target.

A separate five-minute monitor recorded successful attachment and installation of WinHTTP hooks, then finished without capturing the targeted XBL2.0 authorization value. The supported conclusion is **not captured by this monitor during this window**. It does not establish that Imperia never uses XBL2.0, that no authentication occurred, or that all possible networking paths were covered.

## Validation boundaries

Keep package activation, local trial presentation, receipt acceptance, account-picker behavior, endpoint authorization, asset availability, and actual gameplay as separate observations. None of these cases supplies a universal SDK setting, token substitution, or license repair. Existing F1 conclusions remain unchanged.
