# Speedtest Privacy

A modern Xposed module for Ookla Speedtest (`org.zwanoo.android.speedtest`). It suppresses optional permission requests, advertising, telemetry and background measurements while retaining Internet and network access for speed tests. It also fixes the onboarding loop triggered by permanently denied location permission.

Version **1.1.0** discovers 16 important hooks from stable strings and method structure, validates callback contracts, and caches matches for the installed APK set. Remaining obfuscated hooks run only on the verified **Speedtest 7.1.4 / 265370** APK. The settings screen reports compatibility gaps after an update; see [update hardening](https://github.com/Loukious/io.github.loukious.speedtestprivacy/blob/main/docs/update-hardening.md).

[Download v1.1.0](https://github.com/Xposed-Modules-Repo/io.github.loukious.speedtestprivacy/releases/tag/3-1.1.0) · [Source](https://github.com/Loukious/io.github.loukious.speedtestprivacy) · [Support](https://github.com/Loukious/io.github.loukious.speedtestprivacy/issues)

## Controls

All five switches default to **enabled**, including when framework preferences are unavailable. Open **Speedtest Privacy** to change them, then force-stop and reopen Speedtest.

| Switch | Protection |
| --- | --- |
| Block optional permissions | Returns denial for location, phone, notifications, storage and other optional requests; skips the permanent-denial onboarding loop. |
| Block ads | Suppresses banner/native ads, ad SDK initialization and known ad endpoints. |
| Block telemetry and report uploads | Disables analytics, crash/performance collection, map telemetry and optional report uploads. The legacy result flow can continue after local save finishes without uploading the report. |
| Block background measurements | Stops coverage sampling, background report services and known scheduled workers. |
| Hide device identifiers | Redacts advertising/Android IDs, phone identifiers, Wi-Fi identifiers and device fields in known payloads. |

Internet, network/Wi-Fi state, local network access, wake locks and foreground execution remain available for testing. Account synchronization and result sharing may be unavailable while report uploads are blocked.

## Requirements and build

- Android 8.1 or newer, with root and a modern Xposed framework supporting **libxposed API 102**.
- Tested framework: **Vector-SR 1.2**. Compatibility with other frameworks has not been established.
- JDK 21 and Android SDK platform/build tools 36.

Clone the source repository to build the module:

```sh
git clone https://github.com/Loukious/io.github.loukious.speedtestprivacy.git
cd io.github.loukious.speedtestprivacy
./gradlew :app:assembleDebug :app:testDebugUnitTest :app:lintDebug :app:assembleDebugAndroidTest
```

The APK is written to `app/build/outputs/apk/debug/SpeedtestPrivacy-v1.1.0-debug.apk`. Configure your local SDK path separately; `local.properties` is machine-specific. Add `--offline` after dependencies have been cached if you want an offline build.

## Install and enable

Install with the Windows platform-tools executable, including from WSL:

```sh
adb.exe install -r "$(wslpath -w "$PWD/app/build/outputs/apk/debug/SpeedtestPrivacy-v1.1.0-debug.apk")"
```

If `adb.exe` is not on `PATH`, use its path in your Windows platform-tools directory. In the framework manager, enable **Speedtest Privacy** and select only **Speedtest** (`org.zwanoo.android.speedtest`) as its scope. Force-stop and reopen Speedtest.

On the tested Vector installation, its supported CLI can set the owner-profile scope and enable the module:

```sh
adb.exe shell su -c '/data/adb/lspd/cli scope set --json io.github.loukious.speedtestprivacy org.zwanoo.android.speedtest/0'
adb.exe shell su -c '/data/adb/lspd/cli scope ls --json io.github.loukious.speedtestprivacy'
adb.exe shell su -c '/data/adb/lspd/cli modules ls --json --enabled'
adb.exe shell am force-stop org.zwanoo.android.speedtest
```

`/0` selects Android's owner profile. Use the manager for a different profile. Open Speedtest normally after the force-stop.

## Validation and limits

Version **1.1.0** was validated with only Speedtest in its scope on Android 17/API 37 with Vector-SR 1.2 (3131). All 16 semantic targets resolved and installed, 207 hooks were active, and all seven runtime checks passed with no recorded errors. The module UI received a UID-checked Binder report and displayed **Speedtest 7.1.4 · protections matched** with the framework connected.

A speed test completed with both completion flags set and remote ID `-1` in the retained local database. Location, phone, notifications and storage remained denied at the Android level. Mapbox's real native collection getter reported disabled, and maps still rendered.

All **50 unit tests** and **11 Android fixture tests** passed, covering class/method renaming, cache invalidation with unchanged version labels, ambiguous matches, overlapping/shadowed split definitions, and forged-status UID rejection. An additional live test verified the renamed module’s authenticated Binder reply. The real module-to-Speedtest Binder path was checked separately through the settings UI. Lint completed with no errors. See the [validation summary](https://github.com/Loukious/io.github.loukious.speedtestprivacy/blob/main/docs/validation.md).

The earlier native audit found Mapbox's collection flag enabled in 1.0.0. Version 1.0.1 added the actual native disable operation and independent C++ probes confirmed it, including after map use. Version 1.1.0 retains those controls and validates the native HTTP bridge members before installing its guard.

These checks cover the inspected controls and known paths; they are not an exhaustive packet/syscall audit. Stable anchors can survive renaming, but SDK changes, removed anchors, different state contracts or new collection routes still require verification. Unknown builds show partial compatibility and skip remaining version-specific hooks.

- Test servers and other network peers still see your IP address and test traffic.
- Known native telemetry bridges were checked and Mapbox native collection controls were verified; new SDK versions and unexamined transports remain unproven.
- Queued reports are retained and may upload if the relevant protection or module is disabled.
- Hooked permission checks do not revoke Android's existing permission grants. Review actual grants in Android settings.

## Rollback

Disable the module in the framework manager, or uninstall it:

```sh
adb.exe uninstall io.github.loukious.speedtestprivacy
adb.exe shell am force-stop org.zwanoo.android.speedtest
```

Reopen Speedtest to remove hooks from its process. Disabling a switch also requires restarting Speedtest.

## Support development

[Buy me a coffee](https://buymeacoffee.com/loukious)
