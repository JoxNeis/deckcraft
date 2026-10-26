---
name: deckcraft - documentation
status: development
author: JoxNeis
---

# deckcraft

An AI skill for crafting professional pitch decks: structure, story, and slides in one pass.

> **Status: blueprint.** Every file exists, so the structure is fixed, but almost all of them are empty. This README explains what each one is for. [Suggested writing order](#10-suggested-writing-order) gives an order that lets you test the skill early.

## Contents

1. [What deckcraft does](#1-what-deckcraft-does)
2. [How it builds decks](#2-how-it-builds-decks)
3. [Workflow](#3-workflow)
4. [Repository structure](#4-repository-structure)
5. [Sections explained](#5-sections-explained)
6. [The slide plan](#6-the-slide-plan)
7. [Chart detail levels](#7-chart-detail-levels)
8. [Requirement map](#8-requirement-map)
9. [Naming conventions](#9-naming-conventions)
10. [Suggested writing order](#10-suggested-writing-order)

## 1. What deckcraft does

deckcraft turns a brief and some data into a consulting-style `.pptx`, in the look of a McKinsey pitch deck:

- **Plan before building.** The AI writes a slide plan (each slide's layout, message, content, chart and notes) and shows it to the user before any `.pptx` exists.
- **Real charts.** Every chart is drawn with python-pptx, as an editable PowerPoint chart wherever possible. No matplotlib, no images.
- **One message per slide.** Each slide has an *action title*: a full sentence stating the slide's takeaway.
- **The user decides detail.** For every chart, the AI asks whether it should be simple or detailed.
- **Ready to present.** Every slide gets speaker notes.

## 2. How it builds decks

The skill is made of **knowledge, not code**: instructions, reference docs and templates. In the build step, the AI writes and runs python-pptx code itself, guided by the references.

Models already know python-pptx well. What they don't know is your house style, or the tricks for charts that PowerPoint has no built-in type for. The references supply exactly that, and they keep every deck consistent:

| Consistency comes from | Where |
| --- | --- |
| Exact positions, sizes and colours | Region coordinates in `slide-layouts.md`, sizes in `typography.md`, hex values in `color-palette.md` |
| The same build order and building blocks every time | `python-pptx-guide.md` |
| A proven approach for every chart, including the hard ones | One recipe per chart in `references/charts/` |
| Consistent metric definitions | `analytics-recipes.md` (CAGR, price-volume-mix, NPS, …) |
| Plans checked before building | The checklist in `slide-plan-format.md` |

**What to watch:**

- Nothing is enforced automatically. If the AI skips a step, such as a detail question, only its own checklist catches it, so the checklists in `slide-plan-format.md` and `qa-checklist.md` need to be explicit.
- The four charts that need XML edits (bullet, Pareto, box plot, combo) are easy to get wrong. Test the code examples in their recipes before relying on them.

### Where generated files go

Never inside the skill folder. The AI works in the user's project, one folder per deck:

```text
deck-output/<deck-name>/
├── plan.yaml          the approved slide plan
├── deck.pptx
└── thumbnails/        one PNG per slide, for QA
```

## 3. Workflow

What the skill does, start to finish. `SKILL.md` will spell out these steps.

| # | Step | What happens | Reads |
| --- | --- | --- | --- |
| 0 | Setup | Check that `python-pptx` and `pandas` are installed. LibreOffice is optional, for thumbnails | `SKILL.md` |
| 1 | Intake | Ask for audience, objective, length, data files and theme | `SKILL.md`, `color-palette.md` |
| 2 | Storyline | Pick a storyline skeleton; draft only the action titles (a "ghost deck") | `storytelling.md`, `assets/slide-plans/storylines/` |
| 3 | Explore data | Compute the numbers behind each title with pandas | `analytics-catalog.md`, `analytics-recipes.md` |
| 4 | Write the plan | Fill `plan.yaml`: layout, title, content, each chart with its final numbers, source and notes | `slide-plan-format.md`, `slide-layouts.md`, `chart-types.md`, `speaker-notes.md` |
| 5 | Review the plan | Show the plan as a storyboard table in chat (slide, title, layout, chart, notes summary); the user approves or edits | — |
| 6 | Detail questions | For **every** chart: simple or detailed? Write the answers into the plan | `chart-detail-levels.md` |
| 7 | Check the plan | Run the checklist ([section 6](#checking-a-plan)) | `slide-plan-format.md` |
| 8 | Build | Build the deck with python-pptx, following the guide's build order and the recipe for each chart type used | `python-pptx-guide.md`, `charts/NN_<id>.md`, `typography.md` |
| 9 | QA | Render thumbnails, compare charts with their previews, read back each slide's text and notes, fix and rebuild | `qa-checklist.md`, `charts/previews/` |
| 10 | Deliver | Hand over `deck.pptx` with a short summary of the storyline | — |

Users can start at step 4 with their own plan file (see [section 6](#6-the-slide-plan)).

## 4. Repository structure

```text
deckcraft/
├── README.md                        ← this file
├── SKILL.md                         ← skill entry point the AI loads first
├── LICENSE
│
├── references/                      ← everything the AI needs, read on demand
│   ├── storytelling.md              ← pyramid principle, SCR, action titles, MECE
│   ├── slide-plan-format.md         ← every plan field + the pre-build checklist
│   ├── slide-layouts.md             ← layout catalogue with exact region coordinates
│   ├── speaker-notes.md             ← notes structure and the notes-pane format
│   ├── color-palette.md             ← (started) palettes: a hex value per colour role
│   ├── typography.md                ← fonts, fallbacks, size per text role
│   ├── chart-types.md               ← (exists) which of the 30 charts to use when
│   ├── chart-detail-levels.md       ← what simple vs detailed changes; how to ask
│   ├── python-pptx-guide.md         ← build order, building blocks, pitfalls
│   ├── analytics-catalog.md         ← business question → analysis → chart
│   ├── analytics-recipes.md         ← metric definitions + pandas examples
│   ├── qa-checklist.md              ← review checks + how to render thumbnails
│   └── charts/
│       ├── 01_column_highlight.md … 30_funnel.md    ← one build recipe per chart
│       └── previews/
│           └── 01_column_highlight.png … 30_funnel.png   ← target look per chart
│
└── assets/                          ← files the AI copies (not instructions)
    ├── slide-plans/
    │   ├── slide-plan.schema.json   ← formal plan definition (AI reference + editor autocomplete)
    │   ├── blank.plan.yaml          ← every field, commented, nothing filled
    │   ├── example.plan.yaml        ← a complete worked example
    │   └── storylines/              ← investor-pitch · strategy-recommendation ·
    │                                  market-entry · business-review (.plan.yaml)
    └── sample-data/                 ← revenue_by_segment · market_players ·
                                       budget_vs_actual · customer_survey (.csv)
```

## 5. Sections explained

### 5.1 `SKILL.md`

The entry point. A skill loader reads the frontmatter to decide when to trigger the skill, then reads the body. Keep it short (under about 500 lines) and move depth into `references/`. Planned outline:

```markdown
---
name: deckcraft
description: <REQUIRED. What it does and when to use it: pitch decks, consulting-style
  or board presentations, .pptx decks with charts, "make slides from this data", etc.>
version: 0.0.1
---
# deckcraft
## When to use / when not to
## Setup             ← dependencies; where generated files go
## Workflow          ← the steps of section 3, each naming the reference to read
## Rules             ← non-negotiables (below)
## Reference index   ← which references/ file to read at which step
```

Rules the body should state:

- One message per slide; the action title is a full sentence (about 15 words at most).
- Never build before the user has seen the plan.
- Ask the detail question for every chart; never assume the answer.
- Run the plan checklist before building.
- Every slide gets speaker notes.
- Charts are native python-pptx objects or shapes, built from the recipes; never matplotlib or images.
- Every number on a slide has a source line.
- Generated files go in `deck-output/<deck-name>/`, never in the skill folder.

> The current `SKILL.md` frontmatter has `name` and `version` but no `description`. Skill loaders use `description` to decide when to trigger, so the skill won't activate until it's added.

### 5.2 `references/`

The AI reads each file only at the step that needs it, which keeps its context small.

| File | Read at step | Contents |
| --- | --- | --- |
| `storytelling.md` | 2 | Pyramid principle, situation–complication–resolution (SCR), writing action titles, MECE grouping, and checking that the titles alone tell the story. |
| `analytics-catalog.md` | 3 | "What question are you answering?" → analysis → recommended chart. |
| `analytics-recipes.md` | 3 | One definition and pandas example per analysis, so numbers are computed the same way every time: growth and CAGR, mix and mix shift, variance and bridges, price-volume-mix, Pareto, distributions, correlation and trend, segmentation, cohorts, funnels, KPI vs target, simple forecasts, sensitivity, Likert and NPS, financial metrics, weighted scoring. |
| `slide-plan-format.md` | 4, 7 | Every plan field (type, required or optional, default, example) and the checklist the AI runs before building. |
| `slide-layouts.md` | 4, 8 | The layout catalogue: when to use each layout, and the exact position (inches, for 16:9 and 4:3) of every region on it: tracker, title, subtitle, body, chart, takeaways, source, page number. |
| `chart-types.md` | 4 | (Exists.) Which of the 30 charts to use when, grouped by purpose. |
| `speaker-notes.md` | 4 | The notes block (key message, talking points, numbers to say aloud, transition, likely questions, timing), the tone, and the exact text format written into the notes pane. |
| `chart-detail-levels.md` | 6 | What simple and detailed switch on and off ([section 7](#7-chart-detail-levels)), how to explain the choice, and the exact question to ask. |
| `color-palette.md` | 1, 8 | (Started.) Each palette with a hex value for every colour role (primary, secondary, highlight, positive, negative, neutrals, categorical, sequential, diverging), and when to choose it. |
| `typography.md` | 1, 8 | Font pairs, fallbacks, and the size for every text role (action title, subtitle, body, chart label, axis, source, tracker). |
| `python-pptx-guide.md` | 8 | How the AI builds a deck with python-pptx (below). |
| `charts/NN_<id>.md` | 8 | One build recipe per chart ([section 5.3](#53-referencescharts)). |
| `charts/previews/` | 9 | The target look for each chart. |
| `qa-checklist.md` | 9 | Review checks (titles read as a story, numbers match across slides, units and sources present, no overflow, notes on every slide), plus how to render thumbnails (`soffice --headless --convert-to pdf`, then `pdftoppm -png`) and read back each slide's text and notes. |

**`python-pptx-guide.md`** is what makes every deck come out the same way. Planned sections:

- **Build order:** the same sequence for every deck: set the slide size (16:9 is 13.333 × 7.5 in), use the blank layout, take theme values from `color-palette.md` and `typography.md`, build each slide in plan order, save.
- **Building blocks:** exact settings for each standard element: action title, subtitle, body text, bullets, takeaways box, source line, page number, section tracker, speaker notes, table, image, web link and jump-to-slide link, chart styling for each detail level, and number formats.
- **Chart basics:** `CategoryChartData`, `XyChartData` and `BubbleChartData`; placing a chart in a region; per-point colours; custom data labels.
- **XML edits:** reaching the chart XML with lxml (`chart._chartSpace`), and when it's needed.
- **Pitfalls:** python-pptx doesn't measure text, so estimate line lengths to avoid overflow; fonts aren't embedded; there's no API for combo charts, secondary axes, trendlines or error bars; the doughnut hole is fixed at 50%; Office 2016 chart types (waterfall, funnel, histogram, box, Pareto) can't be created.

### 5.3 `references/charts/`

One recipe per chart: a clear written explanation of when to use it and how to build it. All 30 can be built inside PowerPoint with python-pptx (checked against the python-pptx 1.0.2 source); the tier says how.

Every recipe follows the same structure:

```markdown
# 05 Waterfall
Preview: previews/05_waterfall.png · Tier: B (data trick) · Plan id: waterfall

## Use when / avoid when
## Data shape            ← what `chart.data` in the plan must look like
## Build                 ← the approach step by step, with a tested python-pptx code example
## Simple vs detailed    ← exactly what changes for this chart
## Pitfalls
## Talking through it    ← a speaker-note pattern for this chart
```

| Tier | Meaning | Editable with Edit Data in PowerPoint? |
| --- | --- | --- |
| **A** Native | The python-pptx chart API as-is | Yes |
| **B** Data trick | The API plus helper series: invisible bases, negated values, pre-binned data, extra 2-point line series | Yes |
| **C** XML edit | A real chart, then lxml edits for what the API lacks: a second plot (combo), secondary axis, error bars | Yes |
| **D** Shapes/table | Drawn from native rectangles, pie shapes or a table; not a chart object | No, but still native shapes, not images |

| # | Recipe | Tier | python-pptx approach |
| --- | --- | --- | --- |
| 01 | `01_column_highlight.md` | A | Clustered column; per-point fill for the highlighted bar; CAGR arrow drawn as shapes |
| 02 | `02_ranked_hbar.md` | A | Clustered bar, sorted; `category_axis.reverse_order` puts rank 1 on top |
| 03 | `03_lines_direct_labels.md` | A | Line chart, legend off; a custom label on each series' last point names it |
| 04 | `04_stacked_100.md` | A | `COLUMN_STACKED_100` |
| 05 | `05_waterfall.md` | B | Stacked column: no-fill base series plus up/down series; split bases if the total crosses zero |
| 06 | `06_bubble_matrix.md` | A | `BUBBLE`; names as custom labels; quadrants via the axis `crosses_at` |
| 07 | `07_dumbbell.md` | B | XY scatter: a series per end, plus a 2-point line series per row as the connector |
| 08 | `08_diverging_likert.md` | B | Stacked bar with disagree values negated; number format hides the minus sign |
| 09 | `09_clustered_column.md` | A | `COLUMN_CLUSTERED` |
| 10 | `10_stacked_totals.md` | B | Stacked column plus a no-fill "total" series with custom labels |
| 11 | `11_marimekko.md` | D | Rectangles: width = segment share, height = share within the segment |
| 12 | `12_slope.md` | A | `LINE_MARKERS` with two categories; labels left and right |
| 13 | `13_stacked_area.md` | A | `AREA_STACKED` |
| 14 | `14_donut.md` | A | `DOUGHNUT` plus a centre text box |
| 15 | `15_bullet.md` | C | Stacked bar bands plus a narrow actuals bar on a secondary axis; target as a line shape |
| 16 | `16_pareto.md` | C | Column plus a cumulative-% line on a secondary axis |
| 17 | `17_tornado.md` | B | Clustered bar, overlap 100, low case negated, sorted by spread |
| 18 | `18_gantt.md` | B | Stacked bar: no-fill start offset plus duration on a date axis; milestones as shapes |
| 19 | `19_heatmap.md` | D | Native table with cell fills from the sequential or diverging palette |
| 20 | `20_small_multiples.md` | B | A grid of identical charts with shared axis min/max |
| 21 | `21_scatter_trend.md` | A | `XY_SCATTER` plus the fitted trend as a 2-point line series |
| 22 | `22_histogram.md` | B | Column chart with gap width 0 on pre-computed bins |
| 23 | `23_boxplot.md` | C | Stacked bar (no-fill base, Q1→median, median→Q3); whiskers as error bars |
| 24 | `24_range_bar.md` | B | XY scatter: a thick 2-point line per row for min–max, plus markers for the typical value |
| 25 | `25_lollipop.md` | B | XY scatter: a thin stem per item plus a dot series |
| 26 | `26_combo_dual_axis.md` | C | Column chart plus a line chart on a secondary axis |
| 27 | `27_cost_curve.md` | D | Rectangles: width = volume, height = unit cost, sorted by cost |
| 28 | `28_variance.md` | A | Clustered bar with positive/negative fill per point |
| 29 | `29_harvey_balls.md` | D | Native table plus pie autoshapes at 0/25/50/75/100% |
| 30 | `30_funnel.md` | B | Stacked bar: no-fill centring series plus the value series; reversed order |

Totals: 11 tier A, 11 tier B, 4 tier C and 4 tier D. That makes 26 editable chart objects and 4 built from shapes or tables.

### 5.4 `assets/`

Files the AI copies or fills in. They are not instructions.

- **`slide-plans/slide-plan.schema.json`**: the formal definition of a plan. Nothing enforces it automatically; it has two uses:
  - The AI can check a plan against it precisely in step 7.
  - Users writing their own plan get autocomplete and inline errors in VS Code (YAML extension) by adding a `# yaml-language-server: $schema=<path>/slide-plan.schema.json` line at the top of the plan.
- **`slide-plans/blank.plan.yaml`**: every field with a comment and no values. Users start here to write their own plan.
- **`slide-plans/example.plan.yaml`**: a complete deck of about 12 slides using `sample-data/`. Use it as the first end-to-end test of the skill.
- **`slide-plans/storylines/*.plan.yaml`**: skeleton decks for common pitch types, with sections, layouts and placeholder titles. The AI copies one in step 2 and fills it in.

  | Storyline | Arc |
  | --- | --- |
  | `investor-pitch` | problem → solution → market → traction → business model → team → ask |
  | `strategy-recommendation` | situation → complication → options → recommendation → roadmap → next steps |
  | `market-entry` | market size and growth → competitive landscape → customer needs → entry options → business case → plan |
  | `business-review` | headline KPIs → performance vs plan → drivers → risks → priorities for next period |

- **`sample-data/*.csv`**: small made-up datasets for the example plan: a time series by segment, players with size and share, budget vs actual, and a Likert survey.

## 6. The slide plan

The plan is the contract between the AI and the user, and the only input to the build. It's YAML because people can write YAML by hand and add comments.

**Charts carry their final numbers.** The AI computes them in step 3 and writes them into the plan, so the user reviews the real figures before anything is built, and building only draws them. For charts over large raw data (a histogram of 500 values), `data` can point at a file instead, and the recipe says how to compute it.

### Shape

```yaml
deck:
  title: Entering the Southeast Asian EV market     # required
  subtitle: Board discussion document
  audience: Board of directors
  objective: Approve a USD 40M phase-1 entry
  storyline: scr                  # scr | pyramid | custom
  theme: consulting-navy          # a palette from references/color-palette.md
  theme_overrides:                # optional
    palette: { highlight: "#E4572E" }
    typography: { body_font: Arial }
  slide_size: "16:9"              # 16:9 | 4:3
  defaults:
    chart_detail: ask             # ask | simple | detailed. `ask` = the AI must ask per chart

sections:                          # drive the tracker and the agenda slide
  - { id: market, title: Market context }
  - { id: plan,   title: Entry plan }

slides:
  - id: s03
    section: market
    layout: chart-with-takeaways   # required; from references/slide-layouts.md
    action_title: >-               # required: the one message, as a full sentence
      EV sales in Southeast Asia grew 4x since 2021, led by Thailand and Indonesia
    subtitle: EV unit sales, 2021–2025, thousand units
    purpose: Show the market is large and growing fast   # for reviewers; not shown
    content:
      takeaways:
        - Thailand alone is 45% of 2025 volume
        - Indonesia is growing fastest, at 2.3x per year
    chart:
      type: column_highlight       # recipe: references/charts/01_column_highlight.md
      data:                        # the exact numbers the chart shows
        categories: ["2021", "2022", "2023", "2024", "2025"]
        series: { EV units: [120, 190, 260, 370, 480] }
      derived_from: revenue_by_segment.csv, units summed by year   # provenance
      detail: ask                  # replaced by the user's answer in step 6
      highlight: ["2025"]
      annotations: [cagr]
    source: "Source: dummy data for illustration"
    notes:
      key_message: The market is real and accelerating.
      talking_points:
        - Volume quadrupled in four years.
        - Two markets drive most of it, and we start there.
      data_callouts: ["4x since 2021", "41% CAGR", "45% Thailand"]
      transition: So who is already winning this market?
      questions:
        - q: Is growth subsidy-driven?
          a: Partly. Slide 14 shows the scenario without subsidies.
      duration_sec: 90
```

### Writing your own plan

Only three fields are required: `deck.title`, and `layout` plus `action_title` on each slide. This is a valid plan:

```yaml
deck: { title: Q3 business review }
slides:
  - { layout: title, action_title: Q3 business review }
  - layout: chart-with-takeaways
    action_title: Revenue grew 12%, driven by three enterprise deals
    chart:
      type: column_highlight
      data: { categories: [Q1, Q2, Q3], series: { Revenue: [10.0, 11.0, 12.3] } }
      detail: simple
```

When a user hands over a partial plan, the AI checks it, fills the gaps (drafts the missing notes, asks the missing detail questions) and shows the result before building.

### Checking a plan

In step 7 the AI runs this checklist, which `slide-plan-format.md` will spell out in full:

- `deck.title` is set, and every slide has a `layout` and an `action_title`.
- Every layout exists in `slide-layouts.md`; every `section` exists in `sections`.
- Action titles are full sentences of about 15 words or fewer, and reading only the titles tells the story.
- Every `chart.type` has a recipe in `references/charts/`, and its `data` matches that recipe's data shape.
- No chart still has `detail: ask`.
- Every slide has notes, and every slide that shows numbers has a `source`.

### Layout names

`title` · `agenda` · `section-divider` · `executive-summary` · `chart-full` · `chart-with-takeaways` · `two-charts` · `chart-and-table` · `table` · `matrix-2x2` · `timeline` · `text-columns` · `recommendation` · `next-steps` · `appendix` · `closing`

`slide-layouts.md` gives each layout's regions with exact coordinates, so every deck places things identically.

## 7. Chart detail levels

`detail` decides how much each chart shows. Each recipe says what this means for its chart; the general rule is:

| Element | `simple`: for presenting live | `detailed`: for pre-reads and the appendix |
| --- | --- | --- |
| Data labels | Key points only: highlighted, first/last, maximum | Every point |
| Number precision | Rounded (12%, 3.4B) | One more decimal (12.4%, 3.42B) |
| Value axis | Hidden; the labels carry the numbers | Shown, with unit |
| Gridlines | Off | Light, major only |
| Legend | Off; series named by direct labels | On when there are 2 or more series |
| Annotations (CAGR, deltas, benchmark lines) | Only the one supporting the title | All requested |
| Axis titles and footnotes | Off | On |

How the skill asks (step 6): it lists every chart slide with its title and chart type in one message. The user can answer per chart or say "all simple" / "all detailed". The answers are written into each chart's `detail` field.

## 8. Requirement map

| # | Requirement | Where it lives now |
| --- | --- | --- |
| 1 | Make PowerPoint files and add items, especially charts | The AI builds with python-pptx, following `python-pptx-guide.md` and the chart recipes |
| 2 | A slide plan before building, with each slide's content and charts | `assets/slide-plans/`, `slide-plan-format.md`; the AI shows the storyboard in step 5 |
| 3 | Notes on every slide | The plan's `notes:` block, `speaker-notes.md`, the plan checklist, `qa-checklist.md` |
| 4 | A clear explanation of every chart | 30 `.md` recipes in `references/charts/`, each with a tested python-pptx example |
| 5 | Common business data analytics | `analytics-catalog.md` (what to compute) and `analytics-recipes.md` (how, with pandas) |
| 6 | The skill uses its references | Each workflow step names the reference to read ([section 3](#3-workflow)) |
| 7 | Customisable palette, fonts and label detail | `color-palette.md` and `typography.md` presets, `theme_overrides` in the plan, `detail` per chart |
| 8 | Ask whether each chart should be detailed | `detail: ask` default, step 6, and the plan checklist |
| 9 | Users can write their own plan | `blank.plan.yaml`, the schema (editor autocomplete), `slide-plan-format.md`; only 3 required fields |

## 9. Naming conventions

- **One chart id everywhere.** The recipe `charts/05_waterfall.md`, the preview `charts/previews/05_waterfall.png` and the plan's `type: waterfall` all come from the same name; the id is that name without its number.
- **Docs use `kebab-case`** (`slide-plan-format.md`). **Plan files end in `.plan.yaml`.** **Sample data uses `snake_case.csv`.**
- **Generated output goes in `deck-output/<deck-name>/`**, never in the skill folder.

## 10. Suggested writing order

Each step lets you test something.

1. **`SKILL.md`**: the `description`, workflow and rules. The skill activates from this point.
2. **The exact numbers:** `color-palette.md`, `typography.md` and `slide-layouts.md`.
3. **`python-pptx-guide.md`**: the build order and building blocks. Test it by asking the AI to build a 3-slide deck from it.
4. **`slide-plan-format.md`**, `blank.plan.yaml` and the schema.
5. **The 11 tier-A chart recipes.** Test each code example once before trusting it.
6. **`storytelling.md`, `speaker-notes.md`, `chart-detail-levels.md`, `qa-checklist.md`.**
7. **`example.plan.yaml` and `sample-data/`**, then do a full run of the skill end to end.
8. **The tier-B recipes**, plus `analytics-catalog.md` and `analytics-recipes.md`.
9. **The tier-C and tier-D recipes.** Test these carefully; they're the easiest to get wrong.
10. **The four storylines.**
