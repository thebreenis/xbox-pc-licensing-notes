# Research map and already-tested approaches

Primary goal: Windows 8-era games signing into Xbox Live, publishing Presence, and syncing legitimately earned achievements. This index helps avoid repeating unchanged experiments; a negative result is bounded evidence, not proof that every related approach is impossible.

## Where to look

| Question | Read first |
| --- | --- |
| What is pop.bin? Does editing its AppId work? | [Receipt comparison and signature tests](november-4-auth-and-receipts.md#four-game-receipt-comparison) |
| Can another game's receipt work? | [October receipt controls](october-auth-and-achievements.md#receipt-controls-including-a-free-game) and [TY-to-Crash-Course test](november-4-auth-and-receipts.md#ty-receipt-reached-crash-course-authentication-but-was-rejected) |
| Why does sign-in work but Presence/achievements fail? | [Rayman/Adera comparison](october-auth-and-achievements.md#rayman-and-adera-same-platform-field-different-authorization-outcomes) |
| Is the receipt broker using another account? | [Outgoing CLIP account correlation](windows-8-follow-up.md#clip-may-submit-multiple-user-credentials) |
| Why does a correct GUID or signed receipt still fail? | [Receipt versus CLIP decisions](windows-8-follow-up.md#receipt-issuance-and-clip-satisfaction-are-separate-decisions) |
| Is a missing file or RPC error a licensing failure? | [Tracing stability and SDK file lifecycle](windows-8-follow-up.md) |
| Does HTTP 200 or SDK success mean an unlock synced? | [Asynchronous and backend boundaries](october-auth-and-achievements.md#observed-natural-unlock-pipeline-and-delayed-delivery) |

## Tested approaches: do not repeat unchanged

| Approach and context | Recorded result | What would justify another test? |
| --- | --- | --- |
| Edit another title's receipt AppId to Crash Course | Four examined receipts lost signature validity | New legitimate service-issued receipt or new verification evidence, not another string edit |
| Use unchanged Adera receipt in Rayman; TY receipt in Crash Course | Authentication faults; no successful token established | A materially different, documented supported identity relationship |
| Remove Adera receipt input | Tested native path failed | A genuinely different authentication path; do not infer full service authorization from UI sign-in |
| Switch to newAuth=false for Rayman/Crash Course/Taptiles | Sign-in or reads could work; tested service writes/Presence still failed | Capture a different accepted request and independently verify its result |
| Substitute TY's working SDK into Rayman | Same authentication fault; original SDK restored | Evidence of a specific SDK defect that the replacement fixes |
| Treat platform 17 as inherently invalid | Adera used platform 17 successfully; Rayman failed with the same field | Controlled evidence isolating the field, rather than the wording of code 11 alone |
| Add UTC Z to Rayman's history timestamp | Still HTTP 400/code 6; successful Adera control lacked Z | Evidence of a changed endpoint contract or a different malformed field |
| Treat bridge profile HTTP 200 as full authorization | Profile reads worked while Presence/history failed | Endpoint-specific accepted writes and readback |
| Use modern tokens/Title ID or additional Progress variants for PAC-MAN Toshiba Sparkster | Tested variants did not sync the legacy achievement | A new supported contract/credential context and explicit server-side readback |
| Compare only CLIP and broker Authorization hashes | Different opaque credentials were inconclusive; body contained the matching account reference | Inspect scoped body/account references, not more header-hash comparisons alone |
| Treat all-zero/invalid AppId as the sole blocker | Correct GUID could obtain Toshiba receipt, but subsequent stages still failed | Inspect the next boundary after confirming the actual outgoing GUID |
| Treat local SDK success as final achievement acceptance | Minecraft comparison showed local success before later tester-confirmed profile unlocks | Capture transport/backend evidence and allow for delayed earlier submissions |
| Reuse broad LicenseManager hooks after tracing crashes | STATUS_BAD_FUNCTION_TABLE; reduced probe survived two fresh tests | Isolate the unsafe hook/ABI/unwind behavior; do not classify induced RPC failure as entitlement evidence |

Sources: [October cases](october-auth-and-achievements.md), [November 4 cases](november-4-auth-and-receipts.md), [bridge cases](november-4-6-launch-and-bridge.md), and [November 9-10 follow-up](windows-8-follow-up.md). The Minecraft Mobile comparison is explicitly separate from the legacy direct-request paths.

## Still open and worth investigating

- Why Toshiba's service-issued legacy receipt and CLIP satisfaction disagree on the tester-reported owning account.
- Whether specific legacy-to-modern product mappings or OEM applicability explain that discrepancy; neither is proven.
- What Xbox rejects after receiving the apparently correct signed receipt in the failed title contexts.
- Whether the friend's machine sends the same account references; local account correlation is not a substitute for that test.
- The precise unsafe instrumentation interaction and the full CLIP storage schema/integrity model.

For any new test, record the hypothesis, title/build, effective SDK settings, native versus bridged path, request identity, credential source (without credentials), endpoint/contract, response, and independent readback. Change one relevant variable, preserve saves/receipts, and explain what result would distinguish the competing explanations.

## Timing and confidence

The README's 24-hour receipt / four-hour token timing and stale-receipt refresh behavior were explicitly reported by the maintainer from their testing. The published summaries do not include a controlled timing trace proving universal behavior; retain that provenance rather than converting it to a documented Microsoft guarantee.

A useful answer is: "This was tested in this configuration and failed at this boundary; repeating it unchanged is unlikely to add information." Avoid "this can never work" unless the evidence actually establishes that stronger claim.

## Detailed evidence archive

The [local-reference audit](reference-audit-2026-11-10.md) maps all 39 retained reference sections and E01-E29 into the public repository. The [Log Records folder](../Log%20Records/README.md) includes sanitized captured formats; consult its provenance notes before treating summaries as wire captures. The [reference snapshot](local-reference-snapshot.md) preserves historical tests and their later corrections.
