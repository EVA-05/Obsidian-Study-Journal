---
categories:
  - Dashboard
Colors:
  - Ocean
---
Colors: `Ocean` , `Emerald`, `Violet Aurora`, `Rose Gold`, `Neon Heat`, `Midnight Cyber`, `Sunset Blaze`, `Arctic Frost`, `Tropical Punch`, `Monokai Dark`
## Heatmap

```dataviewjs
const pages = dv.pages('"Logs"')
  .where(p => p["study hours"] && p.date);

const data = pages.map(p => ({
  date: p.date.toString().slice(0, 10),
  minutes: p["study hours"],
  subjects: p.Subject ? [].concat(p.Subject) : [],
  subject: p.Subject ? [].concat(p.Subject).join(", ") : "",
  rating: p.Rating || 0
})).array().sort((a, b) => a.date.localeCompare(b.date));

// ── Auto-discover subjects from log data ──
const allSubjects = new Set();
data.forEach(d => d.subjects.forEach(s => allSubjects.add(s)));
const subjectList = [...allSubjects].sort();

// ── Theme registry ──
const rawColors = dv.current().Colors;
const themeName = Array.isArray(rawColors) ? rawColors[0] : (rawColors || "Ocean");
const themes = {
  "Ocean": {
    palettes: [
      ["#80d8ff","#40c4ff","#0091ea"],
      ["#a7ffeb","#64ffda","#00bfa5"],
      ["#b388ff","#7c4dff","#6200ea"],
    ],
    fallback: ["#81d4fa","#29b6f6","#0277bd"],
    generic:  ["var(--background-secondary)","#81d4fa","#29b6f6","#0277bd"]
  },
  "Emerald": {
    palettes: [
      ["#c5e1a5","#8bc34a","#558b2f"],
      ["#ffe082","#ffc107","#f57f17"],
      ["#80cbc4","#26a69a","#00695c"],
    ],
    fallback: ["#a5d6a7","#66bb6a","#2e7d32"],
    generic:  ["var(--background-secondary)","#a5d6a7","#66bb6a","#2e7d32"]
  },
  "Violet Aurora": {
    palettes: [
      ["#f48fb1","#e91e63","#ad1457"],
      ["#ce93d8","#ab47bc","#7b1fa2"],
      ["#90caf9","#42a5f5","#1565c0"],
    ],
    fallback: ["#b39ddb","#7e57c2","#4527a0"],
    generic:  ["var(--background-secondary)","#b39ddb","#7e57c2","#4527a0"]
  },
  "Rose Gold": {
    palettes: [
      ["#ffccbc","#ff8a65","#e64a19"],
      ["#f8bbd0","#f06292","#c2185b"],
      ["#ffe0b2","#ffb74d","#e65100"],
    ],
    fallback: ["#f0c4a8","#d4956a","#b06b3f"],
    generic:  ["var(--background-secondary)","#f0c4a8","#d4956a","#b06b3f"]
  },
  "Neon Heat": {
    palettes: [
      ["#ff80ab","#ff4081","#f50057"],
      ["#ffab40","#ff6d00","#dd2c00"],
      ["#fff176","#ffd600","#ffab00"],
    ],
    fallback: ["#ff8a65","#ff5722","#e64a19"],
    generic:  ["var(--background-secondary)","#ff8a65","#ff5722","#e64a19"]
  },
  "Midnight Cyber": {
    palettes: [
      ["#80deea","#00bcd4","#006064"],
      ["#f8bbd0","#ec407a","#880e4f"],
      ["#b2ff59","#76ff03","#64dd17"],
    ],
    fallback: ["#84ffff","#18ffff","#00b8d4"],
    generic:  ["var(--background-secondary)","#84ffff","#18ffff","#00b8d4"]
  },
  "Sunset Blaze": {
    palettes: [
      ["#ffcdd2","#ef5350","#b71c1c"],
      ["#ffe0b2","#ff9800","#e65100"],
      ["#fff9c4","#ffee58","#f9a825"],
    ],
    fallback: ["#ffab91","#ff7043","#bf360c"],
    generic:  ["var(--background-secondary)","#ffab91","#ff7043","#bf360c"]
  },
  "Arctic Frost": {
    palettes: [
      ["#e1f5fe","#4fc3f7","#0277bd"],
      ["#e0f7fa","#4dd0e1","#00838f"],
      ["#e8eaf6","#7986cb","#283593"],
    ],
    fallback: ["#eceff1","#90a4ae","#455a64"],
    generic:  ["var(--background-secondary)","#eceff1","#90a4ae","#455a64"]
  },
  "Tropical Punch": {
    palettes: [
      ["#ffcc80","#ffa726","#ef6c00"],
      ["#80cbc4","#26a69a","#00695c"],
      ["#ef9a9a","#ef5350","#c62828"],
    ],
    fallback: ["#b2ebf2","#26c6da","#00838f"],
    generic:  ["var(--background-secondary)","#b2ebf2","#26c6da","#00838f"]
  },
  "Monokai Dark": {
    palettes: [
      ["#f9a1bc","#f92672","#c4174f"],
      ["#c8e67e","#a6e22e","#7cb518"],
      ["#ffe5a0","#e6db74","#b8a93e"],
    ],
    fallback: ["#aedcf7","#66d9ef","#3aa5ba"],
    generic:  ["var(--background-secondary)","#aedcf7","#66d9ef","#3aa5ba"]
  },
};
const theme = themes[themeName] || themes["Ocean"];

const subjectColors = {};
subjectList.forEach((name, i) => {
  subjectColors[name] = theme.palettes[i % theme.palettes.length];
});
const fallbackColors = theme.fallback;

function getSubjectColor(subjects, intensity) {
  if (intensity <= 0) return "var(--background-secondary)";
  const primary = subjects.length > 0 ? subjects[0] : null;
  const palette = (primary && subjectColors[primary]) ? subjectColors[primary] : fallbackColors;
  return palette[intensity - 1];
}

function getSubjectAccent(name) {
  const palette = subjectColors[name] || fallbackColors;
  return palette[2];
}

// ── Stats ──
const totalMinutes = data.reduce((sum, d) => sum + d.minutes, 0);
const totalHours = (totalMinutes / 60).toFixed(1);

const dateSet = new Set(data.map(d => d.date));
let streak = 0;
let checkDate = new Date();
if (!dateSet.has(checkDate.toISOString().slice(0, 10))) {
  checkDate.setDate(checkDate.getDate() - 1);
}
while (dateSet.has(checkDate.toISOString().slice(0, 10))) {
  streak++;
  checkDate.setDate(checkDate.getDate() - 1);
}

const statsEl = this.container.createDiv();
statsEl.style.cssText = "display:flex; gap:12px; flex-wrap:wrap; margin-bottom:20px;";

const cardStyle = `
  flex:1; min-width:100px; padding:14px 16px;
  border-radius:10px; text-align:center;
  border:1px solid var(--background-modifier-border);
  background:var(--background-secondary);
`;

const fireEmoji = streak >= 7 ? "🔥🔥🔥" : streak >= 3 ? "🔥🔥" : streak >= 1 ? "🔥" : "";
const cards = [
  { value: "📚 " + totalHours + "h", label: "Total" },
  { value: fireEmoji + " " + streak + (streak === 1 ? " day" : " days"), label: "Current Streak" },
];

cards.forEach(c => {
  const card = statsEl.createDiv();
  card.style.cssText = cardStyle;
  card.innerHTML = `
    <div style="font-size:1.6em; font-weight:700; line-height:1.2;
                color:var(--text-accent);">${c.value}</div>
    <div style="font-size:0.8em; margin-top:4px;
                color:var(--text-muted);">${c.label}</div>
  `;
});

// ── Subject breakdown ──
const subjects = {};
data.forEach(d => {
  const subs = d.subjects.length > 0 ? d.subjects : ["Other"];
  subs.forEach(s => {
    if (!subjects[s]) subjects[s] = { minutes: 0, days: 0 };
    subjects[s].minutes += d.minutes;
    subjects[s].days++;
  });
});
const subjectKeys = Object.keys(subjects).sort((a, b) => subjects[b].minutes - subjects[a].minutes);

if (subjectKeys.length > 0) {
  const subEl = this.container.createDiv();
  subEl.style.cssText = "display:flex; gap:10px; flex-wrap:wrap; margin-bottom:20px;";

  subjectKeys.forEach(s => {
    const pct = totalMinutes > 0 ? Math.round((subjects[s].minutes / totalMinutes) * 100) : 0;
    const accent = getSubjectAccent(s);
    const tag = subEl.createDiv();
    tag.style.cssText = `
      padding:8px 14px; border-radius:8px; min-width:80px;
      background:var(--background-secondary);
      border-left:3px solid ${accent};
      border-top:1px solid var(--background-modifier-border);
      border-right:1px solid var(--background-modifier-border);
      border-bottom:1px solid var(--background-modifier-border);
    `;
    tag.innerHTML = `
      <div style="font-weight:600; font-size:0.85em; color:var(--text-normal); margin-bottom:4px;">${s}</div>
      <div style="font-size:1.2em; font-weight:700; color:${accent}; margin:2px 0;">${pct}%</div>
      <div style="margin-top:6px; height:4px; border-radius:2px; background:var(--background-modifier-border);">
        <div style="height:100%; width:${pct}%; border-radius:2px; background:${accent};"></div>
      </div>
    `;
  });
}

// ── Month calendar heatmap ──
const now = new Date();
const year = now.getFullYear();
const month = now.getMonth();
const monthName = now.toLocaleString("default", { month: "long" });
const daysInMonth = new Date(year, month + 1, 0).getDate();
const firstDayIdx = (new Date(year, month, 1).getDay() + 6) % 7;

const lookup = {};
data.forEach(d => { lookup[d.date] = d; });

const todayStr = now.toISOString().slice(0, 10);
const dayNames = ["Mon", "Tue", "Wed", "Thu", "Fri", "Sat", "Sun"];

const grid = this.container.createDiv();
grid.innerHTML = `
<div style="margin-bottom:6px; font-weight:700; font-size:1.1em; color:var(--text-normal);">
  ${monthName} ${year}
</div>
<div style="display:grid; grid-template-columns:repeat(7, 1fr); gap:4px; max-width:500px;">
  ${dayNames.map(d =>
    `<div style="text-align:center; font-size:0.7em; font-weight:600;
                 color:var(--text-muted); padding-bottom:2px;">${d}</div>`
  ).join("")}
  ${"<div></div>".repeat(firstDayIdx)}
  ${Array.from({ length: daysInMonth }, (_, i) => {
    const day = i + 1;
    const dateStr = `${year}-${String(month + 1).padStart(2, "0")}-${String(day).padStart(2, "0")}`;
    const entry = lookup[dateStr];
    const mins = entry ? entry.minutes : 0;
    const subj = entry ? entry.subject : "";
    const subs = entry ? entry.subjects : [];
    const rating = entry ? entry.rating : 0;

    let intensity = 0;
    if (mins > 0 && mins < 45) intensity = 1;
    else if (mins >= 45 && mins < 75) intensity = 2;
    else if (mins >= 75) intensity = 3;

    const bg = getSubjectColor(subs, intensity);
    const isToday = dateStr === todayStr;
    const border = isToday
      ? "2px solid var(--text-accent)"
      : "1px solid var(--background-modifier-border)";
    const textColor = intensity >= 3 ? "#fff" : "var(--text-normal)";

    const tooltip = mins > 0
      ? `${mins} min` + (subj ? ` · ${subj}` : "") + (rating > 0 ? ` · ${rating}/7` : "")
      : "No study";

    return `<div title="${tooltip}" style="
      aspect-ratio:1; border-radius:6px; background:${bg}; border:${border};
      display:flex; align-items:center; justify-content:center;
      cursor:default; min-height:44px;
    ">
      <span style="font-size:0.75em; font-weight:600; color:${textColor};">${day}</span>
    </div>`;
  }).join("")}
</div>
`;

// ── Legend ──
const legendEl = this.container.createDiv();
legendEl.style.cssText = "display:flex; flex-wrap:wrap; align-items:center; gap:14px; margin-top:12px; font-size:0.8em; color:var(--text-muted);";

const intensityRow = legendEl.createDiv();
intensityRow.style.cssText = "display:flex; align-items:center; gap:4px;";
intensityRow.createSpan().textContent = "Less";
fallbackColors.forEach(c => {
  const s = intensityRow.createSpan();
  s.style.cssText = "width:12px; height:12px; background:" + c + "; border-radius:2px; display:inline-block;";
});
intensityRow.createSpan().textContent = "More";

Object.keys(subjectColors).forEach(name => {
  const row = legendEl.createDiv();
  row.style.cssText = "display:flex; align-items:center; gap:4px;";
  subjectColors[name].forEach(c => {
    const s = row.createSpan();
    s.style.cssText = "width:12px; height:12px; background:" + c + "; border-radius:2px; display:inline-block;";
  });
  row.createSpan().textContent = name;
});
```

---

## Year at a Glance

```dataviewjs
const pages = dv.pages('"Logs"')
  .where(p => p["study hours"] && p.date);
const lookup = {};
pages.forEach(p => {
  const d = p.date.toString().slice(0, 10);
  lookup[d] = {
    minutes: p["study hours"],
    subjects: p.Subject ? [].concat(p.Subject) : [],
    subject: p.Subject ? [].concat(p.Subject).join(", ") : ""
  };
});

// ── Auto-discover subjects ──
const allSubjects = new Set();
Object.values(lookup).forEach(e => e.subjects.forEach(s => allSubjects.add(s)));
const subjectList = [...allSubjects].sort();

// ── Theme registry ──
const rawColors = dv.current().Colors;
const themeName = Array.isArray(rawColors) ? rawColors[0] : (rawColors || "Ocean");
const themes = {
  "Ocean": {
    palettes: [
      ["#80d8ff","#40c4ff","#0091ea"],
      ["#a7ffeb","#64ffda","#00bfa5"],
      ["#b388ff","#7c4dff","#6200ea"],
    ],
    fallback: ["#81d4fa","#29b6f6","#0277bd"],
  },
  "Emerald": {
    palettes: [
      ["#c5e1a5","#8bc34a","#558b2f"],
      ["#ffe082","#ffc107","#f57f17"],
      ["#80cbc4","#26a69a","#00695c"],
    ],
    fallback: ["#a5d6a7","#66bb6a","#2e7d32"],
  },
  "Violet Aurora": {
    palettes: [
      ["#f48fb1","#e91e63","#ad1457"],
      ["#ce93d8","#ab47bc","#7b1fa2"],
      ["#90caf9","#42a5f5","#1565c0"],
    ],
    fallback: ["#b39ddb","#7e57c2","#4527a0"],
  },
  "Rose Gold": {
    palettes: [
      ["#ffccbc","#ff8a65","#e64a19"],
      ["#f8bbd0","#f06292","#c2185b"],
      ["#ffe0b2","#ffb74d","#e65100"],
    ],
    fallback: ["#f0c4a8","#d4956a","#b06b3f"],
  },
  "Neon Heat": {
    palettes: [
      ["#ff80ab","#ff4081","#f50057"],
      ["#ffab40","#ff6d00","#dd2c00"],
      ["#fff176","#ffd600","#ffab00"],
    ],
    fallback: ["#ff8a65","#ff5722","#e64a19"],
  },
  "Midnight Cyber": {
    palettes: [
      ["#80deea","#00bcd4","#006064"],
      ["#f8bbd0","#ec407a","#880e4f"],
      ["#b2ff59","#76ff03","#64dd17"],
    ],
    fallback: ["#84ffff","#18ffff","#00b8d4"],
  },
  "Sunset Blaze": {
    palettes: [
      ["#ffcdd2","#ef5350","#b71c1c"],
      ["#ffe0b2","#ff9800","#e65100"],
      ["#fff9c4","#ffee58","#f9a825"],
    ],
    fallback: ["#ffab91","#ff7043","#bf360c"],
  },
  "Arctic Frost": {
    palettes: [
      ["#e1f5fe","#4fc3f7","#0277bd"],
      ["#e0f7fa","#4dd0e1","#00838f"],
      ["#e8eaf6","#7986cb","#283593"],
    ],
    fallback: ["#eceff1","#90a4ae","#455a64"],
  },
  "Tropical Punch": {
    palettes: [
      ["#ffcc80","#ffa726","#ef6c00"],
      ["#80cbc4","#26a69a","#00695c"],
      ["#ef9a9a","#ef5350","#c62828"],
    ],
    fallback: ["#b2ebf2","#26c6da","#00838f"],
  },
  "Monokai Dark": {
    palettes: [
      ["#f9a1bc","#f92672","#c4174f"],
      ["#c8e67e","#a6e22e","#7cb518"],
      ["#ffe5a0","#e6db74","#b8a93e"],
    ],
    fallback: ["#aedcf7","#66d9ef","#3aa5ba"],
  },
};
const theme = themes[themeName] || themes["Ocean"];

const subjectColors = {};
subjectList.forEach((name, i) => {
  subjectColors[name] = theme.palettes[i % theme.palettes.length];
});
const fallbackColors = theme.fallback;

function getSubjectColor(subjects, intensity) {
  if (intensity <= 0) return "var(--background-secondary)";
  const primary = subjects.length > 0 ? subjects[0] : null;
  const palette = (primary && subjectColors[primary]) ? subjectColors[primary] : fallbackColors;
  return palette[intensity - 1];
}

function getIntensity(m) {
  if (m <= 0) return 0;
  if (m < 45) return 1; if (m < 75) return 2; return 3;
}

const year = new Date().getFullYear();
const todayStr = new Date().toISOString().slice(0, 10);

let html = '<div style="display:grid; grid-template-columns:repeat(3, 1fr); gap:20px 24px;">';

for (let m = 0; m < 12; m++) {
  const mName = new Date(year, m).toLocaleString("default", { month: "long" });
  const dim = new Date(year, m + 1, 0).getDate();
  const firstDay = (new Date(year, m, 1).getDay() + 6) % 7;

  let monthMins = 0, monthDays = 0;
  for (let d = 1; d <= dim; d++) {
    const ds = year + "-" + String(m + 1).padStart(2, "0") + "-" + String(d).padStart(2, "0");
    if (lookup[ds]) { monthMins += lookup[ds].minutes; monthDays++; }
  }
  const subtitle = monthDays > 0 ? monthDays + "d · " + (monthMins / 60).toFixed(1) + "h" : "";

  html += '<div>';
  html += '<div style="display:flex; justify-content:space-between; align-items:baseline; margin-bottom:4px;">';
  html += '<span style="font-weight:700; font-size:0.85em; color:var(--text-normal);">' + mName + '</span>';
  html += '<span style="font-size:0.65em; color:var(--text-muted);">' + subtitle + '</span>';
  html += '</div>';
  html += '<div style="display:grid; grid-template-columns:repeat(7, 1fr); gap:3px;">';

  var dayHeaders = ["M","T","W","T","F","S","S"];
  for (let h = 0; h < 7; h++) {
    html += '<div style="text-align:center; font-size:0.55em; font-weight:600; color:var(--text-muted); padding-bottom:1px;">' + dayHeaders[h] + '</div>';
  }

  for (let p = 0; p < firstDay; p++) html += "<div></div>";

  for (let d = 1; d <= dim; d++) {
    const ds = year + "-" + String(m + 1).padStart(2, "0") + "-" + String(d).padStart(2, "0");
    const entry = lookup[ds];
    const mins = entry ? entry.minutes : 0;
    const subs = entry ? entry.subjects : [];
    const intensity = getIntensity(mins);
    const bg = getSubjectColor(subs, intensity);
    const isToday = ds === todayStr;
    const border = isToday ? "2px solid var(--text-accent)" : "1px solid var(--background-modifier-border)";
    const tc = intensity >= 3 ? "#fff" : "var(--text-muted)";
    const tooltip = mins > 0 ? ds + ": " + mins + " min" + (entry.subject ? " · " + entry.subject : "") : ds;

    html += '<div title="' + tooltip + '" style="aspect-ratio:1; border-radius:4px; background:' + bg + '; border:' + border + '; display:flex; align-items:center; justify-content:center; font-size:0.55em; color:' + tc + ';">' + d + '</div>';
  }

  html += '</div></div>';
}

html += '</div>';

const el = this.container.createDiv();
el.innerHTML = html;

// Legend
const legendEl = this.container.createDiv();
legendEl.style.cssText = "display:flex; flex-wrap:wrap; align-items:center; gap:10px; margin-top:10px; font-size:0.75em; color:var(--text-muted);";
Object.keys(subjectColors).forEach(function(name) {
  const row = legendEl.createDiv();
  row.style.cssText = "display:flex; align-items:center; gap:3px;";
  subjectColors[name].forEach(function(c) {
    const s = row.createSpan();
    s.style.cssText = "width:10px; height:10px; background:" + c + "; border-radius:2px; display:inline-block;";
  });
  row.createSpan().textContent = " " + name;
});
```

---

## GitHub Contribution Graph

```dataviewjs
const pages = dv.pages('"Logs"')
  .where(p => p["study hours"] && p.date);
const lookup = {};
pages.forEach(p => {
  lookup[p.date.toString().slice(0, 10)] = p["study hours"];
});

// ── Theme registry ──
const rawColors = dv.current().Colors;
const themeName = Array.isArray(rawColors) ? rawColors[0] : (rawColors || "Ocean");
const themes = {
  "Ocean":          { generic: ["var(--background-secondary)","#81d4fa","#29b6f6","#0277bd"] },
  "Emerald":        { generic: ["var(--background-secondary)","#a5d6a7","#66bb6a","#2e7d32"] },
  "Violet Aurora":  { generic: ["var(--background-secondary)","#b39ddb","#7e57c2","#4527a0"] },
  "Rose Gold":      { generic: ["var(--background-secondary)","#f0c4a8","#d4956a","#b06b3f"] },
  "Neon Heat":      { generic: ["var(--background-secondary)","#ff8a65","#ff5722","#e64a19"] },
  "Midnight Cyber": { generic: ["var(--background-secondary)","#84ffff","#18ffff","#00b8d4"] },
  "Sunset Blaze":   { generic: ["var(--background-secondary)","#ffab91","#ff7043","#bf360c"] },
  "Arctic Frost":   { generic: ["var(--background-secondary)","#eceff1","#90a4ae","#455a64"] },
  "Tropical Punch": { generic: ["var(--background-secondary)","#b2ebf2","#26c6da","#00838f"] },
  "Monokai Dark":   { generic: ["var(--background-secondary)","#aedcf7","#66d9ef","#3aa5ba"] },
};
const theme = themes[themeName] || themes["Ocean"];
const colors = theme.generic;

function getIntensity(m) {
  if (m <= 0) return 0;
  if (m < 45) return 1; if (m < 75) return 2; return 3;
}

const year = new Date().getFullYear();
const todayStr = new Date().toISOString().slice(0, 10);
const sz = 18;
const gap = 3;
const labelW = 32;
const dayLabels = ["", "Mon", "", "Wed", "", "Fri", ""];

function buildHalf(startMonth, endMonth) {
  const rangeStart = new Date(year, startMonth, 1);
  const startOffset = (rangeStart.getDay() + 6) % 7;
  const gridStart = new Date(year, startMonth, 1 - startOffset);
  const rangeEnd = new Date(year, endMonth + 1, 0);

  const weeks = [];
  const cursor = new Date(gridStart);
  while (cursor <= rangeEnd) {
    const week = [];
    for (let d = 0; d < 7; d++) {
      const m = cursor.getMonth();
      week.push({
        dateStr: cursor.toISOString().slice(0, 10),
        inRange: cursor.getFullYear() === year && m >= startMonth && m <= endMonth,
        month: m
      });
      cursor.setDate(cursor.getDate() + 1);
    }
    weeks.push(week);
  }

  const monthLabels = [];
  let lastM = -1;
  weeks.forEach(function(week, wi) {
    for (var i = 0; i < week.length; i++) {
      var cell = week[i];
      if (cell.inRange && cell.month !== lastM) {
        monthLabels.push({ col: wi, name: new Date(year, cell.month).toLocaleString("default", { month: "short" }) });
        lastM = cell.month;
        break;
      }
    }
  });

  let monthRow = "";
  monthLabels.forEach(function(ml) {
    const x = ml.col * (sz + gap);
    monthRow += '<span style="position:absolute; left:' + x + 'px; font-size:0.65em; color:var(--text-muted);">' + ml.name + '</span>';
  });

  let halfMins = 0;
  weeks.forEach(function(week) {
    week.forEach(function(cell) {
      if (cell.inRange && lookup[cell.dateStr]) halfMins += lookup[cell.dateStr];
    });
  });
  const halfHrs = (halfMins / 60).toFixed(1);

  let cells = "";
  for (var wi = 0; wi < weeks.length; wi++) {
    for (var di = 0; di < 7; di++) {
      var cell = weeks[wi][di];
      if (!cell.inRange) {
        cells += '<div style="width:' + sz + 'px; height:' + sz + 'px;"></div>';
        continue;
      }
      var mins = lookup[cell.dateStr] || 0;
      var intensity = getIntensity(mins);
      var bg = colors[intensity];
      var isToday = cell.dateStr === todayStr;
      var border = isToday ? "2px solid var(--text-accent)" : "1px solid var(--background-modifier-border)";
      var tooltip = mins > 0 ? cell.dateStr + ": " + mins + " min" : cell.dateStr;
      cells += '<div title="' + tooltip + '" style="width:' + sz + 'px; height:' + sz + 'px; border-radius:2px; background:' + bg + '; border:' + border + ';"></div>';
    }
  }

  return '<div style="margin-bottom:16px;">' +
    '<div style="font-weight:600; font-size:0.85em; color:var(--text-muted); margin-bottom:6px;">' +
      new Date(year, startMonth).toLocaleString("default", { month: "short" }) + ' – ' +
      new Date(year, endMonth).toLocaleString("default", { month: "short" }) + ' ' + year +
      '<span style="margin-left:10px; color:var(--text-accent); font-weight:700;">' + halfHrs + 'h</span>' +
    '</div>' +
    '<div style="position:relative; height:16px; margin-bottom:4px; margin-left:' + labelW + 'px;">' + monthRow + '</div>' +
    '<div style="display:flex; gap:0;">' +
      '<div style="display:flex; flex-direction:column; gap:' + gap + 'px; width:' + labelW + 'px;">' +
        dayLabels.map(function(l) { return '<div style="height:' + sz + 'px; line-height:' + sz + 'px; font-size:0.55em; color:var(--text-muted);">' + l + '</div>'; }).join("") +
      '</div>' +
      '<div style="display:grid; grid-auto-flow:column; grid-template-rows:repeat(7, ' + sz + 'px); gap:' + gap + 'px;">' +
        cells +
      '</div>' +
    '</div>' +
  '</div>';
}

const el = this.container.createDiv();
el.innerHTML = buildHalf(0, 5) + buildHalf(6, 11);

const legendEl = this.container.createDiv();
legendEl.style.cssText = "display:flex; align-items:center; gap:5px; font-size:0.75em; color:var(--text-muted);";
legendEl.createSpan().textContent = "Less";
colors.slice(1).forEach(function(c) {
  const s = legendEl.createSpan();
  s.style.cssText = "width:10px; height:10px; background:" + c + "; border-radius:2px; display:inline-block;";
});
legendEl.createSpan().textContent = "More";
```

---

## Monthly Strips

```dataviewjs
const pages = dv.pages('"Logs"')
  .where(p => p["study hours"] && p.date);
const lookup = {};
pages.forEach(p => {
  lookup[p.date.toString().slice(0, 10)] = p["study hours"];
});

// ── Theme registry ──
const rawColors = dv.current().Colors;
const themeName = Array.isArray(rawColors) ? rawColors[0] : (rawColors || "Ocean");
const themes = {
  "Ocean":          { generic: ["var(--background-secondary)","#81d4fa","#29b6f6","#0277bd"] },
  "Emerald":        { generic: ["var(--background-secondary)","#a5d6a7","#66bb6a","#2e7d32"] },
  "Violet Aurora":  { generic: ["var(--background-secondary)","#b39ddb","#7e57c2","#4527a0"] },
  "Rose Gold":      { generic: ["var(--background-secondary)","#f0c4a8","#d4956a","#b06b3f"] },
  "Neon Heat":      { generic: ["var(--background-secondary)","#ff8a65","#ff5722","#e64a19"] },
  "Midnight Cyber": { generic: ["var(--background-secondary)","#84ffff","#18ffff","#00b8d4"] },
  "Sunset Blaze":   { generic: ["var(--background-secondary)","#ffab91","#ff7043","#bf360c"] },
  "Arctic Frost":   { generic: ["var(--background-secondary)","#eceff1","#90a4ae","#455a64"] },
  "Tropical Punch": { generic: ["var(--background-secondary)","#b2ebf2","#26c6da","#00838f"] },
  "Monokai Dark":   { generic: ["var(--background-secondary)","#aedcf7","#66d9ef","#3aa5ba"] },
};
const theme = themes[themeName] || themes["Ocean"];
const colors = theme.generic;

function getIntensity(m) {
  if (m <= 0) return 0;
  if (m < 45) return 1; if (m < 75) return 2; return 3;
}

const year = new Date().getFullYear();
const todayStr = new Date().toISOString().slice(0, 10);
const sz = 16;
const gap = 3;

let headerCells = "";
for (let d = 1; d <= 31; d++) {
  headerCells += '<div style="width:' + sz + 'px; text-align:center; font-size:0.55em; color:var(--text-muted);">' + d + '</div>';
}

let rows = "";
for (let m = 0; m < 12; m++) {
  const mName = new Date(year, m).toLocaleString("default", { month: "short" });
  const dim = new Date(year, m + 1, 0).getDate();

  let monthMins = 0;
  for (let d = 1; d <= dim; d++) {
    const ds = year + "-" + String(m + 1).padStart(2, "0") + "-" + String(d).padStart(2, "0");
    if (lookup[ds]) monthMins += lookup[ds];
  }
  const totalLabel = monthMins > 0 ? (monthMins / 60).toFixed(1) + "h" : "";

  let cells = "";
  for (let d = 1; d <= 31; d++) {
    if (d > dim) {
      cells += '<div style="width:' + sz + 'px; height:' + sz + 'px;"></div>';
      continue;
    }
    const ds = year + "-" + String(m + 1).padStart(2, "0") + "-" + String(d).padStart(2, "0");
    const mins = lookup[ds] || 0;
    const intensity = getIntensity(mins);
    const bg = colors[intensity];
    const isToday = ds === todayStr;
    const border = isToday ? "2px solid var(--text-accent)" : "1px solid var(--background-modifier-border)";
    const tooltip = mins > 0 ? ds + ": " + mins + " min" : ds;

    cells += '<div title="' + tooltip + '" style="width:' + sz + 'px; height:' + sz + 'px; border-radius:3px; background:' + bg + '; border:' + border + ';"></div>';
  }

  rows += '<div style="display:flex; align-items:center; gap:' + gap + 'px;">' +
    '<div style="width:36px; font-size:0.75em; font-weight:600; color:var(--text-muted); text-align:right; padding-right:6px;">' + mName + '</div>' +
    cells +
    '<div style="width:36px; font-size:0.6em; color:var(--text-muted); padding-left:6px;">' + totalLabel + '</div>' +
  '</div>';
}

const el = this.container.createDiv();
el.innerHTML =
  '<div style="display:flex; flex-direction:column; gap:' + gap + 'px;">' +
    '<div style="display:flex; align-items:center; gap:' + gap + 'px;">' +
      '<div style="width:36px;"></div>' + headerCells + '<div style="width:36px;"></div>' +
    '</div>' +
    rows +
  '</div>';

const legendEl = this.container.createDiv();
legendEl.style.cssText = "display:flex; align-items:center; gap:5px; margin-top:10px; font-size:0.75em; color:var(--text-muted);";
legendEl.createSpan().textContent = "Less";
colors.slice(1).forEach(function(c) {
  const s = legendEl.createSpan();
  s.style.cssText = "width:10px; height:10px; background:" + c + "; border-radius:2px; display:inline-block;";
});
legendEl.createSpan().textContent = "More";
```
