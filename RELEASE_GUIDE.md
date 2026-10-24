# Release Guide — Vidonex VPN

This guide provides step-by-step instructions for signing, verifying, and publishing **Vidonex VPN** to the Google Play Store or private distribution channels.

---

## 1. Keystore Creation (Production Signing)

The current release bundle was built with the default Gradle release config without a hardcoded private signing key (or debug signing for testing). For Google Play Store submission, you must sign the bundle using a secure upload keystore.

### Generate an Upload Keystore:

Run the following command using the JDK `keytool`:

```bash
keytool -genkey -v -keystore vidonex-release.keystore -alias vidonex-upload -keyalg RSA -keysize 4096 -validity 10000
```

Store this keystore file and its passwords in a secure password manager. **Never commit the `.keystore` file to Git.**

---

## 2. Configuring Signing in `app/build.gradle.kts`

For secure CI/CD or local builds without exposing passwords in source code, use environment variables:

```kotlin
android {
    ...
    signingConfigs {
        create("release") {
            storeFile = file(System.getenv("VIDONEX_KEYSTORE_PATH") ?: "vidonex-release.keystore")
            storePassword = System.getenv("VIDONEX_KEYSTORE_PASSWORD")
            keyAlias = System.getenv("VIDONEX_KEY_ALIAS") ?: "vidonex-upload"
            keyPassword = System.getenv("VIDONEX_KEY_PASSWORD")
        }
    }
    buildTypes {
        release {
            signingConfig = signingConfigs.getByName("release")
            isMinifyEnabled = true  // Enable when building on CI/CD with >= 4GB RAM
            isShrinkResources = true
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro"
            )
        }
    }
}
```

---

## 3. Verifying the Android App Bundle (AAB) with Bundletool

Before uploading `app-release.aab` to the Google Play Console, verify that it can generate valid APKs for target devices:

### Step 3.1: Generate APKs Archive from AAB

```powershell
$env:JAVA_HOME = "C:\Users\user\.gemini\antigravity\tools\jdk-17.0.20.1+1"
& "$env:JAVA_HOME\bin\java.exe" -jar "C:\Users\user\.gemini\antigravity\tools\bundletool-all.jar" build-apks `
    --bundle="app\build\outputs\bundle\release\app-release.aab" `
    --output="app\build\outputs\bundle\release\app-release.apks" `
    --mode=universal
```

### Step 3.2: Inspect Estimated Download Size

```powershell
& "$env:JAVA_HOME\bin\java.exe" -jar "C:\Users\user\.gemini\antigravity\tools\bundletool-all.jar" get-size total `
    --apks="app\build\outputs\bundle\release\app-release.apks"
```

### Step 3.3: Install on Connected Test Device (via ADB)

```powershell
& "$env:JAVA_HOME\bin\java.exe" -jar "C:\Users\user\.gemini\antigravity\tools\bundletool-all.jar" install-apks `
    --apks="app\build\outputs\bundle\release\app-release.apks"
```

---

## 4. Google Play Console Submission Checklist

### A. App Identification & Store Listing
- **App Name:** Vidonex VPN
- **Short Description:** Fast, secure, and modern WireGuard VPN client.
- **Full Description:** Highlight WireGuard protocol support, clean dark UI, privacy protection, and custom configuration file import capabilities.
- **App Icon:** High-res icon (512x512 PNG, 32-bit).
- **Screenshots:** Phone screenshots (min 2, max 8) for Dashboard, Server List, Diagnostics, and Settings screens.

### B. Google Play VPN Service Policy Compliance
Google Play has strict policies regarding `android.net.VpnService`:
1. **Core Functionality Declaration:** Declare that VPN is the core functionality of Vidonex VPN.
2. **Foreground Service Declaration:** Select `systemExempted` or `specialUse` / `dataSync` in compliance with Android 14 requirements.
3. **User Disclosure:** Ensure the in-app onboarding dialog explicitly notifies users that the app establishes an encrypted VPN tunnel.

### C. Data Safety Form
- **Data Collection:** Select **No** — the app does not collect or share any user data.
- **Data Transfer:** Select **Encrypted in transit** (all VPN tunnel traffic is cryptographically secured via WireGuard).
- **Data Deletion:** Not applicable as no server-side user accounts exist.

### D. Privacy Policy URL
Google Play requires an active public URL pointing to your privacy policy. Host the contents of `PRIVACY_POLICY.md` on your domain (e.g., `https://vidonex.com/privacy`) and enter it in App Content -> Privacy Policy.

---

## 5. Rollout Strategy

1. **Internal Testing Track:** Upload `app-release.aab` and invite internal QA testers. Verify configuration file import and tunnel activation across Android 8, 10, 12, 13, and 14 devices.
2. **Closed Testing Track:** Roll out to external beta testers. Monitor crash reports and device-specific connection logs.
3. **Production Staged Rollout:** Start with 10% -> 25% -> 50% -> 100% rollout to monitor stability in production.