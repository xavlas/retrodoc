---
name: VitePress Expose
description: "Transforms retro-doc synthesis documents into a browsable VitePress documentation site"
user-invocable: false
tools:
  - read_file
  - create_file
  - replace_string_in_file
---

# VitePress Expose

Transforms the synthesis documents produced by the retro-doc orchestrator into a complete, browsable VitePress documentation site. Generates a site structure with configuration, navigation, and reformatted pages ready for interactive exploration and deployment.

## Purpose

- Convert synthesis Markdown pages into VitePress-compatible format with proper frontmatter.
- Generate `.vitepress/config.js` with site title, description, and navigation sidebar.
- Create `package.json` with VitePress dev dependencies and run scripts.
- Organize pages into logical sections (Architecture, Contracts, Business Logic, Migration).
- Produce a self-contained site ready to serve with `npm run docs:dev`.

## Inputs

- **synthesisDocs**: Object mapping document names to paths:
  - `index`: Path to `.retro-doc/index.md`
  - `stack`: Path to `.retro-doc/stack.md`
  - `architecture`: Path to `.retro-doc/architecture.md`
  - `apiContracts`: Path to `.retro-doc/api-contracts.md`
  - `businessLogic`: Path to `.retro-doc/business-logic.md`
  - `migrationReport`: Path to `.retro-doc/migration-report.md`
- **outputDir**: Absolute path to generate the site at (typically `.retro-doc/vitepress/`).
- (Optional) **projectName**: Human-readable project name for site title (inferred from index.md if not provided).
- (Optional) **dashboardHtml**: Absolute path to a pre-generated `dashboard.html` file. When provided, the file is copied into `docs/public/dashboard.html` and linked from the nav. When absent, this agent generates the dashboard itself (see **Step 6: Analytics Dashboard**).

## Output Artifact Structure

Generate the following structure under `outputDir`:

```
vitepress/
├── docs/
│   ├── index.md               # Home page (enhanced index)
│   ├── stack.md               # Technology stack page
│   ├── architecture.md        # Architecture overview page
│   ├── api-contracts.md       # API contracts reference page
│   ├── business-logic.md      # Business logic documentation page
│   ├── migration.md           # Migration strategy page
│   ├── dashboard.md           # Analytics dashboard page (iframe wrapper)
│   ├── public/
│   │   └── dashboard.html     # Self-contained interactive analytics dashboard
│   └── .vitepress/
│       ├── config.js          # VitePress site configuration
│       └── (theme files optional)
├── package.json               # NPM configuration with VitePress deps
└── .gitignore                 # Ignore dist/ and node_modules/
```

The site is ready to run from `vitepress/docs/` with:
```bash
cd docs
npm install
npm run docs:dev
```

## Required Steps

### Step 1: Read Synthesis Documents

1. Read all six synthesis documents from the paths provided in `synthesisDocs`.
2. Extract title, summary, and key sections from each.
3. Infer `projectName` from `index.md` if not provided (extract from title or first heading).

### Step 2: Generate VitePress Config

Create `.vitepress/config.js` with:

```javascript
export default {
  title: '<projectName>',
  description: 'Retro-documentation for <projectName> architecture, contracts, and migration strategy',
  themeConfig: {
    nav: [
      { text: 'Home', link: '/' },
      { text: 'Docs', items: [
        { text: 'Stack', link: '/stack' },
        { text: 'Architecture', link: '/architecture' },
        { text: 'API Contracts', link: '/api-contracts' },
        { text: 'Business Logic', link: '/business-logic' },
        { text: 'Migration', link: '/migration' }
      ]},
      { text: 'Dashboard', link: '/dashboard' }
    ],
    sidebar: {
      '/': [
        { text: 'Overview', items: [
          { text: 'Home', link: '/' },
          { text: 'Stack Summary', link: '/stack' }
        ]},
        { text: 'Architecture', items: [
          { text: 'Layers & Components', link: '/architecture' },
          { text: 'Data Flows', link: '/architecture#data-flows' }
        ]},
        { text: 'Integration', items: [
          { text: 'API Contracts', link: '/api-contracts' },
          { text: 'Data Models', link: '/api-contracts#data-models' }
        ]},
        { text: 'Domain', items: [
          { text: 'Business Logic', link: '/business-logic' },
          { text: 'Workflows', link: '/business-logic#workflows' }
        ]},
        { text: 'Migration', items: [
          { text: 'Strategy & Roadmap', link: '/migration' },
          { text: 'Risks & Gaps', link: '/migration#risks' }
        ]},
        { text: 'Analytics', items: [
          { text: 'Dashboard', link: '/dashboard' }
        ]}
      ]
    },
    socialLinks: [
      { icon: 'github', link: 'https://github.com' }
    ]
  }
}
```

### Step 3: Adapt Markdown Pages for VitePress

For each synthesis document:

1. Add VitePress frontmatter if missing:
   ```yaml
   ---
   title: Page Title
   description: Short page description
   ---
   ```

2. Ensure first heading is `# Page Title` and only appears once (VitePress expects this).
3. Convert relative wikilinks `[[page-name]]` to VitePress links:
   - `[[architecture]]` → `[Architecture](/architecture)`
   - `[[api-contracts]]` → `[API Contracts](/api-contracts)`
4. Preserve code blocks, tables, and Mermaid diagrams as-is.
5. Ensure all headings use proper nesting (`##`, `###`, not skipping levels).

### Step 4: Generate Site Pages

Create the seven documentation pages under `docs/`:

- **index.md** — Home page synthesizing all content areas; extract summary from index.md and add quick-links to main sections. Include a "📊 Analytics Dashboard" card that links to `/dashboard`.
- **stack.md** — Technology inventory and dependency graph; add a table of contents with major sections from stack.md.
- **architecture.md** — Architectural map with diagrams; structure with clear sections for components, layers, entry points.
- **api-contracts.md** — API endpoints and data models; add searchable table of contents for quick reference.
- **business-logic.md** — Domain rules and workflows; organize by domain entity or workflow.
- **migration.md** — Migration strategy; include risk matrix and recommendations.
- **dashboard.md** — Analytics dashboard page; see **Step 6**.

### Step 5: Generate package.json

Create `package.json` in `vitepress/`:

```json
{
  "name": "<projectName>-docs",
  "description": "VitePress documentation for <projectName>",
  "version": "1.0.0",
  "scripts": {
    "docs:dev": "vitepress dev docs",
    "docs:build": "vitepress build docs",
    "docs:preview": "vitepress preview docs"
  },
  "devDependencies": {
    "vitepress": "^1.0.0",
    "vue": "^3.3.0"
  }
}
```

### Step 6: Analytics Dashboard

The dashboard is a self-contained interactive HTML file (Apache ECharts 5 via CDN) that visualises all domains extracted from the synthesis documents across five chart types.

**6a — Source the HTML file**

- If `dashboardHtml` input was provided: copy that file verbatim to `docs/public/dashboard.html`.
- If `dashboardHtml` was NOT provided: generate `docs/public/dashboard.html` directly using the data extraction and chart specs below.

**6b — Data extraction**

Read all five synthesis documents and extract the following per-domain metrics. Use sensible defaults (score = 1, count = 0) and flag estimated values in the tooltip.

| Metric | Source | Scoring |
|--------|--------|---------|
| `complexity` | `migrationReport.md` | Low=1 Medium=2 High=3 Critical=4 |
| `coupling` | `architecture.md` | count unique cross-domain dependencies |
| `apiSurface` | `apiContracts.md` | count endpoints belonging to domain |
| `businessRules` | `businessLogic.md` | count rules / workflows in domain |
| `testRisk` | `migrationReport.md` | Low=1 Medium=2 High=3 Critical=4 |
| `effortSize` | `migrationReport.md` | XS=1 S=2 M=3 L=4 XL=5 |
| `linesOfCode` | `stack.md` / `architecture.md` | raw if available, else `complexity × 100` |

Also extract directed cross-domain flows from `architecture.md` and `apiContracts.md` (source → target, label, weight 1–5).

**6c — Chart specifications**

Generate all five charts inside a single `<script>` block using `echarts.init()`. Attach `window.addEventListener('resize', ...)` to each chart instance. Use the shared colour palette below for all charts.

```javascript
const COMPLEXITY_COLOR = { Low:'#52c41a', Medium:'#faad14', High:'#ff7a45', Critical:'#f5222d' };
const EFFORT_COLORMAP  = ['#52c41a','#a0d911','#faad14','#ff7a45','#f5222d']; // XS→XL
```

| # | Chart | ECharts type | X | Y | Size/Colour |
|---|-------|-------------|---|---|-------------|
| 1 | **Heatmap** — Complexity × Risk | `heatmap` on `cartesian2d` | Complexity level | Risk level | Cell colour = effortSize (green→red) |
| 2 | **Treemap** — Codebase by domain | `treemap` | — | — | Area = linesOfCode; colour = complexity |
| 3 | **Quadrant** — Effort vs. API Surface | `scatter` | effortSize | apiSurface | Bubble radius = coupling; crosshair `markLine` at median X & Y; quadrant labels as `markPoint` |
| 4 | **Sankey** — Cross-domain flows | `sankey` | — | — | Node colour = complexity; link width = flow weight |
| 5 | **Radar** — Domain capability profile | one `series` of `type:'radar'`; all domains as items inside `series[0].data[]`, each with its own `lineStyle`/`itemStyle`/`areaStyle` — **never** one series per domain (ECharts silently ignores all but the first) | 6 spokes: Complexity, Coupling, API Surface, Business Rules, Test Risk, Effort (all normalised 0–100) | — | Legend toggleable via `selectedMode` |

**6d — HTML layout**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>{projectName} — Analytics Dashboard</title>
  <script src="https://cdn.jsdelivr.net/npm/echarts@5/dist/echarts.min.js"></script>
  <style>
    /* CSS Grid: header + 5 chart sections */
    /* Each chart container: min-height 420px */
    /* Dark mode via prefers-color-scheme */
    body { font-family: system-ui, sans-serif; margin: 0; padding: 1rem 2rem; }
    header { border-bottom: 1px solid #eee; padding-bottom: 1rem; margin-bottom: 2rem; }
    .chart-section { margin-bottom: 3rem; }
    .chart-section h2 { font-size: 1.1rem; margin-bottom: 0.25rem; }
    .chart-section p  { color: #666; font-size: 0.85rem; margin-bottom: 0.5rem; }
    .chart-box { width: 100%; }
    @media (prefers-color-scheme: dark) {
      body { background: #1a1a1a; color: #eee; }
      .chart-section p { color: #aaa; }
    }
  </style>
</head>
<body>
  <header>
    <h1>{projectName} — Analytics Dashboard</h1>
    <p>Generated by Retro Doc · {generated date} &nbsp;|&nbsp;
       <a href="/">← Back to docs</a></p>
  </header>
  <main>
    <section id="heatmap"  class="chart-section"><h2>Complexity × Risk Heatmap</h2><p>…</p><div id="c-heatmap"  class="chart-box" style="height:420px"></div></section>
    <section id="treemap"  class="chart-section"><h2>Codebase Composition</h2><p>…</p><div id="c-treemap"  class="chart-box" style="height:420px"></div></section>
    <section id="quadrant" class="chart-section"><h2>Effort vs. API Surface</h2><p>…</p><div id="c-quadrant" class="chart-box" style="height:440px"></div></section>
    <section id="sankey"   class="chart-section"><h2>Cross-Domain Flows</h2><p>…</p><div id="c-sankey"   class="chart-box" style="height:500px"></div></section>
    <section id="radar"    class="chart-section"><h2>Domain Capability Radar</h2><p>…</p><div id="c-radar"    class="chart-box" style="height:500px"></div></section>
  </main>
  <script>/* inline data constants + 5× echarts.init() + resize listeners */</script>
</body>
</html>
```

**6e — VitePress wrapper page**

Create `docs/dashboard.md`:

```markdown
---
title: Analytics Dashboard
description: Interactive heatmap, treemap, quadrant, Sankey, and radar visualisations by domain
---

# Analytics Dashboard

<div style="width:100%;height:85vh;border:none;">
  <iframe src="/dashboard.html"
          style="width:100%;height:100%;border:none;"
          title="Analytics Dashboard">
  </iframe>
</div>

> Prefer the full-screen experience? Open [dashboard.html](/dashboard.html) directly in your browser.
```

> **Why `docs/public/`?** VitePress copies everything in `docs/public/` to the site root at build time, so `/dashboard.html` resolves correctly both in dev and in the built static site.

### Step 7: Generate .gitignore

Create `.gitignore` in `vitepress/`:

```
node_modules/
dist/
.DS_Store
*.swp
*.swo
```

### Step 8: Generate Manifest

Create `subagents/vitepress-manifest.md` documenting:

- Total files generated
- Site entry point (`docs/index.md`)
- Build instructions
- Generated timestamp
- Any warnings or fallbacks applied

## Quality Checks

After generation:

1. Verify all pages have unique titles and descriptions in frontmatter.
2. Confirm all internal links (VitePress format) are valid paths in `docs/`.
3. Check that all Mermaid diagrams are preserved verbatim.
4. Ensure code blocks have language identifiers (`\`\`\`javascript`, etc.).
5. Validate that `config.js` references only existing pages, including `/dashboard`.
6. Confirm `docs/public/dashboard.html` exists and contains all five ECharts chart initialisations.
7. Confirm `docs/dashboard.md` iframes `/dashboard.html` (not a relative `./` path — VitePress serves `public/` at the root).

## Notes

- The generated site is **self-contained** — no external CDN or build steps required beyond the ECharts CDN script in `dashboard.html`.
- VitePress generates static HTML from these Markdown files; the site can be deployed to any static host.
- `docs/public/` is the correct location for the dashboard HTML — VitePress copies it to the site root at build time, making `/dashboard.html` a valid path in both dev and production.
- The iframe in `dashboard.md` uses an absolute path (`src="/dashboard.html"`) so it resolves correctly at every nesting level within VitePress.
- Theme customisation (colors, fonts) can be added to `.vitepress/theme.js` if needed.
- The site is **read-only** — changes should be made to synthesis documents and regenerated via this subagent.
- If both VitePress Expose and Analytics Dashboard subagents run in parallel (orchestrator Phase 3b), VitePress Expose checks whether `docs/public/dashboard.html` already exists before generating it — do not overwrite a pre-generated file.
