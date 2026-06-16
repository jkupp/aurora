# Aurora Borealis Learning Module — Project Handoff

## What this is

A standalone web-based learning module about the aurora borealis, built as a **learning design demonstration project**. The goal is to showcase innovative, beautiful ways of presenting scientific information — more like an immersive explainer experience than a slide deck or LMS course. The intended audience is general/adult learners who are curious but have no science background.

## Design direction

- **Format:** Full-screen scenes that fill the browser window (no scrolling within a scene). Scenes advance via button/keyboard. Think cinematic presentation, not web page.
- **Tone:** Wonder-first. Beautiful. The science is in service of the experience, not the other way around.
- **Aesthetic:** Dark space theme. Aurora color palette (greens, teals, purples, reds). Animated aurora backgrounds. High contrast typography.
- **Opening:** A crossfading slideshow of aurora photos/video with spacey ambient background music, resolving to a title card: *"Aurora Borealis: A Beautiful Mystery in the Sky."* Then a hook: *"For all of human existence, people have marveled in wonder at dancing lights in the sky. We now know a lot about the aurora and why it occurs, but many beautiful mysteries remain."*
- **Knowledge checks:** A fun, interactive activity after each of the 6 content sections (not a quiz — hands-on interactions like sorting, tapping, labeling, scenario reasoning).
- **Assets:** The user is gathering real aurora photos, short timelapse video clips, and ambient background music. Use CSS aurora animations as placeholders where real assets aren't yet placed.

## Assets status

User is sourcing:
- 4–6 aurora photos (variety of colors, at least one with foreground/landscape)
- 1–2 short timelapse video loops (10–20 sec, NASA Goddard public domain is a good source)
- 1 ambient/atmospheric music track (Pixabay "space ambient" is a good free source)

Until assets arrive, use the CSS animated aurora scenes already developed (layered radial gradient divs with skew/scale keyframe animations — see Section 1 prototype code).

## Revised content outline (v2)

### Learning objectives — learners will be able to:
1. **Explain** why the Sun constantly emits charged particles, and what makes that stream sometimes surge dramatically
2. **Describe** how Earth's magnetic field shields the planet — and why the poles are its entry points
3. **Connect** atmospheric gas type and altitude to the specific colors seen in an aurora
4. **Distinguish** between the three timescales of aurora variability (days, hours, minutes)
5. **Correct** the common misconception that Kp index measures intensity rather than geographic reach
6. **Apply** this understanding to evaluate conditions and make a considered viewing forecast

### Section 1 — The Hook: You've Seen the Photos. But What Are You Looking At?
- Opening slideshow → title → hook text
- Gallery of aurora types (green, red, blue-purple) with gas/altitude labels — this seeds Section 4
- Three core questions the module will answer (scroll-reveal)
- Auroral oval world map (animated pulsing oval, key locations marked)
- **Knowledge check:** Unscored pre-assessment poll — "Which of these affect where/when you see an aurora?" (solar activity, season, latitude, moon phase, temperature, magnetic field, cloud cover, luck). Reveals answers with explanations. Temperature is the myth (no effect). Luck gets a warm answer.

### Section 2 — The Source: Our Restless Sun
- Solar wind as the constant; CMEs as dramatic events
- The 11-year solar cycle (we're near solar maximum now)
- **Knowledge check:** Animated sorting — drag events onto "quiet Sun" vs. "stormy Sun" timeline

### Section 3 — The Shield: Earth's Invisible Armor
- The magnetosphere deflects most solar wind
- Field lines converge at the poles — those are the entry points
- **Key analogy:** Neon sign — electricity through gas → colored glow. Same physics.
- **Narrative device:** "Follow a single particle" from the Sun's surface to the atmosphere over Tromsø
- **Knowledge check:** "Tap where on this diagram charged particles can enter Earth's atmosphere" — interactive magnetosphere with pole hotspots

### Section 4 — The Light Show: Why These Colors?
- Oxygen at ~100 km = green (most common)
- Oxygen at 200 km+ = red (rarer, requires more energy)
- Nitrogen = blue/purple
- The aurora is a live readout of the atmosphere's layered structure
- **Knowledge check:** Photo annotation — drag gas labels onto color bands in a real aurora image

### Section 5 — The Rhythm: Three Reasons It's Unpredictable ★ KEY SECTION
Three timescales of variability — most people only know one:

**Days scale:** CME travel time is uncertain (±12 hours). The magnetic orientation of the CME (the critical Bz component) isn't known until it nearly arrives.

**Hours scale:** THE KP MISCONCEPTION. Kp measures the *equatorward geographic reach* of the auroral oval — NOT aurora intensity. A Kp of 7 means the oval has expanded far enough south to reach Scotland or Montana. But if you're already inside the oval (Iceland, Norway, Alaska), a higher Kp tells you almost nothing about how good your show will be.

**Minutes scale:** SUBSTORM CYCLES. Energy builds in the magnetotail (the night side of the magnetosphere, stretched away from the Sun), then releases explosively in a substorm — a sudden magnetic reconnection event. The aurora intensifies dramatically, dances, then quiets. These cycles repeat roughly every 1–3 hours. A local magnetometer graph spikes during a substorm breakout.

**Personal story from the user (use this):** The user went aurora-viewing in Iceland and their guide watched a real-time magnetometer graph. When the graph spiked, the guide would say *"The door is opening!"* — this is a substorm breakout. Use this phrase in the module. It's a perfect description of magnetic reconnection.

- **Knowledge check:** Three-scenario true/false burst — one misconception per timescale. "A Kp of 7 means an intense aurora over Iceland — true or false?" etc.

### Section 6 — The Chase: Where You Are Changes Everything
- Mid-latitude chasers (Montana, Scotland): watch Kp — it genuinely tells you if the oval is reaching you
- High-latitude chasers (Iceland, Norway, Alaska): Kp is nearly irrelevant. Watch: cloud cover (biggest factor), Bz component of solar wind (southward = good coupling), local magnetometer, proximity to magnetic midnight
- Practical synthesis: latitude, season, darkness, light pollution, solar cycle phase
- **Knowledge check:** Branching scenario — "You're planning a trip. Pick your location and month." Two paths (mid vs. high latitude) converge on a shared night-of checklist.

### Final Assessment — Space Weather Forecast Challenge
Four scenario-based reasoning tasks. Learner receives a simulated space weather briefing and must reason through whether conditions favor an aurora:

- **Scenario A (mid-latitude):** High Kp, clear skies, rural Montana → Favorable. Kp is genuinely meaningful here.
- **Scenario B (high-latitude trick):** High Kp, clear skies, Iceland → Kp is irrelevant. Need Bz and substorm data.
- **Scenario C (timing trick):** Perfect conditions, June in Iceland → No darkness. Aurora may be happening invisibly overhead.
- **Scenario D (substorm):** Magnetometer graph shows a spike. Learner must recognize "the door is opening" and say what it means.

## What's been built

A prototype of Section 1 was built as a scrolling HTML page (single widget). **That layout has since been replaced** by the full-screen scene architecture described above. The Section 1 prototype is useful for its content and interactions (CSS aurora scenes, world map SVG, poll logic) but should be rebuilt in the new format.

Useful components from the prototype:
- CSS animated aurora scenes (green, red, blue-purple) using layered radial gradients + skew/scale keyframes
- Auroral oval SVG world map with animated pulsing ellipse
- Poll interaction (toggle checkboxes, reveal answers)

## Technical notes

- Build as a single `index.html` file with embedded CSS and JS (no build step, no dependencies)
- Full-screen scenes: `height: 100vh; width: 100vw; overflow: hidden`
- Scene transitions: CSS crossfade (opacity transition) or slide
- Audio: HTML5 `<audio>` element; must be triggered by user interaction (not autoplay) — a "click to begin" opening screen handles this elegantly
- Keyboard navigation: left/right arrow keys + on-screen buttons
- The user has real aurora assets coming — build with placeholder CSS animations and clear comments marking where `<img>` or `<video>` tags should go

## User context

- The user went aurora-viewing in Iceland — that experience informs the Kp/substorm content and should be woven in
- They care about both learning design craft AND scientific accuracy
- They want this to be a portfolio piece demonstrating innovative instructional design
- Primary working directory for this project: `/Users/digw/Documents/jkupp/AI/claude_code/aurora`
