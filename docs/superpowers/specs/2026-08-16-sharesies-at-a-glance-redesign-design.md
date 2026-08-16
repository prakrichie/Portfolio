# Sharesies "At a glance" — stepper redesign

**Date:** 2026-08-16
**Scope:** `Sharesies.html` only
**Reference:** `redesign.jpg` (wireframe)

## Problem

The at-a-glance panel currently renders as five clickable STAR rows in a left
column beside a process-flow panel on the right. Two things pushed a redesign:

1. The five items have grown very unequal in length. "What I did" is now ~1,800
   characters against ~200 for "Problem Statement". All five render in full,
   simultaneously, in a narrow column, so the panel is dominated by one row.
2. The panel carries no imagery at all, in a portfolio whose subject is visual
   design work.

The wireframe answers both: show one item at a time, give each a visual.

## Design

### Shape

A five-step walkthrough. One step visible at a time.

```
┌─ ● 04  ~/at-a-glance         4 months · team of 4 ─┐
│                                                     │
│              [ image for step 3 ]          ‹  NEXT ›│
│                                                     │
├─────────────────────────────────────────────────────┤
│  ① ② ❸ ④ ⑤                                          │
│                                                     │
│  CHALLENGES                                         │
│  We needed to somehow balance education and         │
│  simplicity. New investors needed enough...         │
│                                                     │
│  PHASE · IDEATE → PROTOTYPE → TEST                  │
│  Concepting, wireframing, gamification. Lo-fi,      │
│  design system, hi-fi. Usability testing.           │
│                                                     │
│  user research · usability testing · affinity...    │
└─────────────────────────────────────────────────────┘
```

Regions, top to bottom:

- **Chrome bar** — retained from the current component. Not in the wireframe,
  kept deliberately: it is the visual signature shared with the homepage work
  cards and the Tribes/Budding case studies. Carries the `04` index, the
  `~/at-a-glance` path, and the `4 months · team of 4` meta.
- **Media area** — fixed 16:9, one image per step. Prev and NEXT controls sit
  top-right, inside the media area, per the wireframe.
- **Step markers** — `① ② ③ ④ ⑤`, clickable for direct access to any step.
- **Text pair 1** — the step's title and body (the existing `name` and `body`).
- **Text pair 2** — the step's process phase. This is where the current
  right-hand flow-band content goes; it changes as the reader steps.
- **Tag strip** — the existing skill tags, as a quiet footer.

The wireframe's two title/paragraph pairs map to *step content* and *process
phase* respectively. Nothing from the current component is discarded; the flow
bands and tags are relocated rather than dropped.

### Where pair 2 comes from

The current `bands` config is keyed `task` / `action` / `result` only. Steps 1
(`situation`) and 5 (`learning`) have no matching band, so deriving pair 2 from
`bands` would leave two of five steps with an empty block.

Pair 2 is therefore authored **explicitly per step** in the config, replacing
`bands` entirely:

| # | Step | Pair 2 title | Pair 2 body |
|---|------|--------------|-------------|
| 1 | Problem Statement | `THE BRIEF` | Four-month capstone embedded with the Sharesies in-house design team. |
| 2 | Role | `PHASE · DISCOVER → DEFINE` | 9 user interviews, onboarding research. Affinity mapping. |
| 3 | Challenges | `PHASE · DEFINE` | Reframing the brief after finding an existing onboarding flow. |
| 4 | What I did | `PHASE · IDEATE → PROTOTYPE → TEST` | Concepting, wireframing, gamification. Lo-fi, design system, hi-fi. Usability testing. |
| 5 | Key learning | `PHASE · DELIVER` | Team presentation. Research-led pivot over an attachment to the first solution. |

Every step gets a filled pair 2. The existing `phase` field (`'2 of 6 phases'`)
is retained on each step and rendered as a small counter beside the step markers.

### Aspect ratio handling

The available assets vary from 2:1 landscape to 0.46:1 portrait. The media box
is **fixed at 16:9 with `object-fit: contain`** over a `--bg-2` backdrop.

Rejected: letting the box resize per step. It makes the whole page below the
component jump on every NEXT click, which is worse than letterboxing a portrait
image.

### Step → image mapping

| # | Step | Image | Size | Ratio |
|---|------|-------|------|-------|
| 1 | Problem Statement | `Sharesies_hs1.jpg` | 5.2M | — |
| 2 | Role | `Icon_assets/workshop_concept.svg` | 2.4M | 1.7:1 |
| 3 | Challenges | `SVGS/q_map.svg` | 100K | 0.46:1 ⚠ portrait |
| 4 | What I did | `SVGS/sha_lofi_p.svg` | 1.3M | 1.2:1 |
| 5 | Key learning | `SHA_Screenshot/SHA_DD.svg` | 80K | 2:1 |

Rationale: step 2 shows the Crazy 8s / workshop concepting the Role text
describes; step 3 shows the questionnaire map, the concept that testing revealed
as friction; step 4 the lo-fi prototype; step 5 the double diamond, matching
"treat design as an iterative process".

`sha_workshopimages/ws1.jpg` was the strongest content match for step 2 but is
8.8MB. Rejected on weight in favour of the 2.4MB workshop concept asset.

### Intro hero

`Sharesies_hs1.jpg` is **removed from its current position** above the panel and
becomes the step 1 visual. Otherwise two large images stack before any text.

This makes first paint weight **neutral against today**: the same 5.2MB image
loads, just inside the component. Steps 2–5 are `loading="lazy"` and fetch only
when reached.

### Interaction and accessibility

Preserved from the current implementation, not rebuilt:

- Step markers are `role="tab"`; the media + text region is the `role="tabpanel"`.
- Left/right arrow keys move between steps; roving `tabindex` (`0` on active,
  `-1` elsewhere).
- `prefers-reduced-motion: reduce` disables the step transitions.
- `:focus-visible` outlines on all controls.

Prev is disabled on step 1, NEXT on step 5. No wraparound: the numbered markers
make jumping back a single click, so silent looping would be more confusing than
a disabled control.

## Blast radius

All within `Sharesies.html`:

| What | Approx. location |
|------|------------------|
| `AT A GLANCE` CSS block | lines 1226–1490 |
| `glanceHTML()` | line 1689 |
| `intro` section `star:` config (extended), `bands:` (removed) | lines 1753–1780 |
| `setupGlance()` | line 2663 |
| Intro hero `<img>` (removed) | line 1751 |

`design-system.css`, `Tribes.html` and `Budding.html` are untouched. Tribes and
Budding keep the current STAR-tabs component until this one is proven live.

## Out of scope

- Porting to Tribes and Budding.
- Colour and motion polish. Structure lands first; polish is a later pass.
- Compressing `Sharesies_hs1.jpg` or the workshop photos.
