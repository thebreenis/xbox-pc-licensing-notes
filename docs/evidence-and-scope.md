# Evidence scope and publication rules

## Evidence labels

- **Observed:** a recorded local result, package/catalog metadata, or inspected implementation behavior.
- **Reported by tester:** acquisition type of the two licensed controls; the final clarification that F1 was tested without a license.
- **External finding:** another researcher's documented result, not reproduced merely by reading it.
- **Hypothesis:** an idea considered without sufficient evidence to establish it locally.

## Documentation consolidation

These pages consolidate local reports written at different stages of the investigation. They are summaries, not verbatim raw logs. Later verified results supersede earlier provisional statements:

- A failed wdapp attempt did not prevent subsequent Windows staging and registration.
- Initial installation notes preceded the later launch/licensing failures; registration was never proof of gameplay.
- Initial filename-string inspection preceded successful parsing of the SMD segment entries.
- Xodus was subsequently built and tested, superseding notes written before it was installed.
- Purchased/redemption ownership was discussed hypothetically. The final case description is an unlicensed F1 negative control, not a verified broken entitlement.
- Native ETW attempts yielded no diagnostic events beyond headers; no protocol-level cause was established from them.

## Publication exclusions

No account purchase history, receipts, device lists, account identifiers, local usernames, computer names, personal filesystem paths, tokens, private license blobs, device secrets, CIK key bytes, PFX files, certificates, raw traces, game assets, executables, dependency installers, or releases are published here.

The Markdown files use public product/package IDs only. The encryption-key ID shown in the metadata table is a lookup identifier, not the secret key. No game package download is hosted here.

The repository intentionally contains only Markdown documentation and an allowlist-style `.gitignore`. Raw local evidence remains outside it. Future changes should preserve that boundary and avoid promoting an untested idea into a claimed working result.

## November Windows 8 follow-up boundary

The [Windows 8/8.1 follow-up](windows-8-follow-up.md) summarizes a separate investigation dated November 9–10, 2026. Its observations must not be attributed retrospectively to the October F1/MSIXVC tests. Local Crash Course account correlation, tester-reported Toshiba ownership, and Adera record inspection are distinct evidence contexts. Detailed captures remain private; this repository publishes no raw receipts, account fingerprints, ticket values, license records, or device identifiers.

A missing SDK file is not automatically evidence of lost gameplay progress. In the separate Crash Course test, the Xbox SDK deleted a `.dat` state file after failed receipt/sign-in and recreated a 20-byte file on normal exit. Its payload semantics remain unknown. Preserve such files and observe their lifecycle rather than deleting them or classifying all `.dat` files as disposable caches.
