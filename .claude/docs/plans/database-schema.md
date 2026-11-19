# Database Schema & Migration Plan: Safe Travel Sri Lanka

**Version:** 1.0.0  
**Target:** SQLite 3 via `sqflite` (read-heavy offline cache) + `sqflite_sqlcipher` (AES-256 for user personal data)  
**Database File:** `safe_travel.db`  
**Compliance:** Strict parameterization (`whereArgs`), zero string interpolation, tri-lingual text columns (`_en`, `_si`, `_ta`).

---

## 1. Architecture & Storage Tiers

The local persistence strategy isolates data into two cryptographic domains:
1. **Unencrypted SQLite (`sqflite`):** High-performance, read-heavy, bundled offline static assets (safety topics, first-aid manuals, emergency contacts, cached hazard alerts).
2. **Encrypted SQLite (`sqflite_sqlcipher` with AES-256):** User private itineraries, notes, and custom emergency pack checklist state. (Key generated via `Random.secure()` and sealed inside `flutter_secure_storage`).

---

## 2. Complete Database DDL (Schema)

```sql
-- ============================================================================
-- 1. METADATA & VERSIONING
-- ============================================================================

CREATE TABLE IF NOT EXISTS db_migrations (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    version INTEGER NOT NULL UNIQUE,
    applied_at TEXT NOT NULL DEFAULT (datetime('now', 'utc')),
    description TEXT NOT NULL
);

CREATE TABLE IF NOT EXISTS content_sync_metadata (
    entity_type TEXT PRIMARY KEY, -- e.g., 'safety_topics', 'first_aid', 'emergency_numbers'
    last_synced_at TEXT NOT NULL,
    etag TEXT,
    record_count INTEGER NOT NULL DEFAULT 0,
    sync_status TEXT NOT NULL DEFAULT 'synced' CHECK(sync_status IN ('synced', 'pending', 'error'))
);

-- ============================================================================
-- 2. OFFLINE STATIC CORE CONTENT (IFRC Aligned, Bundled in Assets)
-- ============================================================================

-- Safety Topic Categories (8 major categories)
CREATE TABLE IF NOT EXISTS safety_topics (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    category_key TEXT NOT NULL UNIQUE CHECK(category_key IN (
        'floods_landslides', 'beach_safety', 'health', 'first_aid',
        'roads_transport', 'wildlife', 'personal_security', 'go_bag_emergency'
    )),
    title_en TEXT NOT NULL,
    title_si TEXT NOT NULL,
    title_ta TEXT NOT NULL,
    description_en TEXT NOT NULL,
    description_si TEXT NOT NULL,
    description_ta TEXT NOT NULL,
    icon_asset_path TEXT NOT NULL,
    sort_order INTEGER NOT NULL DEFAULT 0,
    is_active INTEGER NOT NULL DEFAULT 1 CHECK(is_active IN (0, 1)),
    created_at TEXT NOT NULL DEFAULT (datetime('now', 'utc')),
    updated_at TEXT NOT NULL DEFAULT (datetime('now', 'utc'))
);

-- Deep-dive sections per topic
CREATE TABLE IF NOT EXISTS safety_topic_details (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    topic_id INTEGER NOT NULL,
    section_title_en TEXT NOT NULL,
    section_title_si TEXT NOT NULL,
    section_title_ta TEXT NOT NULL,
    content_en TEXT NOT NULL,
    content_si TEXT NOT NULL,
    content_ta TEXT NOT NULL,
    media_type TEXT NOT NULL DEFAULT 'text' CHECK(media_type IN ('text', 'image', 'video_link')),
    media_url TEXT,
    sort_order INTEGER NOT NULL DEFAULT 0,
    FOREIGN KEY (topic_id) REFERENCES safety_topics(id) ON DELETE CASCADE
);

-- First-Aid Procedures & Protocols
CREATE TABLE IF NOT EXISTS first_aid_guides (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    guide_code TEXT NOT NULL UNIQUE, -- e.g., 'FA_HEATSTROKE', 'FA_SNAKEBITE'
    title_en TEXT NOT NULL,
    title_si TEXT NOT NULL,
    title_ta TEXT NOT NULL,
    severity TEXT NOT NULL CHECK(severity IN ('critical', 'moderate', 'minor')),
    search_keywords TEXT NOT NULL, -- comma-separated search tokens in all languages
    sort_order INTEGER NOT NULL DEFAULT 0,
    is_active INTEGER NOT NULL DEFAULT 1 CHECK(is_active IN (0, 1)),
    created_at TEXT NOT NULL DEFAULT (datetime('now', 'utc')),
    updated_at TEXT NOT NULL DEFAULT (datetime('now', 'utc'))
);

-- Step-by-step procedural emergency instructions
CREATE TABLE IF NOT EXISTS first_aid_steps (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    guide_id INTEGER NOT NULL,
    step_number INTEGER NOT NULL,
    instruction_en TEXT NOT NULL,
    instruction_si TEXT NOT NULL,
    instruction_ta TEXT NOT NULL,
    image_asset_path TEXT,
    warning_text_en TEXT,
    warning_text_si TEXT,
    warning_text_ta TEXT,
    FOREIGN KEY (guide_id) REFERENCES first_aid_guides(id) ON DELETE CASCADE,
    UNIQUE(guide_id, step_number)
);

-- National Emergency Hotlines (pre-warmed into RAM cache)
CREATE TABLE IF NOT EXISTS emergency_numbers (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    service_code TEXT NOT NULL UNIQUE, -- e.g., 'POLICE_119', 'AMBULANCE_1990'
    service_name_en TEXT NOT NULL,
    service_name_si TEXT NOT NULL,
    service_name_ta TEXT NOT NULL,
    phone_number TEXT NOT NULL CHECK(phone_number GLOB '+[0-9]*' OR phone_number GLOB '[0-9]*'),
    category TEXT NOT NULL CHECK(category IN ('police', 'medical', 'fire', 'disaster', 'redcross', 'tourist', 'embassy')),
    is_primary INTEGER NOT NULL DEFAULT 0 CHECK(is_primary IN (0, 1)),
    icon_asset_path TEXT NOT NULL,
    sort_order INTEGER NOT NULL DEFAULT 0
);

-- Pre-defined Go-Bag emergency survival pack items
CREATE TABLE IF NOT EXISTS go_bag_items (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    item_code TEXT NOT NULL UNIQUE, -- e.g., 'WATER_PURIFIER', 'PASSPORT_COPIES'
    item_name_en TEXT NOT NULL,
    item_name_si TEXT NOT NULL,
    item_name_ta TEXT NOT NULL,
    category TEXT NOT NULL CHECK(category IN ('documents', 'medical', 'clothing', 'food_water', 'tools', 'communication')),
    description_en TEXT,
    description_si TEXT,
    description_ta TEXT,
    is_essential INTEGER NOT NULL DEFAULT 1 CHECK(is_essential IN (0, 1)),
    sort_order INTEGER NOT NULL DEFAULT 0
);

-- ============================================================================
-- 3. CACHED ONLINE CONTENT (SWR Caching)
-- ============================================================================

-- Real-time hazard warnings (NBRO, DMC, MetDept, Red Cross)
CREATE TABLE IF NOT EXISTS hazard_alerts (
    id TEXT PRIMARY KEY, -- Remote GUID / ID from .NET Core backend
    title_en TEXT NOT NULL,
    title_si TEXT,
    title_ta TEXT,
    description_en TEXT NOT NULL,
    description_si TEXT,
    description_ta TEXT,
    severity TEXT NOT NULL CHECK(severity IN ('critical', 'high', 'moderate', 'low', 'info')),
    hazard_type TEXT NOT NULL CHECK(hazard_type IN ('flood', 'landslide', 'storm', 'tsunami', 'heatwave', 'wildfire', 'other')),
    affected_area_name TEXT NOT NULL,
    latitude REAL NOT NULL,
    longitude REAL NOT NULL,
    radius_km REAL NOT NULL DEFAULT 10.0,
    source TEXT NOT NULL, -- e.g. 'NBRO', 'DMC', 'SLRC'
    issued_at TEXT NOT NULL,
    expires_at TEXT NOT NULL,
    is_active INTEGER NOT NULL DEFAULT 1 CHECK(is_active IN (0, 1)),
    fetched_at TEXT NOT NULL DEFAULT (datetime('now', 'utc')),
    raw_json TEXT NOT NULL
);

-- Weather Cache per District / Region
CREATE TABLE IF NOT EXISTS weather_cache (
    district_key TEXT PRIMARY KEY, -- e.g. 'colombo', 'kandy', 'galle'
    district_name_en TEXT NOT NULL,
    temperature_celsius REAL NOT NULL,
    feels_like_celsius REAL NOT NULL,
    humidity_percent INTEGER NOT NULL,
    wind_speed_kmh REAL NOT NULL,
    condition_code TEXT NOT NULL,
    condition_text_en TEXT NOT NULL,
    condition_text_si TEXT,
    condition_text_ta TEXT,
    icon_url TEXT,
    fetched_at TEXT NOT NULL DEFAULT (datetime('now', 'utc')),
    expires_at TEXT NOT NULL
);

-- Directory Listings (Hotels, Clinics, Tourist Police)
CREATE TABLE IF NOT EXISTS explore_listings (
    id TEXT PRIMARY KEY, -- Remote backend GUID
    name TEXT NOT NULL,
    category TEXT NOT NULL CHECK(category IN ('accommodation', 'food', 'medical', 'transport', 'embassy', 'attraction', 'shop', 'service')),
    description_en TEXT,
    description_si TEXT,
    description_ta TEXT,
    address TEXT NOT NULL,
    latitude REAL NOT NULL,
    longitude REAL NOT NULL,
    phone TEXT,
    website TEXT,
    rating REAL DEFAULT 0.0,
    review_count INTEGER DEFAULT 0,
    price_level INTEGER DEFAULT 1,
    photo_urls_json TEXT, -- JSON array of image URLs
    is_indoor INTEGER NOT NULL DEFAULT 0 CHECK(is_indoor IN (0, 1)),
    is_emergency_facility INTEGER NOT NULL DEFAULT 0 CHECK(is_emergency_facility IN (0, 1)),
    is_verified INTEGER NOT NULL DEFAULT 1 CHECK(is_verified IN (0, 1)),
    fetched_at TEXT NOT NULL DEFAULT (datetime('now', 'utc'))
);

-- Interactive Map Overlays & Safety Zones
CREATE TABLE IF NOT EXISTS map_points (
    id TEXT PRIMARY KEY,
    type TEXT NOT NULL CHECK(type IN ('hazard_zone', 'safe_zone', 'redcross_point', 'shelter', 'hospital', 'police_station')),
    label_en TEXT NOT NULL,
    label_si TEXT,
    label_ta TEXT,
    latitude REAL NOT NULL,
    longitude REAL NOT NULL,
    radius_km REAL, -- Null for point markers, numeric for circular safety zones
    severity TEXT CHECK(severity IN ('critical', 'high', 'moderate', 'low', 'info')),
    metadata_json TEXT, -- GeoJSON polygon or extra facility metadata
    is_active INTEGER NOT NULL DEFAULT 1 CHECK(is_active IN (0, 1)),
    fetched_at TEXT NOT NULL DEFAULT (datetime('now', 'utc'))
);

-- ============================================================================
-- 4. USER PERSONAL DATA (Encrypted via SQLCipher in Production)
-- ============================================================================

-- User custom trips / itineraries
CREATE TABLE IF NOT EXISTS user_trips (
    id TEXT PRIMARY KEY, -- Client UUIDv4
    trip_name TEXT NOT NULL,
    destination TEXT NOT NULL,
    start_date TEXT NOT NULL,
    end_date TEXT NOT NULL,
    notes TEXT,
    created_at TEXT NOT NULL DEFAULT (datetime('now', 'utc')),
    updated_at TEXT NOT NULL DEFAULT (datetime('now', 'utc'))
);

-- User checklist states for Go-Bag survival items
CREATE TABLE IF NOT EXISTS user_go_bag_checks (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    trip_id TEXT, -- Nullable: null represents default master checklist, or links to user_trips
    item_id INTEGER NOT NULL,
    is_checked INTEGER NOT NULL DEFAULT 0 CHECK(is_checked IN (0, 1)),
    checked_at TEXT,
    FOREIGN KEY (trip_id) REFERENCES user_trips(id) ON DELETE CASCADE,
    FOREIGN KEY (item_id) REFERENCES go_bag_items(id) ON DELETE CASCADE,
    UNIQUE(trip_id, item_id)
);
```

---

## 3. High-Performance Index Definitions

```sql
-- Category & Order Lookups
CREATE INDEX IF NOT EXISTS idx_safety_topics_sort ON safety_topics(sort_order, is_active);
CREATE INDEX IF NOT EXISTS idx_safety_topic_details_topic ON safety_topic_details(topic_id, sort_order);

-- First Aid Triaging & Search
CREATE INDEX IF NOT EXISTS idx_first_aid_guides_severity ON first_aid_guides(severity, sort_order);
CREATE INDEX IF NOT EXISTS idx_first_aid_steps_guide ON first_aid_steps(guide_id, step_number);

-- Emergency Speed Lookups
CREATE INDEX IF NOT EXISTS idx_emergency_numbers_cat ON emergency_numbers(category, is_primary, sort_order);

-- Real-Time Alert Queries
CREATE INDEX IF NOT EXISTS idx_hazard_alerts_active ON hazard_alerts(is_active, severity, issued_at DESC);
CREATE INDEX IF NOT EXISTS idx_hazard_alerts_coords ON hazard_alerts(latitude, longitude);

-- Explore Distance Sorting
CREATE INDEX IF NOT EXISTS idx_explore_listings_coords ON explore_listings(latitude, longitude);
CREATE INDEX IF NOT EXISTS idx_explore_listings_category ON explore_listings(category, is_indoor);

-- Map Rendering Bounds
CREATE INDEX IF NOT EXISTS idx_map_points_coords ON map_points(latitude, longitude, type);

-- User Itinerary Lookups
CREATE INDEX IF NOT EXISTS idx_user_trips_dates ON user_trips(start_date, end_date);
CREATE INDEX IF NOT EXISTS idx_user_go_bag_checks_trip ON user_go_bag_checks(trip_id, is_checked);
```

---

## 4. Migration Strategy & Version Management

Database initialization and upgrades are managed in `lib/config/database/db_config.dart`.

```dart
class DatabaseConfig {
  static const String databaseName = 'safe_travel.db';
  static const int databaseVersion = 1;

  static Future<void> onCreate(Database db, int version) async {
    final batch = db.batch();
    // Execute all DDL statements in batch
    for (final sql in SchemaDefinitions.v1Tables) {
      batch.execute(sql);
    }
    for (final indexSql in SchemaDefinitions.v1Indexes) {
      batch.execute(indexSql);
    }
    // Record initial migration
    batch.insert('db_migrations', {
      'version': 1,
      'applied_at': DateTime.now().toUtc().toIso8601String(),
      'description': 'Initial baseline schema v1.0.0',
    });
    await batch.commit(noResult: true);
  }

  static Future<void> onUpgrade(Database db, int oldVersion, int newVersion) async {
    for (int v = oldVersion + 1; v <= newVersion; v++) {
      final migration = migrations[v];
      if (migration != null) {
        await migration.run(db);
        await db.insert('db_migrations', {
          'version': v,
          'applied_at': DateTime.now().toUtc().toIso8601String(),
          'description': migration.description,
        });
      }
    }
  }
}
```

---

## 5. Seed Content Pipeline & JSON Asset Schema

On first boot (`isFirstLaunch == true`), `OfflineContentService` unpacks JSON assets into SQLite:

1. `assets/data/safety_topics.json`
2. `assets/data/first_aid_guides.json`
3. `assets/data/emergency_numbers.json`
4. `assets/data/go_bag_items.json`

### Example Asset Shape (`assets/data/emergency_numbers.json`):
```json
[
  {
    "service_code": "POLICE_119",
    "service_name_en": "Police Emergency Hotline",
    "service_name_si": "පොලිස් හදිසි ඇමතුම්",
    "service_name_ta": "காவல்துறை அவசர உதவி",
    "phone_number": "119",
    "category": "police",
    "is_primary": 1,
    "icon_asset_path": "assets/images/icons/police.png",
    "sort_order": 1
  },
  {
    "service_code": "AMBULANCE_1990",
    "service_name_en": "1990 Suwa Seriya Ambulance",
    "service_name_si": "1990 සුවසැරිය ගිලන්රථ සේවය",
    "service_name_ta": "1990 சுவ செரிய ஆம்புலன்ஸ்",
    "phone_number": "1990",
    "category": "medical",
    "is_primary": 1,
    "icon_asset_path": "assets/images/icons/ambulance.png",
    "sort_order": 2
  }
]
```

---

## 6. Parameterized Query Patterns (Security Guaranteed)

```dart
// 1. Fetching localized topics safely
Future<List<SafetyTopicModel>> getTopics(String langCode) async {
  final db = await database;
  final results = await db.query(
    'safety_topics',
    where: 'is_active = ?',
    whereArgs: [1],
    orderBy: 'sort_order ASC',
  );
  return results.map((row) => SafetyTopicModel.fromMap(row, langCode)).toList();
}

// 2. Fetching active alerts by severity threshold
Future<List<HazardAlertModel>> getActiveAlerts(List<String> severities) async {
  final db = await database;
  final placeholders = List.filled(severities.length, '?').join(',');
  final results = await db.query(
    'hazard_alerts',
    where: 'is_active = 1 AND severity IN ($placeholders)',
    whereArgs: severities,
    orderBy: 'issued_at DESC',
  );
  return results.map(HazardAlertModel.fromMap).toList();
}
```
