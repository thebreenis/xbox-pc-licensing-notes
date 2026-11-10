# October Xbox authentication and achievement case studies

Observation period: October 2026, with the detailed comparison and tracing records consolidated on October 29. Documentation added November 10, 2026.

These experiments concern specific legacy Windows games and a Minecraft Mobile-identity x64 compatibility build. They extend the diagnostic model; they do not reproduce or change the F1/MSIXVC content-license results. No F1 key, decryption, or playable F1 build resulted from them.

## Evidence and boundaries

**Observed** below means a retained trace, inspected implementation, or recorded service response. **Tester-confirmed** means visible behavior or Xbox-profile state reported during the test. Interpretations and unobserved stages are identified separately. This page publishes reviewed summaries only: no raw receipts, account identifiers, token values, token fingerprints, private keys, captures, or memory dumps.

Keep these stages separate:

`Local startup/trial gate -> Store receipt acquisition/cache -> Xbox authentication -> endpoint-specific authorization -> local achievement processing -> submission/transport -> backend processing -> visible profile result`

That is a diagnostic checklist, not one mandatory implementation sequence for every game. In particular, the observed Minecraft event path and legacy direct achievement requests used different submission mechanisms.

## Local checks and Xbox authorization

Local trial/license-gate changes sometimes left sign-in functional. That did not create a Store entitlement, sign a trusted receipt, issue a content key, or demonstrate achievement-write authority.

In Rayman Fiesta Run package `1.1.0.12` (x86), one runtime experiment kept local trial state true while separately changing selected feature gates and obtaining bridge-based sign-in. The tester could open achievements and leaderboards; presence failed, and gameplay beyond level 3 was not tested. Opening a feature menu did not establish that its writes were accepted.

Another runtime variation of that package, with the bridge disabled, successfully obtained a native legacy JWT using effective SDK settings `newAuth=false`, `useSamlToken=false`. The tester confirmed native sign-in; presence remained absent. Profile achievement synchronization for that exact combination was not established. These settings and feature changes were applied in memory; the installed package alone did not reproduce them after a fresh launch.

In the analyzed legacy SDK, `newAuth=true` introduced receipt-backed authentication-context handling. `newAuth=false` skipped that path; `useSamlToken` was a separate choice. This is a build-specific observation, not a universal meaning of those names across Xbox SDKs. The successful false/false run supersedes any blanket statement that Rayman's native authentication always failed.

Local APPX signing/trust is also distinct from Store receipt authority. A locally trusted package certificate does not make a locally signed receipt trusted by Xbox or supply a missing MSIXVC content key.

## Rayman and Adera: same platform field, different authorization outcomes

The comparison examined Adera `2.5.2.34894` and Rayman Fiesta Run `1.1.0.12`, both x86. Adera supplied the working native receipt/JWT control; Rayman's receipt-backed native path failed, while the tested bridge obtained a modern user token and adapted legacy request handling.

| Boundary | Adera control | Rayman bridge experiment |
| --- | --- | --- |
| Receipt-backed native authentication | Signed receipt supplied; native token issued | Missing-receipt path returned a SOAP fault |
| Profile read | Native working context | HTTP 200 with modern bridge context |
| Natural achievement attempt | Achievement 7 returned HTTP 201 | Starter, achievement 19, returned HTTP 400, code 11 |
| Achievement request fields | Platform 17; contract version 1 | Platform 17; contract version 1 |

The Rayman response described an invalid platform type. Platform 17 was also present in the successful Adera request, so the error alone does not establish that the literal number should be changed. The examined SDK named that value `XboxLIVEOnWindows`. A separate game configuration's platform value belonged to a different field and was not interchangeable.

The tested modern bridge context enabled user/profile access without reproducing Adera's observed native authorization behavior. Missing title/device authority was a hypothesis, not a decoded-claims finding or an established cause of code 11. No successful Rayman profile achievement write was demonstrated in these tests.

The native legacy authorization value included an XML JWT wrapper. Once that wrapper was accounted for, Adera submitted the same token it received. Earlier comparisons that hashed different wrappers had misleadingly suggested token replacement. Simply relabeling a modern header as `XBL2.0` failed a profile-read test and did not reproduce the native wrapper/context. Authorization scheme names alone do not specify permissions or token compatibility.

## Receipt controls, including a free game

In the examined games, `pop.bin` held signed Store receipt XML, separate from the later Xbox authorization token. Adera and Halo Spartan Assault cached receipts verified against the inspected Windows Store Licensing public certificate; package-signing certificates were a separate trust role.

| Control | Observation | Supported conclusion and limit |
| --- | --- | --- |
| Adera receipt argument replaced with an empty string | Native XSTS fault `0x8015DB00`; sign-in failed | Missing receipt input was sufficient to break this tested path; the error is not a universal diagnosis of every failure |
| Adera signature-only mutation in a temporary receipt copy | Original locally verified; mutated signature failed local verification; native authentication failed | Strong evidence signature validity mattered; an immediate restored-signature positive control was requested but not explicitly confirmed in that sequence |
| Unchanged Adera receipt supplied to Rayman's native context | Context grew; Xbox returned fault `0x8015DA87`, without a token | Another title's valid receipt did not authorize Rayman; the precise internal error meaning remained unestablished |
| Free Hexic `1.2.0.36`, x86 | Starting without its standard cached receipt, fresh acquisition returned a signed Full receipt and native sign-in succeeded | A free game can still obtain/use a Full receipt; free does not mean receipt-free |

These tests did not demonstrate an accepted manufactured receipt or interchangeable cross-title receipts. Signature verification against a particular public key, service acceptance, and independent trust-chain validation are distinct checks.

The separate [November 4 case studies](november-4-auth-and-receipts.md) add a four-game signature comparison, TY's cached-receipt removal/restoration control, and a rejected TY-to-Crash-Course receipt substitution. They also document successful legacy Crash Course sign-in with mixed service results and a later Rayman gameplay crash; these outcomes must not be attributed to the October configurations above.

## Minecraft Mobile: separate list and event identities

The examined compatibility build paired a matching desktop x64 engine with Mobile package identity/configuration; it was not ARM instruction translation. Final package revision `1.16.4002.6` used a targeted achievement-list formatter repair.

| Identity | Xbox title ID | Service configuration ID |
| --- | --- | --- |
| Mobile | `1916611344` | `00000000-0000-0000-0000-0000723d2b10` |
| Desktop | `896928775` | `4fc10100-5f7a-4470-899b-280835760c07` |

The list request originally used the desktop title and returned HTTP 200 with an empty result. A targeted formatter change to the Mobile title loaded all 111 achievements. This was a successful-but-wrong-identity query, not an established TLS failure. Removing desktop override fields alone had not corrected it; the final repair retained the original Mobile configuration.

Natural gameplay events already used the Mobile title/SCID and a channel named `Microsoft.XboxLive.T1916611344`. An incorrect read identity therefore did not establish an incorrect write identity. The separate Minutes Played statistic continued using desktop identity and was not repaired or validated by the list change.

## Observed natural-unlock pipeline and delayed delivery

Tracing a natural smelting action established this local path:

```text
Furnace output collected
  -> ItemSmelted / ItemAcquired gameplay events
  -> enabled dispatcher, asynchronous queue and worker
  -> Xbox telemetry listener and buffer
  -> per-player flush and Xbox events API
  -> Windows LoggingFields and Mobile-named LoggingChannel
  -> EventWriteEx success
  -> collection / upload / backend processing [not individually captured]
  -> eventual Xbox-profile unlock [tester-confirmed]
```

SDK `S_OK` and Windows event-write status zero established local acceptance, not upload or backend acceptance. No identifiable Xbox gameplay-event upload was established in the supplied MITM captures. Unrelated telemetry or PlayFab events were not treated as proof of achievement submission.

The tester later confirmed Acquire Hardware, Archer, and Monster Hunter on the profile, including attempts from earlier sessions, and subsequently confirmed a new Bake Bread unlock with the repaired list build. The precise delay location and originating revision for each delayed attempt were not individually established.

Unlocks arrived during an elevated diagnostic that inventoried settings and briefly traced the Windows telemetry collector. It did not flush caches, force uploads, restart telemetry services, or change telemetry policy. Timing alone did not establish that the diagnostic caused the unlocks. System-wide telemetry warnings were not attributed specifically to Minecraft.

These delayed successes differ from Rayman's explicit HTTP rejection. Do not diagnose either case solely from the absence of an immediate notification, and do not describe an explicitly rejected request as an established pending upload.

## Reusable diagnostic lessons

- Measure sign-in, list reads, presence, natural triggers, submission results, and profile state independently.
- Preserve the exact build and request context. Endpoint, title, account, token type, wrapper, and contract matter; a shared numeric field or successful profile read does not establish write authority.
- For a localhost MITM proxy, packaged-app loopback access is separate from TLS trust/pinning. A missing exemption prevented useful traffic in one Minecraft test; adding it did not bypass certificate validation.
- Start clean controls with earlier bridge/proxy helpers stopped. Their interference was suspected after a tester-confirmed recovery, but the precise mechanism was not isolated.
- Use validated instruction boundaries and scoped hooks. An incorrectly placed Minecraft breakpoint and Hexic's shared-COM-thunk hook caused instrumentation failures; those runs were not clean evidence of game behavior.
- Observe the correct collector with appropriate permissions. An incomplete non-elevated session list or a quiet capture does not establish that collection is disabled.
- Preserve timing across sessions so delayed earlier submissions are not automatically attributed to the newest patch or diagnostic.

None of these case studies establishes a universal achievement repair, a safe cross-game SDK swap, or decryption of unlicensed encrypted content.
