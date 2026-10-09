# Slides Investigation: 2026_DUW_Analytics_DiracX

## Overview

This is a **Slidev** presentation for "Analytics in DiracX", presented at the 12th Dirac(X) Users' Workshop (DUW12), 13–16 October 2026, FZU Prague.

## Slidev Core Setup

### CLI Version
- **@slidev/cli**: `^52.6.0` (devDependency) — [`package.json:35`](package.json#L35)

### Entry Point
- **slides.md** at repo root — the main Slidev markdown file with 1046 lines of content — [`slides.md`](slides.md)

### Frontmatter (global defaults)
Defined on the first slide (`slides.md:1-11`):
- `colorSchema: light` — light mode default
- `favicon: /public/images/diracx-logo-square.svg`
- `color: diracx-light` — custom color scheme
- `layout: cover` — cover layout for the first slide
- `routerMode: hash` — hash-based routing (important for SPA hosting)
- `title: Analytics in DiracX`
- `theme: neversink` — custom theme
- `neversink_string: "DIRAC(X) DUW12"` — theme-specific string
- `download: true` — enables PDF download button

## Theme

### slidev-theme-neversink
- **Version**: `^0.4.0` — [`package.json:31`](package.json#L31)
- A custom theme that provides:
  - Multiple color schemes (dark, light, green variants)
  - Custom layouts: `cover`, `section`, `top-title`, `top-title-two-cols`, `credits`
  - The `neversink_string` frontmatter key for branding text
  - Component: `<SpeechBubble>`, `<AdmonitionType>`

### Custom Color Schemes (style.css)
Four DiracX-branded schemes defined in [`style.css:29-88`](style.css#L29-L88):

| Scheme | CSS Class | Short Class | Background | Border/Highlight |
|--------|-----------|-------------|------------|------------------|
| diracx (dark) | `neversink-diracx-scheme` | `ns-c-dx-scheme` | `#0f1a1e` | `#00afca` (teal) |
| diracx-light | `neversink-diracx-light-scheme` | `ns-c-dx-lt-scheme` | `#f5f9fa` | `#00afca` (teal) |
| diracx-green (dark) | `neversink-diracx-green-scheme` | `ns-c-dx-gr-scheme` | `#1a2420` | `#77b52c` (green) |
| diracx-green-light | `neversink-diracx-green-light-scheme` | `ns-c-dx-gr-lt-scheme` | `#f6faf2` | `#77b52c` (green) |

### Typography (style.css)
- **Headings**: Montserrat (wght 400, 600, 700, 800) — [`style.css:1-2`](style.css#L1-L2), [`style.css:22-26`](style.css#L22-L26)
- **Body**: Roboto (wght 300, 400, 500, 700) — [`style.css:3`](style.css#L3), [`style.css:17-19`](style.css#L17-L19)
- **Code**: Roboto Mono — [`style.css:3`](style.css#L3), [`style.css:12`](style.css#L12)
- **Neversink string**: Montserrat 800 weight — [`style.css:29-32`](style.css#L29-L32)

## UnoCSS Configuration

### unocss.config.ts
- **Preset**: `@unocss/preset-uno` — [`unocss.config.ts:2,8`](unocss.config.ts#L2)
- **Icons**: `@unocss/preset-icons` with scale 1.2 — [`unocss.config.ts:3,9-12`](unocss.config.ts#L3)
- **Transformer**: `@unocss/transformer-directives` — [`unocss.config.ts:4,36`](unocss.config.ts#L4)

### Safelist Entries
Classes that must be included even if not detected in source — [`unocss.config.ts:14-35`](unocss.config.ts#L14-L35):
- All neversink color scheme classes (long and short forms)
- DiracX accent classes (`diracx-accent`, `diracx-green-accent`)
- Background gradient (`bg-diracx-gradient`)
- Logo styling (`diracx-logo`)
- Section styling (`section-diracx`)
- Icon classes: `logos-grafana`, `logos-mysql`, `logos-postgresql`, `logos-opensearch`, `simple-icons-duckdb`, `simple-icons-amazons3`, `simple-icons-opentelemetry`

## Icon Packs

Five iconify collections installed as dependencies — [`package.json:26-32`](package.json#L26-L32):

| Package | Version | Usage |
|---------|---------|-------|
| `@iconify-json/simple-icons` | `^1.2.99` | Tech logos (DuckDB, S3, OpenTelemetry) |
| `@iconify-json/logos` | `^1.2.15` | Brand logos (Grafana, MySQL, PostgreSQL, OpenSearch) |
| `@iconify-json/devicon` | `^1.2.68` | Dev icons |
| `@iconify-json/devicon-plain` | `^1.2.61` | Plain dev icons |
| `@iconify-json/mdi` | `^1.2.3` | Material Design icons (e.g., `mdi-open-in-new`) |

Icons are used inline via UnoCSS icon classes, e.g., `<span class="i-simple-icons:duckdb">` ([`slides.md:373`](slides.md#L373)) and `<mdi-open-in-new />` ([`slides.md:24`](slides.md#L24)).

## Custom Components Used in Slides

### From neversink theme
- `<SpeechBubble>` — speech bubble callout with color, shape, maxWidth, position props — e.g., [`slides.md:88-90`](slides.md#L88-L90)
- `<AdmonitionType>` — admonition blocks with `type='important'` or `type='note'` — e.g., [`slides.md:114-125`](slides.md#L114-L125)
- `<Email>` — email display component — e.g., [`slides.md:16`](slides.md#L16)

### Slidev built-in
- Slot-based layouts using `:: title ::`, `:: content ::`, `:: left ::`, `:: right ::` syntax — used throughout

## Layouts Used

| Layout | First Used | Purpose |
|--------|------------|---------|
| `cover` | [`slides.md:5`](slides.md#L5) | Title slide |
| `section` | [`slides.md:27`](slides.md#L27) | Section divider slides |
| `top-title` | [`slides.md:34`](slides.md#L34) | Single-column content with title |
| `top-title-two-cols` | [`slides.md:60`](slides.md#L60) | Two-column layout with title |
| `credits` | [`slides.md:1018`](slides.md#L1018) | Credits/people slide with `loop: true` and `speed: 1.4` |

## Slidev Features in Use

### Mermaid Diagrams
Three Mermaid flowchart diagrams used:
1. DuckLake deployment architecture — [`slides.md:510-555`](slides.md#L510-L555)
2. Power user visualization architecture — [`slides.md:656-683`](slides.md#L656-L683)
3. Generic user visualization architecture (Grafana) — [`slides.md:716-763`](slides.md#L716-L763)

Each uses custom theme variables for DiracX colors.

### Code Blocks
Python code block with syntax highlighting — [`slides.md:782-793`](slides.md#L782-L793)

### Markdown Extensions
- Tables with embedded icon spans — throughout
- HTML components inline — throughout
- `<br>` tags for spacing — e.g., [`slides.md:154-158`](slides.md#L154-L158)
- External links with custom styling — e.g., [`slides.md:24`](slides.md#L24)

## Public Assets

Located in `public/images/`:
- `diracx-logo-square.svg` — favicon and summary slide logo
- `diracx-logo-full.svg` — full logo (referenced but not used in slides)
- `DuckLake_Logo-horizontal.svg` — DuckLake logo on DuckLake slide — [`slides.md:466`](slides.md#L466)

## No setup/ or .slidev/ Directories

- **No `setup/` directory** — no custom Vue components, layouts, or global setup
- **No `.slidev/` directory** — no Slidev-specific config overrides
- **No `components/` directory** — all components come from the neversink theme
- All customization is done via `style.css` and `unocss.config.ts` at the root level

## Build & Deployment

### npm Scripts
- `build`: `slidev build --download` — builds static site with PDF download enabled — [`package.json:6`](package.json#L6)

### Netlify Configuration ([`netlify.toml`](netlify.toml))
- **Publish directory**: `dist` — [`netlify.toml:2`](netlify.toml#L2)
- **Build command**: Installs all dependencies explicitly then runs `npm run build` — [`netlify.toml:3`](netlify.toml#L3)
  - Includes `playwright-chromium` for PDF export
  - Includes `@slidev/theme-default` (though not used — the theme is neversink)
- **Node version**: 20 — [`netlify.toml:6`](netlify.toml#L6)
- **SPA redirects**: All routes redirect to `/index.html` with status 200 — [`netlify.toml:8-11`](netlify.toml#L8-L11) (needed for hash router mode)

## Content Structure

The presentation has ~30 slides organized into 7 sections:

1. **Terminology** — OLTP, OLAP, ELT, Data Warehouse/Lake/Lakehouse, OTEL
2. **Scope** — Purpose of the proposal
3. **Background** — Related GitHub discussions and issues
4. **Requirements** — User stories for 4 user types
5. **Architecture and Technology Stack** — Parquet, S3, DuckDB, DuckLake, Grafana
6. **ELT** — Extract, Load, Transform details
7. **Visualizing what is in the ducklake** — Power user and generic user access patterns
8. **Changes WRT DIRAC** — Comparison table
9. **Conclusions** — Rejected ideas, summary, next steps

## Summary

This is a well-structured Slidev presentation using:
- **slidev-theme-neversink** (v0.4.0) as the base theme
- **Custom DiracX branding** via CSS variables and UnoCSS safelist
- **5 iconify collections** for inline tech logos
- **Mermaid diagrams** for architecture visualization
- **Netlify** for static hosting with SPA fallback
- **No custom Vue components** — all customization via CSS/theme configuration
- **Hash router mode** for compatibility with static hosting
