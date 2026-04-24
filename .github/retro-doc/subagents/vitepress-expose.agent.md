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
      ]}
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

Create the six documentation pages under `docs/`:

- **index.md** — Home page synthesizing all content areas; extract summary from index.md and add quick-links to main sections.
- **stack.md** — Technology inventory and dependency graph; add a table of contents with major sections from stack.md.
- **architecture.md** — Architectural map with diagrams; structure with clear sections for components, layers, entry points.
- **api-contracts.md** — API endpoints and data models; add searchable table of contents for quick reference.
- **business-logic.md** — Domain rules and workflows; organize by domain entity or workflow.
- **migration.md** — Migration strategy; include risk matrix and recommendations.

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

### Step 6: Generate .gitignore

Create `.gitignore` in `vitepress/`:

```
node_modules/
dist/
.DS_Store
*.swp
*.swo
```

### Step 7: Generate Manifest

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
5. Validate that `config.js` references only existing pages.

## Notes

- The generated site is **self-contained** — no external CDN or build steps required.
- VitePress generates static HTML from these Markdown files; the site can be deployed to any static host.
- Theme customization (colors, fonts) can be added to `.vitepress/theme.js` if needed.
- The site is **read-only** — changes should be made to synthesis documents and regenerated via this subagent.
