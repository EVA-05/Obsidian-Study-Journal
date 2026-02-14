---
categories:
  - Dashboard
---
# Study Dashboard — Documentation

## Implement

1.  [Dataview plugin](https://github.com/blacksmithgu/obsidian-dataview) installed and enabled
2. add Templates folder to your vault
3. Create Folder `Logs`
4. Keep your study logs in Logs folder
5. go to settings, Templates, under Template folder location select Templates (important part here is that you would use study logs template for journal entries and in order to create a dashboard you would insert Template_v2 to the note)
6. create new note, call it ex. "Dashboard"
7. `Ctrl + p` in command palette select insert template, choose Template_v2
8. Done

## What Is This?

A study-tracking dashboard for Obsidian that visualizes your daily study sessions as color-coded heatmaps. It reads your log entries and renders four different calendar views — all from a single template file, with **10 switchable color themes** controlled by one frontmatter property.

No plugins beyond Dataview are required. No CSS snippets. No separate theme files. Just change a single word in your note's properties and the entire dashboard recolors itself instantly.

---

## How Color Switching Works (The Core Innovation)

Obsidian's built-in template system has a fundamental limitation: templates are just text insertion. They can't accept parameters. So if you wanted "this dashboard but in blue" and "this dashboard but in green," you'd traditionally need two completely separate template files with hundreds of lines of duplicated code. Every bug fix or feature change would need to be replicated across all variants.

This project solves that with a **runtime theme resolution** pattern:

### The Mechanism

1. **A `Colors` property lives in the dashboard note's frontmatter** — not in the template code, not in a settings file. It's right there in the YAML metadata of each dashboard note you create.

2. **The DataviewJS code reads its own note's frontmatter at runtime** using Dataview's `dv.current()` API. This is the key insight — the code doesn't use hardcoded colors. Instead, every time the note renders, it asks: "What theme does my own frontmatter say to use?"

3. **A theme registry inside the code maps theme names to color palettes.** It's a plain JavaScript object — `themes["Ocean"]` returns the Ocean color set, `themes["Monokai Dark"]` returns the Monokai set.

4. **Fallback behavior** ensures the dashboard always works. If the `Colors` property is blank, misspelled, or missing entirely, it defaults to Ocean.

### The Code Pattern

Every DataviewJS block in the template starts with this:

```javascript
// Read the Colors property from this note's frontmatter
const rawColors = dv.current().Colors;

// Handle both YAML formats: string ("Ocean") or list (["Ocean"])
const themeName = Array.isArray(rawColors) ? rawColors[0] : (rawColors || "Ocean");

// Look up the theme, fall back to Ocean if not found
const theme = themes[themeName] || themes["Ocean"];
```

The `Array.isArray` check exists because Obsidian's YAML parser treats `Colors: Ocean` (string) and `Colors:\n  - Ocean` (list) differently. The code handles both transparently.

### Why This Works Without an Extension

Dataview already provides everything needed:
- `dv.current()` — reads the current note's frontmatter properties
- `dv.pages('"Logs"')` — queries all notes in the Logs folder
- `this.container.createDiv()` — renders HTML directly into the note

The "theme switching" is really just a JavaScript object lookup that happens on every render. There's no state management, no configuration files, no plugin API — just pure DataviewJS reading a YAML property from its own note.

---

## Technical Details

### Architecture

The template contains **4 independent DataviewJS code blocks**, one for each visualization:

| Block | Section | What It Renders |
|-------|---------|-----------------|
| 1 | Heatmap | Monthly calendar grid for the current month |
| 2 | Year at a Glance | All 12 months in a 3x4 grid |
| 3 | GitHub Contribution Graph | Trailing 365-day grid (GitHub-style) |
| 4 | Monthly Strips | Compact horizontal bars for each month |

Each block runs independently — DataviewJS code blocks don't share variables. This means the theme registry is defined in each block separately. Blocks 1 and 2 use the **full theme format** (with subject-specific palettes), while Blocks 3 and 4 use a **compact format** (generic intensity only).

### Theme Object Structure

**Full format** (Blocks 1 & 2 — supports per-subject coloring):

```javascript
"Ocean": {
  palettes: [                          // Color sets for auto-discovered subjects
    ["#80d8ff", "#40c4ff", "#0091ea"], // Palette 1 (assigned to first subject)
    ["#a7ffeb", "#64ffda", "#00bfa5"], // Palette 2 (assigned to second subject)
    ["#b388ff", "#7c4dff", "#6200ea"], // Palette 3 (assigned to third subject)
  ],
  fallback: ["#81d4fa", "#29b6f6", "#0277bd"], // For subjects beyond palette count
  generic:  ["var(--background-secondary)",     // Intensity 0: no study
             "#81d4fa",                         // Intensity 1: light (<45 min)
             "#29b6f6",                         // Intensity 2: medium (45-74 min)
             "#0277bd"]                         // Intensity 3: heavy (75+ min)
}
```

**Compact format** (Blocks 3 & 4 — no subject breakdown):

```javascript
"Ocean": { generic: ["var(--background-secondary)", "#81d4fa", "#29b6f6", "#0277bd"] }
```

### Subject Auto-Discovery

Subjects are never hardcoded. The code scans all log entries and builds a unique set:

```javascript
const allSubjects = new Set();
data.forEach(d => d.subjects.forEach(s => allSubjects.add(s)));
const subjectList = [...allSubjects].sort();
```

Each subject is then assigned a palette from the theme using modulo indexing:

```javascript
subjectList.forEach((name, i) => {
  subjectColors[name] = theme.palettes[i % theme.palettes.length];
});
```

If you have 5 subjects but only 3 palettes, subjects 4 and 5 wrap around and reuse palettes 1 and 2. If a subject isn't in the registry at all, the `fallback` palette is used.

### Intensity Mapping

Study minutes are bucketed into 4 levels:

| Intensity | Minutes | Meaning |
|-----------|---------|---------|
| 0 | 0 | No study that day |
| 1 | 1–44 | Light session |
| 2 | 45–74 | Medium session |
| 3 | 75+ | Heavy session |

Each intensity level maps to a color in the palette (index 0 = lightest, index 2 = darkest).

### Data Flow

```
Logs/ folder
    │
    ├── 2026-02-10.md  (date, study hours, Subject, Rating)
    ├── 2026-02-11.md
    └── ...
    │
    ▼
dv.pages('"Logs"')  →  Filter entries with study hours + date
    │
    ▼
Auto-discover subjects  →  Assign palettes from active theme
    │
    ▼
Render HTML into note  →  Calendars, stats, legends
```

### Stats & Streaks

Each visualization section calculates:
- **Total hours** — sum of all study minutes, converted to hours
- **Current streak** — consecutive days with study entries, counting backwards from today (or yesterday if today has no entry yet)
- **Subject breakdown** — percentage and hours per subject with progress bars

---

## How to Use This in Your Obsidian Vault

### Prerequisites

- [Obsidian](https://obsidian.md/) (any recent version)
- [Dataview plugin](https://github.com/blacksmithgu/obsidian-dataview) installed and enabled
- In Dataview settings: **Enable JavaScript Queries** must be turned on

### Setup Steps

**1. Create the folder structure**

Your vault needs a `Logs/` folder (or change the folder name in the code). Place the template files in a `Template/` folder or wherever you keep templates.

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

You can use the `Log Template.md` to create new entries quickly.

**3. Create a dashboard**

Apply the `Template_v2.md` template to a new note (or copy-paste its contents). The dashboard will immediately start reading your log entries and rendering visualizations.

**4. Choose a color theme**

Edit the `Colors` property in the dashboard note's frontmatter:

```yaml
---
Colors: Midnight Cyber
---
```

Available themes: `Ocean`, `Emerald`, `Violet Aurora`, `Rose Gold`, `Neon Heat`, `Midnight Cyber`, `Sunset Blaze`, `Arctic Frost`, `Tropical Punch`, `Monokai Dark`

### Customizing for Your Use Case

- **Different subjects?** No changes needed — subjects auto-discover from your logs. Just use whatever `Subject` values you want in your log entries.
- **Different intensity thresholds?** Search for `if (m < 45)` and `if (m < 75)` in the code and adjust the minute thresholds.
- **Different folder name?** Replace `'"Logs"'` with your folder name in each `dv.pages()` call.
- **Different property names?** Replace `"study hours"`, `"Subject"`, `"Rating"` with your property names.

---

## Files

| File | Purpose |
|------|---------|
| `Template_v2.md` | Main dashboard template — contains all 4 visualization blocks and the full theme registry |
| `Log Template.md` | Frontmatter template for creating new daily study log entries |
| `Color variants.md` | Visual reference showing all 10 color themes with hex swatches |
| `Documentation.md` | This file |

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

See `Color variants.md` for full hex color swatches of each theme.

---

## Future Plans

### Obsidian Plugin / Extension

The current approach works well but has limitations inherent to DataviewJS:
- The theme registry is duplicated across 4 code blocks (they can't share variables)
- Adding a new theme means editing JavaScript in 4 places
- No UI for theme selection — you have to manually type the theme name

A dedicated **Obsidian plugin** could solve all of these:
- **Theme picker UI** — a dropdown or modal to preview and select themes instead of editing YAML by hand
- **Live preview** — see color swatches before committing to a theme
- **Single source of truth** — theme definitions stored once, not duplicated per block
- **Custom theme builder** — let users create their own palettes through a color picker interface
- **Settings panel** — configure intensity thresholds, folder paths, and property names without touching code

- Instead of keeping logs in folders, They should be added through `Journal` property

### Potential Features

- **Weekly / custom date range views** — zoom into a specific week or arbitrary date range
- **Goal tracking** — set daily/weekly study targets and visualize progress against them
- **Exportable reports** — generate shareable study summaries
- **Multi-metric support** — track more than just study minutes (e.g., pages read, problems solved)
- **Animated transitions** — smooth color fade when switching themes
- **Community theme gallery** — share and download themes from other users
- **Dark/light mode awareness** — auto-adjust palette brightness based on Obsidian's current theme
- **Mobile optimization** — responsive layouts that work better on phone screens

### Contributing

If you want to add a new theme to the current template:

1. Pick a name and design 3 subject palettes (each with 3 hex colors: light, medium, dark)
2. Choose a fallback palette (3 colors) and a generic intensity scale (4 values)
3. Add the theme to the `themes` object in **all 4** DataviewJS blocks in `Template_v2.md`
4. Update the theme list on line 7 of the template
5. Optionally add a visual swatch to `Color variants.md`
