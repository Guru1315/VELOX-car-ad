# How the VELOX site was built

## Starting point

The original VELOX site is a Next.js 14 app with four routes: a landing page and three model pages. It ran locally from its source.

The rebuild uses a reusable website-cloning template:
- Next.js 16 with the App Router and React 19.
- TypeScript in strict mode.
- Tailwind CSS v4.
- shadcn/ui on Base UI primitives.

The aim was to match the original 1:1 first, then improve it.

| Original route | Rebuilt route | Page |
|---|---|---|
| `/` | `/` | Landing page: VELOX wordmark, light-streak shader, model selector, model cards |
| `/car` | `/car` | GT-X supercar |
| `/moto` | `/moto` | RS1 moto racer |
| `/legend` | `/legend` | LX1 hypercar |

## Process

1. **Observe:** each page was inspected in a browser at desktop (1440×900) and mobile (390×844) sizes. Measured:
   - Computed fonts, letter spacing, colours.
   - Element positions and page heights.
   - Hover states.
   - Scroll-driven behaviour.

   The findings are in [`research-brief.md`](research-brief.md).
2. **Assets:** images and the frame sequences were copied from the original source, so they're byte-identical with no re-encoding:
   - The three model-card images.
   - 695 JPG frames (1920×1080) for the scroll animations.

   See [`../assets`](../assets).
3. **Build:** shared pieces came first: fonts, colour palette, smooth scrolling and the frame-sequence engine. Then each page section by section.
4. **Compare and fix:** the original and the clone were captured side by side with headless Chrome at the same viewport and scroll position. At 1440px wide, every section on every page starts at the same pixel and has the same height as the original. See [`../screenshots/comparison`](../screenshots/comparison).
5. **Check:** lint, type-check and production build (`npm run check`) pass. A scripted click test of every button and link passed 45 of 45 checks.

## Matching details that were easy to miss

The original was built with Tailwind v3. A few of its classes behave differently or don't exist in v3, and the rebuild reproduces what v3 actually rendered:
- **Divider width:** `md:w-xs` isn't a v3 class, so the hero divider stays 96px wide.
- **Transition speed:** `duration-400` isn't a v3 class, so those transitions run at the default 150ms.
- **Text opacity:** `text-white/28` isn't a valid v3 opacity, so that text renders solid white.
- **Colours:** v3 colours are sRGB hex, while v4's are oklch. The v3 values for the red, purple and gold shades are redefined so colours match exactly.
- **Gradients:** they use sRGB interpolation, like v3.

## Deliberate changes from the original

- **Fonts:** the original's local dev server failed to download Inter and Cormorant Garamond, so it showed Arial and Times. The rebuild uses the intended fonts. Layout is identical. Glyph shapes differ, and two mobile pages are 24–43px shorter from text wrapping.
- **Smooth scrolling:** Lenis inertial scrolling, synced to the scroll animations.
- **Performance:** the original drew five or six full-resolution WebGL backgrounds per page constantly. On an Intel UHD GPU, `/car` scrolled at about 4 frames in 4 seconds. Now:
  - Backgrounds pause offscreen and render at a fraction of the pixels; they're dark blurs, so they look the same.
  - The hero shader lowers its own resolution and frame rate when the GPU struggles.

  Result: about 60fps on every page.
- **Working controls:** about 50 buttons and links did nothing on the original. Now:
  - Order and booking buttons open an enquiry form.
  - The heart button saves a favourite.
  - Nav and footer links go to real sections or pages.

  See [how it works](how-it-works.md).
  
## No source code provided in this repo for the security issues
