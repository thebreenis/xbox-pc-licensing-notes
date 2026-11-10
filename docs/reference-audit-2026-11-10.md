# Local-reference audit, 2026-11-10

Compared the public repository with all 39 sections of the maintained Windows 8 licensing reference and retained evidence E01-E29. The [sanitized reference snapshot](local-reference-snapshot.md) preserves the detailed historical sequence; [Log Records](../Log%20Records/README.md) supplies machine-readable evidence and protocol examples. Original F1 and earlier October/November case studies remain intact.

## Important details added or made accessible

- **The bad AppId is not always zero.** Allocation/constructor/refresh/cache-copy observations found unchanged stale bytes on the no-license path, including zero and nonzero results. The first observed propagation is established; the prior owner/writer of recycled heap memory is not. E03/E06/E07 retain the investigation, with sensitive/uncatalogued identifiers zeroed in publication.
- **Cached authentication and fresh licensing differ.** In the controlled native comparison, both Standard DX+ and Toshiba returned `0x803F8001` without a license object and failed fresh receipt acquisition, while Adera succeeded. Standard's earlier successful Xbox authentication used its cached receipt; it was not a successful fresh Store acquisition control (E01/E04).
- **The request is a bodyless POST.** The observed AcquireReceipt request used `/Licensing/License/AcquireReceipt/6.2/0?productId=<GUID>&receiptType=Full`, zero body bytes, and WinHTTP flags `0x00800000`. All three responses were HTTP 200; the XML envelope distinguished error `0xc03f300c` from success with `ReceiptBase64Encoded`. Asynchronous read completion was needed to capture the response.
- **Native lookup uses contentType 1.** The initial explicit contentType 0 probe is historical, not the final native-equivalent control. Successful wrapper HRESULTs must be checked alongside object presence and output values (E10-E12).
- **The CLIP content ID is reproducibly derived.** For the examined contentType 1 path, lowercase `<package name>:<publisher suffix>:win8xappx`, encode UTF-16LE without NUL, SHA-256, and interpret the first 16 digest bytes as a little-endian Windows GUID. Toshiba yields `79027a65-d615-f76d-a36d-e56124399ad5`; Adera yields `a813df52-acd9-9362-9d5f-79d4452447a9`. Version/architecture are not inputs; this is an identifier, not a secret key (E14).
- **The missing registry value was not the explanation.** The examined registry branch belonged to a different category; native contentType 1 used the correct derived key. A cache miss alone was insufficient evidence, but the later worker explicitly reported no key document and online replenishment returned zero keys/leases with satisfaction code 1 (E13/E14).
- **Adera's policy JSON was decoded, not decrypted.** Exact Base64 payloads were located in ASCII and UTF-16LE inside tokens.dat. Surrounding Version 5 License XML records verified their digests/signatures against embedded keys; root trust and whole-database integrity were not established. Static ClipInstallLicense anchors do not prove a successful acquisition-to-disk transaction (E15/E16).
- **The public catalog already contains a legacy mapping.** Toshiba product `9WZDNCRDMWQF`, SKU `0010`, maps its content ID to satisfying keys including `wes:App:2d4168ad-4092-4ea7-92fd-1c81e8191304:Full`. Thus a completely absent published legacy mapping is not supported; a missing effective account entitlement, base-app versus add-on distinction, or applicability problem remains unresolved. The receipt's durable product string is not itself proof of the base-app entitlement used by CLIP (E19/E21).
- **The full records preserve negative experiments.** Temporary source/cache correction changed the actual request but did not grant a license; modern Store license API queries differed between Adera and Toshiba; native-only friend tests still had no satisfying keys; and Standard-token replay in VAIO produced service failures rather than proving title-independent write authority. The retained VAIO trace includes leaderboard POST 401 responses alongside a GET 200; those are not achievement acceptance evidence.
- **Diagnostic mistakes are retained as corrections.** `0x80073DF6` is the observed unlicensed-user mapping, not registration failure `0x80073CF6`. Failed IAsyncInfo inspection did not negate subsequently delivered Toshiba receipt XML. Broad instrumentation caused invalid-function-table crashes; two reduced-probe runs survived, but the exact original hook defect remains unresolved (E23-E28).

## Coverage ledger

| Reference area | Public coverage |
| --- | --- |
| Components, identities, files, XML signatures and token distinction (1-7) | Snapshot 1-7; receipt XML, decoded policy JSON and E04/E05/E10-E12 |
| Bad GUID, request protocol, cached/fresh comparison, temporary correction (8-13) | Snapshot 8-13; three Store wire captures and E01/E03/E06/E07 |
| Presence, achievement failures, error boundaries (14-15) | Follow-up, research map, E09 and native authentication traces |
| Tool coverage, preservation, historical hypotheses (16-21) | Snapshot; dated supersession warning; E02/E17 |
| Async load, content derivation, persistent records (22-25) | Snapshot; E13-E16 and decoded Adera policies |
| Friend tests, catalog mapping, account comparisons (26-32) | Snapshot; E18-E23; public catalog snapshot |
| Error correction, account body correlation, tracing stability, SDK .dat (33-39) | Follow-up; E24-E29 |

No new live authentication, license-store modification, or achievement submission was performed for this publication audit. This is a curated archive of retained evidence, not a claim that every historical raw capture survived cleanup or that missing wire bodies have been reconstructed.
