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

The repository contains Markdown documentation and explicitly reviewed sanitized JSON, JSONL and XML examples under `Log Records`, with an allowlist-style `.gitignore`. Unmodified private evidence remains outside it. Do not promote an untested idea into a claimed working result.

## Separate October case-study boundary

The [October authentication and achievement case studies](october-auth-and-achievements.md), added November 10, summarize separate title-specific tests. Minecraft's eventual profile unlocks are tester-confirmed; its local tracing did not individually capture the subsequent collection, upload, and backend stages. Rayman's failed requests are not generalized to every authentication configuration. F1's trial-acquisition API records establish `NotPurchased`; the UI error does not independently establish the backend cause. None of these additions changes the original F1 negative-control conclusion.

## November Windows 8 follow-up boundary

The separate [November 4 case studies](november-4-auth-and-receipts.md) distinguish traced Crash Course endpoint results and receipt rejection, offline receipt-signature checks, tester-reported TY login restoration, and Rayman's recorded level-select crash. Receipt dependence in TY does not establish universal portability, and an HTTP 200 response does not by itself establish successful authentication.

The initial Rayman crash account is followed by the verified asset-filename repair and tester-confirmed gameplay in build `1.1.0.17`; that repair does not establish service authorization. Later Rayman tests verified receipt signature integrity but still observed authentication faults, including after a TY SDK substitution. A timestamp change did not resolve the history failure. Taptiles sign-in and locally earned achievements likewise did not establish accepted writes; its captured service errors did not include numeric HTTP status.

The [Windows 8/8.1 follow-up](windows-8-follow-up.md) summarizes a separate investigation dated November 9â€“10, 2026. Its observations must not be attributed retrospectively to the October F1/MSIXVC tests. Local Crash Course account correlation, tester-reported Toshiba ownership, and Adera record inspection are distinct evidence contexts. Detailed captures remain private; this repository publishes no raw receipts, account fingerprints, ticket values, license records, or device identifiers.

A missing SDK file is not automatically evidence of lost gameplay progress. In the separate Crash Course test, the Xbox SDK deleted a `.dat` state file after failed receipt/sign-in and recreated a 20-byte file on normal exit. Its payload semantics remain unknown. Preserve such files and observe their lifecycle rather than deleting them or classifying all `.dat` files as disposable caches.

The [November 4â€“6 local launch and bridge follow-up](november-4-6-launch-and-bridge.md) adds a pinned Rayman bridge test with profile success but failed presence/history, plus Dragon's Lair, Minion Rush, and Imperia observations. The proposed paired native-versus-bridge presence test was never executed. Dragon's Lair's verified UI change was not a full playthrough or service-authorization result; Minion Rush's validated asset import did not establish progress beyond loading. Imperia's bounded negative token capture does not establish absence of the protocol, and its account-switch experiment did not confirm that the account picker reappeared. These findings leave the F1 results unchanged.

## Sanitized capture publication, November 10

[Log Records](../Log%20Records/README.md) expands the earlier documentation-only publication policy: it includes sanitized captured events and receipt/policy examples, plus retained summaries. It does not publish original usable receipts, tokens or license documents. Each newly replaced sensitive string is zero-filled at its original character length; source redactions of unknown original length remain labeled rather than reconstructed. Signatures are invalid after sanitization, and all-zero fields must not be mistaken for actual service output. Public product identities and selected binary hashes remain where needed to interpret the experiment. The manifest hashes the sanitized exports only.
