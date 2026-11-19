# Project Brief

## What This Project Is

Safe Travel Sri Lanka — a Flutter/Dart mobile application developed in affiliation with
the **Sri Lanka Red Cross Society**. Designed primarily for international tourists and
local travelers, the app provides:

- Real-time hazard warnings (landslides, floods, storms, rough seas)
- Offline-accessible first-aid instructions and safety guides
- Local emergency dispatch integration (SOS one-tap dialer)
- Trusted neighborhood/business recommendations
- Trip planning with emergency go-bag checklist
- Context-aware UI that adapts during active weather warnings

## Nature / Stage

**New project, production-bound.** Starting from scratch with Flutter/Dart. The web
prototype exists at https://safetravelbyredcross.netlify.app/ as the design reference.

## Primary Goals

1. Deliver a reliable offline-first safety companion app for travelers in Sri Lanka
2. Integrate with national emergency services and Red Cross Society resources
3. Provide context-aware safety recommendations based on location and weather conditions
4. Ship on Android first, iOS second

## Target Users

- International tourists visiting Sri Lanka
- Local Sri Lankan travelers
- Humanitarian workers and Red Cross field staff

## Key Constraints

- **Offline-first**: core safety content must work without any connectivity
- **Stress-ready UI**: app must be usable in emergencies (high contrast, large touch
  targets, minimal steps to critical actions)
- **Bilingual**: English primary, Sinhala support planned
- **Red Cross branding**: follows IFRC visual identity guidelines
- **Android-first**: primary target platform, iOS support follows

## App Architecture & Screen Flow

```
[Welcome] ──> [Onboarding] ──> [Location & Alerts] ──> [All Set] ──> [Home Dashboard]
                                                                        │
     ┌──────────────────┬─────────────────┬─────────────────┬───────────┴──────────┐
     ▼                  ▼                 ▼                 ▼                      ▼
[Safety Map]      [Explore Tabs]    [Topic Details]    [SOS & Numbers]     [Trip & Go-Bag]
     │            (Normal/Warning)        │             (Emergency)            │
     ▼                  ▼                 ▼                                    ▼
[Place Detail]                          [First Aid / Guide]               [Live Alerts]
                                                                               │
                                                                               ▼
                                                                      [Profile & Settings]
```

## Core Feature Set

### 1. Welcome & Onboarding Flow
- Brand introduction with Red Cross emblem
- Carousel: offline guides, live alerts, emergency response
- Location & notification permissions setup
- Offline cache download confirmation

### 2. Home Dashboard
- User greeting with location, temperature, weather, offline status
- Active hazard alert banner (e.g., LANDSLIDE WATCH)
- Quick-access grid: Map, First Aid, Explore, I'm Safe
- Trip plan & go-bag summary widget
- Explore near you section (local businesses with distance)
- Safety topics offline card grid (8 categories)
- Bottom nav: Home, Map, SOS (red center), Explore, Alerts

### 3. Safety Map & Warnings
- Interactive map with hazard zones, safe zones, Red Cross points
- Filter toggles: weather, health, transit
- Place detail view with safety advisories
- Live alerts chronological feed

### 4. Explore & Warning States
- Directory of traveler-friendly services by distance/safety
- Dynamic warning-state adaptation (indoor alternatives, clinics)

### 5. Safety Topics & First Aid
- Deep-dive content pages per risk category
- First aid index (alphabetical/severity)
- Step-by-step guides aligned with IFRC standards

### 6. Emergency SOS & Quick Numbers
- One-tap dialer for: Police, Ambulance, Tourist Police, DMC, Red Cross

### 7. Trip Planning & Go-Bag
- Itinerary tracker
- Emergency supply checklist (go-bag packer)

### 8. Profile & Settings
- Emergency contacts (ICE), language, offline cache status, permissions
