# How the VELOX site works

## Landing page (`/`)

- **Hero background:** a full-screen WebGL2 fragment shader. Layered fractal-noise "clouds" with eleven light streaks, animated over time, rendered at half the device pixel ratio.
  - It pauses when scrolled offscreen.
  - On slow GPUs it measures its own frame time, steps its resolution down to 55%, then draws every other frame, so page scrolling stays at 60fps.
- **Model selector:** three tiles (GT-X, RS1, LX1), each in its own accent colour: gold `#c9a84c`, red `#e8003d`, purple `#a855f7`.
- **Model cards:** full-height cards with the first frame of each car's sequence. On hover:
  - The photo slowly zooms (1.06× over 1s).
  - An accent glow, border and underline appear.
- **Navigation:** every model link goes to that model's page.

## Model pages (`/car`, `/moto`, `/legend`)

### Scroll-driven hero

The hero is a **frame sequence**, not a video:
1. A loading screen shows progress while all 215–240 JPG frames download.
2. The frames are decoded once into `ImageBitmap`s.
3. The hero is a sticky full-screen `<canvas>` inside a container 6000px taller than the screen.
4. A GSAP ScrollTrigger maps scroll position through that container to a frame number, and the canvas draws that frame "cover"-fitted. Scrolling down plays the car animation; scrolling up rewinds it.
5. Overlays sit on top of the canvas:
   - Vignettes.
   - The model counter (`01 / 03`).
   - A progress bar that fills as you scroll.
   - The info panel with the model name and actions.
6. The decoded frames are freed when you leave the page.

The RS1 hero also has animated red speed lines.

### Sections

- **Entrance:** each section fades and slides in when it scrolls into view.
- **Count-up stats:** statistics count up from zero.
- **Backgrounds:** a slowly moving near-black mesh gradient, using `@paper-design/shaders-react`. It only animates while visible.

### Smooth scrolling

[Lenis](https://github.com/darkroomengineering/lenis) turns wheel and trackpad input into eased, inertial scrolling. It runs on GSAP's ticker and reports every position to ScrollTrigger, so the car animations scrub smoothly instead of jumping per wheel notch.
- In-page links such as **▶ EXPLORE** glide to their section.
- Visitors who prefer reduced motion get normal scrolling.

### Buttons and links

| Control | What it does |
|---|---|
| Configure / Reserve / Build & Configure / Book a Test Drive / Book a Track Day / Reserve Your LX1 | Opens the enquiry form for that model, with the request type preselected |
| ♡ | Saves the model as a favourite (♥), remembered in this browser |
| ▶ EXPLORE / RACE / DISCOVER | Glides to the page's first section |
| Nav links | Glide to that section, or go to the landing page |
| Footer entries | Go to a section or page, or open the enquiry form with that topic (e.g. "Warranty") |

**Enquiry form:** name, email, request type and an optional message, with validation.
- The site has no backend, so submitting doesn't send anything. It explains this and offers **Open email**, which drafts an email with the details.
- The address is a placeholder, `enquiries@velox.example`, to be replaced before real use.
- While the form is open, the page behind it doesn't scroll. Escape closes it.

## Stack

| Layer | Used for |
|---|---|
| Next.js 16 (App Router), React 19, TypeScript | Pages, routing, static prerendering |
| Tailwind CSS v4 | Styling, with the original's Tailwind v3 colour palette restored |
| GSAP 3.15 + ScrollTrigger | Scroll → frame mapping for the heroes |
| Lenis 1.3 | Smooth inertial scrolling |
| WebGL2 (hand-written shader) | Landing-page light streaks |
| @paper-design/shaders-react 0.0.76 | Section mesh-gradient backgrounds |
| Base UI Dialog | Accessible enquiry form |
| Inter + Cormorant Garamond (Google Fonts via `next/font`) | Typography |

## Assets in this repository

| Folder | Contents |
|---|---|
| `assets/gt-x.jpg`, `rs1.jpg`, `lx1.jpg` | Landing-page model card images (1920×1080) |
| `assets/frames/gt-x/` | 215 hero frames for the GT-X |
| `assets/frames/rs1/` | 240 hero frames for the RS1 |
| `assets/frames/lx1/` | 240 hero frames for the LX1 |
