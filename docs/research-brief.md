# VELOX landing page brief

Source: `http://localhost:3100/` (Next.js 14 dev server for GitHub `alagappan567/car`, code at `F:\car-source`).
Local: `/` → `src/app/page.tsx` → `src/components/sites/velox/`.

## Route map

| Source | Local | Notes |
|---|---|---|
| `/` | `/` | `velox-landing.tsx` |
| `/car` | `/car` | `car/` (GT-X): hero + Performance, Design, Technology, Specs, CTA, footer |
| `/moto` | `/moto` | `moto/` (RS1): hero + Performance, Engineering, Heritage, CTA, footer |
| `/legend` | `/legend` | `legend/` (LX1): hero + Performance, Design, CTA, footer |

The shared model-page parts are in `shared/`:
- `use-frame-sequence.ts`: GSAP ScrollTrigger scrub over a canvas image sequence.
- `model-hero.tsx`: the hero, built from a per-model config.
- `reveal.tsx` and `count-up.tsx`: the section entrance and counting effects.
- `section-background.tsx`: the paper-design `MeshGradient` background.
- `model-footer.tsx`.

## Model pages

- **Hero:** a 100vh + 6000px scroll container with a sticky 100vh panel. All frames are preloaded behind a full-screen loader, then decoded to ImageBitmaps. Scroll scrubs through the frames (`scrub: 0.5`) on a canvas sized at up to 2× DPR. Vignettes, model counter, scroll hint, left progress bar and right info panel sit on top.
  - GT-X uses frames 002–216 (215 frames).
  - RS1 and LX1 use frames 001–240.
  - RS1 also has 8 animated red speed lines. Its source sets `animationDelay` before the `animation` shorthand, so every line runs without a delay.
- **Section backgrounds:** `@paper-design/shaders-react@0.0.76` `MeshGradient` with the "black" palette. Speeds range from 0.2 to 0.4 depending on the section.
- **Dependencies:** pinned to the source's installed versions, `gsap@3.15.0` and `@paper-design/shaders-react@0.0.76`.

## More Tailwind v3 quirks reproduced

- `duration-400` is not a v3 class, so `transition-all` keeps its 150ms default. The clone uses `duration-150`.
- `text-white/28` is not a v3 opacity step, so RS1 Engineering body copy renders solid white. The clone keeps it white.
- v3 palette hex values (`red-*`, `purple-*`, `violet-300`) and the custom `gold` colors are redefined in `globals.css` `@theme`.
- Gradients use `bg-linear-*/srgb` to match v3's sRGB interpolation.

## Sections (top to bottom)

1. **Fixed header**: VELOX wordmark (display serif, light, 0.4em/0.5em tracking), three model links (11px desktop / 10px mobile), "2024 Collection" (desktop only).
2. **Hero** (min-h-screen): full-bleed WebGL2 shader canvas with fbm clouds and light streaks, half-DPR, time-driven. Eyebrow 9px/0.65em; H1 `clamp(2rem,12vw,9rem)`, tracking `clamp(0.2em,3vw,0.7em)`, weight 100 (renders as the lightest loaded weight); a 96px gradient rule; italic serif "Choose Your Machine"; a three-cell fleet selector (672px wide at desktop, 320px max on mobile); "03 / 03" counter at bottom 28% (desktop only).
3. **Product grid**: three cards, 1-col below 768px (55vh) and 3-col at desktop (100vh), separated by 1px gaps. Hover: background scales to 1.06 over 1s, accent bloom, inset 1px accent border, accent rule grows 24→56px and CTA rule 16→40px.
4. **Footer**: wordmark and copyright, `bg-black/80`, top border white/5%.

## Measurements (1440×900)

H1 box: 730×144 on the source (Arial fallback) and 677×144 on the clone (Inter). Selector at y=543/547. Grid starts at y=900. Document height is 1879 on both. At 390px both are 2298 tall.

## Assets

| Source | Local |
|---|---|
| `F:\car-source\public\frames\ezgif-frame-002.jpg` | `public/sites/velox/gt-x.jpg` (1920×1080) |
| `F:\car-source\public\moto-frames\ezgif-frame-001.jpg` | `public/sites/velox/rs1.jpg` |
| `F:\car-source\public\legend-frames\ezgif-frame-001.jpg` | `public/sites/velox/lx1.jpg` |
| `F:\car-source\publicrames\ezgif-frame-002…216.jpg` | `public/sites/velox/frames/gt-x/` (215 frames, 12 MB) |
| `F:\car-source\public\moto-frames\ezgif-frame-001…240.jpg` | `public/sites/velox/frames/rs1/` (240 frames, 14 MB) |
| `F:\car-source\public\legend-frames\ezgif-frame-001…240.jpg` | `public/sites/velox/frames/lx1/` (240 frames, 14 MB) |

## Fonts (known deviation)

The source declares `next/font/google` Inter (300–600) and Cormorant Garamond (300–600, normal and italic). Its dev server failed to fetch them, so localhost:3100 renders the generated fallbacks: Arial at 107% size-adjust and Times New Roman at 97%. The clone loads the declared fonts. Geometry still matches; glyph shapes and widths differ. Mono text uses the system mono stack (Consolas on Windows) on both.

## Tailwind v3 → v4 notes

- On the source, `md:w-xs` on the hero rule does nothing in v3, so the rule is 96px at every width. In v4 it would be 320px, so the clone omits it.
- `bg-gradient-to-*` is written as `bg-linear-to-*` in v4.

Screenshots: `docs/design-references/velox/` (`source-*` / `clone-*`, at 1440×900 and 390×844).

## Customizations beyond the source

- **Smooth scrolling (requested after the 1:1 pass):** `shared/smooth-scroll.tsx`, mounted in `src/app/layout.tsx`.
  - Runs Lenis `1.3.27` (`lerp: 0.08`, anchor links enabled) on GSAP's ticker and feeds every update to `ScrollTrigger.update`, so the hero frame sequences scrub with the eased position.
  - The source uses native scrolling with CSS `scroll-behavior: smooth`. The clone removes that rule so it doesn't conflict with Lenis.
  - Lenis respects `prefers-reduced-motion`.
- **Scroll performance:** measured on Intel UHD with wheel scrolling. On `/car`, scrolling went from 4 frames in 4s to 60fps.
  - Section mesh gradients pause offscreen and render at ≤0.46 MP. The library default is a 2× pixel ratio, up to 8.3 MP per section, always animating.
  - The homepage light-streak shader pauses offscreen. On slow GPUs it steps its render scale down to 0.55, then draws every other frame.
  - The hero frame scrub uses `scrub: true`, since Lenis already eases the scroll.
  - Decoded frame bitmaps are freed when leaving a model page.
- **Working controls (requested; the source's buttons and `#` links do nothing):**
  - **Enquiry dialog:** `shared/enquiry.tsx`, built on Base UI Dialog. The hero Configure/Reserve buttons, the CTA buttons, the "Race" and LX1 "Configure" nav items, and the dealer/service/configure footer entries open it with the request type and topic prefilled. The site has no backend, so on submit it says nothing was sent and offers a prefilled email to the placeholder `ENQUIRY_EMAIL` (`enquiries@velox.example`). Replace it before launch. Smooth scrolling pauses while the dialog is open.
  - **Favourite ♡:** toggles to ♥ and is remembered per model in `localStorage`.
  - **Nav links:**
    - GT-X "Models" → `/`.
    - LX1 "Heritage" → `/`.
    - The others → their sections.
  - **Footer links:**
    - Entries with a page or section go there; LX1's heritage entries go to `/moto#heritage`.
    - Legal entries are plain text, since no legal pages exist.
    - The GT-X social chips are decorative, since no real accounts exist.
