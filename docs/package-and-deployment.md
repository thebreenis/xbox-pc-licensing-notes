# Package acquisition and Windows deployment

## Separate stages

The observed pipeline was:

`Public catalog metadata -> authenticated package lookup -> encrypted CDN package -> Windows staging -> registration -> native prelaunch checks -> licensed content access -> game process`

Each stage answers a different question. Finishing an earlier stage does not establish that a later one can succeed.

## Package acquisition

The Store ProductId resolved to a PC content ID and an MSIXVC package. The Xbox package service's `GetBasePackage` response supplied package metadata and a CDN path. The downloaded package was 43,476,717,568 bytes, version `1.0.6.0`.

Header inspection and GDK `PackageUtil` identified the expected PC package, content ID, build ID, and encryption-key ID. The complete 40.5 GiB file was not hashed during this investigation; header and metadata checks are not a full integrity verification.

Neither obtaining the CDN package nor reading its identifiers supplied a content-decryption key. Full download URLs and authentication material are deliberately omitted here.

## Why the Windows route progressed further than wdapp

The tested `wdapp install` attempt rejected the submission-signed package with `0x80070057`. That was a limitation of the attempted installation route, not proof that Windows could not deploy the package.

The inspected XboxDownload implementation invoked Windows `Add-AppxPackage`; it did not implement a separate game-decryption system. The Windows deployment path staged files but then failed registration with `0x80073CF6`: the manifest's `splash.png` could not be located.

The successful registration experiment was:

1. Stage the MSIXVC with `PackageManager.StagePackageAsync`, without registering it.
2. Supply missing root-level presentation images in the staged installation.
3. Register the staged package with `RegisterPackageByFullNameAsync`.

The added PNG placeholders were `splash.png` (620 × 300), `logo.png` (150 × 150), `small_logo.png` (44 × 44), and `store_logo.png` (50 × 50). No existing image was overwritten. The original downloaded MSIXVC was not repacked or re-signed.

Registration returned success and the package reported status `Ok`. Subsequent launches still failed licensing before gameplay. This experiment repaired presentation-file registration, not entitlement or encryption.

## Exposed files and protected content

The installation was exposed under the XboxGames layout, with its WindowsApps package path pointing to the content directory. Approximately 16,697 files were visible in enumeration. The launcher helper and manifest were readable, while reads of `game.dat` and a sampled ERP asset were denied, including from an elevated process.

Administrator ownership and full-control ACL entries were already present on sampled files. No ACL changes were made. These observations did not identify ordinary folder ownership as the cause of the protected reads.

Parsing SMD metadata later established that the main executable was explicitly marked to remain encrypted on disk. A filename found in metadata is not a readable executable, and a filename-string offset is not a payload offset.

## Cleanup

The Windows package was subsequently uninstalled, and its installed XboxGames directory was verified absent. Removal of the separate downloaded MSIXVC was blocked by the automation policy and left for manual deletion; that deletion was not verified as part of the documented tests.
