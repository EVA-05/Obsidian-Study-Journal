# Obsidian Study Journal

A study-tracking dashboard for [Obsidian](https://obsidian.md/) that visualizes your daily study sessions as color-coded heatmaps. It reads your log entries and renders four different calendar views — all from a single template file, with **10 switchable color themes** controlled by one frontmatter property.

No plugins beyond Dataview are required. No CSS snippets. No separate theme files. Just change a single word in your note's properties and the entire dashboard recolors itself instantly.

---

## How Color Switching Works

Obsidian's built-in template system can't accept parameters — so if you wanted "this dashboard but in blue" and "this dashboard but in green," you'd need two completely separate template files with duplicated code.

This project solves that with a **runtime theme resolution** pattern:

1. A `Colors` property lives in the dashboard note's frontmatter.
2. The DataviewJS code reads its own note's frontmatter at runtime using `dv.current()`.
3. A theme registry inside the code maps theme names to color palettes.
4. Fallback behavior ensures the dashboard always works — defaults to Ocean if the property is blank or misspelled.

```javascript
// Read the Colors property from this note's frontmatter
const rawColors = dv.current().Colors;

// Handle both YAML formats: string ("Ocean") or list (["Ocean"])
const themeName = Array.isArray(rawColors) ? rawColors[0] : (rawColors || "Ocean");

// Look up the theme, fall back to Ocean if not found
const theme = themes[themeName] || themes["Ocean"];
```

---

## Visualizations

The template contains **4 independent DataviewJS code blocks**:

| Block | Section | What It Renders |
|-------|---------|-----------------|
| 1 | Heatmap | Monthly calendar grid for the current month |
| 2 | Year at a Glance | All 12 months in a 3x4 grid |
| 3 | GitHub Contribution Graph | Trailing 365-day grid (GitHub-style) |
| 4 | Monthly Strips | Compact horizontal bars for each month |

Each block calculates:
- **Total hours** — sum of all study minutes, converted to hours
- **Current streak** — consecutive days with study entries
- **Subject breakdown** — percentage and hours per subject with progress bars

---

## Intensity Mapping

Study minutes are bucketed into 4 levels:

| Intensity | Minutes | Meaning |
|-----------|---------|---------|
| 0 | 0 | No study that day |
| 1 | 1-44 | Light session |
| 2 | 45-74 | Medium session |
| 3 | 75+ | Heavy session |

---

## Getting Started

### Prerequisites

- [Obsidian](https://obsidian.md/) (any recent version)
- [Dataview plugin](https://github.com/blacksmithgu/obsidian-dataview) installed and enabled
- In Dataview settings: **Enable JavaScript Queries** must be turned on

### Setup

**1. Create the folder structure**

```
Your Vault/
├── Logs/
│   ├── 2026-02-10.md
│   ├── 2026-02-11.md
│   └── ...
├── Template/
│   ├── Template_v2.md      ← The dashboard template
│   ├── Log Template.md     ← Template for new log entries
│   └── Color variants.md   ← Color reference (optional)
└── My Dashboard.md         ← Created from the template
```

**2. Create log entries**

Each log entry needs these frontmatter properties:

```yaml
---
categories:
  - Journal
Subject:
  - Math        # Your subject(s) — can be anything
study hours: 60 # Minutes studied (despite the property name)
date: 2026-02-14
Rating: 5       # Optional: 1-7 scale
---
```

Use the included `Log Template.md` to create new entries quickly.

**3. Create a dashboard**

Apply `Template_v2.md` to a new note (or copy-paste its contents). The dashboard will immediately start rendering visualizations from your log entries.

**4. Choose a color theme**

Edit the `Colors` property in the dashboard note's frontmatter:

```yaml
---
Colors: Midnight Cyber
---
```

---

## Available Color Themes

| Theme | Vibe |
|-------|------|
| **Ocean** | Blues, teals, purples — calm and focused |
| **Emerald** | Greens, ambers, teals — natural and grounding |
| **Violet Aurora** | Pinks, purples, blues — vibrant and creative |
| **Rose Gold** | Peach, pink, amber — warm and elegant |
| **Neon Heat** | Hot pink, orange, yellow — bold and energetic |
| **Midnight Cyber** | Cyan, hot pink, neon green — dark and electric |
| **Sunset Blaze** | Reds, oranges, yellows — fiery gradient |
| **Arctic Frost** | Ice blue, slate, indigo — minimal and clean |
| **Tropical Punch** | Orange, teal, coral — bright and playful |
| **Monokai Dark** | Pink, green, yellow — code-editor aesthetic |

See `Template/Color variants.md` for full hex color swatches of each theme.

---

## Customization

- **Different subjects?** No changes needed — subjects auto-discover from your logs.
- **Different intensity thresholds?** Search for `if (m < 45)` and `if (m < 75)` in the code and adjust.
- **Different folder name?** Replace `'"Logs"'` with your folder name in each `dv.pages()` call.
- **Different property names?** Replace `"study hours"`, `"Subject"`, `"Rating"` with your property names.

---

## Files

| File | Purpose |
|------|---------|
| `Template/Template_v2.md` | Main dashboard template with all 4 visualization blocks and theme registry |
| `Template/Log Template.md` | Frontmatter template for creating new daily study log entries |
| `Template/Color variants.md` | Visual reference showing all 10 color themes with hex swatches |

---

## Technical Details

### Subject Auto-Discovery

Subjects are never hardcoded. The code scans all log entries and builds a unique set, then assigns palettes from the active theme using modulo indexing. If you have more subjects than palettes, they wrap around.

### Theme Object Structure

**Full format** (Blocks 1 & 2 — per-subject coloring):

```javascript
"Ocean": {
  palettes: [
    ["#80d8ff", "#40c4ff", "#0091ea"],
    ["#a7ffeb", "#64ffda", "#00bfa5"],
    ["#b388ff", "#7c4dff", "#6200ea"],
  ],
  fallback: ["#81d4fa", "#29b6f6", "#0277bd"],
  generic:  ["var(--background-secondary)", "#81d4fa", "#29b6f6", "#0277bd"]
}
```

**Compact format** (Blocks 3 & 4 — intensity only):

```javascript
"Ocean": { generic: ["var(--background-secondary)", "#81d4fa", "#29b6f6", "#0277bd"] }
```

---

## Future Plans

- **Obsidian plugin** with theme picker UI, live preview, and settings panel
- Weekly / custom date range views
- Goal tracking with daily/weekly targets
- Multi-metric support (pages read, problems solved, etc.)
- Dark/light mode awareness
- Mobile-optimized layouts

### Contributing

To add a new theme:

1. Design 3 subject palettes (each with 3 hex colors: light, medium, dark)
2. Choose a fallback palette and a generic intensity scale (4 values)
3. Add the theme to the `themes` object in **all 4** DataviewJS blocks in `Template_v2.md`
4. Update the theme list on line 7 of the template
5. Optionally add a visual swatch to `Color variants.md`
