# Bush Study Guide — Project Context

> **This file is automatically loaded by Antigravity at the start of every session.**
> **You MUST keep this file updated whenever the app's architecture, data model, features, or tech stack changes.**

## Project Overview

**Bush Study Guide** is a mobile-first Progressive Web App (PWA) for studying African wildlife and trees — built for safari field guide training. It is a single-user personal study tool hosted on **GitHub Pages**.

- **Repository**: `bush-study`
- **Hosting**: GitHub Pages (static, free)
- **Live URL**: GitHub Pages site for this repo

## Current Architecture

The app is built as a single-page PWA using vanilla web technologies. It previously hardcoded data but now uses a decoupled JSON backend with in-app editing capabilities that commit directly to the repository via the GitHub REST API.

| File | Purpose |
|---|---|
| `index.html` | Entire app UI: markup, styles, app logic, rendering, and GitHub sync engine |
| `data.json` | Contains all study data arrays and objects (Mammals, Tracks, Trees, etc.) |
| `manifest.json` | PWA manifest (standalone mode, theme color, icon) |
| `sw.js` | Service worker — cache-first for fonts, network-first for app/data, offline support |

### Tech Stack

- **Frontend**: Vanilla HTML/CSS/JS (no framework, no build step)
- **Fonts**: Google Fonts (Inter)
- **Styling**: CSS custom properties, glassmorphism, dark theme
- **External APIs**: 
  - iNaturalist REST API for species photos (on-demand, online only)
  - GitHub REST API for saving user data edits directly to the repository
- **Persistence**: 
  - `localStorage` for SRS flashcard progress (`bush_srs`)
  - `localStorage` for offline data edits queue/cache (`bush_data_local`)
  - `localStorage` for GitHub PAT (`bush_github_pat`)
- **Offline**: Robust two-tier offline data system. The Service Worker caches `data.json`, and `localStorage` acts as a higher-priority cache for uncommitted edits made while offline, auto-syncing when connectivity returns.

## Data Model

All data is stored in `data.json` and loaded asynchronously on startup. 

### Mammals (`mammals` array)

40 species. Each entry:

```javascript
{
  "name": "Lion",                              // Common name (unique key)
  "cat": "predator",                           // Category: predator|primate|herbivore|small|other
  "_w": 190,                                   // Sortable weight in kg (male)
  "_g": 110,                                   // Sortable gestation in days
  "_l": 14,                                    // Sortable max lifespan in years
  "lifespan": "10–14 yrs",                     // Display text
  "weight": "♂ 190 kg / ♀ 130 kg",            // Display text with sexual dimorphism
  "gestation": "110 days (~3.5 months)",        // Display text
  "weaning": "6–7 months",                     // Display text
  "territorial": "Yes – males defend...",       // Territorial behavior description
  "social": "Prides of related females...",     // Social structure description
  "food": "Large ungulates..."                  // Diet and digestive type
}
```

### Tracks (`tracks` array)

14 entries with inline SVG footprint art:

```javascript
{
  "name": "African Elephant",
  "group": "megaherbivore",                    // megaherbivore|herbivore|predator|small
  "size": "Front: ~45 cm / Hind: ~40 cm",
  "toes": "Round, cushion-padded...",
  "features": ["Largest track...", "Toenails visible..."],  // Diagnostic ID points
  "svg": "<svg viewBox=\"0 0 80 80\">...</svg>"               // Inline vector art
}
```

### Trees (`trees` array)

17 species:

```javascript
{ "common": "Leadwood", "sci": "Combretum imberbe" }
```

Supplemented by `treeNotes` object with detailed field notes per common name (bark, toxicity, leaves, ecology).

### iNaturalist IDs (`inatIds` object)

57 pre-resolved taxon IDs mapping common names to iNaturalist taxonomy IDs for photo fetching.

## Features

### Content Editing & Sync
- **In-App Editing**: Fully functional edit forms for Mammals and Trees. Add new species or edit existing ones.
- **GitHub Sync**: When online, edits are automatically committed to the `data.json` file on GitHub using a Personal Access Token (PAT).
- **Offline Resilience**: If edits are made while offline, they are saved to `localStorage` and automatically queued to sync the next time the app comes online.

### Reference & Browse
- **Mammals section**: Category filter pills, search bar, sort by name/weight/gestation/lifespan
- **Tracks section**: Group filters, inline SVG track art, diagnostic feature lists
- **Trees section**: Common names with tap-to-reveal scientific names, toggle-all, detailed field notes

### Study Tools
- **Inline Study Mode (🧠)**: Blurs mammal stats on cards — tap to reveal and self-test
- **Species Comparison (⚖️)**: Side-by-side comparison of two mammals with difference highlighting and optional iNaturalist photo comparison

### Quiz Engine (7 modes)
1. **Multiple Choice** — lifespan, weight, gestation, weaning, diet, social, territorial
2. **Flashcards (SRS)** — Leitner spaced repetition with 3-button rating (Again/Hard/Easy)
3. **Fill in the Blank** — type numerical values with error tolerance
4. **Match Pairs** — connect 5 mammals to stats
5. **Rank & Order** — sort 5 animals by a stat (lightest→heaviest, etc.)
6. **Rapid-fire True/False** — with explanations
7. **Specialized quizzes** — territoriality, track ID, tree scientific/common names

### Photo Gallery
- Fetches up to 8 photos per species from iNaturalist API
- Caches in memory (`galleryCache`)
- Shows attribution and licensing
- Gracefully disabled when offline

## Known Limitations

- **Tracks not editable**: Track entries contain complex inline SVG strings which are difficult to edit safely via a mobile form, so they remain read-only for now.
- **No deletion**: Entries can only be added or edited, not deleted (to prevent accidental data loss in the field).
- **No personal notes/annotations**: Cannot attach arbitrary observations to species outside the formal data structure.
- **SRS progress is device-locked**: `localStorage` for quiz progress doesn't sync across devices.
- **No image uploads**: Cannot add personal photos.
- **Monolithic UI Logic**: All UI rendering and sync logic remains housed within the ~3,180 line `index.html`.

## Development Notes

- The user is the sole developer and sole user
- The app targets mobile/phone use (in the bush)
- Near-zero cost is a hard requirement (hence the GitHub API backend approach)
