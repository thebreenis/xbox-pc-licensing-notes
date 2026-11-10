# External research review and open questions

The [xboneafide/xboxREsearch repository](https://github.com/xboneafide/xboxREsearch), reviewed at commit `78e4fbc249a0d1175aa4ecbda873e1b8d0c5f332`, contains relevant notes on CIK extraction, PC package acquisition, native licensing, and console-specific experiments.

Useful distinctions include wrapped license material versus a usable content key, package lookup versus key issuance, and native Gaming Services credentials versus third-party authentication flows. The repository also describes inspecting an executable already mapped into a running process. That last prerequisite was absent in our F1 test.

Its characterization of the content-license endpoint as effectively retired should not be generalized: our two licensed control titles succeeded against that endpoint. Its console-specific extraction and mount findings likewise do not establish a compatible key or working bypass for a Windows package.

Some statements describe proposed or unresolved paths. In particular, a route described as deterministic should not be treated as reproduced here when the same notes discuss unresolved device provisioning. The repository was used as a source of hypotheses; its tools, interception setup, and console techniques were not executed in this investigation.

## Hypotheses considered locally

- **Cached F1 license:** ruled out as a direction by the tester; no cached-license extraction was attempted in this phase.
- **Alternative legacy content identity:** the tested legacy lookups resolved to the same current PC product; no alternative was established.
- **Native device-attestation issue:** possible in other environments, but the cited device-claim errors were not observed here.
- **Running-process inspection:** no main game process was found, so this could not supply an executable image.
- **Steam comparison:** a readable Steam build could help compare common game logic, asset formats, and platform integration. No Steam build was inspected. It would not by itself supply the Store package's content key.
- **Adding Xbox services to another build:** discussed only as a feasibility question. It would require appropriate title identity, authentication, and compatible game/service integration; copying DLLs or obtaining generic sign-in does not establish working Xbox achievements. No integration or achievement submission was attempted.

## Unresolved

The tests did not obtain a native request/response trace, recover an F1 key, decrypt its executable, or produce a playable modified build. There is no demonstrated license bypass in these notes. The unlicensed F1 result is not a defect that this documentation claims to have repaired.

## Separate October authentication and achievement cases

For the separate October Rayman/Adera comparison and Minecraft's event-based achievement observations, see [October authentication and achievement case studies](october-auth-and-achievements.md). In particular, identical platform fields did not imply identical authorization outcomes, and local event acceptance did not establish immediate upload. These are title-specific experiments, not additional F1 results.

## Later Windows 8 account-context finding

A separate native Crash Course Go trace found multiple user tickets inside the outgoing CLIP request body. One transmitted `localTicketReference` matched the receipt broker's selected Store account identifier. Comparing only outer Authorization headers would miss this relationship; different opaque ticket hashes do not establish different Microsoft accounts.

This narrows one account-mismatch hypothesis for that local Windows 8 run. It neither establishes how F1's native client serialized its request nor proves backend entitlement acceptance. See the [dated follow-up](windows-8-follow-up.md).
