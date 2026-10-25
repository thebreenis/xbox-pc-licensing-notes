# Xbox PC package and licensing notes

Documentation of Xbox PC / Microsoft Store package installation, MSIXVC metadata, launch diagnostics, and content-license acquisition. The aim is to distinguish package availability, installation success, Xbox sign-in, and authorization to decrypt and run a game.

**F1 2018 PC GP was the test case, tested without a valid license.** It was an unlicensed negative control, not evidence that a purchased Windows entitlement was broken. Minecraft for Windows and Minecraft Dungeons supplied successful purchased and subscription controls, respectively.

The tests documented here were performed on Windows on October 24–25, 2026. Results are observations from that environment, not universal guarantees about Microsoft's services.

## Main findings

- A downloadable encrypted package is not a game license.
- Windows could stage and register the test MSIXVC after missing presentation images were supplied. Registration did not make the game playable.
- The native launcher stopped before the main game executable started. Its update/licensing path returned `0x803F8001`.
- Xodus authenticated successfully and obtained content keys for both licensed controls. The unlicensed test case received an entitlement refusal and no content key.
- File enumeration, administrator ownership, Xbox presence, and package status `Ok` did not establish successful content decryption or gameplay.
- No working license bypass, F1 executable patch, or decryption without the required key was produced.

## Documentation

1. [Package acquisition and Windows deployment](docs/package-and-deployment.md)
2. [Content licensing and CIK files](docs/licensing-and-cik.md)
3. [Native launch diagnostics](docs/launch-diagnostics.md)
4. [F1 test-case metadata and legacy identifiers](docs/test-case-and-identifiers.md)
5. [External research review and open questions](docs/research-review.md)
6. [Tools, versions, and references](docs/tools-and-references.md)
7. [Evidence scope and publication rules](docs/evidence-and-scope.md)

## What is included

Only documentation, public package identifiers, and summarized observations. There are no game files, binaries, dependency installers, releases, private keys, certificates, authentication tokens, account records, memory dumps, or raw network/event captures.

Public content IDs and encryption-key **identifiers** identify packages or keys; they are not secret decryption keys.

Maintained by [TheBreenis](https://github.com/thebreenis). This is independent documentation, unaffiliated with Microsoft, Xbox, Codemasters, EA, or the referenced tool authors.
