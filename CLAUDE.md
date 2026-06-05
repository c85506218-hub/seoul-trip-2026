# CLAUDE.md — Seoul Trip 2026

## Project Overview

A self-contained static travel itinerary website for a 5-day Seoul trip (May 23–27, 2026) departing from Kaohsiung, Taiwan. The UI is in Traditional Chinese (zh-TW) with Korean station/place names inline.

**No build system, no dependencies, no package manager.**

Hosted on GitHub Pages; auto-deployed via GitHub Actions on every push to `main`.

### Files
| File | Purpose |
|------|---------|
| `index.html` | Main itinerary — all HTML, CSS, JS in one file |
| `pharmacy.html` | Korean pharmacy group-order system — Tailwind CSS via CDN, Lucide icons |
| `.github/workflows/deploy.yml` | GitHub Actions workflow: push to `main` → deploy to GitHub Pages |
| `.nojekyll` | Disables Jekyll processing on GitHub Pages |

---

## index.html — Trip Itinerary

### Layout
- **Left panel** (`.left`): day-tab navigation + timeline content, scrollable
- **Right panel** (`.right`): sticky map panel with Google Maps iframe and a custom metro diagram, `position:sticky;top:0`
- Responsive breakpoint at `900px`: stacks to single column, right panel becomes `380px` fixed height

### Key Structural Sections
| Section | HTML | Purpose |
|---------|------|---------|
| Header | `<header>` | Trip title, flight badge, hotel badges, link to `pharmacy.html` |
| Edit bar | `#editBar` | Shown only in edit mode, has "clear all" button |
| Day tabs | `.tabs > .tab[d="N"]` | Switches active day (1–5) |
| Day content | `#dayN` / `.day` | Each day's timeline, hidden unless `.on` |
| Day header | `.dh.dN` | Coloured gradient card at top of each day |
| Flight card | `.flight` | Outbound (Day 1) and return (Day 5) flight details |
| Tip box | `.tip` | Orange left-border callout for warnings/tips |
| Timeline | `.timeline > .ti` | Vertical timeline; `.hl` = red dot, `.star` = gold dot |
| Item card | `.ic` | Individual activity card with time, title, desc, tags |
| Shopping grid | `.grid2 > .mini-card` | 2-col convenience store buy list |
| Map panel | `.right > .map-body` | Google Maps iframe OR metro diagram |
| Notes | `#noteN` (injected by JS) | Per-day textarea synced to cloud |
| Edit FAB | `#editFab` | Fixed bottom-right button to toggle edit mode |

### Day Color Scheme
```
--d1: #0052A4  (blue)
--d2: #00A650  (green)
--d3: #EF7C1C  (orange)
--d4: #00A4E3  (cyan)
--d5: #996CAC  (purple)
```

### Tag Classes
| Class | Color | Meaning |
|-------|-------|---------|
| `.t-food` | yellow | Food / meals / coffee |
| `.t-tp` | blue | Transport |
| `.t-act` | green | Activity / sightseeing |
| `.t-hotel` | purple | Hotel / accommodation |
| `.t-tip` | red | Warning / skip note |
| `.t-shop` | pink | Shopping |
| `.t-beauty` | magenta | Beauty / medical aesthetics |

### JavaScript Features

#### Cloud Sync (JSONBlob)
- **Blob URL**: `https://jsonblob.com/api/jsonBlob/019e4048-aa27-7a2f-8fe0-f31ca1f13742`
- Stores two keys: `notes` (object, keys `"1"`–`"5"`) and `edits` (object, keys `"c0"`, `"c1"`, …)
- Write strategy: debounced 800ms, then `PUT` to JSONBlob. Falls back silently on network failure.
- Read strategy: load from `localStorage` cache first (instant), then fetch cloud and apply (may override).
- `cloudLoad()` / `cloudSave()` / `applyData()` — main sync functions
- `showSyncStatus(msg)` — transient toast injected at `bottom:72px;right:24px`

#### Edit Mode
- Toggle via `toggleEdit()` (FAB button)
- Sets `contentEditable=true` on all `.ic-title` and `.ic-desc` elements
- Elements identified by `data-eid="cN"` (assigned on `DOMContentLoaded`)
- `body.edit-mode` class enables dashed-orange outlines on editable elements
- On exit: calls `cloudSave()` and shows "已儲存！" for 1800ms

#### Day / Map Switching
- `switchDay(n)` — removes/adds `.on` class on tabs and `.day` containers
- `switchMap('map'|'metro')` — toggles iframe (`.hide`) vs `.metro-view` (`.on`)

#### Notes
- Injected dynamically in `DOMContentLoaded` loop (days 1–5) as `<textarea id="noteN">`
- `oninput` calls `saveNote(d)` which delegates to `cloudSave()`

### CSS Conventions
- All CSS in the `<style>` block, kept compact (no blank lines, semicolons on same line)
- Shorthand class names: `.ic`, `.ti`, `.dh`, `.fr`, `.fc`, `.fa`, `.fi`, `.mt`, `.st`, etc.
- No external frameworks or icon fonts — emojis used for icons
- `box-sizing:border-box` globally
- CSS custom properties in `:root` for day accent colors only

### Metro Diagram (right panel)
- Fully hand-coded HTML tables/divs — no SVG, no image
- Line color classes: `.line-arex`, `.line-1`, `.line-2`, `.line-4`, `.line-5`, `.line-9`, `.line-gimpo`
- Station dot classes: `.st-dot.arex`, `.st-dot.l1`, `.st-dot.l2`, `.st-dot.l4`, `.st-dot.l5`, `.st-dot.l9`
- Route badge classes: `.rs.arex`, `.rs.l1`, etc. (used in both metro view and route boxes)
- Station name tri-lingual table: Chinese / English / 한국어 + line badges

### Content Conventions

#### Adding a New Timeline Item
```html
<div class="ti [hl|star]">
  <div class="ic">
    <div class="ic-time">HH:MM [label]</div>
    <div class="ic-title">Title text</div>
    <div class="ic-desc">Description text</div>
    <div class="ic-tags">
      <span class="tag t-food">🍜 label</span>
    </div>
  </div>
</div>
```
- `.ti` = default grey dot
- `.ti.hl` = red/highlight dot (major events)
- `.ti.star` = gold/star dot (recommended / starred)

#### Adding a Tip Box
```html
<div class="tip">⚠️ <strong>Label：</strong>Body text</div>
```

#### Adding a Mini-Card Grid
```html
<div class="grid2">
  <div class="mini-card">
    <div class="store">Store name</div>
    <div class="item-name">Product</div>
    <div class="item-note">Note</div>
  </div>
</div>
```

---

## pharmacy.html — Korean Pharmacy Group Order System

A standalone two-tab web app for collecting and aggregating coworker cosmetic/medicine orders from a Korea trip. Uses **Tailwind CSS (CDN)** and **Lucide icons (CDN)** — unlike `index.html`, this file has external dependencies.

### Architecture
- **Tab 1 "我要跟團"** (`#view-order`): product selection grid + orderer name input + sticky submit bar
- **Tab 2 "主揪管理後台"** (`#view-dashboard`): live stats, order table, bar chart, CSV export

### Cloud Sync
- **Blob URL**: `https://jsonblob.com/api/jsonBlob/019e6c33-baac-7c7f-8dc4-84a7f74965c8` (separate blob from `index.html`)
- Stores key `pharmacyOrders` — array of order objects
- Each order: `{ id, time, name, items: [{id, name, price, qty}], total }`
- All submitters write to the same shared blob; no auth

### Key Functions
| Function | Purpose |
|----------|---------|
| `init()` | Render product cards, load cloud orders |
| `renderProductCards()` | Build product grid from `products[]` array |
| `adjustQty(id, val)` / `setQty(id, val)` | Stepper ± buttons, highlights card when qty > 0 |
| `submitOrder()` | Validate → cloud read-modify-write → clear form → switch to dashboard |
| `updateDashboardUI()` | Populate table, stats cards, bar chart |
| `renderChart(counts, total)` | Horizontal bar chart sorted by qty descending |
| `exportToCSV()` | BOM-prefixed CSV download (Excel-compatible) |
| `refreshDashboard()` | Re-fetch cloud and re-render dashboard |
| `deleteSingleOrder(id)` | Read-modify-write removing one order by id |
| `clearAllOrders()` | Wipe `pharmacyOrders` array in cloud |
| `switchTab(name)` | Toggle between order/dashboard views |
| `copyShareLink()` | Copy `window.location.href` to clipboard |
| `getThemeColors(color)` | Returns Tailwind class strings for a named color key |
| `escapeHTML(s)` | XSS-safe HTML entity encoding for user-supplied strings |

### Product Catalogue (`products[]`)
18 Korean pharmacy/beauty products. Each entry:
```js
{ id, name, tagline, price, color, category, note? }
```
To add a product, append to the `products` array in the `<script>` block. The `id` must be unique (used as DOM id prefix `card-`, `input-`, `subtotal-`).

### Tailwind Color Keys
Each product has a `color` key (e.g. `"teal"`, `"rose"`) mapped by `getThemeColors()` to three Tailwind class strings: `text`, `badge`, `bgBar`. Supported keys: `zinc emerald blue amber rose pink indigo teal cyan violet purple sky slate lime`.

---

## Deployment

### GitHub Actions (`deploy.yml`)
- Trigger: push to `main`, or manual `workflow_dispatch`
- Steps: checkout → `configure-pages` (enables Pages if not already) → upload artifact (entire repo root) → deploy
- The `concurrency: group: pages` setting cancels in-progress deployments when a new push arrives.

### Local Development
- Open `index.html` directly in a browser — no server required.
- `pharmacy.html` loads Tailwind and Lucide from CDN, so it needs network access.
- Cloud sync requires network; offline use falls back to `localStorage` (index.html only).

---

## Trip Reference Data
| Item | Value |
|------|-------|
| Outbound flight | TW672, KHH→ICN, 16:10→20:00 |
| Return flight | TW671, ICN→KHH, 13:05→15:10 |
| Hotel 1 (Day 1–2) | SR Hotel Seoul Magok, near 발산역 (Line 5, exit 3) |
| Hotel 2 (Day 3–5) | G3 Hotel Chungmuro, near 충무로역 (Lines 3/4) |
| Medical aesthetics | DORÉ Clinic, Day 2 10:00, Gangnam Stn exit 9, ARA Tower 4F |
| Spa | Dragon Hill Spa, 龍山站, Day 4 |

## Key Constants (update these if rotated)
| Constant | File | Location |
|----------|------|----------|
| `BLOB` (itinerary notes/edits) | `index.html` | `const BLOB = '...'` in `<script>` |
| `BLOB_URL` (pharmacy orders) | `pharmacy.html` | `const BLOB_URL = '...'` in `<script>` |
