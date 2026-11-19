# Developer Agent Security Guidelines & Quality Gates

<!--
MANDATORY FOR ALL CODING AGENTS AND DEVELOPERS.
Read this file before generating, modifying, or reviewing code.
Authority: User Global Rules > CLAUDE.md > AGENTS.md > This File > Coding Patterns.
-->

## 1. Golden Rules (Non-Negotiable)

1. **Zero Secrets in Source Code or Git**: Never hardcode API keys, tokens, endpoints, secrets, or passwords. All environment-specific variables must pass through `--dart-define` / `--dart-define-from-file` and be read via `AppEnvironment`.
2. **Never Log Sensitive Information**: Never output PII (names, phone numbers, ICE contacts), authentication tokens, bearer headers, or exact GPS coordinates into `AppLogger`, `print()`, or crash reporting breadcrumbs.
3. **No Raw SQL String Interpolation**: Every `sqflite` query must use parameterized queries (`whereArgs`). String concatenation in SQL statements is an instant rejection.
4. **Strict Cleartext HTTP Ban**: All endpoints must use HTTPS. Android `network_security_config.xml` and iOS `Info.plist` must explicitly forbid cleartext traffic.
5. **Sanitize Tel/Dialer Schemes**: All phone numbers for SOS or emergency hotlines must be validated with a strict regex (`^\+?[0-9]{3,12}$`) before passing to `url_launcher`. Never execute strings containing `*`, `#`, or URL parameters.
6. **Defense in Depth**: Even though the .NET backend enforces rate limiting, authentication, and validation, the Flutter app must validate inputs client-side, sanitize display outputs, and handle network anomalies gracefully without crashing or leaking state.

---

## 2. Storage Classification Matrix for Agents

When implementing features that persist data locally, agents MUST consult this table:

| Data Type | Storage Mechanism | Prohibited Storage | Reason |
|---|---|---|---|
| **Passwords / Auth Credentials** | `flutter_secure_storage` | `shared_preferences`, SQLite plaintext | Stored in Android KeyStore (AES-256 GCM) / iOS Keychain |
| **JWT Access / Refresh Tokens** | `flutter_secure_storage` | `shared_preferences`, SQLite plaintext | Stored in OS Keychain (iOS) / KeyStore (Android) |
| **Emergency Contacts (ICE)** | `flutter_secure_storage` | Plain SQLite, SharedPreferences | Personal PII must be protected at rest |
| **Medical Info / Blood Group** | `flutter_secure_storage` | Plain SQLite, SharedPreferences | Sensitive personal health data |
| **Database Encryption Master Key**| `flutter_secure_storage` | Plain SQLite, SharedPreferences | 256-bit CSPRNG key for SQLCipher |
| **User GPS Coordinates (Realtime)** | In-memory State ONLY | Any persistent disk storage | Do not build unconsented movement logs |
| **Bundled Offline Safety Guides** | SQLite (`sqflite`) / Assets | N/A | Public static data, read-only |
| **Personal Offline Trip/Journal** | SQLite with SQLCipher (AES-256) | Plain SQLite | Encrypted with KeyStore-backed key |
| **Cached Weather / Regional Alerts**| SQLite (`sqflite`) | `flutter_secure_storage` | Non-sensitive, high-volume cached data |
| **ThemeMode, LanguageCode, Flags** | `shared_preferences` | SQLite, SecureStorage | Low-sensitivity lightweight key-value pairs |

### Correct Implementation Examples

#### `flutter_secure_storage` Configuration
Always pass platform-specific security options:
```dart
import 'package:flutter_secure_storage/flutter_secure_storage.dart';

class SecureStorageService {
  static const FlutterSecureStorage storage = FlutterSecureStorage(
    aOptions: AndroidOptions(
      // Hardware-backed KeyStore + AES-256 GCM
      encryptedSharedPreferences: true,
      resetOnError: true,
    ),
    iOptions: IOSOptions(
      // Hardware-backed Secure Enclave Keychain
      accessibility: KeychainAccessibility.first_unlock_this_device,
    ),
  );
}
```

#### SQLCipher Database Encryption at Rest
For offline databases storing sensitive user profiles or personal trip logs:
```dart
// 1. Retrieve or generate a 256-bit CSPRNG key from KeyStore:
final dbKey = await SecureStorageService.getOrGenerateDbKey();

// 2. Open SQLite database using 256-bit AES encryption:
final db = await openDatabase(
  dbPath,
  password: dbKey, // SQLCipher AES-256 CBC encryption
);
```

#### Safe SQLite Parameterization
```dart
// ALWAYS DO THIS:
Future<List<Map<String, Object?>>> getAlertsBySeverity(String severity) async {
  return await db.query(
    'alerts',
    where: 'severity = ?',
    whereArgs: [severity],
  );
}

// NEVER DO THIS (VULNERABLE):
// return await db.rawQuery("SELECT * FROM alerts WHERE severity = '$severity'");
```

---

## 3. Network & API Client Security Standards

Because our hand-rolled `http` client communicates with a .NET Core API, developers must follow these patterns:

### 3.1 Standard Request Headers
Every outgoing HTTP request to our backend must pass:
```dart
Map<String, String> buildSecureHeaders({String? token}) {
  return {
    'Accept': 'application/json',
    'Content-Type': 'application/json; charset=utf-8',
    'X-Request-ID': const Uuid().v4(),
    'X-Client-Timestamp': DateTime.now().toUtc().toIso8601String(),
    'X-Client-Version': AppConstants.kAppVersion,
    if (token != null) 'Authorization': 'Bearer $token',
  };
}
```

### 3.2 Secure Token Handling & 401 Refresh Mutex
When implementing authenticated requests:
- Never log the `Authorization` header.
- Token refresh operations must use a mutex/lock to prevent duplicate simultaneous refresh requests ("thundering herd" problem).
- Clear tokens immediately from `flutter_secure_storage` upon logout or unrecoverable 401 response.

### 3.3 Strict Timeout & Resource Protection
Never make unbounded network calls. All requests must carry an explicit timeout:
```dart
final response = await client
    .get(uri, headers: headers)
    .timeout(const Duration(seconds: 15));
```

---

## 4. Input Validation & Defense Against Injection

### 4.1 SOS Emergency Dialer Validation
SOS functionality triggers phone calls. To prevent malicious intent execution or USSD string hijacking:
```dart
class PhoneNumberSanitizer {
  static final RegExp _safeNumberRegex = RegExp(r'^\+?[0-9]{3,12}$');

  static bool isSafeToDial(String rawNumber) {
    final clean = rawNumber.replaceAll(RegExp(r'\s+|-'), '');
    // Rejects #, *, commas, query parameters, semicolons
    return _safeNumberRegex.hasMatch(clean);
  }
}
```

### 4.2 DTO Parsing Safety
Never trust remote JSON payloads. Ensure null-safety and safe type-casting:
```dart
factory AlertDto.fromJson(Map<String, dynamic> json) {
  return AlertDto(
    id: json['id'] as String? ?? '',
    title: json['title'] as String? ?? AppConstants.kDefaultFallbackText,
    severity: json['severity'] as String? ?? 'low',
    latitude: (json['latitude'] as num?)?.toDouble() ?? 0.0,
    longitude: (json['longitude'] as num?)?.toDouble() ?? 0.0,
  );
}
```

---

## 5. Logging & Error Handling Rules for Agents

### 5.1 Forbidden Log Contents
Any agent review must fail if the following appear in log statements:
- Password, PIN, or auth tokens (JWTs)
- Emergency contact names, phone numbers, or relationships
- Precise GPS coordinates (Latitude/Longitude)
- Full API response bodies containing personal details

### 5.2 Correct Logging Practice
```dart
// CORRECT: Redacted or summary logging
AppLogger.info('Fetched ${alerts.length} hazard alerts successfully.');
AppLogger.error('Failed to dispatch SOS telemetry. Status: ${response.statusCode}');

// FORBIDDEN:
// AppLogger.debug('User GPS: $lat, $lng');
// AppLogger.debug('Authorization: Bearer $token');
// AppLogger.debug('ICE contact saved: ${contact.phone}');
```

---

## 6. Manifest & OS Configuration Checklist

Before shipping platform configuration changes:

### Android (`android/app/src/main/AndroidManifest.xml`)
- [ ] `android:allowBackup="false"` (Prevents ADB extraction of app data)
- [ ] `android:fullBackupContent="false"`
- [ ] `android:networkSecurityConfig="@xml/network_security_config"` configured
- [ ] Exported activities/receivers explicitly declare `android:exported="false"` unless needed by OS launcher.
- [ ] No dangerous unused permissions declared.

### iOS (`ios/Runner/Info.plist`)
- [ ] `NSAppTransportSecurity` has `NSAllowsArbitraryLoads = false`.
- [ ] Permission description strings (`NSLocationWhenInUseUsageDescription`) clearly explain why the Red Cross app needs location (e.g., "Safe Travel uses your location to display nearby hazard alerts and safe shelters during emergencies").

---

## 7. Pre-Commit Agent Verification Checklist

Run through this checklist before presenting any code or feature completion:

```markdown
### Security Checklist
- [ ] 1. No hardcoded keys, secrets, or dev URLs in code?
- [ ] 2. All SQLite queries parameterized using `whereArgs`?
- [ ] 3. Sensitive items (passwords, tokens, ICE contacts, DB keys) stored in `flutter_secure_storage` using Android KeyStore (AES-256 GCM) / iOS Keychain?
- [ ] 4. Any sensitive offline SQLite database encrypted using SQLCipher (AES-256)?
- [ ] 5. Non-sensitive settings only in `shared_preferences`?
- [ ] 6. SOS phone dialer validates numbers against `^\+?[0-9]{3,12}$`?
- [ ] 7. Zero PII, GPS coordinates, or Bearer tokens in `AppLogger`?
- [ ] 8. All network calls carry an explicit timeout?
- [ ] 9. DTO parsing uses safe casting with default fallbacks?
- [ ] 10. Android allowBackup is false and cleartext HTTP traffic is blocked?
- [ ] 11. Release build command includes `--obfuscate --split-debug-info`?
```
