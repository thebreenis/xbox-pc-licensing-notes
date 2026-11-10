# Windows 8/8.1 licensing follow-up

Observation dates: November 9–10, 2026. Published as a separate comparison to the October F1/MSIXVC investigation.

## Scope and evidence

These findings concern legacy Windows 8/8.1 games examined on a current Windows installation: PAC-MAN Championship Edition DX+ variants, Adera, and Crash Course Go. They do not establish the behavior of Windows 8 itself, every SDK revision, F1's native licensing request, or every MSIXVC generation.

The original F1 negative control, Minecraft licensed controls, package metadata, CIK layout for the pinned Xodus implementation, and deployment/launch results remain unchanged. No F1 key, executable decryption, or new playable F1 build resulted from this follow-up.

The observations below are summaries of retained private traces and offline checks. No raw receipts, account/device identifiers, identifier fingerprints, tickets, secret keys, or memory dumps are published here. Ownership reported by a tester is distinguished from what a service response establishes.

## Receipt issuance and CLIP satisfaction are separate decisions

**Observed:** On a tester-reported owning Toshiba account, correcting the legacy product GUID in the receipt request yielded a signed XML receipt asserting a full app license and a durable product entitlement. Separate CLIP requests returned zero keys/leases and no satisfying entitlement. Xbox authentication subsequently rejected the receipt-bearing request.

**Interpretation:** A successful legacy receipt request is not proof that a separate CLIP content-ID request must succeed, nor that Xbox will accept the resulting receipt. A correct GUID identifies the requested product; it does not grant an entitlement.

**Unknown:** The exact reason for the Toshiba cross-service discrepancy remains unresolved. These results do not establish that the tester lacks all ownership, that OEM applicability is the cause, or that a signed AppId format difference alone explains Xbox rejection.

For local Crash Course, the correct legacy GUID was observed in the outgoing receipt request, yet receipt acquisition still failed. That local result must not be substituted for a test on another person's account.

## CLIP may submit multiple user credentials

**Observed:** The local Crash Course probe captured two returned MSA ticket records. Both ticket values matched corresponding `identityValue` fields in the outgoing CLIP request's `users` collection. Their paired references matched `localTicketReference` fields. One transmitted reference matched the receipt broker's `PrimaryWebAccountId` in the same session.

**Conclusion:** That Store account reference and its paired ticket were included in the outbound CLIP body; the account was not merely present in an unused local collection. The request also carried another account reference. The outer Authorization credential did not match the body-ticket fingerprints, so treating it as the sole user identity would have obscured this evidence.

**Limits:** Inclusion does not prove which entitlement the backend evaluated or accepted. It does not independently decode every credential, prove a universal single-account selection rule, or establish the friend's account configuration. Different ticket hashes alone never prove different accounts.

## Three different kinds of licensing artifact

| Artifact | Established role and limits |
| --- | --- |
| `pop.bin` in the examined legacy games | Signed Store receipt XML, not itself a SAML assertion, JWT, CIK, or Xbox authorization token |
| Examined Adera records in `tokens.dat` | Larger XML license documents containing Base64-encoded JSON policies and XML signatures |
| `.cik` from the pinned Xodus implementation | Key-GUID/content-key artifact described in the original [CIK notes](licensing-and-cik.md); not the same format as receipt XML or a CLIP policy document |

**Observed:** Exact Base64 representations of captured Adera policy JSON were located in `tokens.dat`. For the two examined surrounding XML license documents, reference digests and RSA-SHA256 signatures verified against their embedded public keys. Independent certificate/root trust validation was not established by those checks.

Readable policy JSON is therefore not evidence that the surrounding license is unsigned or freely editable. Changing signed policy content invalidates the original signature. Whether `tokens.dat` additionally has whole-file integrity protection remains unknown. Do not generalize these two record checks into a complete specification of the database.

Receipt expiry, durable-product expiry, and Xbox token lifetime are separate concepts. The follow-up does not establish a universal 24-hour receipt validity rule or that issuing a new receipt invalidates every older one. It also does not establish that SDK `newAuth`/`useSamlToken` settings select GUID-versus-package-family receipt AppIds.

## Error boundaries and tracing reliability

The examined Store implementation mapped `0xC03F300C` to `0x80073DF6`, `STORE_ERROR_UNLICENSED_USER`, during a live receipt failure. This is distinct from `0x80073CF6`, package registration failure. A failed receipt request prevented the SDK from obtaining an Xbox token in the observed Crash Course attempts.

Broad service instrumentation caused LicenseManager failures with `STATUS_BAD_FUNCTION_TABLE`, followed by RPC errors. Removing one suspected throw-path hook did not resolve that failure. Replacing the broad set with bounded content-scope, ticket-record, and request-entry observations survived two fresh launch/sign-in tests without another service crash.

This is a validated replacement on the examined build, not an isolated root cause or universal instrumentation fix. Internal offsets were guarded by exact binary hashes. Reduced coverage was explicitly recorded; absent telemetry must not be interpreted as absent service activity. Instrumentation-induced RPC failures were excluded from clean entitlement conclusions.

## SDK state-file lifecycle

**Observed in Crash Course:** The SDK itself successfully deleted `ED2EAF28-7260-4872-A8BB-F9EA9189FD79.dat` after failed receipt/sign-in. On normal game exit, it recreated the file and successfully wrote 20 bytes. The filename is an SDK constant, not a published account identifier. Forwarded API hooks produced duplicate observations of the same operation chain; these were not counted as independent deletions.

This explains temporary absence in that test. The precise payload semantics remain unknown; the file has not been established as a particular token or gameplay save. Investigators backed up existing state and did not delete or restore the original to force this behavior.

## Open questions

- Why does Toshiba's legacy receipt entitlement differ from CLIP satisfaction on the tester-reported owning account?
- Which exact backend product, account, device, and entitlement conditions govern that discrepancy?
- What caused the original instrumentation/unwind failure?
- What is the complete schema and integrity model of the persistent CLIP store?

None of these unknowns changes the original F1 conclusion: the unlicensed negative control received no content key, while the two licensed Xodus controls succeeded.
