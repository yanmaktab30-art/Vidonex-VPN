# Security Policy — Vidonex VPN

## Threat Model

### Assets Protected
- User WireGuard private keys and preshared keys (stored on device)
- User DNS queries (routed through VPN tunnel when connected)
- User network traffic (encrypted through WireGuard tunnel when connected)
- App configuration and server profiles

### Threats Considered

| Threat | Mitigation |
|--------|-----------|
| Private key leakage via logs | Keys are redacted before any logging via `sanitizeForLog()` |
| Private key leakage via crash reports | Keys never stored in plain fields that appear in stack traces |
| Fake connection state | VPN state tracked via `VpnService` callbacks, not UI animation |
| DNS leak | DNS configured in WireGuard tunnel builder; AllowedIPs=0.0.0.0/0 routes all traffic |
| Traffic outside tunnel on failure | Android VpnService blocks non-tunnel traffic by default when established |
| Unauthorized VPN activation | Requires explicit Android VPN consent dialog per OS requirement |
| Malicious .conf import | Parser validates required fields before accepting configuration |
| Clipboard key exposure | No clipboard actions on private key fields |
| Third-party telemetry | No analytics, ads, or tracking SDKs included |
| Insecure backup | allowBackup=false in AndroidManifest; dataExtractionRules configured |

### Threats NOT Mitigated by This App
- Compromise of the WireGuard server itself
- Network-level traffic analysis (timing attacks, traffic shape analysis)
- Endpoint compromise (malware on the Android device)
- Legal compulsion of the server operator to disclose logs
- Physical device access
- Anonymity (WireGuard is a VPN, not an anonymization network)

---

## Security Controls

### Key Material Handling
- Private keys exist only in memory during connection setup
- Keys imported from .conf files are stored in SharedPreferences
  (production hardening: migrate to Android Keystore or EncryptedSharedPreferences)
- `sanitizeForLog()` redacts PrivateKey and PresharedKey values in all log output
- Keys are never placed in Bundle extras, Intent data, or debug screen text fields

### Permissions (Minimal Set)
- `android.permission.INTERNET` — required for VPN tunnel socket
- `android.permission.ACCESS_NETWORK_STATE` — detect network changes for reconnect
- `android.permission.FOREGROUND_SERVICE` — required for persistent VPN service
- `android.permission.FOREGROUND_SERVICE_SYSTEM_EXEMPTED` — VPN service type (Android 14+)
- `android.permission.POST_NOTIFICATIONS` — VPN status notification (Android 13+, runtime)
- No location, contacts, camera, microphone, or storage permissions

### No Telemetry
- No analytics SDK (no Firebase, Amplitude, Mixpanel, etc.)
- No advertising SDK
- No crash reporting SDK transmitting data off-device
- No usage tracking or behavioral profiling

### Network Security
- No cleartext HTTP traffic permitted (enforced via network security config defaults)
- Certificate verification not disabled anywhere in the codebase
- No custom TrustManager that accepts all certificates

### Build Security
- `allowBackup=false` prevents ADB backup of app data
- `extractNativeLibs=false` uses compressed native libraries
- Release builds use AAB format (Google Play signing)
- ProGuard rules keep WireGuard JNI classes; no broad wildcard keep rules

---

## Known Limitations and Caveats

1. **SharedPreferences storage** — WireGuard configs are stored in plain SharedPreferences.
   Production hardening should use `EncryptedSharedPreferences` from Jetpack Security.

2. **No perfect forward secrecy for stored keys** — If the device is compromised, stored
   private keys may be recoverable. Use hardware-backed key storage for highest security.

3. **DNS leak prevention** — The app configures DNS in the WireGuard tunnel. If the OS or
   a third-party app forces alternative DNS resolution outside the tunnel, leaks may occur.
   Use Android's private DNS (DNS-over-TLS) setting as an additional layer.

4. **VPN always-on** — The app does not force always-on VPN. This is a user-controlled
   Android system setting. Explain to users who require it how to enable it in Settings.

5. **No audit** — This application has not been independently audited. Use at your own risk.

---

## Responsible Disclosure

If you discover a security vulnerability in Vidonex VPN:

1. **Do NOT** file a public GitHub issue or post on social media.
2. Email: security@vidonex.com *(placeholder — update with real security contact)*
3. Provide a clear description, reproduction steps, and impact assessment.
4. Allow reasonable time for investigation and remediation before disclosure.
5. We will acknowledge receipt within 72 hours and provide a timeline for resolution.

We do not currently operate a bug bounty program but appreciate responsible disclosure.

---

## Dependency Security

All third-party dependencies are declared in `gradle/libs.versions.toml`.
See the dependency/license inventory in `BUILD_REPORT.md`.

Regularly update dependencies and check for known CVEs:
```
gradlew dependencyUpdates    # requires com.github.ben-manes.versions plugin
```