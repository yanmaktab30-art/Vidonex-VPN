# Vidonex VPN — Android Application

[![Build Status](https://img.shields.io/badge/build-passing-brightgreen)](#)
[![Min SDK](https://img.shields.io/badge/minSdk-26%20(Android%208.0)-blue)](#)
[![Target SDK](https://img.shields.io/badge/targetSdk-34%20(Android%2014)-blue)](#)

**Vidonex VPN** is a production-oriented Android VPN application built with Jetpack Compose and Material 3. It uses the WireGuard protocol via the official wireguard-android tunnel library.

## Features
- WireGuard-based VPN — fast, modern, audited protocol
- Premium dark UI — navy/cyan Material 3 Jetpack Compose design
- Server management — import .conf files, manage multiple profiles
- Privacy-first — no analytics, no trackers, no telemetry
- Uzbek + English localization
- Foreground service notification showing connection state
- Accessible — TalkBack labels and sufficient touch targets

## Prerequisites

| Tool | Version |
|------|---------|
| JDK | 17 (Temurin) |
| Android SDK | API 34 + build-tools 34.0.0 |
| Gradle | 8.6 (via wrapper) |

## Building on Windows

Set environment variables in PowerShell:
```
$env:JAVA_HOME = "C:\path\to\jdk-17"
$env:ANDROID_HOME = "C:\path\to\android-sdk"
$env:PATH = "$env:JAVA_HOME\bin;" + $env:PATH
```

Debug APK:
```
cmd.exe /c "gradlew.bat assembleDebug --no-daemon"
```
Output: app/build/outputs/apk/debug/app-debug.apk

Release AAB:
```
cmd.exe /c "gradlew.bat bundleRelease --no-daemon"
```
Output: app/build/outputs/bundle/release/app-release.aab

Unit tests:
```
cmd.exe /c "gradlew.bat testDebugUnitTest --no-daemon"
```

## Connecting to a VPN Server

This app requires a valid WireGuard server configuration to connect.
1. Obtain a WireGuard .conf file from your VPN server administrator.
2. Open the app -> tap Servers -> tap the Import button.
3. Select the .conf file from your device storage.
4. The imported profile appears in the server list, marked as ready.
5. Tap the profile -> return to Dashboard -> tap Connect.

See SETUP_SERVER.md for instructions to deploy your own WireGuard server.

## Security

See SECURITY.md for the threat model and responsible disclosure policy.
Private keys and WireGuard configurations are never committed to source control,
never printed in logs, and never exposed in UI debug screens.

## Support

Email: support@vidonex.com (placeholder - update with real contact)
Website: https://vidonex.com (placeholder - update with real URL)