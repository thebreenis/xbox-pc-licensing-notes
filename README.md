# Windows 8-era Xbox Live licensing research

**PRIMARY SCOPE: Windows 8 and Windows 8.1 games - Xbox Live sign-in, working Presence, and server-synced achievements.**

This repository preserves experiments, successful controls, rejected approaches, and unresolved questions so researchers and coding agents can build on existing work. These legacy games were often tested on newer Windows installations; this is not a claim about every Windows 8 operating-system build or Xbox SDK.

## Top 10 things learned

1. **`pop.bin` is a signed Store receipt, not an Xbox token.** In the games examined, it lives at `%LOCALAPPDATA%\Packages\<PackageFamilyName>\LocalState\Microsoft.Xbox\pop.bin`; changing signed fields such as AppId or expiration invalidates the original signature and does not produce an accepted replacement.

2. **A usable receipt is required by the receipt-backed SDK authentication path we traced.** The SDK can read a cached `pop.bin` or obtain fresh receipt XML, but acquisition failure stopped those attempts before Xbox token issuance; alternate modes sometimes displayed a signed-in state without full service functionality.

3. **The Store broker obtains receipts using its selected Store account.** Do not automatically equate that account with the one displayed by the Xbox app; the latest local Crash Course trace linked the broker's account reference to a user ticket actually included in CLIP's outgoing request.

4. **Receipts carry a game-specific application identity.** PAC-MAN Championship Edition DX+ Standard, VAIO, and Toshiba have distinct identities, and copying Standard's working receipt into VAIO or Toshiba did not enable successful modern Xbox authentication; universal account portability has not been established.

5. **The maintainer observed a 24-hour receipt lifetime and a four-hour Xbox-token lifetime in the tested setup.** In those observations, a stale receipt auto-refreshed when the Store account was licensed for its AppId, while an unlicensed account failed refresh and Xbox sign-in; these are tester-reported timings, not a published universal service contract.

6. **The correct receipt-request GUID matters, but it does not grant ownership.** PAC-MAN Toshiba's failed package-license lookup produced an invalid GUID; substituting its correct legacy GUID enabled receipt issuance on the friend's reportedly licensed account but did not resolve modern Xbox authentication.

7. **A signed PAC-MAN Toshiba receipt did not guarantee Xbox authentication.** A possible missing or unsupported link between legacy entitlements and modern licensing identities could affect OEM editions such as PAC-MAN VAIO and Toshiba, but this remains a hypothesis; the receipt/CLIP discrepancy was directly observed for Toshiba.

8. **PAC-MAN Toshiba showed that legacy receipt issuance and CLIP satisfaction can disagree.** The legacy service returned a full-license receipt while CLIP reported no satisfying entitlement, so recognition by one path does not establish recognition by the other or the account's entire purchase history.

9. **A gamertag, an Xbox token, working Presence, and synced achievements are separate milestones.** Verify each independently: a signed-in UI, successful list read, or accepted Presence heartbeat does not prove that an achievement write was accepted or appeared on the Xbox profile.

10. **An `XBL3.0` token is not automatically universal, and changing SDK flags does not confer new authority.** Tokens that worked for Presence did not necessarily authorize legacy achievement writes; tested token reuse, Progress variants, and a modern Title ID did not establish a working route to PAC-MAN Toshiba's legacy achievement list.

### Final determination

**For the receipt-backed path studied here, the practical requirement for full Xbox Live functionality is a fresh, authentic `pop.bin` for the specific game, obtained through an account that genuinely owns that game's license. The receipt is presented to Microsoft's servers during authentication; editing, fabricating, or substituting another game's receipt did not produce an accepted substitute in our tests.**

This is a necessary starting point, not a guarantee of full functionality: PAC-MAN Toshiba demonstrates that even receipt issuance can be followed by Xbox rejection. No working receipt forgery or spoof was demonstrated; that is an empirical conclusion, not proof that every conceivable future method is impossible. It also does not assert that the Xbox account consuming a receipt must always be the account that obtained it.

## Start here: questions and continued research

- [Research map and already-tested approaches](docs/research-map.md): find the relevant result before proposing another experiment.
- [Instructions for Codex and other coding agents](AGENTS.md): evidence rules, scope, preservation, and when a repeat test is justified.
- [October authentication and achievement cases](docs/october-auth-and-achievements.md): Rayman/Adera controls, Hexic receipts, token wrappers, and a separately scoped Minecraft event-pipeline comparison.
- [November 4 authentication and receipt cases](docs/november-4-auth-and-receipts.md): signature mutations, cross-game receipts, SDK swaps, and rejected achievement writes.
- [November 4-6 launch and bridge cases](docs/november-4-6-launch-and-bridge.md): local launch fixes versus online authorization, and capture limitations.
- [November 9-10 Windows 8 follow-up](docs/windows-8-follow-up.md): CLIP records, account correlation, service-tracing stability, and SDK file lifecycle.
- [Evidence scope and publication rules](docs/evidence-and-scope.md).

Findings are build-, account-, endpoint-, and date-specific. Read the linked case before treating a short takeaway as a universal rule; successful local behavior is not proof of server authorization.

## Secondary research: F1/MSIXVC package deployment

The October 24-25, 2026 F1 2018 PC GP study is a separate, smaller part of this repository. F1 was tested without a valid license; Minecraft for Windows and Minecraft Dungeons supplied purchased/subscription controls, and none of the later legacy-game experiments changes those results.

The original findings remain intact: downloading, staging and registering the encrypted package did not grant a license; native launch failed before the main game process; licensed Xodus controls obtained keys while F1 did not. No F1 decryption, playable modified build, or license bypass was produced.

- [Package acquisition and deployment](docs/package-and-deployment.md)
- [Content licensing and CIK files](docs/licensing-and-cik.md)
- [Native F1 launch diagnostics](docs/launch-diagnostics.md)
- [F1 metadata and identifiers](docs/test-case-and-identifiers.md)
- [External research and open questions](docs/research-review.md)
- [Tools, versions, and references](docs/tools-and-references.md)

## What is included

Only documentation, public package identifiers, and summarized observations. There are no game files, binaries, dependency installers, releases, private keys, certificates, authentication tokens, account records, memory dumps, or raw network/event captures.

Public content IDs and encryption-key **identifiers** identify packages or keys; they are not secret decryption keys.

Maintained by [TheBreenis](https://github.com/thebreenis). This is independent documentation, unaffiliated with Microsoft, Xbox, Codemasters, EA, or the referenced tool authors.
