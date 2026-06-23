# Aurora Borealis Learning Module — Handoff & Next-Module Notes

*Updated at project close, 2026-06-23. Replaces the pre-build version.*
*Use this to jump-start the next immersive learning module.*

---

## What Shipped

A 30-scene, single-file (`index.html`, ~168 KB, ~4,000 lines) cinematic learning module.
Deployed at https://aurora-jkupp.vercel.app

### Scene inventory
- **Opening** (0–5): Begin screen, photo slideshow, title card + video background, hook text, three questions, pre-assessment poll
- **Part 1 — Our Restless Sun** (6–9): animated solar wind, CME photo, 11-year cycle wave chart, drag-and-drop KC
- **Part 2 — Earth's Invisible Armor** (10–13): magnetosphere diagram, Follow-a-Particle, auroral oval map, tap-the-magnetosphere KC
- **Part 3 — Why These Colors?** (14–18): colors intro, Neon Sign analogy, altitude chart, energy/depth chart, photo-matching KC
- **Part 4 — The Rhythm** (19–23): timescale intro, Days/CME, Hours/Kp misconception, Minutes/magnetometer, True/False KC
- **Part 5 — The Chase** (24–25): seasonal/solar-cycle intro, branching "Plan Your Chase" KC
- **Final Assessment** (26–27): intro card, 4 scenario-based space-weather briefings
- **Conclusion + Credits** (28–29): 5-part visual recap with staggered badges + crossfading photo background, credits with links, "For Toko" dedication

### Knowledge-check mechanics built (all reusable)
| # | Mechanic | Scene | Notes |
|---|---|---|---|
| 1 | Drag-and-drop sort | 9 | Cards → labeled buckets. Full keyboard + pointer. |
| 2 | Tap-to-find | 13 | Tap correct spots on SVG. Keyboard-accessible. |
| 3 | Photo matching | 18 | Drag labels onto photos. Same DnD engine as #1. |
| 4 | True/False | 23 | Sequential statements with reveal. |
| 5 | Branching path | 25 | Location × season decision tree. |
| 6 | Scenario assessment | 27 | Briefing card + Yes/No + feedback reveal. |

---

## The Most Important Architectural Insight for Next Time: Separate Engine from Content

The code splits cleanly into two things:

**Engine** (reusable across any topic): scene manager, nav bar, CSS transitions, KC mechanics, accessible DnD, deferred-asset loading, video lifecycle management

**Content** (topic-specific): scene markup, text, image/video paths, KC question data, part names

Right now they're interleaved in one 4,000-line file. For the next module, extract them:

### Proposed structure
```
player.html          ← engine only (scene manager, all CSS, nav, KC mechanics)
content/
  scenes.js          ← declarative scene config (see schema below)
  assets/            ← images, video, audio
```

### Sketch of a scene config schema
```js
const MODULE = {
  title: "Plate Tectonics: Forces Below",
  parts: ["The Plates", "The Edges", "The Evidence", "The Forecast"],
  scenes: [
    {
      id: 'intro',
      type: 'section-card',    // uses existing .section-card CSS
      part: 0,
      eyebrow: 'Part 1',
      headline: 'The Plates',
      body: 'Earth's outer shell is broken into...'
    },
    {
      id: 'kc-sort',
      type: 'sort-kc',
      part: 0,
      prompt: 'Convergent or divergent?',
      items: [
        { text: 'Two plates collide, forming a mountain range.', bucket: 'convergent' },
        { text: 'Two plates pull apart, creating a rift valley.', bucket: 'divergent' },
        // ...
      ],
      buckets: [
        { id: 'convergent', label: 'Convergent', sub: 'plates collide' },
        { id: 'divergent',  label: 'Divergent',  sub: 'plates separate' }
      ]
    },
    // ...
  ]
};
```

The player renders scenes from the config — no more manual HTML per scene, no more renumbering bugs, no stale `#scene-14` selector errors.

**Why not a full framework?** For solo portfolio work, React/Vue adds more friction than it removes — build toolchain, npm, different collaboration mental model. The flat-file content/engine split gives most of the reusability benefit without any of that overhead. Revisit a full framework if the team grows or complexity increases significantly.

---

## SCORM / LMS Path

Adding basic SCORM tracking requires no rebuild of the module — just a wrapper.

### Completion tracking only (~20 lines of code)
1. Add [`pipwerks-scorm-api-wrapper`](https://github.com/pipwerks/scorm-api-wrapper) — a single vanilla-JS file, no build step
2. `SCORM.init()` when the module loads
3. `SCORM.set("cmi.core.lesson_status", "completed")` + `SCORM.quit()` when the learner reaches the conclusion scene or completes the Final Assessment

### Score reporting
Add `cmi.core.score.raw` and `cmi.core.score.max` calls at the Final Assessment completion point.

### Packaging
Wrap the content in a ZIP with `imsmanifest.xml`. [scormify.io](https://scormify.io) can auto-wrap a working Vercel URL; or do it manually (well-documented, straightforward).

Use **SCORM 1.2** for broadest LMS compatibility.

---

## Technical Patterns to Carry Forward

### Scene manager
```js
const TOTAL = N;
const SCENE_PART = [null, null, ..., 0, 0, ..., 1, 1, ...]; // null = no dot active

function goTo(n) {
  // 1. loadSceneAssets(n)   — loads deferred images/backgrounds for destination scene
  // 2. onSceneLeave(cur)    — unloads videos for scenes that use them
  // 3. unloadSceneAssets(cur) — strips src from images in the scene being left
  // 4. onSceneActivate(n)   — scene-specific hooks (init KC, start animation, etc.)
}
```

### Deferred asset loading — CRITICAL for mobile memory management
All `<img>` tags use `data-src` (not `src`) in markup. `loadSceneAssets(n)` / `unloadSceneAssets(n)` swap them at scene entry/exit.

```html
<!-- In HTML — no src, only data-src -->
<img data-src="assets/images/aurora.jpg" alt="Green aurora over Estonia">

<!-- In CSS — no background-image, only data-bg -->
<div class="bg-photo" data-bg="assets/images/aurora.jpg" style="animation-delay:0s;"></div>
```

Without this, iOS Safari accumulates decoded bitmaps for the entire session, causing the OS to kill the page under memory pressure — especially visible when users pinch-zoom, which spikes compositing memory.

### Video lifecycle management
```html
<video id="my-video" data-src="assets/video/clip.mp4" preload="none" muted loop playsinline></video>
```
```js
function loadAndPlayVideo(id) {
  const v = el(id);
  if (!v.getAttribute('src') && v.dataset.src) v.src = v.dataset.src;
  v.play().catch(() => {});
}
function unloadVideo(id) {
  const v = el(id);
  if (!v.getAttribute('src')) return;
  v.pause();
  v.removeAttribute('src');
  v.load(); // fully releases decoder buffers
}
```

### Accessible DnD engine
`initDragDrop(root, { onDrop, describeItem, describeTarget })`

Key implementation notes:
- Use **Pointer Events** (not native HTML5 `draggable`) — HTML5 DnD is unreliable on iOS touch
- Use a **ghost clone appended to `<body>`** as the drag visual, NOT the source element moved to `position:fixed` — CSS animations ending on `transform: translateY(0)` with `fill-mode: both` make ancestors the fixed-position containing block instead of the viewport, causing a ~150px drag offset
- **`e.stopPropagation()` on every handled key** in KC keyboard handlers — the global nav handler also binds to Space and ArrowLeft/Right; without stopPropagation those keys will accidentally navigate the module while a user is interacting with a KC

### CSS design system
```css
/* Color variables */
--green: #50ff8c;  --teal: #00dcd2;  --purple: #a050ff;  --red: #ff503c;
--text-bright: #ffffff;  --text-primary: #e8eeff;  --text-secondary: #9ba8cc;

/* Font stack */
/* Cinzel — headings, KC prompts, badge labels */
/* Inter — body text, instructions, data labels */
/* Pacifico — only for neon-sign analogy */
```

### Dev console shortcut
```js
// Add this in dev builds for fast scene jumping during testing
window._go = goTo;
// Then in DevTools: _go(18) jumps straight to Name That Color
```

### Viewport meta (iOS-specific)
```html
<meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover">
```
- `viewport-fit=cover` is required for `env(safe-area-inset-bottom)` to return non-zero values (needed for the nav bar and slideshow continue button to clear the home-indicator area on notched devices)
- Use `100dvh` for scene height with `100vh` as fallback — `100dvh` properly tracks the real visible viewport as iOS toolbar height changes
- Note: `maximum-scale` and `user-scalable=no` are **deliberately ignored by iOS Safari** since iOS 10 for accessibility reasons — you cannot prevent pinch-zoom on iOS via meta tags

---

## Accessibility State at Project Close

- ✅ All `<img>` tags: meaningful, specific alt text
- ✅ Content-bearing SVG diagrams: `role="img"` + `aria-label` (solar cycle chart, magnetosphere, Follow-a-Particle, altitude chart, energy chart, both Kp oval comparison maps)
- ✅ Magnetometer canvas: `role="img"` + `aria-label` describing the trace narrative
- ✅ Three drag-and-drop KCs: full keyboard path + `aria-live` progress announcements
- ✅ Tap-the-magnetosphere KC: keyboard path + `aria-pressed` state + `aria-live` progress
- ✅ Nav buttons, mute button: `aria-label`
- ⚠️ Decorative ambient elements (stars, aurora glow waves) not marked `aria-hidden` — accepted, low impact (no text content)
- ⚠️ Not fully WCAG 2.1 AA compliant — acceptable for a proof-of-concept portfolio piece

---

## Process Lessons (For the Next Module)

### What worked well
1. **Prototype cheaply, discard freely.** Treating every artifact as disposable kept the collaboration fast and concrete. The fastest path to a good design is a rough first pass you can react to, not a careful first attempt.
2. **Specific feedback over vague feedback.** "The blue border looks like a boat" and "the labels should be a much bigger font" are actionable. "Make it better" is not.
3. **Verify, don't guess.** Compute SVG geometry before writing it (Python one-liners for Bezier math). Use the live preview to actually click through interactions rather than declaring them done from code inspection alone.
4. **Real-device testing is not optional for mobile.** Chromium DevTools simulation missed the iOS `100vh` safe-area bug, the memory-pressure crash, and the drag-offset bug caused by `animation-fill-mode: both`. Test on real hardware.
5. **Precise symptom reporting enables real diagnosis.** "Title flashes, white screen, title flashes again, then 'a problem repeatedly occurred'" is the level of detail that enables root-cause analysis. Vaguer reports cost extra investigation rounds.

### Where to improve
1. **Front-load reference materials.** When you already have a specific visual in mind, share it at the start of the request — not after a first attempt misses. The iteration is most useful when you're genuinely open; it wastes a round when you already know what "right" looks like.
2. **Distinguish open from specific before making a request.** "I'm open on form, here's the goal" → let Claude run with it. "I have something specific in mind" → say what it is, even roughly.
3. **Generalize bug fixes across their whole class.** When a bug is found (e.g., missing `stopPropagation`), immediately check all similar handlers — not just the one that surfaced it.
4. **Raise uncertainty before implementing.** If a proposed fix has a real risk of not working on the target platform, flag the doubt before spending an implementation + test cycle on it.

---

*Aurora project completed 2026-06-23. Module content: aurora borealis, magnetosphere, solar wind, space weather forecasting.*
