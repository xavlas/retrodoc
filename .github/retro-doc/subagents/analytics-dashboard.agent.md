---
name: Analytics Dashboard
description: "Generates an interactive analytics dashboard with heatmap, treemap, quadrant chart, Sankey diagram, and radar lines grouped by domain"
user-invocable: false
tools:
  - read_file
  - create_file
  - replace_string_in_file
---

# Analytics Dashboard

Generates a self-contained, interactive HTML analytics dashboard from the retro-doc synthesis documents. The dashboard provides five complementary visualisations — heatmap, treemap, quadrant chart, Sankey diagram, and radar lines — each sliced by domain to give a multi-dimensional view of the codebase's complexity, coupling, and migration risk.

## Purpose

- Surface patterns that static documents hide (hot spots, structural debt, cross-domain coupling).
- Give the rewrite team a single-page overview they can share without running the VitePress site.
- Enable domain-level comparison across complexity, coverage, coupling, and migration effort.
- Produce a file that requires zero build steps — opens directly in any browser.

## Inputs

- **synthesisDocs**: Object mapping document names to paths:
  - `stack`: Path to `.retro-doc/stack.md`
  - `architecture`: Path to `.retro-doc/architecture.md`
  - `apiContracts`: Path to `.retro-doc/api-contracts.md`
  - `businessLogic`: Path to `.retro-doc/business-logic.md`
  - `migrationReport`: Path to `.retro-doc/migration-report.md`
- **outputFile**: Absolute path to write the dashboard (`.retro-doc/vitepress/docs/dashboard.md` — the agent writes an HTML file alongside it at `.retro-doc/dashboard.html`).
- **projectName**: Human-readable project name (inferred from `architecture.md` if not provided).

## Output Artifact

A single self-contained file: `.retro-doc/dashboard.html`

- Zero external runtime dependencies — all JS/CSS inlined or loaded from a public CDN (jsDelivr / unpkg).
- Uses **Apache ECharts 5** for all charts (one library, consistent API, zero build).
- Responsive layout via CSS Grid; works at 1280 px width and above.
- Dark-mode aware via `prefers-color-scheme`.
- A companion Markdown stub `.retro-doc/vitepress/docs/dashboard.md` that iframes the HTML file so it appears in the VitePress navigation.

## Data Extraction Protocol

Before generating any chart, read all five synthesis documents and extract the following structured data. Record what you find; use sensible defaults (score = 1, count = 0) when a field is not mentioned.

### Domain List

Extract every named domain / bounded context / functional module from `architecture.md` and `businessLogic.md`. A domain is any top-level heading that groups multiple components. Normalise to a short label (e.g., "Pricing", "Auth", "Notification"). Aim for 5–15 domains; merge very small ones.

### Per-Domain Metrics

For each domain, extract or estimate:

| Metric | Source document | Scoring guide |
|--------|----------------|---------------|
| `complexity` | `migrationReport.md` | Low=1 Medium=2 High=3 Critical=4 |
| `coupling` | `architecture.md` | count unique cross-domain dependencies |
| `apiSurface` | `apiContracts.md` | count endpoints belonging to domain |
| `businessRules` | `businessLogic.md` | count rules / workflows in domain |
| `testRisk` | `migrationReport.md` | Low=1 Medium=2 High=3 Critical=4 |
| `effortSize` | `migrationReport.md` | XS=1 S=2 M=3 L=4 XL=5 |
| `linesOfCode` | `stack.md` or `architecture.md` | raw number if available, else estimate from complexity×100 |

### Cross-Domain Flows

From `architecture.md` and `apiContracts.md`, identify directed dependencies between domains:
`source domain → target domain: flow label, approximate call volume or weight (1–5 if unknown)`

---

## Chart Specifications

### Chart 1 — Heatmap: Complexity × Risk Grid

**What it shows:** Each domain plotted as a cell in a grid whose axes are migration complexity (X) and test risk (Y). Cell colour intensity encodes `effortSize`. Size of cell label encodes `apiSurface`.

**ECharts type:** `heatmap` on a `cartesian2d` grid with category axes.

**X axis:** Complexity levels — Low / Medium / High / Critical  
**Y axis:** Risk levels — Low / Medium / High / Critical  
**Value (colour):** `effortSize` (1–5, palette: green→yellow→red)  
**Tooltip:** domain name, complexity, risk, effort, API surface

**Why useful:** Immediately flags "Critical complexity + High risk" hot spots that must be addressed first in the rewrite roadmap.

---

### Chart 2 — Treemap: Codebase Composition by Domain

**What it shows:** A nested treemap where tile area encodes `linesOfCode` and tile colour encodes `complexity` (green→red gradient).

**ECharts type:** `treemap`

**Hierarchy:**
```
root (project name)
└── domain (one tile per domain)
    └── value: linesOfCode
    └── itemStyle.color: complexity colour
```

**Tooltip:** domain, lines of code, complexity rating, number of business rules

**Why useful:** Shows where the bulk of the code lives and whether large modules are also the most complex.

---

### Chart 3 — Quadrant Chart: Effort vs. API Surface

**What it shows:** Each domain as a bubble. X = `effortSize`, Y = `apiSurface`, bubble radius = `coupling`. A reference crosshair divides the space into four quadrants labelled:
- Top-right: **High Exposure, High Effort** (tackle last, risk of regression)
- Top-left: **High Exposure, Low Effort** (quick wins — migrate early)
- Bottom-right: **Low Exposure, High Effort** (internal complexity — plan carefully)
- Bottom-left: **Low Exposure, Low Effort** (simple utilities — migrate any time)

**ECharts type:** `scatter` with `symbolSize` proportional to coupling, plus two `markLine` elements for the crosshair at median X and median Y.

**Tooltip:** domain, effort, API endpoints, coupling score

**Why useful:** Directly informs the migration sequencing decision — surfaces the "quick win" domains a team should tackle in Phase 1.

---

### Chart 4 — Sankey Diagram: Cross-Domain Data Flows

**What it shows:** Directed flows between domains. Node width encodes total in/out flow weight; link width encodes individual flow weight.

**ECharts type:** `sankey`

**Nodes:** one per domain  
**Links:** each cross-domain dependency from the extracted flow list, with `value` = weight (1–5)

**Colour scheme:** nodes coloured by domain complexity (same green→red palette as treemap); links inherit source node colour at 60 % opacity.

**Tooltip:** source → target, flow label, weight

**Why useful:** Exposes tight coupling and hidden fan-out that raises integration risk during the rewrite.

---

### Chart 5 — Radar Lines: Domain Capability Profile

**What it shows:** A radar (spider) chart with one polyline per domain. Each spoke represents one metric: Complexity, Coupling, API Surface, Business Rules, Test Risk, Effort. Values normalised to 0–100 for comparability.

**ECharts type:** one single `series` of `type: 'radar'` with **all domains as items inside `series[0].data[]`**. Each item carries its own `lineStyle`, `itemStyle`, and `areaStyle`. Do NOT produce one series per domain — ECharts ignores all but the first series on a radar and the chart renders empty.

**Spokes (indicators):**
1. Complexity (max 4)
2. Coupling (max: highest coupling found across all domains)
3. API Surface (max: highest apiSurface)
4. Business Rules (max: highest businessRules)
5. Test Risk (max 4)
6. Effort (max 5)

**Series:** one line per domain, each with a distinct colour from ECharts' default palette. Legend is clickable to toggle domains on/off.

**Why useful:** Enables side-by-side domain comparison across all six dimensions simultaneously — useful for prioritisation meetings.

---

### Chart 6 — Horizontal Bar Chart: Quality & Maturity Scores (0–100)

**What it shows:** Ten scored dimensions of codebase health, each rendered as a horizontal bar from 0 to 100. Bars are colour-coded by score band: red (0–39), orange (40–59), amber (60–74), green (75–100). A vertical reference line at 75 marks the "acceptable threshold".

**ECharts type:** `bar` with `orient: 'horizontal'` (i.e. `xAxis` = value 0–100, `yAxis` = category).

**The 10 dimensions and how to score them from the synthesis documents:**

| # | Dimension | Source signals | Scoring guide |
|---|-----------|---------------|---------------|
| 1 | **Code Quality** | Complexity ratings in `migration-report.md`; High/Critical modules; code smells mentioned | 100 − (criticalCount×25 + highCount×10 + mediumCount×3) capped 0–100 |
| 2 | **Architecture Maturity** | Layer separation, pattern discipline in `architecture.md`; circular deps, mixed concerns | Start 100; −20 per anti-pattern found (mixed layers, God class, circular dep) |
| 3 | **API Design** | REST maturity, versioning, OpenAPI docs in `api-contracts.md` | +25 per: OpenAPI present, versioned paths, standard HTTP codes, consistent naming |
| 4 | **Test Coverage** | Test risk ratings, test file presence signals in `migration-report.md` and `stack.md` | Low risk = 80–100; Medium = 50–79; High = 20–49; Critical = 0–19 |
| 5 | **Observability** | Actuator, logging framework, metrics, tracing in `stack.md` and `architecture.md` | +20 per signal: health endpoint, structured logging, metrics, distributed tracing, alerting |
| 6 | **Security Posture** | Auth mechanism, input validation, secret management in `api-contracts.md` and `stack.md` | +25 per: JWT/OAuth present, HTTPS enforced, no secrets in code, input validation |
| 7 | **Domain Modeling** | Business logic isolation, bounded context clarity in `business-logic.md` | Start 100; −15 per: logic in controller, anemic model, missing domain boundary |
| 8 | **Dependency Health** | Dependency age, known issues, transitive coupling in `stack.md` | Start 100; −10 per outdated major version; −20 if CVEs or deprecated libs mentioned |
| 9 | **Migration Readiness** | Documentation completeness, overall migration complexity inverse in `migration-report.md` | Low overall = 80–100; Medium = 55–79; High = 30–54; Critical = 0–29 |
| 10 | **Operational Readiness** | CI/CD config, health checks, externalized config, Docker/K8s in `stack.md` and `architecture.md` | +20 per signal: CI pipeline, containerisation, externalized config, health check, graceful shutdown |

**Score derivation rules:**
- Compute each score from the signals found in the synthesis documents. Do not invent scores.
- If a signal is absent (e.g. no mention of tracing), treat it as not present (score contribution = 0).
- Clamp all scores to [0, 100].
- Flag each score as `confident` (signal explicitly found) or `estimated` (inferred from absence/context); show this in the tooltip.

**ECharts configuration:**

```javascript
// Score band colour
function scoreColor(v) {
  if (v >= 75) return '#52c41a';   // green
  if (v >= 60) return '#faad14';   // amber
  if (v >= 40) return '#ff7a45';   // orange
  return '#f5222d';                 // red
}

const SCORES = [
  { name: 'Code Quality',          score: <computed>, confident: true|false },
  { name: 'Architecture Maturity', score: <computed>, confident: true|false },
  // … all 10 …
];

const option = {
  title: { text: 'Quality & Maturity Scores', left: 'center', top: 8 },
  tooltip: {
    trigger: 'axis', axisPointer: { type: 'shadow' },
    formatter: params => {
      const d = SCORES.find(s => s.name === params[0].name);
      const band = d.score >= 75 ? '✅ Good' : d.score >= 60 ? '⚠️ Fair' : d.score >= 40 ? '🔶 Weak' : '❌ Poor';
      return `<b>${params[0].name}</b><br/>Score: ${d.score}/100 ${band}<br/>${d.confident ? '(from source)' : '(estimated)'}`;
    }
  },
  grid: { left: 160, right: 60, top: 50, bottom: 30 },
  xAxis: { type: 'value', min: 0, max: 100,
    axisLabel: { formatter: v => v },
    splitLine: { lineStyle: { color: '#eee' } }
  },
  yAxis: { type: 'category', data: SCORES.map(s => s.name), axisLabel: { fontSize: 11 } },
  series: [{
    type: 'bar',
    data: SCORES.map(s => ({ value: s.score, itemStyle: { color: scoreColor(s.score) } })),
    barMaxWidth: 32,
    label: { show: true, position: 'right', formatter: p => `${p.value}`, fontSize: 11 }
  }, {
    // Reference line at 75 — rendered as a ghost bar series using markLine
    type: 'bar', barWidth: 0, silent: true,
    data: SCORES.map(() => 0),
    markLine: {
      silent: true, symbol: 'none',
      lineStyle: { color: '#4361ee', type: 'dashed', width: 1.5 },
      data: [{ xAxis: 75, label: { formatter: 'Threshold 75', position: 'insideEndTop', fontSize: 10 } }]
    }
  }]
};
```

**Why useful:** Gives a single-glance health scorecard that stakeholders can read without any domain knowledge of the codebase. Drives prioritisation: dimensions below 40 need immediate attention before or during migration.

---

## HTML Structure

Generate the file using the following layout:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>{projectName} — Analytics Dashboard</title>
  <script src="https://cdn.jsdelivr.net/npm/echarts@5/dist/echarts.min.js"></script>
  <style>
    /* CSS Grid layout: header + 5-chart grid */
    /* Dark mode via prefers-color-scheme */
    /* Chart containers: min-height 400px, responsive width */
  </style>
</head>
<body>
  <header>
    <h1>{projectName} — Analytics Dashboard</h1>
    <p class="subtitle">Generated by Retro Doc · {generated date}</p>
    <nav><!-- anchor links: heatmap, treemap, quadrant, sankey, radar, scores --></nav>
  </header>

  <main>
    <section id="heatmap">
      <h2>Complexity × Risk Heatmap</h2>
      <p class="description"><!-- one sentence explaining the chart --></p>
      <div id="chart-heatmap" style="height:420px"></div>
    </section>

    <section id="treemap">
      <h2>Codebase Composition — Treemap</h2>
      <p class="description"><!-- one sentence --></p>
      <div id="chart-treemap" style="height:420px"></div>
    </section>

    <section id="quadrant">
      <h2>Effort vs. API Surface — Quadrant Chart</h2>
      <p class="description"><!-- one sentence --></p>
      <div id="chart-quadrant" style="height:440px"></div>
    </section>

    <section id="sankey">
      <h2>Cross-Domain Data Flows — Sankey Diagram</h2>
      <p class="description"><!-- one sentence --></p>
      <div id="chart-sankey" style="height:500px"></div>
    </section>

    <section id="radar">
      <h2>Domain Capability Profile — Radar Lines</h2>
      <p class="description"><!-- one sentence --></p>
      <div id="chart-radar" style="height:500px"></div>
    </section>

    <section id="scores">
      <h2>Quality &amp; Maturity Scores</h2>
      <p class="description"><!-- one sentence --></p>
      <div id="chart-scores" style="height:420px"></div>
    </section>
  </main>

  <script>
    // Inline all extracted data as JS constants
    // Initialise each chart in its own echarts.init() call
    // Attach window.addEventListener('resize', ...) for responsiveness
  </script>
</body>
</html>
```

---

## ECharts Colour Palette

Use this consistent palette throughout all charts so domains are recognisable across views:

```javascript
const COMPLEXITY_COLOR = {
  Low:      '#52c41a',  // green
  Medium:   '#faad14',  // amber
  High:     '#ff7a45',  // orange
  Critical: '#f5222d'   // red
};

const EFFORT_COLORMAP = [
  '#52c41a', '#a0d911', '#faad14', '#ff7a45', '#f5222d'
]; // index 0 = XS (green), 4 = XL (red)
```

Domain identity colours should be drawn from ECharts' built-in `colorBy: 'series'` palette so they stay consistent across radar, quadrant, and Sankey charts.

---

## VitePress Integration Stub

After writing `dashboard.html`, create `.retro-doc/vitepress/docs/dashboard.md`:

```markdown
---
title: Analytics Dashboard
description: Interactive visualisation of domain complexity, coupling, and migration effort
---

# Analytics Dashboard

<div style="width:100%;height:80vh;">
  <iframe src="../../dashboard.html" style="width:100%;height:100%;border:none;"></iframe>
</div>

> Open [dashboard.html](../../dashboard.html) directly in your browser for the full interactive experience.
```

Update `.retro-doc/vitepress/docs/.vitepress/config.js` to add a Dashboard entry to the sidebar and nav if the file already exists — append; do not overwrite the entire file.

---

## Required Steps

### Step 1: Read Synthesis Documents

Read all five synthesis documents in full. Extract the domain list and per-domain metrics table using the protocol in **Data Extraction Protocol**. Record confidence level (high / estimated) for each metric so the tooltip can flag estimated values.

### Step 2: Build Data Constants

Write the extracted data as inline JavaScript constants — one object per domain, one array for flows. These constants are the single source of truth for all five charts.

### Step 3: Generate Charts

Implement all five charts in the order listed. Each chart must:
- Use the shared colour palette.
- Show a tooltip with all relevant metrics.
- Include a descriptive title rendered inside the chart via ECharts `title` option.
- Gracefully handle missing data (empty domain list → render a "No data extracted" placeholder instead of crashing).

### Step 4: Write Files

1. Write the complete self-contained HTML to `.retro-doc/dashboard.html`.
2. Write the VitePress stub to `.retro-doc/vitepress/docs/dashboard.md`.
3. If `.retro-doc/vitepress/docs/.vitepress/config.js` exists, patch in the Dashboard nav/sidebar entry.

### Step 5: Return Summary

Return to the parent orchestrator:
- Path to `dashboard.html`
- List of domains extracted (count)
- Any metrics that had to be estimated (flag them)
- Whether the VitePress config was patched

---

## Required Protocol

1. Never hallucinate metric values — if a metric cannot be found, use the default and flag it as estimated in the tooltip.
2. All five charts must be present in the final file; partial output is not acceptable.
3. The HTML file must open in a browser with no network requests beyond the single ECharts CDN script.
4. Do not include inline `onclick` handlers that could be blocked by CSP — use ECharts event API instead.
5. Normalise all radar values to a 0–100 scale before rendering; raw values in different units are not comparable on a shared radar.
6. The Sankey diagram must show at least one inter-domain flow; if none are found in the synthesis docs, synthesise one plausible flow based on architectural layers (e.g., API layer → Service layer → Repository layer).
7. Log every domain and every flow in the manifest returned to the orchestrator so it can be audited.
