# PROMPT FOR CLAUDE FABLE — BOJACK HORSEMAN DATA ANALYST PORTFOLIO

Build a single-file `index.html` (all CSS and JS inline) that is a fully animated, graphically rich, scroll-driven personal portfolio for **Somya Vishnoi** — a data analyst and B.Tech CSE student. The visual language is the **Bojack Horseman animated TV show**: flat vector illustration, bold outlines, saturated color fields, slightly surreal suburban-California meets data world. No gradients that feel digital. Everything feels hand-drawn but crisp. The horse IS Bojack — same dead eyes, same slouch, same existential dread — but instead of actor, he's a data analyst. He carries a laptop. He drinks from a mug that says "NULL". He mutters things.

---

## TECH STACK — MANDATORY

- **Three.js r128** via CDN (`https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js`)
- **GSAP 3** + **ScrollTrigger** plugin via CDN
- **Canvas API** for the horse character (drawn programmatically, animated via `requestAnimationFrame`)
- **Intersection Observer API** for section reveals
- Zero external images. Everything drawn in code (SVG inline or Canvas).
- One HTML file. CSS in `<style>`. JS in `<script>`. Nothing external except CDN libs.

---

## COLOR PALETTE — EXACT HEX

```
Background sky day:    #F5C842  (Bojack-universe warm yellow)
Background sky dusk:   #E8724A  (burnt orange)
Background sky night:  #1A1A3E  (dark indigo)
Ground/grass:          #4CAF50 → #2D5016 (flat, no gradient within shapes)
Road:                  #4A4A4A
Building fill:         #C0392B, #2980B9, #8E44AD (flat blocks)
Outline stroke:        #1A1A1A (thick, 3-4px feel)
Accent yellow:         #F39C12
Accent teal:           #1ABC9C
Horse skin:            #C8860A (Bojack tan)
Horse mane:            #6B2D8B (Bojack purple tones)
Horse shirt:           #2C3E50 (dark casual)
Text primary:          #1A1A1A
Text on dark:          #F5F5F5
Card bg:               #FFFDF0
```

---

## THE HORSE CHARACTER — ALWAYS ON SCREEN

The horse (`HorseCanvas`) sits **fixed bottom-left**, `width: 220px height: 260px`, and is ALWAYS visible. He is drawn on a `<canvas>` element using 2D context. He must be animated constantly — not static for more than 0.5 seconds.

### Drawing instructions (approximate — make him look like Bojack):

**Body**: Large oval torso, hunched shoulders (suggest defeat). Tan/brown fill (`#C8860A`), thick black outline stroke `3px`. 
**Head**: Large horse head, elongated snout, lidded eyes with heavy bags underneath (draw the bags — 2px dark arc below each eye). Ears pointed. Mane flopped to one side (purple-ish `#6B2D8B`).
**Shirt**: Untucked button-down, dark (`#2C3E50`), one button undone. 
**Laptop**: He holds a laptop (flat rectangle `#2C3E50`) with a tiny glowing screen (`#1ABC9C` rectangle inside).
**Mug**: In other hand, mug saying `NULL` in tiny white text.

### Idle animations (loop these):

1. **Breathing**: Torso slightly scales Y 1.0 → 1.02 → 1.0, period 2.5s.
2. **Ear flick**: Left ear rotates ±5° randomly every 3-7 seconds.
3. **Eye blink**: Every 4-6 seconds, eyes close (fill eyes with skin color) for 150ms.
4. **Typing fingers**: On the laptop keyboard, tiny finger-shaped rectangles tap in sequence at 120bpm.
5. **Mug sip**: Every 8-12 seconds, mug raises to mouth and lowers. During sip, horse's expression changes to mild disgust (eyebrows furrow, mouth curves down).
6. **Head bob**: Subtle Y oscillation ±3px, period 3s.

### Scroll-triggered behaviors:

- **Scrolling down fast**: Horse looks left, expression shifts to "ugh, another section"
- **On PROJECTS section**: Horse looks at laptop screen intently. Screen glows brighter.
- **On SKILLS section**: Horse raises one eyebrow skeptically (one eyebrow lifts, one stays).
- **On ACHIEVEMENTS section**: Horse does a tiny, resigned half-shrug. No smile. 
- **On CONTACT section**: Horse looks at phone. Puts it down. Picks it up again. Puts it down.

### Speech bubble (popup, not permanent):

Horse occasionally mutters. CSS tooltip-style bubble above his head. Appears for 3s then fades. Lines rotate randomly every 15-25s:

```
"This data cleaned itself... JK, it never does."
"Oh look, another null value. Great."
"Six projects. Six existential crises."
"Power BI? More like Power... BI... whatever."
"I should've been a novelist."
"The model's F1 is 0.88. My happiness is 0.12."
"Someone's reading my portfolio. Don't get excited."
"LPU. It's fine. Everything is fine."
"printf('help');"
```

---

## SCROLL-DRIVEN LANDSCAPE (THE MAIN THEATRICAL TRICK)

The **background is a continuous parallax world** that scrolls as the user scrolls. It is drawn on a `<canvas id="worldCanvas">` that fills the entire viewport, `position: fixed, z-index: 0`. Everything else has `position: relative, z-index: 1+`.

The world is split into **5 biomes** that smoothly transition as sections come into view:

| Section | Biome | Time of Day | What's in Background |
|---|---|---|---|
| HERO | Suburban California street | Day/Golden hour | Hills, palm trees (flat triangles), highway, Hollywoo sign parody |
| ABOUT | Interior of an apartment | Night | Window with city lights, overflowing bookshelf, data charts on wall |
| SKILLS | Data center / server room | Blue-tinted night | Racks of servers, blinking LEDs, cable spaghetti |
| PROJECTS | Construction site + city | Dusk/orange | Buildings being built (scaffolding), cranes, progress bars as windows |
| CONTACT | Rooftop at night | Deep night | City panorama, stars, neon signs |

**Transitions**: Biomes cross-fade with 40% overlap. Use `lerp()` on the canvas draw parameters, driven by `window.scrollY / totalScrollHeight`. Draw 3 parallax layers per biome (far/mid/near) moving at 0.1x, 0.3x, 0.5x scroll speed.

**Background is NEVER static**: Add these perpetual animations to whichever biome is active:
- Clouds drift left (reset when offscreen)
- Leaves/particles drift down
- Stars twinkle (opacity oscillates 0.4 → 1.0)
- LED lights in server room blink on staggered timers
- City windows light up/turn off randomly

---

## SECTIONS — CONTENT AND ANIMATION SPEC

### 1. HERO SECTION

**Height**: 100vh. Background: Biome 1.

**Elements** (all scroll-in with GSAP, but hero elements animate in on `DOMContentLoaded`):

```
[Big bold title] 
SOMYA VISHNOI
→ Font: 'Georgia' or serif, 96px, weight 900, color #1A1A1A
→ Animate in: letters stagger-drop from Y:-60px, opacity 0 → 1, 0.05s delay between chars

[Subtitle — typed effect]
Data Analyst · Problem Architect · B.Tech CSE @ LPU
→ Use a typewriter animation, cursor blinks after

[Three floating stat bubbles — CSS 3D cards, float animation loop]
  🏆 6 Projects  |  📄 Research Paper  |  🎤 TED-Ed Talk

[CTA Buttons — flat Bojack-style, thick border]
  [View Projects]  [Download CV]  [GitHub]
  → Hover: translate Y:-3px, box-shadow deepens
  
[Scroll indicator]
  Horse silhouette pointing down + "scroll, dude" text
```

---

### 2. ABOUT SECTION

**Height**: 100vh. Background: Biome 2 (apartment interior).

Two-column layout: Left = text, Right = "data portrait" (abstract bar chart shaped like a human face, drawn in SVG/Canvas).

**Left column text** (reveal via Intersection Observer, slide from left):

```
ABOUT ME

First-year B.Tech CSE (Data Science) student at Lovely Professional University.
I build things that turn messy data into decisions.
Co-authored a research paper on non-generative AI and semantic classification.
I also go by Gokul. I run a Hinglish zine called Burger & Brownies.
Currently: learning, shipping, and trying to keep my CGPA above 6.4.
```

**Right column**: An animated "data portrait" — a face made of bar charts. Eyes are scatter plots. Mouth is a line chart trending up. Nose is a histogram. It animates via `requestAnimationFrame` — bars grow, scatter dots pulse.

**Floating detail cards** (appear on scroll, stagger-in):
```
📍 Phagwara, Punjab (via Bhilwara, Rajasthan)
🎓 CGPA: 6.40 (and climbing)
💻 Stack: Python · JS · SQL · Power BI
📧 somyavishnoi32@gmail.com
```

---

### 3. SKILLS SECTION

**Height**: 120vh. Background: Biome 3 (server room).

**Layout**: Skills float in as 3D CSS cards (`transform-style: preserve-3d`, `perspective: 800px`). On hover, cards flip to show proficiency bar + a short sarcastic description in Bojack's voice.

**Skills data**:
```js
const skills = [
  { name: "Python", level: 88, bojack: "It gets the job done. Like me. Barely." },
  { name: "Power BI", level: 82, bojack: "DAX is just Excel's emo phase." },
  { name: "SQL", level: 80, bojack: "SELECT sanity FROM life WHERE 1=0;" },
  { name: "Pandas", level: 85, bojack: "Not the animal. Though both are lazy." },
  { name: "Scikit-learn", level: 78, bojack: "F1 score: my personality has none." },
  { name: "Machine Learning", level: 75, bojack: "Fit the model, not your expectations." },
  { name: "FastAPI", level: 72, bojack: "Fast. Unlike my motivation Mondays." },
  { name: "React / Next.js", level: 70, bojack: "Components of my unraveling." },
  { name: "PostgreSQL", level: 75, bojack: "Relations. I understand tables, not people." },
  { name: "LightGBM", level: 68, bojack: "Light. My existence is anything but." },
  { name: "ETL Pipelines", level: 74, bojack: "Extract. Transform. Load. Repeat. Forever." },
  { name: "C++ / Java", level: 65, bojack: "Memory management for when you forget things." },
]
```

**Visual**: Cards arranged in a 3D grid, slightly rotated (X: -5deg, Y: varies). They float with subtle sinusoidal Y animation. On hover → flip 180deg Y axis → back shows proficiency fill bar animating from 0% to N%.

**Section header**: `<SKILLS />` in monospace, blinking cursor. Background server LEDs blink in rhythm.

---

### 4. PROJECTS SECTION

**Height**: auto (at least 200vh). Background: Biome 4 (construction/city at dusk).

**Layout**: Projects stack vertically, each is a large "billboard" card that slides in from alternating sides (left/right) on scroll. Each billboard is styled like an outdoor ad — thick border, worn corners (CSS border-radius hack), and the project info inside.

**Projects data**:

```js
const projects = [
  {
    name: "WhenTho",
    subtitle: "AI Revenue Recovery Model",
    date: "Sept 2026",
    stack: ["Python", "FastAPI", "LightGBM", "React", "Razorpay API", "Gemini API"],
    bullets: [
      "LightGBM classifier predicting invoice payment risk — 0.88+ weighted F1",
      "Automated Razorpay UPI recovery workflows with Gemini 2.5 tone-calibrated follow-ups",
      "30-day risk projection timeline with Promise-to-Pay tracking",
    ],
    github: "https://github.com/Somya-Vishnoi/WhenTho.git",
    live: "https://whentho-tawny.vercel.app/",
    bojack: "Predicting who'll pay their bills. Spoiler: nobody does.",
    color: "#E74C3C"
  },
  {
    name: "Trip-Split",
    subtitle: "Group Travel Expense Tracker",
    date: "Jun–Jul 2026",
    stack: ["Next.js 14", "TypeScript", "Tailwind", "Supabase", "PostgreSQL", "FastAPI"],
    bullets: [
      "Knapsack DP to optimize hotel/restaurant/activity selection via Overpass API",
      "Debt-resolution algorithm minimizing total transactions across group members",
      "FastAPI backend with Gemini API fallback for venue scoring",
    ],
    github: "https://github.com/Somya-Vishnoi/Trip-Split.git",
    live: "https://trip-split-xi-drab.vercel.app/",
    bojack: "Fair cost splitting. Nothing in life is fair. But the algorithm is.",
    color: "#3498DB"
  },
  {
    name: "GigShift",
    subtitle: "B2B2C Gig-Economy Dispatch Platform",
    date: "Apr–May 2026",
    stack: ["Next.js", "Role-Based Access", "BFS/Dijkstra", "Priority Queue", "Greedy Algorithms"],
    bullets: [
      "Dispatch platform for last-mile delivery riders across India",
      "3 distinct Next.js apps for riders, operators, and ops teams — 30% engagement increase",
      "10,000-row synthetic dataset with surge patterns (peaks, weather, events)",
    ],
    github: "https://github.com/Somya-Vishnoi/Gig-Shift-v4.git",
    live: "https://gig-shift-v4.vercel.app/",
    bojack: "Moving packages. I can't even move on.",
    color: "#9B59B6"
  },
  {
    name: "Crop Production Analysis",
    subtitle: "Agricultural Data Analysis — India",
    date: "Mar–Apr 2026",
    stack: ["Power BI", "EDA", "ETL", "DAX", "Data Visualization"],
    bullets: [
      "Two decades of agricultural data — crop production trends across Indian states",
      "Interactive Power BI dashboard with slicers, KPIs, and yield metrics",
      "ETL pipeline from data.gov.in with DAX-powered seasonal contribution measures",
    ],
    github: null,
    live: null,
    bojack: "Farming data. The crops grow. I do not.",
    color: "#27AE60"
  },
  {
    name: "AQI Analysis & Prediction",
    subtitle: "Air Quality ML Model",
    date: "Mar–Apr 2026",
    stack: ["Python", "Pandas", "Scikit-learn", "Matplotlib", "Linear Regression"],
    bullets: [
      "End-to-end EDA on India's AQI dataset — pollution patterns across cities and seasons",
      "Full data cleaning + validation pipeline across pollutant features",
      "Linear Regression model for AQI prediction + feature correlation analysis",
    ],
    github: "https://github.com/Somya-Vishnoi/AQI-Analysis-And-Prediction.git",
    live: null,
    bojack: "The air is bad. Much like the general vibes.",
    color: "#F39C12"
  },
  {
    name: "IPC Debugger",
    subtitle: "GUI-Based Inter-Process Communication Tool",
    date: "Dec 2025",
    stack: ["Python", "GUI Development", "Systems Programming", "IPC"],
    bullets: [
      "GUI-based IPC debugger for system-level troubleshooting",
      "Concurrent monitoring of pipes, message queues, and shared memory",
      "Live channel state visualization with dynamic attach/detach",
    ],
    github: "https://github.com/Somya-Vishnoi/IPC-DEBUGGER.git",
    live: null,
    bojack: "Debugging processes. Still can't debug myself.",
    color: "#1ABC9C"
  }
]
```

**Animation**: Each project card triggers on Intersection Observer. Slides in from the side, slight 3D tilt (rotateY: 8deg → 0deg). Stack pills animate in one by one after the card. Bojack quote fades in last.

**Progress bars** in background buildings light up as you scroll past projects (windows illuminate in the building silhouettes, suggesting completion/data flowing).

---

### 5. ACHIEVEMENTS & CERTS SECTION

**Height**: auto. Background: transition toward Biome 5.

**Layout**: Timeline — vertical dashed line down the center, cards alternate left/right. Cards animate in with a "stamp" effect (scale 1.2 → 1.0, rotation ±3deg → 0).

**Achievements**:
```
🔬 Research Paper (Under Review) — Apr 2026
   "Non-Generative Cognitive Tracking: Temporal Decay Modelling and Semantic 
    State Classification for Real-Time Idea Analysis" — 82.6% classification accuracy

🎤 TED-Ed Talk — Sep 2022
   "Healthy Criticism" — student talk on constructive feedback frameworks
```

**Certifications** (render as stacked cards, fanned out like playing cards, click to spread):
```
Oracle — AI Database Certified Foundations Associate (Aug 2026)
Oracle — Agentic AI Certified Foundations Associate (Aug 2026)
Infosys — C++ Programming Language (Jul 2026)
Infosys — Database Management System (Jul 2026)
Neocolab — Programming in Java (May 2026)
Neocolab — Data Structure and Algorithm (Jan 2026)
Neocolab — Object Oriented Programming (Jan 2026)
Neocolab — C Programming (May 2025)
Skillera — Introduction to AI & ML (Apr 2025)
Skillera — Introduction to DSA (Apr 2025)
```

---

### 6. CONTACT SECTION

**Height**: 100vh. Background: Biome 5 (rooftop night).

Center-aligned. Large neon-sign aesthetic for the heading: `LET'S TALK DATA`. Neon flickering animation (opacity alternates slightly, CSS `@keyframes neon-flicker`).

**Links block** (large, clickable cards, hover → glow):
```
📧 somyavishnoi32@gmail.com
🐙 github.com/Somya-Vishnoi
💼 linkedin.com/in/somya-vishnoi
```

**Footer**: Small text — `© 2026 Somya Vishnoi · Made with existential dread and Python`

Horse in the corner is at his most animated here — looking at the city, taking a long sip from the NULL mug.

---

## GLOBAL UI REQUIREMENTS

### Navigation

Fixed top nav, semi-transparent dark (`rgba(26,26,62,0.85)`), backdrop-blur. Links: `HOME · ABOUT · SKILLS · PROJECTS · CONTACT`. On scroll, nav condenses slightly (height shrinks via GSAP). Active section highlighted.

### Typography

- Display/headlines: `Georgia, serif` — big, confident
- Body: `'Courier New', monospace` — data-analyst nerd energy
- UI labels: `system-ui, sans-serif`

### Scroll Behavior

`scroll-behavior: smooth`. Every section triggers via `ScrollTrigger` pinning, not just CSS.

**Scroll progress bar**: Thin `3px` line at top of page, fills left-to-right as user scrolls. Color: `#F39C12`.

### Responsive

Desktop-first. At `<768px`, horse scales to `140px`, font sizes drop, grid switches to single column. Background biomes still animate.

### Performance

- Cap world canvas FPS to 60 using `requestAnimationFrame` with timestamp delta check.
- Biome layers drawn off-screen then composited.
- Horse canvas is separate, never redraws the world canvas.

---

## SPECIAL EFFECTS

1. **Cursor trail**: Custom cursor leaves brief orange trail (`#F39C12`) that fades in 500ms.
2. **Section entrance flash**: Each section briefly flashes a 1px border pulse (like a camera shutter) when entering viewport.
3. **Data particles**: Floating `01` binary particles drift across the hero section slowly.
4. **Hover on project cards**: 3D tilt follows mouse position (use `mousemove` → `rotateX/Y` via CSS transform).
5. **Horse shadow**: Dynamic shadow beneath horse that scales with breathing animation.

---

## OUTPUT

A single `index.html` file. Fully functional when opened locally. No build steps. No missing assets. Comments in code marking each section clearly. Total file should be playable as-is.

Begin with the HTML boilerplate, then implement in order:
1. CSS (all styles)
2. World canvas + biome system
3. Horse canvas + animations
4. Section content + GSAP scroll triggers
5. Special effects
6. Mobile responsive overrides

Do not truncate. Do not add placeholders. Do not say "you can add more later." Build the whole thing.
