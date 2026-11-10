# Android Compatibility Audit

## Scope

This audit is based on the supplied `SAMP-Mobile-Launcher-RN-English.zip` archive. The GitHub repository currently contains only its short README, so these findings are from the supplied archive rather than a checkout of the repository source.

## Confirmed baseline

- React Native: `0.70.3`
- React: `18.1.0`
- Android minimum SDK: `21` (Android 5.0)
- Android compile SDK: `33`
- Android target SDK: `28`
- Android Gradle Plugin: `7.2.2`
- Gradle wrapper: `7.5.1`
- Application ID: `com.touch.mobile.dark`
- The app module explicitly filters native builds to `armeabi-v7a`.

## Native game libraries

The archive contains these shared libraries only under:

`android/app/src/main/jniLibs/armeabi-v7a/`

- `libSCAnd.so`
- `libGTASA.so`
- `libbass.so`
- `libsamp.so`
- `libImmEmulatorJ.so`

No matching `arm64-v8a` game libraries were present in the supplied archive.

## Compatibility implications

1. The React Native layer is older and its Android toolchain should be upgraded as a coordinated, tested migration rather than by changing Gradle and Android Gradle Plugin versions independently.
2. The app's explicit ABI filter restricts the packaged native app to 32-bit ARM.
3. Adding `arm64-v8a` to the ABI filter alone is not a valid 64-bit migration. The GTA SA / SA-MP native libraries and every other required native dependency must have compatible ARM64 builds.
4. A 64-bit-only Android device cannot load the supplied 32-bit game client libraries. Some 64-bit devices still support 32-bit apps, but that is device/vendor dependent.
5. The current target SDK is old. Raising it may change storage, notification, background execution, and package installation behavior; these areas require device testing.

## Recommended implementation sequence

1. Bring the complete source project into this repository (including lockfiles and required assets, excluding credentials and signing secrets).
2. Establish a reproducible baseline build with the existing dependency/toolchain versions.
3. Add CI build checks and document the required JDK, Android SDK, NDK, and package manager versions.
4. Upgrade the React Native / Android build toolchain in small, reviewable steps, validating each native module.
5. Audit scoped storage, notification permission, download/install flows, and Android package installation behavior against supported Android releases.
6. Obtain legitimate ARM64 builds of the game client and all required native libraries from their maintainers. Do not relabel or copy ARM32 binaries into an ARM64 directory.
7. Produce separate test builds and verify installation, game startup, audio, controls, server query, downloads, and updates on representative devices.

## Release gate

Do not advertise full ARM64 game support until an APK containing real `arm64-v8a` game-client libraries has been built and tested on a 64-bit-only Android device.

## Security note

Before making this repository public or uploading the full source, review `.env`, Firebase configuration, debug keystores, signing files, API credentials, and any distribution URLs. Keep private signing keys and tokens out of Git history. Rotate any credential that has already been exposed publicly.
