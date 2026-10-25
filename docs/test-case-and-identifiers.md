# F1 test case and public identifiers

F1 2018 PC GP was used to test an encrypted Microsoft Store PC package without a valid game license. This page records public package identifiers for reproducibility, not account identifiers or secret keys.

| Field | Observed value |
| --- | --- |
| Store ProductId | `9NTZV7H85TW9` |
| Package family | `CodemastersSoftwareCompan.F12018PCGP_4cfye3zbe1gaw` |
| Package version | `1.0.6.0` |
| Architecture / format | x64 / MSIXVC |
| Downloaded bytes | `43476717568` |
| Content ID | `f5b2ad33-1ca5-4d69-a0bf-5c305c5fef82` |
| Package ID | `7e12dfcb-8468-46a4-a156-b0522c41fe4d` |
| Build ID | `770a078f-f991-4835-bc8e-d2f2c54e5fd5` |
| Encryption-key ID, not key bytes | `7b621323-5018-ac12-e18d-9b3c9858fea7` |
| Xbox title ID | `1901507130` / `0x7156b23a` |
| Legacy Windows Store product GUID | `a497ddd3-7da2-446f-acfe-cf6f2da3f8cb` |
| Legacy Windows Phone product GUID | `59889225-83d9-4c87-87b4-2965e5dc38bf` |
| Windows Update category | `16a6acc6-3654-4f83-8127-65ae66c93d85` |
| Windows Update bundle | `6d88a6ec-af4b-4f99-910f-6c61bd88bd90` |

## SMD metadata inspection

A parser based on pinned Xodus source examined 17,088 SMD segment entries. Selected observations:

| File | Segment index | Bytes | Keep encrypted on disk |
| --- | --- | --- | --- |
| `MicrosoftGame.config` | 0 | 1,887 | False |
| `splash.png` | 4 | 3,749,524 | False |
| `F1_2018.exe` | 8 | 30,350,848 | True |
| `game.dat` | 6,544 | 1,846,250,752 | False |

The main executable's filename string occurred at SMD offset `0x42D90`. That offset locates the name, not the executable payload. An earlier exploratory note had not mapped the filename to a segment; the later parsed segment table supersedes that uncertainty. No executable bytes were decrypted or extracted.

The original package reported two chunks: `0x40000003` at 7,950,336 bytes and `0x3E8` at 43,130,019,840 bytes. Those sizes alone do not locate a specific file payload.

## Legacy lookup results

The catalog's full and trial SKUs (`0010` and `0011`) referenced the same PC content ID. Official alternate-ID lookups for both legacy Windows GUIDs returned the current `9NTZV7H85TW9` product. A legacy GUID is consequently not evidence of a separate historical APPX release.

Public satisfying-entitlement entries referenced current product SKUs, the two legacy Windows GUIDs, the legacy Xbox product GUID, and `BWWMGZJVPXT8`. Queries for the last identifier returned not found in the tested markets. Its precise product identity was not established. Public satisfying-entitlement entries describe accepted catalog relationships, not ownership by a particular account.

Windows Update queries for the known category returned no package links. Exploratory queries using the legacy Store GUID and the bundle GUID also returned none; those exploratory IDs were not established Windows Update categories.

No older downloadable APPX or alternate Windows content ID was found. This is a search result, not proof that no older build ever existed. Microsoft's [October 2019 PC Game Pass announcement](https://news.xbox.com/en-us/2019/10/09/xbox-game-pass-for-pc-october-2019/) confirms the historical PC offering, but does not establish its original package format.
