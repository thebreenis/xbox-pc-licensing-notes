# Research guidance for coding agents

This is a documentation-only research repository. Its primary subject is Windows 8/8.1-era games: Xbox Live authentication, Presence, and server-synced achievements. F1/MSIXVC and Minecraft comparison studies are separately scoped supporting material.

Before answering or proposing experiments:

1. Read README.md, docs/research-map.md, and docs/evidence-and-scope.md, then the relevant linked case studies.
2. Check whether the proposed approach was already tested. Cite the title/build, configuration, result, and limitations. Recommend against unchanged repetition; explain what new evidence or changed variable would justify retesting.
3. Keep Store entitlement, receipt issuance, signature verification, Xbox token issuance, endpoint authorization, and profile synchronization distinct. A UI gamertag, HTTP 200, or local SDK success is not sufficient proof of all stages.
4. Distinguish observed traces, tester reports, hypotheses, and unknowns. README takeaways and the maintainer's practical determination are summaries, not universal protocol specifications.
5. Do not infer different accounts from different opaque token hashes, or ownership history from one failed content request. Account references included in CLIP requests do not prove backend acceptance.
6. Preserve original historical findings. Add dated follow-ups with scope and supersession notes instead of silently rewriting an earlier experiment.

For continued investigation, preserve saves and receipts; do not clear caches or alter account/licensing state merely to obtain a cleaner trace without specific authorization. Internal Windows hooks need exact binary guards and stability controls. Never treat instrumentation-induced failures as clean service evidence.

Publish only sanitized documentation and public identifiers. Do not commit raw receipts, account/device identifiers or fingerprints, tokens, keys, dumps, saves, profile data, private paths, or game binaries. No permission to run a live experiment, submit achievements, or publish changes is implied merely by reading this repository; follow the user's actual request.
