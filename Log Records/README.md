# Log records and protocol examples

These sanitized records can be supplied to Codex or another research assistant together with the [research map](../docs/research-map.md) and [reference audit](../docs/reference-audit-2026-11-10.md). They preserve observed formats, event ordering, public game identities, status codes, and useful negative results without publishing usable credentials or personal data.

## How to interpret the files

- Every newly replaced sensitive string uses **only `0` characters, with the same character count as the replaced value**. Entire sensitive GUIDs are 36 zeroes, including replacement of their hyphens. This preserves length, not GUID/date/Base64 semantics or cryptographic validity.
- Existing source redactions such as `REDACTED`, hashes-only fields, or recorded lengths remain distinguishable. Their unknown original values/lengths are not invented. Newly zeroed fingerprints cannot be compared to establish account equality; use recorded comparison results.
- XML signatures and receipt digests no longer validate after sanitization. These examples are not usable receipts, tokens, licenses, or installation payloads. Protected/encoded payloads are zeroed; a separately exported decoded receipt shows its XML structure.
- JSON is reserialized for readability, so whole-file byte counts are not original wire lengths. Recorded request/response byte counts refer to the original capture. In Store wire files, integer `bytes` arrays represent sanitized response text and can be decoded as UTF-8.
- Public package/product/content/Title IDs are retained. Selected exact Windows/SDK binary hashes needed to scope analysis are retained; account, receipt, token, device and private-file fingerprints are not.
- E01-E29 are retained evidence snapshots: some are event excerpts, others are analytical summaries or validation reports. They are **not all raw logs**. Native authentication JSONL records include SOAP element/attribute inventories and lengths, not an invented complete SOAP packet.
- [manifest.json](manifest.json) identifies each source by basename, record type, number and character lengths of replacements, and SHA-256 of the sanitized published file. It does not contain the original private-file hashes or a reversible replacement map.

Earlier hypotheses must be read alongside later corrections. In particular, use E24 for error arithmetic and E28 for stable account correlation; do not treat instrumentation-induced RPC failure as an entitlement result.

## Files

| Record | Provenance/type |
| --- | --- |
| [E01-comparison-sanitized.json](E01-comparison-sanitized.json) | retained evidence snapshot; summary or event excerpt, not necessarily raw wire |
| [E02-preservation-final.json](E02-preservation-final.json) | retained evidence snapshot; summary or event excerpt, not necessarily raw wire |
| [E03-toshiba-guid-first-write-live.jsonl](E03-toshiba-guid-first-write-live.jsonl) | retained evidence snapshot; summary or event excerpt, not necessarily raw wire |
| [E04-pacman-standard-toshiba-receipt-auth-comparison.json](E04-pacman-standard-toshiba-receipt-auth-comparison.json) | retained evidence snapshot; summary or event excerpt, not necessarily raw wire |
| [E05-adera-toshiba-receipt-signatures.json](E05-adera-toshiba-receipt-signatures.json) | retained evidence snapshot; summary or event excerpt, not necessarily raw wire |
| [E06-toshiba-guid-source-findings.json](E06-toshiba-guid-source-findings.json) | retained evidence snapshot; summary or event excerpt, not necessarily raw wire |
| [E07-toshiba-native-guid-source-proof.json](E07-toshiba-native-guid-source-proof.json) | retained evidence snapshot; summary or event excerpt, not necessarily raw wire |
| [E08-pacman-original-comparison.json](E08-pacman-original-comparison.json) | retained evidence snapshot; summary or event excerpt, not necessarily raw wire |
| [E09-toshiba-progress-presence-final-findings.json](E09-toshiba-progress-presence-final-findings.json) | retained evidence snapshot; summary or event excerpt, not necessarily raw wire |
| [E10-store-modern-license-results.json](E10-store-modern-license-results.json) | retained evidence snapshot; summary or event excerpt, not necessarily raw wire |
| [E11-toshiba-contenttype1-probe-result.json](E11-toshiba-contenttype1-probe-result.json) | retained evidence snapshot; summary or event excerpt, not necessarily raw wire |
| [E12-adera-license-manager-positive-smoke.json](E12-adera-license-manager-positive-smoke.json) | retained evidence snapshot; summary or event excerpt, not necessarily raw wire |
| [E13-license-service-record-comparison.json](E13-license-service-record-comparison.json) | retained evidence snapshot; summary or event excerpt, not necessarily raw wire |
| [E14-content-key-async-load.json](E14-content-key-async-load.json) | retained evidence snapshot; summary or event excerpt, not necessarily raw wire |
| [E15-adera-clip-document-storage.json](E15-adera-clip-document-storage.json) | retained evidence snapshot; summary or event excerpt, not necessarily raw wire |
| [E16-clip-signed-records.json](E16-clip-signed-records.json) | retained evidence snapshot; summary or event excerpt, not necessarily raw wire |
| [E17-toshiba-full-clip-build-validation.json](E17-toshiba-full-clip-build-validation.json) | retained evidence snapshot; summary or event excerpt, not necessarily raw wire |
| [E18-friend-toshiba-clip-results.jsonl](E18-friend-toshiba-clip-results.jsonl) | retained evidence snapshot; summary or event excerpt, not necessarily raw wire |
| [E19-toshiba-catalog-mapping.json](E19-toshiba-catalog-mapping.json) | retained evidence snapshot; summary or event excerpt, not necessarily raw wire |
| [E20-toshiba-mapping-update-tests.json](E20-toshiba-mapping-update-tests.json) | retained evidence snapshot; summary or event excerpt, not necessarily raw wire |
| [E21-friend-native-only-results.jsonl](E21-friend-native-only-results.jsonl) | retained evidence snapshot; summary or event excerpt, not necessarily raw wire |
| [E22-account-update-tests.json](E22-account-update-tests.json) | retained evidence snapshot; summary or event excerpt, not necessarily raw wire |
| [E23-toshiba-crashcourse-account-results.json](E23-toshiba-crashcourse-account-results.json) | retained evidence snapshot; summary or event excerpt, not necessarily raw wire |
| [E24-receipt-error-mapping-static.json](E24-receipt-error-mapping-static.json) | retained evidence snapshot; summary or event excerpt, not necessarily raw wire |
| [E25-crashcourse-live-receipt-mapping.json](E25-crashcourse-live-receipt-mapping.json) | retained evidence snapshot; summary or event excerpt, not necessarily raw wire |
| [E26-clip-ticket-account-correlation.json](E26-clip-ticket-account-correlation.json) | retained evidence snapshot; summary or event excerpt, not necessarily raw wire |
| [E27-crashcourse-ui-signin.json](E27-crashcourse-ui-signin.json) | retained evidence snapshot; summary or event excerpt, not necessarily raw wire |
| [E28-stable-clip-probe-and-sdk-file-lifecycle.json](E28-stable-clip-probe-and-sdk-file-lifecycle.json) | retained evidence snapshot; summary or event excerpt, not necessarily raw wire |
| [E29-crashcourse-stable-full-installer.json](E29-crashcourse-stable-full-installer.json) | retained evidence snapshot; summary or event excerpt, not necessarily raw wire |
| [adera-store-wire.json](adera-store-wire.json) | captured event records; response byte arrays contain sanitized text |
| [toshiba-store-wire.json](toshiba-store-wire.json) | captured event records; response byte arrays contain sanitized text |
| [standard-store-wire.json](standard-store-wire.json) | captured event records; response byte arrays contain sanitized text |
| [adera-receipt.xml](adera-receipt.xml) | captured receipt; signature no longer valid after redaction |
| [toshiba-receipt.xml](toshiba-receipt.xml) | retained supplied receipt; signature no longer valid after redaction |
| [adera-decoded-policies.json](adera-decoded-policies.json) | captured decoded policies, not a fabricated schema |
| [pacman-standard-native-auth.jsonl](pacman-standard-native-auth.jsonl) | captured SDK/SOAP structural trace; source already omitted some values |
| [pacman-vaio-persistent-native-auth.jsonl](pacman-vaio-persistent-native-auth.jsonl) | captured SDK/SOAP structural trace; source already omitted some values |
| [standard-cached-receipt.xml](standard-cached-receipt.xml) | captured cached receipt from comparison backup; redacted, not signature-verifiable |
| [adera-receipt-response-1.xml](adera-receipt-response-1.xml) | extracted sanitized response chunk, not reconstructed; encoded receipt redacted |
| [toshiba-receipt-response-1.xml](toshiba-receipt-response-1.xml) | extracted sanitized response chunk, not reconstructed; encoded receipt redacted |
| [standard-receipt-response-1.xml](standard-receipt-response-1.xml) | extracted sanitized response chunk, not reconstructed; encoded receipt redacted |
