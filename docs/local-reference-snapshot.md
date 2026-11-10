# Retained local reference: publication snapshot

Published 2026-11-10. Sanitized historical reference, retained for completeness. **Later dated sections supersede earlier unresolved questions and hypotheses.** In particular, sections 23-25 resolve earlier content-key/storage questions, and sections 32-39 correct error arithmetic and account/tracing conclusions. Read the current [follow-up](windows-8-follow-up.md) and [research map](research-map.md) first. Local work/output paths are provenance labels, not files included in this repository. Redacted values use same-length zero strings; signatures cannot be validated from this copy.

# Windows 8 Store and Xbox licensing investigation reference

Last updated: 2026-11-09. Revision: 6. Maintained reference for this workspace.

This reference preserves the investigation of Windows 8 and 8.1 game licensing, Store receipt acquisition, `pop.bin`, Xbox authentication, and the diagnostic tooling used to distinguish failures at each layer. The strongest evidence currently concerns PAC-MAN DX+ Standard, Toshiba, and Adera. Other games appear where their results illuminate the same mechanisms.

The central finding is that **Store licensing, receipt acquisition, Xbox authentication, and Xbox service authorization are separate stages**. Success at one does not prove success at another. Standard can authenticate using an existing receipt while a fresh Store receipt request fails. Adera provides a successful fresh-acquisition control. Toshiba's bad request GUID originates in the Store broker's licensing context before the receipt URL is constructed.

## Reading and updating this reference

Evidence labels used below:

- **Observed:** recorded by a local trace, comparison report, or verification result.
- **Interpretation:** explanation supported by observations but not independently proven at every boundary.
- **Historical:** an earlier test or user report; it must not be treated as current state without rechecking.
- **Unknown:** a question that remains open or a format whose internal schema has not been decoded.

This is an investigation record, not a complete specification of Microsoft's private interfaces. Internal names, COM layouts, and offsets must be tied to the examined Windows binary. Do not turn an observed numeric argument into a universal enum definition without further evidence.

The [evidence index](../Log%20Records/README.md) contains retained, small supporting artifacts and their hashes. Full tokens, signed receipts, account-specific cache data, and game assets are deliberately not embedded here. Private originals may remain in `work` or the user's application data. Preservation of this document does not authorize deleting those originals.

For every update: record the date, affected build/account context, exact observation, evidence filename, preservation checks, and whether a statement supersedes an earlier conclusion. Add a dated entry to the change history. Keep negative results, but do not present obsolete hypotheses as current conclusions.

## 1. Components and boundaries

| Component | Role in the observed system | What its success does not prove |
|---|---|---|
| Installed APPX and package registration | Establish package name, publisher identity, version, executable and resources | That the current account has a valid Store licensing record |
| Game's Store startup check | Consult licensing APIs or gate startup | That bypassing the gate creates a license or repairs Store binding |
| Trial and product ownership checks | Determine trial/full behavior and optional content availability | That Xbox authentication will accept a receipt |
| Windows Store client API | Exposes `CurrentApp.AppId`, licensing information and asynchronous receipt acquisition | That every successful HRESULT contains meaningful data |
| Package-associated RuntimeBroker | Hosts observed Store commerce/licensing implementation and sends receipt HTTP requests | That the license lookup returned an object |
| Application license manager | Looks up a license object for a package and content type | That an online receipt will be issued or accepted by Xbox |
| Store receipt service | Returns an error or signed XML receipt | That Xbox accepts the receipt for the requested title |
| Xbox SDK | Uses receipt/account/title information in its authentication path | That all service calls are authorized by the resulting token |
| Xbox Presence and achievement services | Apply endpoint-specific contracts and authorization | That success on Presence implies an achievement write succeeds |

Observed logical flow:

```text
Registered package identity
  -> Store licensing context
  -> GetLicenseForInstalledPackage(packageFullName, contentType, flags)
  -> license object, if found
  -> content information -> Windows Store AppId
  -> broker cached AppId -> CurrentApp.AppId
  -> GetAppReceiptAsync -> AcquireReceipt HTTP request
  -> ReceiptResponse -> signed Receipt XML
  -> game/SDK receipt cache, commonly pop.bin
  -> Xbox authentication request with title and account context
  -> issued token, or SOAP/authentication fault
  -> Presence / achievement requests and independent verification
```

An existing `pop.bin` can allow the SDK to begin from the cached-receipt stage. Calling `GetAppReceiptAsync` directly exercises acquisition even when `pop.bin` exists; the latest Adera test did not require deleting its receipt.

## 2. Identities that must not be conflated

| Identity | Form and role |
|---|---|
| Package name | Manifest identity name, such as `NAMCOBANDAIGamesInc.2685271D3260E` |
| Package family name, PFN | Package name plus publisher-derived suffix; does not contain the package version |
| Package full name | Includes name, version, architecture, resource field and publisher suffix; used in the observed license lookup |
| Legacy Store AppId | GUID used by the legacy Store API and receipt request's `productId` |
| Receipt `AppId` | Signed XML field; observed as either PFN or legacy GUID depending on the product/receipt path |
| Store catalog Product ID / SKU | Catalog identifiers; not automatically an Xbox Title ID or a legacy AppId |
| Receipt `ProductId` | Product/add-on identifier within a `ProductReceipt`; may be a human-readable legacy product string |
| Xbox Title ID | Numeric title identity; may have a different ID exposed by newer services |
| XUID | Xbox user identity; does not identify the game or Store product |
| Receipt IDs | Receipt/app/product transaction identifiers; not interchangeable with package or Xbox title identity |

An uppercase versus lowercase GUID is not itself an identity mismatch. A GUID versus a PFN is a substantive identity-format difference. A correct request GUID can still produce a receipt with a PFN `AppId`, as Adera demonstrates.

### Known identity catalog

| Item | Standard DX+ | Toshiba DX+ | Adera |
|---|---|---|---|
| Package name | `NAMCOBANDAIGamesInc.PAC-MANChampionshipEditionDX` | `NAMCOBANDAIGamesInc.2685271D3260E` | `Microsoft.Adera` |
| Publisher suffix | `gdy2aq6ez762w` | `gdy2aq6ez762w` | `8wekyb3d8bbwe` |
| Examined installed version | `1.0.1.2` | `1.0.1.2` | `2.5.2.34894` |
| Architecture | x86 | x86 | x86 |
| Legacy Store GUID confirmed in retained evidence | Not recorded here | `2d4168ad-4092-4ea7-92fd-1c81e8191304` | `77f0d9b5-87ac-4816-87d5-76b75afcb6db` |
| Legacy Xbox Title ID in auth comparison | `1313676122` | `1313676124` | Not needed for this comparison |
| Modern-service title identity tested | Not recorded here | `2016983674` | Not needed for this comparison |

Example Toshiba full name:

```text
NAMCOBANDAIGamesInc.2685271D3260E_1.0.1.2_x86__gdy2aq6ez762w
```

VAIO is a separate package, `NAMCOBANDAIGamesInc.PAC-MANChampionshipEditionDXfo_gdy2aq6ez762w`. Distinct package families allow separate installations. Do not infer interchangeable receipts from similar executables or shared assets.

## 3. Files, directories and formats

| File or location | Format / purpose | Handling rule |
|---|---|---|
| `.appx` | Packaged application archive containing executable, resources, manifest and signing metadata | Distinguish original Microsoft-signed package from locally rebuilt/signed package |
| `AppxManifest.xml` | Package identity, dependencies, applications and capabilities | Version and identity changes affect registration; retain original version by user requirement |
| `AppxBlockMap.xml` | Package block/hash metadata | Rebuilding payload requires coherent packaging/signing, not arbitrary ZIP edits |
| `AppxSignature.p7x` and `AppxMetadata/CodeIntegrity.cat` | Package signature and integrity-related artifacts | A package signature is separate from a receipt signature |
| `StoreManifest.xml` | Store-related package metadata | Presence alone does not prove a valid installed license binding |
| Game `.exe` | PE executable containing game logic and references to platform/SDK components | Game patches do not automatically repair broker-side state |
| Xbox SDK DLL | Native Xbox client implementation | Record SHA-256 and architecture before applying hooks or comparing offsets |
| `xbl.spa` | Title-specific Xbox configuration/metadata resource | Full binary schema is not established here; do not equate it with a receipt or token |
| `pop.bin` | In the examined cases, signed Store receipt XML despite the `.bin` extension | Preserve exact original bytes; never infer a token format from the filename |
| `receipt.txt` | Diagnostic export of full receipt XML | Can contain the receipt before it is cached as `pop.bin`; not automatically proof a cache write occurred |
| `*.log` | Human-readable diagnostic lines with timestamps, thread/process details and events | Identify logger build and coverage before interpreting an absence |
| `*.jsonl` | One JSON event per line; helpers and instrumentation | Correlate by process, call ID, origin and time |
| `*.json` | Summaries, metadata, comparison reports, hashes | A summary may omit raw fields; follow its source links |
| `Settings/settings.dat`, `.LOG1`, `.LOG2` | Windows application settings storage and associated logs | May be locked while app is running; not disposable just because called a log |
| `Microsoft.Xbox/*.dat`, `.txt`, XUID subdirectories, `ReceiptKey` | SDK caches and state; complete schemas are not decoded | Treat as user data, not as safe-to-delete diagnostics |
| `.dmp` under `Crash Dumps` | Windows process crash dump | May contain private memory; separate from Xbox SDK text logs |
| `Files.sha256.json` | Redistribution payload integrity manifest | Do not accidentally require a self-referential hash of the manifest itself |

Per-user game data is normally under:

```text
%LOCALAPPDATA%\Packages\<PackageFamilyName>\
    LocalState\Microsoft.Xbox\pop.bin
    LocalState\Xbox SDK Logs\
    LocalState\Crash Dumps\
    RoamingState\
    Settings\
```

The package's executable install location is obtained from package registration. It is not necessarily `%LOCALAPPDATA%\Packages`; current examined packages are registered under `C:\Program Files\WindowsApps\...`. The user-data directory and executable directory must not be confused.

Downloaded DLC and guides can live in LocalState alongside saves. Adera has `Episode1` through `Episode5`, `SG101` through `SG105`, and other content directories. A broad LocalState copy can therefore include both user state and large downloadable content. A broad deletion can destroy both.

## 4. Receipt XML anatomy

Schematic only; values below are placeholders and the example is not a valid signed receipt:

```xml
<Receipt Version="1.0"
         ReceiptDate="YYYY-MM-DDThh:mm:ssZ"
         CertificateId="certificate-identifier"
         ReceiptDeviceId="00000000000"
         PublisherDeviceId="000000000000000000000">
  <AppReceipt Id="0000000000"
              AppId="package-family-or-legacy-guid"
              PurchaseDate="00000000000000000000"
              LicenseType="Full" />
  <ProductReceipt Id="000000000000000000"
                  ProductId="legacy-product-identifier"
                  PurchaseDate="00000000000000000000"
                  ExpirationDate="9999-12-31T23:59:59Z"
                  ProductType="Durable"
                  AppId="app-identity" />
  <Signature xmlns="http://www.w3.org/2000/09/xmldsig#">...</Signature>
</Receipt>
```

`ProductReceipt` is optional in the observed examples. Its absence is not proof of a trial license. Standard's working receipt and Adera's successful receipt had no product records; the examined Toshiba legacy receipt had one.

| Field | Interpretation and limits |
|---|---|
| `ReceiptDate` | Receipt issue timestamp. `Z` denotes UTC. This is not the purchase date |
| `PurchaseDate` | Record-specific date supplied by the service. Do not assume every returned date is the original retail purchase time |
| `LicenseType="Full"` | Signed receipt assertion of full license for the indicated identity; not proof Xbox accepted it |
| `AppId` | Receipt's signed app identity. Its format must be inspected, not inferred from the request |
| `ExpirationDate` | Belongs to the element containing it. A durable product's far-future expiry does not establish receipt freshness or Xbox token lifetime |
| `ProductType="Durable"` | Product record describes a durable entitlement; distinct from the app record and authentication token |
| `CertificateId` | Used to identify the signing certificate. Matching the identifier and verifying a signature are distinct checks |
| Device identifiers | Service-issued device-related identifiers; no complete derivation is established in this investigation |

### Signature structure observed

The pasted Toshiba example used XMLDSIG with:

- Exclusive canonicalization: `http://www.w3.org/2001/10/xml-exc-c14n#`.
- RSA-SHA256 signature method: `http://www.w3.org/2001/04/xmldsig-more#rsa-sha256`.
- Empty reference URI and enveloped-signature transform.
- SHA-256 digest method, a Base64 digest, and a Base64 signature value.

Changing a signed AppId, expiry or license field changes signed content and invalidates the original signature. Editing XML cannot create a new Microsoft-signed identity. Formatting changes require care because validation uses the specified XML transforms and canonicalization, not a casual string comparison.

Separate these questions: Is the XML well formed? Does the digest match? Does the signature verify against the selected key? Is the certificate the intended trusted signer? Does the consuming service accept this identity, freshness and license context? A yes to one is not a yes to all.

The retained Adera/Toshiba signature report records `SignatureValid=true` and `CertificateIdMatches=true` for both examined receipts. That strengthens the conclusion that the GUID-versus-PFN distinction can exist in signed service output; it does not prove that distinction caused Xbox rejection. See evidence E05.

### Dates, expiry and replacement receipts

Historical Toshiba output included an app PurchaseDate exactly 30 minutes before ReceiptDate, a much older product PurchaseDate, and a far-future product expiry. These are observations of that output, not a universal rule for legacy receipts or `newAuth=false`.

The user adopted a redistribution policy of never including `pop.bin`, noting approximately 24-hour usefulness in their tests. **A universal 24-hour validity rule has not been demonstrated for all receipts, products and consumers.** Likewise, issuance of a new receipt has not been proven to invalidate every older receipt. A historical Hexic test reportedly continued to work after restoring an older receipt. Preserve these as separate observations, not a single expiry model.

## 5. Receipt XML is not an Xbox authentication token

`pop.bin` in these tests is a signed purchase/license receipt. A SAML assertion and a JWT are authentication-token representations. An `XBL2.0` or `XBL3.0` authorization string belongs to a later Xbox authentication/service stage. They must not be categorized solely by whether they contain XML or Base64.

`Receipt` XML is not a SAML token merely because SAML also uses XML. Standard's working `pop.bin` is not a JWT merely because it leads to successful authentication. Receipt acquisition and Xbox token issuance can involve different services, identities, error codes, and caches.

`newAuth` and `useSamlToken` are SDK/configuration controls examined in this project. Changing them can select different Xbox authentication behavior. Current evidence does **not** establish that they choose GUID-versus-PFN receipt XML. The direct `CurrentApp.GetAppReceiptAsync` comparisons did not depend on those SDK controls.

Historical Toshiba tests with `newAuth=false` displayed a signed-in state, but this was not equivalent to confirmed Presence and achievement authorization. Reusing a token from a different title or client does not automatically change the service's authenticated title context. The prefix `XBL3.0` alone is not evidence that a token is universally authorized.

## 6. What the license-instance object is

The object referred to in the traces is a runtime COM interface returned by the application license manager. It is an in-memory object/interface, not `pop.bin` and not a directly identified disk file.

The observed client path creates or obtains the license manager, asks it for a package license, and, when an object exists, queries content information for the Windows Store AppId. The underlying service's persistent storage, index schema, and exact account-to-package binding logic have not yet been fully traced.

A Store startup check may trigger these APIs, but bypassing a game's startup check does not create a genuine license-manager object. Conversely, returning a convenient value from a game-side getter does not necessarily change the broker's independently cached GUID.

The local user does not claim ownership of Toshiba; the friend reports owning it. Those are different test contexts. The friend's reported receipt issuance after request-GUID correction is evidence about that request/account combination, not proof the local installed-package lookup must succeed on either machine.

## 7. Observed COM interfaces and native lookup

These identifiers and slots are from the current diagnostic source, not a promised stable public ABI:

| Interface or class | Identifier | Observed operation |
|---|---|---|
| ApplicationLicenseManager class | `22f5b1df-7d7a-4d21-97f8-c21aefba859c` | Created through COM |
| IApplicationLicenseManager | `90e2000c-b946-42fa-892f-94506f30ca4f` | Slot 6: installed-package license lookup |
| Content information interface | `19a543ba-bad2-4739-83c8-177b3beac2ef` | Slot 4: Windows Store AppId |
| Returned license interface | IID not catalogued here | Slot 13: GetContentInfo with requested IID |
| CurrentApp statics interface | `D52DC065-DA3F-4685-995E-9B482EB5E603` | Examined x86 slots 8 AppId and 12 app receipt acquisition |
| IAsyncInfo | `00000036-0000-0000-C000-000000000046` | Status/error inspection for receipt operation |

Observed lookup arguments:

```text
GetLicenseForInstalledPackage(
    packageFullName = exact registered package full name,
    contentType = 1,
    flags = 0x20000000,
    out licenseObject
)
```

The explicit probe was initially using `contentType=0`. It has been corrected to **1**, matching the native trace. Old positive-smoke reports may still show 0 and must be read as historical tests. The complete semantic enum mapping of content types remains unverified; do not call 0 or 1 a token format, MIME type, or new/old authentication switch.

The probe also mirrors observed COM impersonation/proxy settings. It uses `CoCreateInstance` with class context `0x15`, then `CoSetProxyBlanket` with default authentication/authorization selections, impersonation level 4 and capabilities `0x800`. Differences here can change results and should be captured in comparative tests.

An explicit probe is marked `origin=explicit-read-only-probe`; intercepted game/Store activity is marked `native-store-call`. Equivalent arguments do not make an explicit diagnostic call proof of every timing-dependent native behavior.

## 8. Where the incorrect GUID originates

The latest first-write trace observed Toshiba's AppId bytes at allocation return, constructor entry/exit, license refresh and server cache initialization. In that run they were:

```text
Raw GUID memory bytes: 00000000000000000000000000000000
Formatted GUID:       000000000000000000000000000000000000
```

Windows GUID memory uses little-endian storage for the first three fields; raw hex and textual GUID therefore differ in ordering.

Observed sequence:

1. A 0x1e0-byte context allocation returned with those bytes already in the GUID field.
2. The constructor left that field unchanged; no direct GUID-field writes were observed in the instrumented constructor range.
3. License refresh returned success at its wrapper boundary but had no license object and did not replace the GUID.
4. `InitializeIdsFromLicense` copied the bytes into the commerce server's AppId cache.
5. `CurrentApp.AppId` returned the same value with success.
6. The receipt request used it as `productId`.

An earlier fresh process had all-zero bytes at allocation and propagated all zeroes. Another run had a different nonzero tail. Standard later returned bytes resembling stale UTF-16 characters. These observations support **uninitialized memory exposed on a no-license path**, not an intentionally selected fallback identity.

The original writer of recycled heap bytes was not captured. The trace identifies the first observed propagation into the server AppId cache, not the full prior lifetime of the heap allocation. Constructor instrumentation covered the identified constructor instruction range; static analysis and before/after snapshots support its missing initialization.

### Local binary details

These offsets apply only to the examined x64 broker DLL:

```text
C:\Windows\System32\Windows.ApplicationModel.Store.dll
SHA-256: 3f0fae916cbda1e3c556970d9a9282cd5d43957d3b5b26d691f09565460f8f48
PE TimeDateStamp: 0x1952801d
```

| Location | Meaning in this analysis |
|---|---|
| Context size `0x1e0` | Allocation size |
| Context `+0xe0` | Package-name HSTRING used in the lookup path |
| Context `+0xf0` | AppId GUID storage |
| Context `+0x100` | Content type loaded for lookup |
| Context `+0x128` | License-manager interface pointer |
| Context `+0x130` | License-instance pointer; null in failed path |
| Server `+0x80` | Context pointer |
| Server `+0xa0` | Cached AppId GUID |
| RVA `0xddd54` | Allocation-return observation before constructor |
| RVA `0xdd5ec` through `0xdd7b3` | Context constructor range examined |
| RVA `0xdf558` | License refresh routine |
| RVA `0xdf988` through `0xdf9a0` | Successful content-info AppId output path |
| RVA `0x794a0` | InitializeIdsFromLicense |
| RVA `0x7954e` | `movdqu xmmword ptr [rsi + 0xa0], xmm0`, server GUID copy |
| RVA `0x76f50` | Server AppId getter |

ASLR changes absolute addresses. These are RVAs, not portable addresses. The local first-write script guards architecture and PE timestamp; a hash check is stronger. Do not distribute these offsets as a universal Windows hook.

## 9. Receipt request and response protocol observed

The three-title comparison used the native Store receipt API without substituting request identities:

```text
POST https://licensingwindows.mp.microsoft.com/Licensing/License/AcquireReceipt/6.2/0
     ?productId=<GUID>&receiptType=Full

Request body length: 0
Total body length:   0
WinHTTP flags:      0x00800000
```

Application-added headers observed:

| Header | Comparison result |
|---|---|
| `MS-CorrelationId` | Per-request GUID, different between requests |
| `MS-RequestId` | Per-request GUID, different between requests |
| `MS-SiteName` | `www.microsoft.com` in all three |
| `MS-CV` | Correlation vector differed as expected |
| `Authorization` | Byte-identical across Toshiba, Standard and successful Adera in this run; 1,453 characters |

The sanitized report retains an authorization-value hash, not the token. This comparison covers captured application request arguments and headers; it is not a byte-for-byte TLS capture of every transport-generated header.

All three received HTTP 200. The service result is inside the XML envelope:

```xml
<ReceiptResponse Version="1.0" ErrorCode="0xc03f300c" />
```

Successful Adera response:

```xml
<ReceiptResponse Version="1.0" ErrorCode="0x0">
  <ReceiptBase64Encoded>000</ReceiptBase64Encoded>
</ReceiptResponse>
```

Decode the Base64 payload to obtain the signed Receipt XML; preserve its original bytes separately. Do not confuse this envelope with `pop.bin` itself. In the latest successful sample, the decoded receipt contained 1,288 characters in the API result and used Adera's PFN as AppId.

WinHTTP operations can be asynchronous. Observing only `WinHttpReadData` return values missed response data in the first pass. The improved observer captures read-completion callbacks as well. Absence of a synchronous response chunk was an instrumentation gap, not an empty server response.

## 10. Latest controlled comparison

Date: 2026-11-09. Local account and current installed packages. Same content type and lookup flags. No receipt GUID substitution in this comparison.

| Result | Toshiba | Standard DX+ | Adera |
|---|---|---|---|
| Package name passed to lookup | Correct registered full name | Correct registered full name | Correct registered full name |
| Content type / flags | `1` / `0x20000000` | Same | Same |
| License-manager HRESULT | `0x803F8001` | `0x803F8001` | `0x00000000` |
| License object returned | No | No | Yes |
| AppId | Uninitialized bytes | Uninitialized bytes | `77f0d9b5-87ac-4816-87d5-76b75afcb6db` |
| HTTP response | 200 | 200 | 200 |
| ReceiptResponse ErrorCode | `0xC03F300C` | `0xC03F300C` | `0x0` |
| Async Store operation | `0x80073DF6` | `0x80073DF6` | Receipt returned |
| Existing pop.bin at test | Absent | Present | Present |

No additional malformed field was identified in the captured requests. This is a bounded conclusion: the broker's entire internal object and backend state were not exhaustively audited.

Standard's earlier successful Xbox login used a cached receipt. The current fresh Store lookup fails. There is no contradiction: these are different stages. Adera is the actual successful fresh-acquisition control for this comparison.

Adera's receipt was not removed. The diagnostic invoked the receipt API directly. Startup changed some SDK caches/settings; after closing the diagnostic Adera launch, pre-existing tracked files were restored and hash-verified. Final verification: Toshiba 6 files, Standard 35 files, Adera 249 files, zero differences. Standard/Adera pop.bin remained present and unchanged. Toshiba had no pop.bin before this test. Downloads/assets were not part of that count and were not altered by the test.

## 11. Request identity versus returned receipt identity

Observed Adera request: correct legacy GUID. Returned receipt: PFN AppId. Earlier Adera comparisons also obtained successful PFN-format output through the tested alternatives. Toshiba's corrected legacy-GUID request on the friend's reported licensed account returned a receipt whose AppId remained the GUID and included a product record.

This establishes that the service may map a legacy request identifier to a package-family receipt identity for one product and behave differently for another. It does not establish why. Candidate explanations include catalog relationships, legacy product provisioning, package identity mapping, and account/license history. None is yet isolated as the cause.

Changing the signed receipt's GUID into a PFN is not an equivalent experiment: it destroys signature validity. The decisive future test would obtain a genuinely signed Toshiba PFN receipt and submit it through an otherwise unchanged Toshiba authentication path.

## 12. Source correction and helper substitution are different tests

Three distinct operations have been discussed or performed:

1. **Observe without changes:** trace native GUIDs and receipt arguments. This is the latest comparison.
2. **Substitute the URL's productId:** package-scoped helper rewrites only the outgoing AcquireReceipt productId to Toshiba's legacy GUID. It does not edit signed receipt XML.
3. **Temporarily correct the broker's source/cache fields:** prior local proof changed the context GUID and server GUID in memory, then called the unmodified receipt API. The native URL used the correct GUID without URL rewriting; the original fields were restored.

The third test proved the request follows the broker's cached identity. It still failed locally with `0x80073DF6`. A correct GUID alone does not grant a license or guarantee successful receipt issuance. No permanent repair of genuine Store licensing binding has been demonstrated.

Patching only the game executable's AppId getter would not necessarily repair the broker's separate source/cache. A package-scoped broker compatibility helper is still instrumentation, not restoration of an authentic Store license record.

## 13. Xbox authentication comparison

Retained historical comparison, not a claim that these exact token lengths or receipts are current:

| Field | Standard successful native authentication | Toshiba helper recipient trace |
|---|---|---|
| Title ID | `1313676122` | `1313676124` |
| Receipt characters | 1,321 | 1,546 |
| Receipt AppId | Standard PFN | Toshiba legacy GUID |
| AppId matches installed PFN | Yes | No |
| LicenseType | Full | Full |
| ProductReceipt count | 0 | 1 |
| Request bytes | 6,633 | 7,249 |
| Response bytes | 9,807 | 459 |
| RequestedSecurityToken present | Yes | No |
| Issued authorization scheme | XBL2.0 | No token issued in that trace |
| Issued token characters | 7,291 | 0 |
| Result | Authentication succeeded | `0x8015DA87`, SOAP Sender fault without explanatory Reason |

Common endpoint recorded:

```text
https://auth.xboxlive.com/XSts/xsts.svc/IWSTrust13
```

The comparison records the SDK passing receipt and title into a protected payload. The protected payload is not fully explained by the wire summary. The AppId mismatch is confirmed, but causation of Xbox rejection is **plausible, not proven**. Accounts, titles and receipt sources differed, and the Toshiba trace did not capture every outer SOAP field.

The compared Xbox SDK binary SHA-256 was:

```text
c60cfb494d795d31aa1cb18d963bc7a302091ff63b9ee3ee664667ea4d62973e
```

The original three PAC-MAN packages shared 1,412 identical files totaling 538,489,912 bytes, while executables, manifests and other title-specific files differed. Shared code/assets do not imply interchangeable license or title identity. See E08 for the package comparison.

## 14. Presence and achievements are separate authorization tests

The retained Toshiba test report records successful modern-service Presence using a token obtained from the running Xbox PC app for the verified account:

- Endpoint host: `presence-heartbeat.xboxlive.com`.
- Contract version: 3.
- Title ID: `2016983674`.
- Heartbeat HTTP 200.
- Independent UserPresence and PeopleHub readbacks showed Toshiba active under that ID.
- Cleanup request returned HTTP 200.

The legacy-ID heartbeat returned HTTP 200 but did not show Toshiba in the independent readbacks during that test. Other tested token sources described as simple/new Sisu tokens returned HTTP 403. This is direct evidence that the XBL3.0 prefix alone does not describe identical authorization capabilities.

The Sparkster legacy Progress variants recorded in the same report returned 404 or 405 and did not sync. Presence success therefore must not be represented as achievement-write success or as proof that a modern-to-legacy achievement translation exists.

User-visible gamertag, token issuance, accepted heartbeat, independently visible Presence, accepted achievement write, and independent achievement readback are separate milestones. Record each independently.

## 15. Error catalog with layer and limits

| Value | Where observed | What is established |
|---|---|---|
| `S_OK` / `0x00000000` | CurrentApp.AppId and wrapper refresh | Can occur while AppId is invalid or no license object exists; inspect outputs |
| `0x803F8001` | GetLicenseForInstalledPackage | Current Toshiba/Standard probes returned no object; do not infer a complete account entitlement inventory from this one local lookup |
| `0x80073DF6` | Async receipt API result | Operation failed; inspect service envelope for more specific response |
| `0xC03F300C` | Latest ReceiptResponse XML | Server error for unmodified malformed-GUID Toshiba/Standard requests; exact symbolic definition not established here |
| `0xC03F300A` | Earlier investigation target in Crash Course discussion | Keep distinct from 300C; this reference does not assign an unverified symbolic meaning |
| `0x8015DA87` | Toshiba Xbox authentication response | Recorded Xbox error with Sender SOAP fault; not a Store receipt HTTP status |
| HTTP 200 | Store or Presence transport response | Inspect payload and independent outcome; not sufficient alone |
| HTTP 403 | Some Presence token tests | Authorization failure in that endpoint/token context |
| HTTP 404 / 405 | Achievement request variants | Tested resource/method unsuccessful; does not alone establish title ownership status |

Also distinguish Store UI device-limit messages from an Xbox login failure. Bypassing the visible Store startup gate can remove the UI without repairing receipt acquisition.

## 16. Diagnostic tooling and coverage

| Source file in `work` | Purpose |
|---|---|
| `toshiba-license-manager.js` | Package-scoped COM manager, lookup, content-info and AppId observation; explicit contentType 1 probe |
| `toshiba-guid-first-write.js` | Local Windows-build-specific allocation/constructor/refresh/cache-copy trace |
| `trace-toshiba-guid-first-write.py` | Fresh-process attachment orchestration and JSONL capture |
| `store-appid-invoke.js` | Invokes native CurrentApp AppId and asynchronous receipt acquisition in the package process |
| `receipt-wire-readonly.js` | Read-only WinHTTP request/header/body/response observation; includes asynchronous completion capture |
| `compare-store-request-fields.py` | Original three-title comparison orchestration; broad backup implementation was interrupted during Adera asset copying |
| `compare-store-request-adera.py` | Adera follow-up with asset exclusions; trace completed, initial final hash pass encountered locked settings |
| `compare-store-request-repeat.py` | PAC-MAN repeat used to obtain asynchronous response envelopes |
| `toshiba-receipt-broker.js` | Diagnostic helper's package-scoped legacy productId substitution; not read-only observation |
| `toshiba-xbl-metadata.js` | SDK receipt metadata/full receipt and authentication fault instrumentation |
| `toshiba-receipt-test.py` | Helper process scoping, attachment, output and lifetime management |
| `ToshibaLicenseTrace.spec` | Frozen-helper packaging inputs |

The final preservation report supersedes the intermediate locked-settings verification failure. All instrumentation sessions from the latest comparison were detached.

**Source versus distribution:** the contentType correction is present in the source and was tested live. Existing full redistribution ZIPs were not rebuilt by the latest tracing turns. A helper build number does not automatically imply all source changes are in a previously packaged binary. Verify frozen resources, launcher text, source copies, file hashes and build metadata together before redistribution.

The earlier helper source identifies build 5; SDK logger build numbering is separate. Do not call a helper build number an SDK logger version. Full `receipt.txt` capture is an explicitly requested diagnostic feature, not proof that bearer-token capture is enabled.

## 17. Preservation and redistribution requirements

Current user requirements and workspace rules:

- Keep original game version numbers when patching or packaging.
- Preserve saves, profile state and existing pop.bin during diagnostic updates; hash-check backups/restoration.
- Never bundle pop.bin, saved games, profile data, tokens or private diagnostic captures in redistribution.
- Installer offers the user's own pop.bin through explicit typed Y or N. Enter repeats the instruction; Y opens a file picker. N skips import rather than deleting an existing receipt.
- Enable no-suspension behavior during normal installation for the playing account, tracked per package full name/version; no separate user action.
- Enable crash dumps during installation. Use `LocalState\Crash Dumps`, game-specific latest dump filename and up to four prior dumps; no desktop crash-dump folder.
- Xbox SDK logs and full receipt capture are included when requested for the diagnostic build; do not confuse crash dump configuration with SDK logging.
- Preserve downloaded DLC/guides separately from saved progress when packaging them.

Backup discipline: enumerate only relevant state when possible, exclude diagnostics and large assets unless they are being changed, record hashes before launching, account for file locks, and restore only test-induced changes after the diagnostic process is closed. Retain a final verification result. Do not delete a whole LocalState directory to force receipt acquisition.

The latest comparison inadvertently began a broad Adera content backup. That copy was stopped; redundant copied asset directories were removed only from the verified diagnostic backup root. Original installed/downloaded content was untouched. This is a workflow correction worth retaining so future probes do not repeat expensive asset copying.

## 18. What is not yet established

1. Which persistent licensing record or index the application license manager searches for these packages, and how account, registration and sideload history affect that lookup.
2. Why the friend's reported licensed Toshiba account also produces an empty/unusable native AppId path.
3. Whether genuine Store installation/binding restoration can make Toshiba return a license object without instrumentation.
4. Why Adera's legacy GUID maps to a PFN receipt while Toshiba's examined legacy response retains the GUID.
5. Whether Toshiba can obtain a genuinely signed PFN-format receipt through an available legitimate service path.
6. Whether the receipt AppId mismatch is the actual cause of the recorded Xbox SOAP rejection.
7. Complete schemas for SDK cache files and xbl.spa, and exact persistence/refresh rules for each token/receipt cache.
8. A universal receipt lifetime, old-receipt invalidation rule, or account/device portability rule.
9. Whether any supported achievement route accepts Toshiba writes under its modern identity and reflects them in its legacy list; tested variants did not establish this.
10. Complete internal contentType enum semantics and the full licensing-context field layout.

## 19. Next experiments and acceptance criteria

The next useful target is **the failed package-license lookup**, rather than another blind change to receipt XML or Xbox token format.

| Experiment | Required evidence | What would count as progress |
|---|---|---|
| Friend's licensed Toshiba machine with matching probe | Exact package full name, native contentType/flags, COM HRESULT, object presence, returned AppId, account context without secrets | Establish whether failure reproduces before online receipt acquisition |
| Follow license-manager server boundary | Scoped caller/service trace and symbols where available; exact binary hashes | Identify the persistent lookup path and failure branch |
| Compare genuine Store binding against rebuilt install | Registration/signature/deployment context and same-account controlled comparison | Show which change causes a real object and GUID to appear |
| Signed Toshiba PFN receipt test | Original signed XML, signature-validation result, native auth outcome | Isolate receipt identity format without editing signed data |
| Repeat Xbox auth comparison | Same SDK/configuration, complete nonsecret metadata, all status/fault layers | Separate account/title/receipt/configuration differences |

Do not change several factors at once. Preserve native traces before introducing compatibility substitutions. Record absence of an object separately from zero or garbage contents in a caller-owned output field.

## 20. Corrections to earlier assumptions

- “Only all zeroes are being sent” is incomplete: zeroes and nonzero stale bytes have both been observed.
- “Standard works, so its Store licensing lookup works” is false for the latest local comparison; cached Xbox receipt authentication and fresh acquisition differ.
- “Receipt XML is SAML” and “working receipt is JWT” conflate different artifacts.
- “Legacy GUID request always returns GUID AppId” is contradicted by Adera.
- “newAuth selects the receipt AppId format” is not established by direct receipt API tests.
- “HTTP 200 means receipt acquisition succeeded” is contradicted by error-bearing ReceiptResponse envelopes.
- “Presence success guarantees achievement sync” is contradicted by the recorded endpoint tests.
- “Source correction restores licensing” overstates the temporary memory proof; it changes request identity but does not create entitlement.
- “All pop.bin files expire exactly after 24 hours” remains an unproven generalization; no-pop redistribution is still the user's policy.

## 21. Change history

### 2026-11-09 revision 5

Located enclosing signed XML License documents around both decoded JSON payloads. Verified RSA-SHA256 signatures against embedded keys and reference digests. Identified static ClipInstallLicense dispatch and ClipSVC verification imports. Documented licensed-account recovery test and limits; no license or game state modified. See section 25 and E16.

### 2026-11-09 revision 4

Captured Adera decoded document JSON privately and retained sanitized shapes. Matched the exact compact JSON, Base64-encoded, to tokens.dat at specific offsets. This establishes physical payload locations without claiming full record boundaries or whole-store encryption semantics. See section 24 and E15.

### 2026-11-09 revision 3

Verified content-key derivation and traced asynchronous missing-key lookup, online replenishment failure, and Adera positive control. See section 23 and E14. Saves and receipts preserved.

### 2026-11-09 revision 2

Followed the broker COM proxy into the LicenseManager service, identified distinct internal lookup keys, compared the in-memory lookup table and the per-user ContentId registry mapping, and recorded the asynchronous failure boundary. Adera was launched with its receipt temporarily removed, successfully acquired a receipt, and had its original receipt/state restored. See section 22 and E13. The underlying persistent license-store read remains unresolved.

### 2026-11-09 revision 1

Created the consolidated reference from retained local comparisons, native first-write evidence, helper source, receipt-signature report, Presence/achievement results, user requirements and the latest three-title Store request trace. Recorded the contentType 1 correction, uninitialized GUID path, Standard cached-versus-fresh distinction, successful Adera control, asynchronous WinHTTP response capture, and final data-preservation results. Retained small evidence snapshots separately from private receipts and tokens.

## 22. Service-side lookup investigation

The broker's IApplicationLicenseManager interface is a `combase.dll` stubless COM proxy. Its slot 6 forwards to the Windows LicenseManager service, hosted in `svchost.exe`, with implementation in `LicenseManager.dll`. This is a separate process from the package-associated RuntimeBroker.

Examined LicenseManager.dll SHA-256: `a9119eb185eb378b3579699596ce508cba6ffeda60d191648d2fecfbfef64605`. The following RVAs apply only to this binary.

Observed service path:

```text
ServiceCreateApplicationLicenseManager
 -> IApplicationLicenseManager slot 6, RVA 0x36450
 -> inner implementation thunk RVA 0xa0c90 -> RVA 0x23160
 -> package/content-type to internal lookup key, RVA 0x23424
 -> manager lookup RVA 0x4980
 -> storage/cache query RVA 0x10a60 -> RVA 0x10ab0
 -> asynchronous record load/result
 -> license selection if the preceding load succeeds
```

The internal lookup key is another GUID-shaped identifier, **not the legacy Store AppId returned by the license object**:

| Identity | Toshiba | Adera |
|---|---|---|
| Internal lookup key observed | `79027A65-D615-F76D-A36D-E56124399AD5` | `A813DF52-ACD9-9362-9D5F-79D4452447A9` |
| Legacy AppId expected/returned | `2d4168ad-4092-4ea7-92fd-1c81e8191304` | `77f0d9b5-87ac-4816-87d5-76b75afcb6db` |

The storage query searches an in-memory vector of entries, with 0x30-byte entry stride and a UTF-16 string key. In the initial service trace the vector contained three keys: Adera's key and two other unidentified keys. Toshiba's key missed; Adera's matched. The miss path can insert/load an entry, so **a cache miss by itself does not prove that the persistent license is absent**.

Toshiba then reached an asynchronous-result failure check in `managercore.cpp`, recorded line 171, RVA return site `0x3f703`. The stored negative result was `0x803F8001`. It propagated through `onestoreapplicensemanager.cpp`, line 125, return site `0x23253`, to the COM caller. The examined check waits on an event and examines a saved result; it is a propagation point, not necessarily the original storage failure. Adera continued to license selection at RVA `0x52d4` and returned an object successfully.

A read-only registry inspection found:

```text
HKCU\Software\Microsoft\Windows\CurrentVersion\Store\ContentId
  Microsoft.Adera_8wekyb3d8bbwe
    REG_SZ {a813df52-acd9-9362-9d5f-79d4452447a9}
  NAMCOBANDAIGamesInc.2685271D3260E_gdy2aq6ez762w
    value absent
```

Adera's registry value exactly matches its observed internal lookup key. This is a concrete local mapping difference. It is not proof that writing this value would restore Toshiba licensing; the service still needs a valid backing license. No registry or licensing-store data was modified.

CLIPC.dll contains references to `%PROGRAMDATA%\Microsoft\Windows\ClipSvc` and `tokens.dat`; that file exists locally. These are leads from static inspection, **not a traced proof that the particular failed Toshiba request reads a particular record there**. Revision 3 resolves the key derivation and online origin of the failure (section 23). The physical persistent-record schema remains open.

### Fresh receipt test and diagnostic incident

Adera's original pop.bin was backed up, temporarily removed before launch, then restored. Adera still returned its correct AppId, a real license object and a 1,288-character signed receipt. This rules out the existing game-local pop.bin as a prerequisite for that successful local Store lookup in this test. It does not clear every other Store/SDK cache.

An expanded CLIPC worker-thread hook attempt coincided with a LicenseManager service crash: Application event 1000, exception `0xc00000ff` in ntdll.dll. That broader hook was removed. Subsequent COM activation restarted the service, and ordinary probes reproduced the original Toshiba failure and Adera success. A narrower service trace completed afterward. All trace sessions were detached. Do not rerun the superseded broad hook indiscriminately.

Final preservation verification matched all 6 tracked Toshiba files and all 315 tracked Adera files to their pre-test hashes after restoring startup-induced changes. Adera's original pop.bin is restored; Toshiba had no pop.bin. The game executables and installed licensing databases were not patched. See E13 for retained service-side evidence and preservation metadata.

## 23. Verified content-key derivation and asynchronous failure

Observed 2026-11-09 on the LicenseManager.dll hash in section 22. Sanitized evidence: E14-content-key-async-load.json. No license data or return values were modified.

### What creates the content key

LicenseManager derives the identifier from `<package name>:<publisher suffix>:win8xappx` for native contentType 1. It lowercases the text, encodes UTF-16LE without NUL, computes SHA-256, and interprets the first 16 digest bytes as a Windows little-endian GUID. Independent Python calculations exactly matched both live outputs:

| Game | Hash input before lowercasing | Content ID |
|---|---|---|
| Toshiba | NAMCOBANDAIGamesInc.2685271D3260E:gdy2aq6ez762w:win8xappx | 79027a65-d615-f76d-a36d-e56124399ad5 |
| Adera | Microsoft.Adera:8wekyb3d8bbwe:win8xappx | a813df52-acd9-9362-9d5f-79d4452447a9 |

Version and architecture are not inputs. This is an identifier, not a cryptographic secret or the legacy Store AppId. The license key document is a separate object indexed using it.

Static anchors: package conversion 0x23424; category mapping 0x23730; string construction 0x23870; lowercase/hash 0x25300. Observed contentType mappings: 1->category 0 (win8xappx), 2->1 (wp8xappx), 3->2 (uapp), 5->4 (win32). The category table also contains xap at index 3. These are binary-specific observations, not a complete public enum. Hash algorithm ID is 0x800c.

The inspected registry-lookup branch belongs to category 2. Native contentType 1 derives the hash. Therefore the absent Toshiba ContentId registry value does not explain an incorrect key in this path: the observed key is correct for Toshiba's package identity.

### Missing logical record and online response

The asynchronous worker reports, from clipstorage.cpp line 269, return RVA 0xf99e:

`No key document found for contentid 79027A65-D615-F76D-A36D-E56124399AD5`

This identifies the missing logical CLIP key document, beyond the earlier in-memory cache miss. It does not establish a physical tokens.dat offset or prove rejection of a corrupt/signature-invalid document.

The worker then captures account tickets successfully and requests a license online. Response telemetry records:

- Zero leases and zero keys for that content ID.
- Satisfaction error code 1: **Users do not possess any satisfying entitlements for the content id in question.**
- Token broker status 0x00000000, two MSA tickets, zero AAD tickets.
- ValidExistingKey False; HRESULT 0x803F8001.

The online response failure is recorded at proxy.cpp line 358, return RVA 0x18298, before propagation through managercore and the COM lookup. Further fallback content-ID searches find no key documents; E14 retains their IDs, but their individual roles remain undecoded.

The event's package-family pointer could not be read and is recorded null; this does not establish that the serialized outbound request omitted the PFN. Full wire serialization was not captured here.

**Interpretation:** On the local account, Toshiba lacks a usable local key document and online replenishment fails because the service reports no satisfying entitlement. Account-ticket acquisition succeeded. This is consistent with the user's stated lack of Toshiba ownership. The friend's reported licensed account needs the same response-level trace; its cause must not be inferred from this account.

### Adera control and preservation

Adera derives its expected key, emits three document-decoding events from clipdocument.cpp line 50, and successfully returns a license object and AppId 77f0d9b5-87ac-4816-87d5-76b75afcb6db. No equivalent online failure was observed. This run did not flush service caches or prove a new physical disk read for Adera; the prior receipt-removal test is separate evidence.

Neither receipt was removed or replaced in this experiment. All six tracked Toshiba files and 382 tracked Adera files matched pre-test hashes after restoring startup-induced changes. Adera's original pop.bin remained intact; Toshiba had no existing pop.bin. Downloaded assets were untouched. Trace sessions were detached and only game processes started by the harness were closed. No package rebuild or registry/license-database edit was performed.

Remaining unknowns: physical CLIP storage schema and exact record/index location; why the friend's licensed-account lookup fails. The logical missing record, deterministic key creation and local account's online rejection are now established.

## 24. Adera decoded JSON and physical payload locations

Read-only follow-up, 2026-11-09. Adera's clipdocument.cpp decoding events contain JSON, not Store Receipt XML. Two distinct documents were observed: a 58-character object containing legacyProductId, and a 669-character lease/entitlement metadata object (decoded twice). Full JSON is retained privately at `work/private-adera-document-capture/adera-decoded-documents.json`, not in distributable evidence.

The first is:

```json
{"legacyProductId":"77f0d9b5-87ac-4816-87d5-76b75afcb6db"}
```

The second contains entitlementId, entitlementSatisfaction (`Device`), isOffline (`true`), leaseEnforcement (`RefreshOnStartup`), leaseUri, keyIds, kind (`Content`), packages, pollAt, refreshOnStartup (`true`), and version (`7`). Its package entry identifies content ID a813df52-acd9-9362-9d5f-79d4452447a9, type appx, product 9WZDNCRFJB4P, SKU 0010, and an empty productAddOns array. These metadata fields alone do not establish the entirety of the key document or the account's entitlement history. isOffline does not mean the game has no internet connection, and pollAt is not established as a receipt expiration.

### Physical matches

Read `C:\ProgramData\Microsoft\Windows\ClipSVC\tokens.dat` without modifying it. Snapshot size 1,907,122 bytes; SHA-256 recorded in E15. Both captured JSON objects, serialized compactly and Base64-encoded, matched exact bytes inside this file:

| JSON payload | Base64 stored as ASCII offset | Base64 stored as UTF-16LE offset |
|---|---|---|
| legacyProductId object | 0x1a06b8 | 0x1a0ffe |
| Lease/entitlement metadata | 0x17a78b | 0x1a493e |

Adera's content ID also occurs as ASCII at 0x1a0482 and UTF-16LE at 0x1a0f88. Toshiba's content ID was absent in the searched GUID little/big-endian binary, lowercase ASCII/UTF-16LE and uppercase UTF-16LE forms. Absence in those forms is not a complete database enumeration.

This establishes **physical locations of the exact encoded Adera payloads**, improving the previous unresolved-storage statement. It does not establish record headers, complete record boundaries, the active copy among duplicate representations, or which offset a particular cached lookup read.

### Decoding versus decryption

At the observed boundary these are Base64-encoded UTF-8 JSON payloads converted to readable text and parsed. Exact Base64 matches in the persistent file show that these specific payloads are not ciphertext requiring a secret to recover. This is not evidence that every license field or all of tokens.dat is unencrypted; signature, integrity and secret key material are separate concerns. No cryptographic keys were extracted and no license-store data was modified.

Static storage wrapper RVA 0x3baf0 calls ClipGetQueryResults and explicitly maps 0xc03f6508 to successful HRESULT with a null result. This is a binary-specific behavior, not a claimed public name for that status. LicenseManager then reports no key document. Prior broader CLIPC hooks were not reused.

All six tracked Toshiba and 383 tracked Adera files matched pre-test hashes after restoring startup changes. Existing pop.bin remained intact; large downloaded assets were untouched; traces detached. Evidence E15 omits full private entitlement identifiers, lease URI and document payloads.

## 25. Signed records, installation and licensed-account restoration

The readable JSON from section 24 is Base64 text inside `CustomPolicies` of larger XML `License` documents, namespace `urn:schemas-microsoft-com:windows:store:licensing:ls`, Version 5. The key metadata document spans file offsets 0x1a03b0 to 0x1a0e1a (exclusive), with ContentID a813df52-acd9-9362-9d5f-79d4452447a9. The lease metadata document spans 0x17a4e7 to 0x17b21d (exclusive); its root has no ContentID attribute. These are XML document boundaries, not complete database-record boundaries.

Both documents contain XMLDSIG. Each RSA-SHA256 SignatureValue verified against its embedded RSA public key. The empty-URI reference digest matched SHA-256 after the declared enveloped-signature transform and implicit inclusive canonicalization. SignedInfo uses exclusive canonicalization. Certificate/root trust was not independently validated by this check. Sanitized results are in E16.

**Consequences:** Changing CustomPolicies JSON or ContentID changes the signed document and invalidates its digest/signature. Merely encoding replacement JSON as Base64 does not produce an authentic Toshiba license. Do not conflate readable policy data with unsigned authorization. Whether tokens.dat additionally has a whole-file signature/MAC or other integrity mechanism remains unestablished; what is now proven is record-level signed XML.

Static code evidence: LicenseManager.dll wrapper RVA 0x61e00 tail-calls imported ClipInstallLicense at 0x61e21. CLIPC.dll exports ClipInstallLicense, ClipRefreshLicense and ClipGenerateDeviceLicenseRequest. ClipSVC.dll imports CryptXmlVerifySignature and BCryptVerifySignature. These observations support the licensing-service install/refresh route, rather than games writing arbitrary JSON. They do not by themselves prove every export's ABI or trace a successful transaction through the final disk write. No undocumented write API was invoked.

The previous live trace establishes LicenseManager's online request and response handling. The supported interpretation is that successful service-issued license acquisition supplies the documents for the CLIP installation path. A fresh successful acquisition-to-persistence trace is still needed to establish all internal steps. Adera currently has cached working records; deleting them merely to force a write would be an unnecessary destructive test.

### Friend's licensed Toshiba account: discriminating test

The account must be tested on the friend's machine; the local account lacks Toshiba ownership. Preserve saves/pop.bin and license database before any proposed repair, and start with observation rather than database changes. Capture the native contentType 1 lookup for Toshiba's exact registered package, derived content ID, account-ticket status, license response counts, satisfaction code/text, and any subsequent ClipInstallLicense return status. Do not substitute a receipt GUID during this native lookup test.

- If the service returns nonzero keys/leases and installation succeeds, repeat the native lookup and check for a real license object and the expected legacy AppId. This would demonstrate restoration rather than a forced return value.
- If keys/leases arrive but installation fails, investigate the exact verification/binding/persistence error.
- If the service returns zero keys/leases and the same unsatisfied-entitlement message, there is no local signed document to install from that response. Check the Store account, exact Toshiba SKU/package identity and OEM/device applicability. These are hypotheses to test, not established causes. A signed legacy receipt alone does not prove this separate CLIP content-ID lookup is satisfied.
- If the Store Library still offers installation on the owning account, genuine Store acquisition is a candidate for restoring the binding; availability for this delisted OEM title has not been verified. No guarantee of recovery is made.

No manual JSON injection or cross-title record copying was performed. No game was launched in this follow-up; saves and pop.bin were untouched. The only environment addition was lxml in the workspace virtual environment for offline XML canonicalization/signature checks.

Microsoft's general service documentation identifies ClipSVC as Microsoft Store infrastructure and notes that disabling it disrupts purchased apps: https://learn.microsoft.com/en-US/windows-server/security/windows-services/security-guidelines-for-disabling-system-services-in-windows-server . This contextual documentation does not specify this private database format.

## 26. Full CLIP diagnostic redistribution

2026-11-09: prepared Toshiba Full CLIP Diagnostics installer, unchanged game version 1.0.1.2, helper 6 and native SDK logger 7. Adds hash-guarded service observation of requests, key/lease counts, satisfaction failures, ClipInstallLicense results and HRESULT source locations. SDK/COM/receipt/SOAP diagnostics and the diagnostic-launcher-only legacy receipt GUID override remain. No signed license data is edited. Unsupported LicenseManager.dll builds explicitly skip internal CLIP coverage while retaining other logs.

The installer uses the three-choice completion menu in Logging-Build-Template.md. All diagnostic sessions/captures use distinct filenames; Enter ends the external trace, waits for detachment, cleans its markers and opens the logs folder. The native SDK logger continues until game exit. Existing game data is preserved by replacement logic; a prior manual uninstall is outside its recovery capability.

Validation: unsigned data-free source audit, signed APPX/dependency verification, menu branch/invalid-input tests, two native SDK log sessions, two frozen helper sessions with marker cleanup, and a live Toshiba/Adera observer comparison. Toshiba returned 0x803f8001 and Adera returned its real license object. All six tracked Toshiba and 383 tracked Adera files matched original hashes after test-induced changes were restored. During the comparison, broker detachment waited until test-created game processes ended; the user-facing launcher has a bounded shutdown wait and reports any timeout. A successful Toshiba license installation on the friend's account remains untested. No helper binary can claim internal coverage for an unsupported Windows build.

## 27. Friend-provided Toshiba full diagnostic results

2026-11-09 update (E18): analysis of supplied SDK logger 7 and helper 6 logs at 20:33–20:34 UTC. The friend reports owning Toshiba; ownership/account selection was not independently verified. No installed files, saves or receipts were changed in this analysis.

Both the explicit contentType 1 probe and native path derive Toshiba content ID 79027A65-D615-F76D-A36D-E56124399AD5. Both online responses contain zero leases and zero keys, service code 1, "Users do not possess any satisfying entitlements for the content id in question.", tokenBrokerStatus 0, four MSA tickets, and HRESULT 0x803f8001. No usable local key document is found. Explicit lookup returns 0x803f8001/no object; native lookup separately returns 0x800706be/no object. The latter must not be collapsed into the former or treated as a proven service crash. No successful license installation is established.

CurrentApp.AppId reports 000000000000000000000000000000000000. The helper replaces the outbound receipt productId with the known legacy GUID, without modifying returned XML. Two HTTP 200 responses yield 1546-character receipts. AppReceipt and ProductReceipt use the legacy GUID; the durable full-version product has PurchaseDate 2013-10-01 and far-future expiry. App purchase timestamps are exactly 30 minutes before issuance. The two pasted receipts differ in issue/app-purchase times, AppReceipt IDs, digests and signatures; their product entitlement fields match. Signature presence is logged, validation is explicitly false (not performed), not a finding of invalidity.

SDK GetXblTokenResponse receives 1546 receipt characters and 1122 ticket characters for title 1313676124. Both requests return HTTP 200 but X-Err 0x8015DA87 and a 459-byte SOAP fault; requestedToken, SAML and JWT indicators are zero and credentials remain empty. SOAP text gives no specific reason. The receipt/AppId mismatch remains a hypothesis, not a demonstrated cause.

The IAsyncInfo diagnostic reports 0x8000000E despite subsequent successful receipt delivery. It cannot establish failed receipt acquisition in this run; diagnostic object lifetime/query timing needs review. Native logger says it does not override requests, while the separate helper does: these describe different components, not contradictory evidence.

This extends section 25's zero-key/lease branch to the friend's reported owning account. It does not disprove historical purchase ownership: exact Store account, base-app versus durable add-on entitlement, OEM applicability and catalog/content-ID mapping remain unresolved. Receipt issuance does not repair or prove satisfaction of the separate CLIP lookup. Next useful comparison is account/SKU/content-ID applicability and a known-good owning-account acquisition, not editing signed XML or assuming another forced GUID repairs licensing.

## 28. Public catalog entitlement mapping and small native-only update

2026-11-09, revision 6. Fresh unauthenticated Microsoft DisplayCatalog v7 responses for US/en-US establish the following mapping (E19). These are catalog definitions, not account ownership queries. Source snapshots remain in `work/toshiba-mapping-investigation`.

| Edition | Store Product ID | Legacy Windows Store AppId | Windows 8 package ContentId |
|---|---|---|---|
| Toshiba | 9WZDNCRDMWQF | 2d4168ad-4092-4ea7-92fd-1c81e8191304 | 79027a65-d615-f76d-a36d-e56124399ad5 |
| Standard | 9WZDNCRDMWQH | 03385fc3-0d41-46b1-a450-ee24675c1031 | 26c5c07d-b3f5-2231-e5b9-1b83d574f74e |
| Adera | 9WZDNCRFJB4P | 77f0d9b5-87ac-4816-87d5-76b75afcb6db | a813df52-acd9-9362-9d5f-79d4452447a9 |

Recomputing each content ID using the lowercase UTF-16LE package identity hash matches its catalog package ContentId. Toshiba's catalog explicitly names the friend's exact full package identity/version. Both Toshiba full SKU 0010 and trial SKU 0011 contain the same ContentId: it is not a separate trial-versus-full content key. Changing category/contentType or substituting the legacy GUID for this key would depart from the catalog evidence.

### Stronger evidence: SatisfyingEntitlementKeys

Toshiba full SKU 0010's desktop License availability lists:

- `big:9WZDNCRDMWQF:0010`
- `wes:App:2d4168ad-4092-4ea7-92fd-1c81e8191304:Full`
- `wes:App:af5d47b9-41cd-4071-81d0-67ba43c95f04:Full` (its catalog Windows Phone alternate ID)
- `big:9WZDNCRDMWQF:0001`
- `big:9WZDNCRDMWQF:0002`

Each group maps to LicensingKeyIds `1`. The named durable product `PAC-MAN CEDX for TOSHIBA full version` does not appear among these satisfying keys. The trial SKU has no SatisfyingEntitlementKeys in this response. Empty/missing catalog arrays do not by themselves establish a service's complete private policy.

Adera has the same structure, using its own product/GUID IDs. Standard additionally lists SKU 0017 as satisfying its SKU 0010 licensing availability. Standard's cached working receipt/authentication is not evidence that its current fresh CLIP acquisition succeeds. Adera remains the established successful local license-object control.

**Observed conclusion:** There is no mismatch between Toshiba's catalog package ContentId, the independently derived ContentId and the friend's CLIP request. The catalog also explicitly supports its legacy *app* full entitlement. This rules out the simple hypotheses that the derived key is wrong or the public catalog omits that legacy app mapping.

**Interpretation, not yet proved:** A recognized durable upgrade purchase may coexist with a missing/inapplicable base-app entitlement in CLIP's account context. The receipt also says AppReceipt LicenseType Full, so it would be incorrect to conclude from ProductReceipt alone that the friend owns only an add-on. The recurring issue-minus-30-minutes app purchase date is not the same evidence as a durable purchase timestamp from 2013. The exact receipt-service policy producing that app assertion has not been traced.

Still unresolved: whether the receipt broker and CLIP evaluate the same account; actual server-side account entitlement records; legacy entitlement migration/normalization; OEM/device applicability; region/platform-specific private policy. Four MSA tickets and successful token-broker status do not prove account equality. The fetched public conditions do not identify an explicit Toshiba-manufacturer restriction; OEM restriction remains a hypothesis. These results do not justify rewriting tokens.dat, signed receipts or licensing keys.

Public sources: https://displaycatalog.mp.microsoft.com/v7.0/products/9WZDNCRDMWQF?market=US&languages=en-us (and the Standard/Adera product IDs above); https://learn.microsoft.com/en-us/windows/uwp/monetize/use-receipts-to-verify-product-purchases describes AppReceipt and ProductReceipt separately and GetAppReceiptAsync returning app plus add-on information. Neither public source exposes this friend's account entitlement state.

### Mapping diagnostic update 1

Prepared `outputs/Toshiba-CLIP-Mapping-Update.zip`, a roughly 20 KB helper-script delta for the Full CLIP Diagnostics installer (helper executable 6 / SDK logger 7). Rebuild using `work/build-toshiba-mapping-update.py`; updater and launcher sources are `work/Update-Toshiba-Mapping.ps1` and `work/Start-Toshiba-Mapping-Trace.ps1`.

No APPX rebuild or reinstall. The delta replaces two external JavaScript files, verifies all 84 existing runtime files against the retained baseline/updated hashes, retains rollback copies and updates the original installer integrity manifest without a self-hash entry. Existing game version, data, crash dumps and no-suspension setup are unchanged. No pop.bin, saves, profile data, account tickets or tokens are bundled.

The helper's explicit COM probe is skipped and labeled. Native hooks stay active. This isolates a possible confound: the earlier explicit probe precedes the native call, and their HRESULTs differed. It is not a finding that instrumentation caused the failure. Existing receipt-GUID override, metadata/full receipt capture, SOAP/SDK logging, hash-guarded service observations, key/lease counts and install-result hooks remain. Bounded native telemetry string reads improve short-string observation; request endpoint host/redacted fixed route metadata is added without query values or credentials. Windows/package/signature-kind and OEM manufacturer/model context is written separately; no serial or account name is collected. This is not full decrypted CLIP wire capture and cannot itself prove account equality.

`Update.cmd` remembers or asks once for the extracted original Full CLIP Diagnostics installer folder, applies the small update, then offers logging/normal/exit. Logging runs directly from this update window; subsequent Update.cmd runs can launch again. Original folder remains necessary only to supply its runtime. Unique per-session context/SDK/helper/receipt files are retained, confirmed helper shutdown cleans markers and opens Explorer. A timeout reports failure and retains markers.

Validation E20: baseline runtime hash comparison; PowerShell parsing; updater verification-only/no mutation, apply, manifest hash consistency, repeat apply, modified-payload and unsupported-runtime rejection; all three menu choices and invalid inputs; two unique context files and marker cleanup using mocked OS/process boundaries; JavaScript syntax and bounded field/redaction/native-probe-skip fixture tests; ZIP hashes/private-data exclusion. No game was launched, installed data or pop.bin modified, or live service hooked during this follow-up. Real recipient native-only results and endpoint coverage remain untested. This is a diagnostic update, not a demonstrated repair.


## 29. Friend native-only follow-up, 21:39 UTC

2026-11-09 (E21). Mapping update 1 ran successfully with probeInvoked=false. Native contentType 1 lookup still produces the correct Toshiba ContentId, no local key document, zero online keys/leases, service code 1 unsatisfied-entitlement text and tokenBrokerStatus 0 with four MSA tickets. Native COM result is now 0x803f8001/no object (previous run: 0x800706be). CurrentApp.AppId and the original outbound receipt productId are now all zeroes (previous run: string-like nonzero GUID). These changes are consistent with nondeterministic failed-lookup state or probe/context effects; one run does not prove the extra probe caused the earlier anomaly. The licensing rejection clearly persists without the extra probe.

The retained helper still substitutes the known legacy receipt GUID. Two successful 1546-character receipt responses reach GetXblTokenResponse for title 1313676124 with 1122-character tickets. Both Xbox responses carry X-Err 0x8015DA87, SOAPFault=1, requestedToken=0; credentials remain empty. No successful XBL2.0 or XBL3.0 token is observed. Receipt times are 21:39:30 and 21:39:35 UTC; product entitlement fields match the previous samples, app purchase dates remain issue-minus-30-minutes. Signature validation was not performed. SDK IAsyncInfo error still must not override demonstrated receipt delivery.

Context reports unchanged version 1.0.1.2, packageStatus Ok, signatureKind Developer, isDevelopmentMode false, receiptCacheExists false. Manufacturer is Micro-Star International Co., Ltd., model MS-7C80: this is not Toshiba-branded hardware, which makes OEM applicability a concrete test dimension, not a proven cause. Developer signature alone is not proof of rejection either. accountIdentityCaptured=false and accountEqualityEstablished=false remain explicit limitations.

Endpoint metadata returned recognizedUrl=false. Thus the added telemetry argument reader did not establish the CLIP endpoint; do not infer that the actual request URL is invalid. Field meaning/representation needs confirmation before claiming endpoint coverage. Supplied helper excerpt has no helper-stopped event, so detachment/marker cleanup cannot be verified from these attachments.

Next discriminating evidence: establish whether the Store receipt broker and CLIP request use the same account, and compare entitlement satisfaction with the original eligible device if available. A validated account/context comparison is more useful than another content-ID substitution. These findings do not prove that a durable upgrade alone accounts for AppReceipt Full, or that OEM binding caused the failure. No game/data/receipt was modified while analyzing these attachments.

## 30. Account-context tracing update, no Toshiba-device requirement

2026-11-09. The user cannot test on Toshiba hardware. That route is not a prerequisite for this investigation. Prepared outputs/Toshiba-Account-Trace-Update.zip (~30 KB), supporting the original Full CLIP Diagnostics runtime and Mapping Update 1. Rebuild with work/build-toshiba-account-update.py. Update.cmd applies the helper-only delta, then launches logging directly via its completion menu. Existing runtime is selected once and remembered; no APPX download or reinstall. Installed game, saves, pop.bin, crash settings and suspension exemption were not changed or launched during preparation.

Static inspection of the supported LicenseManager.dll shows TokenBrokerInternal, PrimaryWebAccountId, StoreAADAccountId and telemetry strings for capturing multiple MSA/AAD tickets and "Licensing with user ticket %s (Active)". The latter calls the existing telemetry routine 0xacc0 at 0x7ad63. These are account-context investigation leads, not evidence that a particular raw value is a PUID or proof of the friend's selected account. No new internal RVA is hooked: the existing hash-guarded telemetry hook forwards only a fingerprint of that active-ticket field during the Toshiba content-ID observation window.

New observer hooks named WinHTTP exports in the verified package broker and supported CLIP service, scopes receipt requests to licensingwindows.mp.microsoft.com/AcquireReceipt and CLIP requests to licensing.mp.microsoft.com within a 15-second Toshiba observation window, and hashes Authorization values after removing known WLID1.0 t= or Bearer wrappers. Only exact normalized credential equality is asserted. It also hashes explicitly named user-ID headers and reads of PrimaryWebAccountId/StoreAADAccountId, with distinct namespaces and context-only labels. Registry reads may be cached/absent and need not identify the final serialized request account. HTTP bodies and bearer values are not logged. CLIP credential transport not passing those exports or absent telemetry will produce incomplete coverage, not evidence of different accounts.

Identifiers and tickets use SHA-256 domain-separated by kind/namespace, computed inside the observed process; no raw IDs, emails, authorization headers or tickets leave it through this observer. Hashes are stable/linkable across sessions and are diagnostic metadata, not anonymization guarantees. Existing full receipt capture remains as requested. SDK/Store/SOAP/CLIP logs and labeled legacy receipt GUID override remain; the extra COM probe stays disabled. No license data is edited.

After confirmed detachment, Summarize-Accounts.ps1 writes a unique account-comparison JSON for the current session. Outcomes are matching observed account context, shared normalized credential, or inconclusive. Different ticket hashes NEVER establish different Microsoft accounts; an account can have multiple differently issued/scoped tickets. Matching registry context alone does not prove final wire account equality. A shared-service observation window is not per-request attribution. The summary explicitly retains those limitations. Summary failure does not block marker cleanup/Explorer opening.

Validation E22: updater verify/apply/repeat, baseline compatibility and unchanged runtime hash checks, tamper rejection, all completion-menu branches and invalid inputs, mocked two-session launcher lifecycle and marker cleanup, PowerShell/JavaScript parsing, ZIP hashes and excluded private payloads. Separate synthetic observer tests verified SHA-256 against Node crypto (ASCII, Unicode and long input), request-handle scoping, normalized credential matching and no raw synthetic secret/ID emission. No game was launched and no live credential captured in this preparation. Actual friend-side interception coverage and account correlation remain to be tested; this package is not proof of account equality or a licensing fix.

## 31. Crash Course Go account/CLIP diagnostic update
2026-11-10: Prepared outputs/CrashCourseGo-Full-Logging-Update.zip as a self-contained helper update for the existing Xbox Logging 1.1.37.4 installation. Update.cmd verifies payload and installed diagnostic prerequisite, then offers full logging, normal launch or exit. No APPX or installed-data replacement. CrashCourseReceiptTest uses unique helper/receipt files, native-only COM observation, the retained labeled legacy GUID receipt override, supported-hash-only CLIP hooks and hashed account-context comparison. Content-key scope is derived as 000000000000000000000000000000000000 from microsoft.crashcourse2:8wekyb3d8bbwe:win8xappx; this is a derived value, not newly observed catalog evidence. Metadata instrumentation retains Crash Course-specific SDK offsets.
Each stopped session creates its own ZIP, retaining loose originals. The installed legacy SDK logger remains build 6 with its pre-existing cumulative/rotation behavior; Capture-SdkDelta.ps1 exports only the current run's appended bytes into a unique file. This helper update does not alter that embedded logger. Runtime self-test, script syntax, payload hashes, mocked two-run launcher cleanup/archive isolation and SDK delta extraction tested; real licensed-account results remain untested. Installed saves/pop.bin, version, suspension and crash-dump configuration were not changed. Rebuild helper with work/crashcourse-full-diagnostics-source/CrashCourseFull.spec, then run work/build-crashcourse-full-update.py; tests in work/test-crashcourse-full-update.py.


Full installer companion: outputs/CrashCourseGo-1.1.37.4-Full-Diagnostics-Installer.zip includes the unchanged installed diagnostic game payload, locally signed APPX, original Microsoft-signed VC110 x86 dependency and the same helper runtime/Update.cmd. Install.cmd preserves verified existing data on replacement, offers explicit Y/N receipt import, enables crash dumps and suspension exemption, and invokes the logging/normal/exit menu after success. No local installation or game launch was performed. Rebuild with work/prepare-crashcourse-full-installer.py, MakeAppx/sign using existing local certificate, then work/finalize-crashcourse-full-installer.py. Validation report: work/crashcourse-full-diagnostics-validation.json. Private data are excluded from both archives.


## 32. Received Toshiba/Crash Course traces, 2026-11-10
E23: Toshiba 02:15 UTC receives a 1546-character legacy-GUID receipt but Xbox rejects its request with 0x8015DA87 and no requested token. SOAP reason is unspecified. Its failed IAsyncInfo inspection 0x8000000E must not override observed receipt delivery. Crash Course 02:20 UTC receives HTTP 200 responses but GetAppReceiptAsync fails four times with outer 0x80073DF6; no XML capture or SDK GetXblTokenResponse call is observed. CLIP returns zero keys/leases and satisfaction code 1 for each game's own content ID, with four MSA tickets and token broker status zero. Receipt broker PrimaryWebAccountId fingerprint and receipt Authorization fingerprint match across the two games; CLIP Authorization fingerprint also matches across games. Neither trace exposes a CLIP account ID. Thus account consistency across games is observed separately per path, but CLIP-versus-receipt account equality remains unproven. No raw identifiers or fingerprints retained in E23.
Correction to the earlier analysis (supersedes its mistaken correction): build-6 logger CORRECTLY labels 0x80073DF6 STORE_ERROR_UNLICENSED_USER. Decimal 15862 is 0x3DF6; decimal 15606 is 0x3CF6, making registration failure 0x80073CF6. The earlier claim of 0x80073EF6/registration failure was arithmetic error, not experimental evidence. Both supported Store DLLs explicitly map service 0xC03F300C to 0x80073DF6. No conclusion of an app-registration defect follows from these traces. The receipt/account entitlement cause remains unresolved. Crash Course archive has four expected new files and helper-stopped; Toshiba excerpt has no stop event. No receipt signature verification or game/data modification performed during E23 analysis.

## 33. Corrected arithmetic and receipt-service mapping probes
2026-11-10, E24. Original STORE_ERROR_UNLICENSED_USER label was correct; no registration failure established. Explicit disassembly of both exact supported Store DLLs confirms conversion 0xC03F300C -> 0x80073DF6, with embedded x86 diagnostic ChkHr(HRESULT_FROM_WIN32(15862L)). Logger source now spells out decimal/hex constants, separately labels true registration failure 0x80073CF6, and builds as 8; this DLL was compiled but NOT installed/repackaged, because the distributed build-6 original label is correct. Do not claim the installed logger changed.
Prepared CrashCourseGo-Receipt-Trace-Update.zip (~3 MB), a delta for either existing full diagnostics or full logging update folder. Update.cmd validates old/new runtime hashes, rolls back file replacements on failure, refreshes both manifest formats, remembers target and directly invokes its launch menu. It changes only helper runtime/scripts, never the game or app data. Adds bounded request/response inspection with numeric ReceiptResponse error fields, hashed allowlisted JSON account fields, opaque credential format labels, and architecture-and-hash-guarded native error conversion probes. No raw wire bodies/tokens emitted. Original receipt capture remains separate. Fresh local helper ready after fixing initial x86-offset-on-x64 coverage error: initial hook failed to attach, then helper detached and was rebuilt with correct x86/x64 guards. No fresh user-driven sign-in captured yet. Six preexisting non-log data files remain hash-identical; no pop.bin present at baseline, none removed.
Rebuild delta: work/prepare-crashcourse-receipt-delta.py, updater script retained in output, work/test-package-crashcourse-receipt-delta.py finalizes hashes/ZIP. Tests verify repeat application, manifest consistency, rejection of modified payload and ZIP integrity; numeric XML response parsing and identifier allowlists have synthetic tests. Account equality remains unresolved. Pending user interaction should not be described as completed validation.

## 34. Live receipt error conversion confirmed
2026-11-10 07:34 UTC (E25): Native CurrentApp AppId was zero; helper substituted correct Crash Course legacy GUID. Read-only x64 Store probe observed 0xC03F300C -> 0x80073DF6 at 07:34:08.048, followed by SDK receipt error/empty credentials and no token exchange. This confirms receipt-service authorization failure propagation, not package registration failure. Raw response capture was unavailable because the connection predated attachment; do not claim the raw response was read in this run. Receipt broker account fingerprint observed, but no comparable CLIP account ID, so equality remains unresolved. External helper stopped normally and its markers were removed by its own cleanup. Existing data verification reports 2 matching, 1 changed, 3 locked files while game remains open; no data/receipt removed. Local per-run investigation ZIP produced.

## 35. CLIP ticket collection account correlation; tracing limitation
2026-11-10 07:44 UTC, E26. Static inspection of the exact supported LicenseManager.dll identified a ticket-collection call at RVA 0x4c194. Its successful output is an array of 24-byte records: DWORD type at +0, HSTRING fields at +8 and +16. The caller processes type 1 into the MSA collection. A bounded, hash-only observer captured two type-1 records, with +8 string lengths 1445/1405 and +16 identifiers of 16 hexadecimal characters each. Field names remain positional: their full semantics have not been independently established.

The first +16 identifier exactly matches the receipt broker PrimaryWebAccountId fingerprint observed in prior local runs (including 07:22 session). This establishes that the broker-selected account identifier is represented in CLIP's returned ticket collection. It does not prove that only this account is evaluated, that a particular ticket is selected for the entitlement, or that these observations apply to the friend's machine. The outgoing 1893-character Authorization credential does not fingerprint-match either returned +8 value; different scope/wrapping or a separate credential remain hypotheses. A differing token hash is not evidence of a differing account.

The service subsequently crashed at 07:44:49 UTC: svchost.exe_LicenseManager, ntdll.dll+0x1d3df, exception 0xc00000ff (local SDK names STATUS_BAD_FUNCTION_TABLE). Instrumentation is suspected; exact offending hook is unresolved. External helper was stopped/detached and its markers cleaned. Do not interpret this run's 0x800706be lookup failure as evidence of an entitlement rejection, and do not distribute the experimental added hooks as validated. No game package or existing distribution archive was updated. The new WinHttpQueryOption URL fallback and ticket-record hook remain experimental source changes.

The preceding healthy receipt trace remains the authorization evidence: correct legacy GUID sent, native service mapping 0xc03f300c to 0x80073df6 STORE_ERROR_UNLICENSED_USER. Correct product identity is necessary but is not an entitlement grant. The precise server-side reason for denial remains unknown. Next isolate tracing stability, then link the returned ticket records to the serialized CLIP request using hashed identifiers; do not clear account/license caches or alter receipts to force this. Saves and receipts were not removed, replaced, or reset. Preservation status is recorded in E26; mutable files may be locked/changed by the running game.

Follow-up preparation: disassembly shows RVA 0x4e4fc calls an error routine and has no normal return (call followed by INT3). Its interception was removed from the experimental helper as a suspected unwind hazard; this is not yet a proven crash fix. A new helper session was attached to the existing game for a user-driven sign-in. No new sign-in result is claimed yet. All four tracked baseline files matched after the first session; pop.bin was absent before and afterward, and none was removed.

## 36. Computer-use sign-in and same-session account correlation
2026-11-10 11:19 UTC, E27. Launched Crash Course and clicked the title-screen continue control. The game displayed its Xbox connection failure without a separate authentication dialog. The helper replaced the invalid reported product GUID with 273de536-b14a-469b-a1cc-4f1179dc18d6 in the actual broker request. Native mapping again observed 0xc03f300c -> 0x80073df6, followed by SDK 0x82bc000c, empty credentials and no token acquisition. Raw receipt response body capture remains unavailable.

Within this same helper session, CLIP ticket record 0's 16-character identifier fingerprint matched the receipt broker PrimaryWebAccountId fingerprint. This strengthens earlier cross-run correlation; it proves inclusion of the broker account identifier in the CLIP collection, not final server account attribution or entitlement ownership.

The service tracing crash recurred before sign-in despite removing RVA 0x4e4fc: STATUS_BAD_FUNCTION_TABLE at ntdll+0x1d3df. The previous suspected single-hook cause is NOT confirmed. Do not use this CLIP lookup failure as clean entitlement evidence. The source helper now skips internal service instrumentation by default, requiring an explicit experimental flag; existing distributable archives were not changed and the frozen helper has not yet been rebuilt with that safeguard. Further CLIP live tracing should wait for isolation of the unwind failure. The external helper was stopped after the UI attempt. No saves or receipts were removed/replaced, and package files were unchanged. See E27 for tracked file verification; settings may be locked by the running game.

## 37. Stable replacement service probe and outgoing account correlation
2026-11-10 14:06-14:10 UTC, E28. Replaced the broad service interception set with two hash-guarded internal observations: target content-key entry establishes a time scope; the ticket-collection return captures bounded HSTRING metadata. Added one entry-only WinHttpSendRequest observer, parsing bounded JSON and emitting hashes only. Final source additionally requires the exact Crash Course contentId and limits emitted fields to the users subtree. No response callbacks, service failure/throw hook, service telemetry varargs hook, or install-return hook is active. SDK, receipt broker and COM lookup-result logging remain. This is a reduced-coverage replacement, not a claim that every historical service log layer is restored.

Two fresh game launches/sign-in attempts completed with the same LicenseManager process 36156 surviving, no new Application crash event, and native lookup returning 0x803f8001 rather than RPC termination. The helper detached normally each time. The previous broad hook set is retained at work/crashcourse-clip-service-unstable.js for offline analysis; the exact hook/unwind interaction responsible for STATUS_BAD_FUNCTION_TABLE remains unisolated. The replacement is enabled by default in source and rebuilt helper, with --disable-clip available. Exact DLL hash guard remains mandatory. Rebuilt helper at work/crashcourse-full-dist/CrashCourseReceiptTest; existing redist ZIPs are NOT updated.

CLIP request JSON includes users entries containing identityType, identityValue and localTicketReference. Both collected +8 HSTRINGs fingerprint-match identityValue fields; both +16 HSTRINGs match corresponding localTicketReference fields. One transmitted localTicketReference matches the same-session receipt broker PrimaryWebAccountId. Thus the broker Store account reference and its paired ticket are actually included in CLIP's outbound body, not merely present in an unused collection. The request carries a second account/ticket reference too. The 1893-character Authorization header was a misleading place to seek a one-to-one match with the returned MSA tickets: the returned values are carried in the body. This does not prove backend acceptance, entitlement grant, or the friend's account configuration. Correct receipt GUID still produces 0xc03f300c -> 0x80073df6 locally.

## 38. Missing SDK .dat lifecycle resolved
At the start of the follow-up, LocalState/Microsoft.Xbox/ED2EAF28-7260-4872-A8BB-F9EA9189FD79.dat had reappeared (20 bytes, timestamp 14:02:56 UTC). All six non-log app-data files were backed up before further testing to work/crashcourse-preserve-20261110-fixtrace. No original was deleted or replaced.

Read-only file hooks observed Microsoft.Xbox.dll+0x14886a call DeleteFileW successfully on this exact file immediately after receipt/sign-in failure, repeated in both tests. On normal game exit at 14:10:27 UTC, the SDK reopened/created it and Microsoft.Xbox.dll+0x148936 successfully wrote 20 bytes. The file is present afterward. Kernel32/KernelBase forwarding produced duplicate observations of the same API chain; do not count these as separate independent deletions. This explains temporary absence during a failed sign-in. The SDK manages this file; its precise payload semantics are not decoded, so do not label it a specific token or gameplay save. The investigator did not delete, restore or alter it. Current settings and SDK state can change through ordinary launch/exit; baseline comparison is in E28. pop.bin was absent at baseline and remained absent; none was removed. No gameplay was entered, package replacement performed, or career progress reset.

## 39. Complete stable-logging installer
2026-11-10, E29. Produced outputs/CrashCourseGo-1.1.37.4-Stable-Logging-Full-Installer.zip (167,259,408 bytes), including game APPX, VC110 dependency and standalone rebuilt helper. Game payload and embedded logger 6 unchanged; version 1.1.37.4 retained. No saves, profiles, pop.bin, tokens or prior logs bundled. Installer preserves existing data on replacement and includes explicit Y/N receipt picker, crash logs, suspension exemption and completion menu. Update.cmd remains the later logging/menu entry without requiring reinstallation.

Stable reduced CLIP probe replaces unstable broad service hooks; README explicitly describes coverage limits. Account summary now recognizes outbound localTicketReference matching receipt-broker PrimaryWebAccountId while retaining backend-evaluation uncertainty. SDK file lifecycle observation retained. Verification: all 2355 game payload files match installed/staged build, APPX/dependency/certificate signatures passed VerifyOnly, helper self-test and PowerShell parsing passed, repeat-session launcher tests confirmed separate ZIPs, prior-log exclusion, SDK delta and marker cleanup. No local installation was performed.

Rebuild: PyInstaller using work/crashcourse-full-diagnostics-source/CrashCourseFull.spec; copy its dist tree into the complete installer TestTools/CrashCourseReceiptTest, regenerate Updated-Runtime.json, then run work/finalize-crashcourse-full-installer.py. Retain the complete-folder README, launcher correction and account-summary extension. Finalizer validates unchanged APPX and stages before creating ZIP; rename its ZIP to the stable full-installer name. Test with work/test-crashcourse-stable-full.py before renaming.
