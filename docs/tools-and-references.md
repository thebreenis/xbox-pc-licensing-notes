# Tools, versions, and references

These tools were used or inspected; none of their binaries or dependency installers is included in this repository.

| Tool | Version / revision | Role |
| --- | --- | --- |
| Microsoft GDK | `2604.5.7903` | `PackageUtil`, `wdapp`, packaging/deployment inspection |
| Windows SDK | `10.0.22621` tools already available; 26100 SDK also installed with build prerequisites | Windows development tooling |
| Process Monitor | `4.12` | Manual launch/process/file capture |
| XboxDownload | Inspected local build `3.0.0.65` | Compared its Windows deployment route with wdapp |
| Xodus | `514dd56f360a4bdf12f4acde5148bac5ce5f957f` | Authentication, controlled content-license tests, SMD parser reference |
| Gaming Services | `39.118.21002.0` | Native prelaunch/update/licensing path |
| Rust / Cargo | `1.99.1` | Local Xodus Windows build |
| Visual Studio Build Tools 2022 | `17.14.42`, C++ workload | Local native build prerequisites |
| Protobuf compiler | `36.2` | Local Xodus build prerequisite |

Xodus additionally needed CMake and OpenSSL development inputs during the local build. Build dependencies are distinct from runtime prerequisites. The separate [Xodus Windows repository](https://github.com/thebreenis/xodus-windows) documents that build and hosts its artifacts; this repository does not duplicate them or create a release.

The local Xodus change added failure-response diagnostics without printing response bodies, tokens, or key bytes. It did not alter the entitlement decision. An upstream assessment that treated on-demand executable decryption as universally unavailable was superseded by inspection of the implementation's encrypted-file handling; extraction of this F1 package nevertheless remained untested because no key was issued.

## Primary references

- [Official F1 PC Store listing](https://www.xbox.com/en-us/games/store/F1-2018-PC-GP/9NTZV7H85TW9)
- [Microsoft public catalog product response](https://displaycatalog.mp.microsoft.com/v7.0/products/9NTZV7H85TW9?market=US&languages=en-us)
- [Microsoft package-license acquisition error reference](https://learn.microsoft.com/en-us/xbox/gdk/docs/reference/system/xstore/functions/xstoreacquirelicenseforpackageresult)
- [Microsoft MSIXVC2 encryption-model comparison](https://learn.microsoft.com/en-us/gaming/gdk/docs/features/common/packaging/packaging-encryption-msixvc2): useful context, but this test package was MSIXVC, not MSIXVC2.
- [MicrosoftGame.config and title identity](https://learn.microsoft.com/en-us/gaming/gdk/docs/features/common/game-config/microsoftgameconfig-overview)
- [Microsoft event-based versus title-managed achievements](https://learn.microsoft.com/en-us/xbox/gdk/docs/services/player-data/achievements/live-achievements-eb-vs-tm)
- [Xodus pinned source](https://github.com/xodus-gaming/xodus/tree/514dd56f360a4bdf12f4acde5148bac5ce5f957f)
- [XboxDownload source](https://github.com/skydevil88/XboxDownload)
- [CikExtractor upstream](https://github.com/LukeFZ/CikExtractor)
- [External Xbox research](https://github.com/xboneafide/xboxREsearch/tree/78e4fbc249a0d1175aa4ecbda873e1b8d0c5f332)

External findings are attributed to their authors. Links and service behavior may change after the observation date.
