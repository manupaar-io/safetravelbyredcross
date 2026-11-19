# Security Policy & Architecture: Safe Travel Sri Lanka

**Version:** 1.0.0  
**Target Platform:** Flutter 3.x / Dart 3 (Android & iOS)  
**Backend:** .NET Core RESTful API  
**Affiliation:** Sri Lanka Red Cross Society  
**Last Updated:** 2026-11-19  

---

## 1. Executive Summary & Security Posture

Safe Travel Sri Lanka is an offline-first mobile application providing life-critical travel advisory, real-time hazard alerts (floods, landslides, severe weather), IFRC-aligned first-aid guides, and one-tap emergency SOS dispatch.

While the .NET backend API serves as the primary system of record and enforces backend-level authorization, rate limiting, and business validation, the mobile client holds substantial security responsibility. A mobile app is an untrusted client operating on an untrusted device. This document defines the security boundaries, threat model, compliance targets, and defensive architecture for the Flutter/Dart codebase.

---

## 2. Threat Model & Asset Classification

### 2.1 Critical Client-Side Assets
| Asset | Classification | Storage Location | Threat Exposure |
|---|---|---|---|
| **Emergency Contacts (ICE)** | High (PII) | `flutter_secure_storage` | Unauthorized exfiltration, stalking, data mining |
| **User GPS Coordinates** | High (Real-time PII) | In-memory only (never cached to plaintext disk) | Location tracking, physical surveillance |
| **Auth & Refresh Tokens** | High (Credentials) | `flutter_secure_storage` | Session hijacking, account takeover |
| **Offline Safety Content** | Medium (Integrity) | SQLite (`sqflite`) / bundled assets | Content tampering, malicious first-aid modification |
| **API Keys (Maps, Weather)** | Medium (Cost/Usage) | Encrypted compile-time flags (`--dart-define`) | API quota exhaustion, billing abuse |
| **App Settings / UI Theme** | Low (Public) | `shared_preferences` | Tampering with preferences |

### 2.2 Primary Threat Vectors
1. **Man-in-the-Middle (MitM) Attacks:** Intercepting hazard warnings, alerts, or emergency telemetry over public Wi-Fi or compromised mobile networks.
2. **Local Data Extraction (Physical / ADB Access):** Extracting unencrypted SQLite databases, SharedPreferences, or cached JSON files via `adb backup` or device physical acquisition.
3. **App Tampering & Reverse Engineering:** Decompiling APK/IPA, extracting hardcoded secrets, modifying safety instructions, or injecting spoofed emergency numbers.
4. **Dialer / URI Scheme Injection:** Exploiting SOS quick-dial buttons to execute arbitrary USSD codes (e.g., `*21*`, `##002#`) or premium numbers.
5. **SQL Injection in Local Cache:** Exploiting offline search queries to corrupt local SQLite databases.
6. **Information Leakage via Logs & Screenshots:** Exposing user coordinates, phone numbers, or tokens in Android Logcat or iOS Console logs.

---

## 3. Network & Transport Layer Security

All communication with the .NET backend and external third-party services must adhere to strict transport-level hardening.

### 3.1 TLS Enforcement
- **Minimum Protocol:** TLS 1.3 preferred, TLS 1.2 strict minimum. Any TLS 1.0/1.1 or SSL connection will be rejected at the socket layer.
- **Cleartext Traffic Policy:** Cleartext (`http://`) traffic is strictly forbidden across all environments, including local development (use self-signed or mkcert HTTPS for dev).

#### Android Configuration (`android/app/src/main/res/xml/network_security_config.xml`)
```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <!-- Global default: Disallow all cleartext HTTP traffic -->
    <base-config cleartextTrafficPermitted="false">
        <trust-anchors>
            <certificates src="system" />
        </trust-anchors>
    </base-config>
    <domain-config cleartextTrafficPermitted="false">
        <domain includeSubdomains="true">api.safetravel.redcross.lk</domain>
        <!-- Pin SPKI fingerprint in release builds -->
    </domain-config>
</network-security-config>
```

#### iOS Configuration (`ios/Runner/Info.plist`)
```xml
<key>NSAppTransportSecurity</key>
<dict>
    <key>NSAllowsArbitraryLoads</key>
    <false/>
</dict>
```

### 3.2 Certificate Pinning Architecture
For production communication with the .NET backend (`api.safetravel.redcross.lk`), the hand-rolled `http` client or custom `SecurityContext` must validate the certificate public key (SPKI pinning).

```dart
// lib/config/security/secure_http_client.dart
import 'dart:io';
import 'package:http/io_client.dart';

class SecureHttpClient {
  static IOClient createClient({required List<String> trustedCertFingerprints}) {
    final SecurityContext context = SecurityContext(withTrustedRoots: true);
    
    final HttpClient rawClient = HttpClient(context: context)
      ..badCertificateCallback = (X509Certificate cert, String host, int port) {
        // Reject invalid certs; in production, verify cert.sha256 matches trusted list
        return false;
      }
      ..connectionTimeout = const Duration(seconds: 15)
      ..idleTimeout = const Duration(seconds: 30);

    return IOClient(rawClient);
  }
}
```
*Note: Always provide a certificate rotation strategy with backup pinning fingerprints to prevent app lockouts upon certificate renewal.*

### 3.3 Request Integrity & Correlation
Every request dispatched from the mobile app to the .NET backend must include:
- `X-Request-ID`: UUIDv4 generated per request for end-to-end tracing in .NET Serilog/Application Insights.
- `X-Client-Timestamp`: ISO 8601 UTC timestamp to detect clock drift and deter replay attacks.
- `X-Client-Version`: App build version (e.g., `1.0.0+1`) to allow the backend to reject outdated, vulnerable clients.
- `User-Agent`: Structured string (e.g., `SafeTravelLK/1.0.0 (Android 14; Mobile; RedCross)`).

---

## 4. Local Data Storage & Secrets Architecture

The app uses a strict 3-tier local storage partition based on sensitivity:

```
┌─────────────────────────────────────────────────────────────┐
│                 Local Storage Architecture                  │
├──────────────────────────┬──────────────────────────────────┤
│ 1. flutter_secure_storage│ Credentials, ICE Contacts, Keys  │
│    (OS Keychain / Keystore AES-256 GCM)                     │
├──────────────────────────┼──────────────────────────────────┤
│ 2. sqflite               │ Offline Guides, Cached Alerts    │
│    (SQLite local database, fully parameterized queries)     │
├──────────────────────────┼──────────────────────────────────┤
│ 3. shared_preferences    │ ThemeMode, Locale, UI Flags      │
│    (Zero PII, Zero Tokens, Non-sensitive settings only)     │
└──────────────────────────┴──────────────────────────────────┘
```

### 4.1 Sensitive Data in `flutter_secure_storage`
- **Android:** Uses Android KeyStore with EncryptedSharedPreferences (`encryptedSharedPreferences: true`) backed by hardware security modules (TEE / StrongBox where available). Keys are encrypted with AES-256-GCM.
- **iOS:** Uses Keychain with `kSecAttrAccessibleAfterFirstUnlockThisDeviceOnly` backed by Apple's Secure Enclave. Prevents keys from syncing to iCloud or restoring onto an unverified device.
- **Items strictly stored here:**
  - Passwords, JWT Access & Refresh Tokens (if authenticated)
  - Emergency contact phone numbers, names, and relationships (ICE details)
  - User's blood type, allergies, and critical medical notes
  - 256-bit AES master encryption keys for offline databases

### 4.2 SQLite Security (`sqflite` & `sqflite_sqlcipher`)
- **SQL Injection Prevention:** Absolute ban on string concatenation in SQL commands. Always use parameterized queries with `whereArgs`.
  ```dart
  // CORRECT: Parameterized
  await db.query('safety_topics', where: 'category = ?', whereArgs: [category]);

  // VULNERABLE: FORBIDDEN
  await db.rawQuery("SELECT * FROM safety_topics WHERE category = '$category'");
  ```
- **Database Encryption at Rest (SQLCipher):** For offline personal data (custom trip itineraries, offline medical logs):
  - The SQLite database is encrypted page-by-page using 256-bit AES-CBC (`sqflite_sqlcipher`).
  - The encryption passphrase is generated on initial app launch via a CSPRNG (`Random.secure()`, 256-bit entropy) and stored exclusively in `flutter_secure_storage` (Android KeyStore / iOS Keychain).
- **Android Backup Prevention:** Prevent local databases and app files from leaking via Google Cloud backup or USB ADB commands:
  In `android/app/src/main/AndroidManifest.xml`:
  ```xml
  android:allowBackup="false"
  android:fullBackupContent="false"
  ```

### 4.3 In-Memory Lifecycle for High-Risk Data
- **Live GPS Coordinates**: Kept in volatile memory (RAM) only during active navigation or map display. Never written to persistent storage (no disk tracking).
- **Passwords & Decrypted Tokens**: Cleared from memory immediately upon logout or failure. Never persisted in static or global variables longer than active usage.

### 4.3 Zero Secrets in Source Code
- **Rule:** Never commit secrets, private endpoints, or staging credentials into git.
- **Build-Time Injection:** Use `--dart-define` or `--dart-define-from-file=.env.json` at build time:
  ```bash
  flutter build apk --dart-define=API_BASE_URL=https://api.safetravel.redcross.lk --dart-define=MAPS_API_KEY=AIzaSy...
  ```
- Access compile-time variables strictly through `AppEnvironment`:
  ```dart
  class AppEnvironment {
    static const String apiBaseUrl = String.fromEnvironment('API_BASE_URL');
    static const String mapsApiKey = String.fromEnvironment('MAPS_API_KEY');
  }
  ```

---

## 5. Reverse Engineering & Anti-Tampering Defenses

Because mobile apps are distributed as binaries to potentially adversarial devices:

### 5.1 Dart Code Obfuscation
Release builds must compile with symbol obfuscation and separate debug info to prevent decompilation into legible Dart source:
```bash
flutter build apk --release --obfuscate --split-debug-info=build/app/outputs/symbols
flutter build ipa --release --obfuscate --split-debug-info=build/ios/outputs/symbols
```

### 5.2 Native Android R8 / ProGuard
Ensure code shrinking, resource shrinking, and ProGuard obfuscation are active in `android/app/build.gradle.kts`:
```kotlin
buildTypes {
    release {
        isMinifyEnabled = true
        isShrinkResources = true
        proguardFiles(
            getDefaultProguardFile("proguard-android-optimize.txt"),
            "proguard-rules.pro"
        )
    }
}
```

### 5.3 Screen & Task Switcher Protection
For sensitive screens (Emergency Contacts, Personal Medical Profile):
- Android: Enable `FLAG_SECURE` to block screenshots and screen recording.
- iOS: Obscure or blur the snapshot view when the app transitions to the background in the multitasking app switcher.

---

## 6. Input Validation & Platform Interop Security

### 6.1 SOS Dialer & Telephony Hardening
The SOS module triggers phone calls to national emergency dispatchers (119, 110, 1990).
- **Tel URI Sanitization:** All numbers launched via `url_launcher` must pass strict regex validation to prevent USSD code injection:
  ```dart
  static final RegExp _validEmergencyNumber = RegExp(r'^\+?[0-9]{3,12}$');

  static bool isValidEmergencyNumber(String number) {
    // Rejects #, *, commas, or URL query parameters
    return _validEmergencyNumber.hasMatch(number.trim());
  }
  ```
- **Confirmation Guard:** High-consequence dials must require an intentional confirmation tap or slide interaction to prevent accidental pocket-dials.

### 6.2 External URLs & Web Links
When opening external links (e.g., Red Cross donation or NBRO meteorological reports):
- Whitelist approved hostnames (`*.redcross.lk`, `*.meteo.gov.lk`, `*.dmc.gov.lk`).
- Disallow arbitrary or unvalidated deep links.

---

## 7. Logging, Telemetry & Privacy Standards

### 7.1 PII Logging Sanitization (`AppLogger`)
- **Strict Prohibition:** Production logs must never output:
  - Personal Names, Phone Numbers, or Emergency Contacts (ICE).
  - Exact GPS coordinates (Latitude/Longitude).
  - Authorization tokens, Bearer headers, or cookies.
  - Raw exception stack traces containing user input.
- **Log Gating:** All verbose and debug logs are stripped in release builds:
  ```dart
  import 'package:flutter/foundation.dart';

  class AppLogger {
    static void debug(String message) {
      if (kDebugMode) {
        // print or developer.log only during active development
      }
    }
  }
  ```

### 7.2 Location Privacy & Consent
- **Principle of Least Privilege:** Request `ACCESS_COARSE_LOCATION` by default. Only request `ACCESS_FINE_LOCATION` when precision mapping or active SOS dispatch requires it.
- **Background Location:** Never request `ACCESS_BACKGROUND_LOCATION` unless an active, foreground-notified safety tracking session is explicitly activated by the user.
- **Data Minimization:** Location coordinates sent to the .NET backend for regional weather/alerts must be truncated or generalized to regional boundaries (e.g., district level) unless triggering an SOS dispatch.

---

## 8. Incident Response & Vulnerability Disclosure

Security researchers and developers who discover a vulnerability in Safe Travel Sri Lanka should report it responsibly.

- **Security Contact:** `security@safetravel.redcross.lk`
- **PGP Key:** Available upon request or published on official domain.
- **Disclosure Policy:** We request 30 days to remediate issues before public disclosure.
- **Prohibited Actions:** Denial of Service (DoS) attacks, social engineering, or testing against production emergency responder dispatch systems.
