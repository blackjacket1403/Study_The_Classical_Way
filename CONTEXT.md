# Vivaldi *Four Seasons* Visualizer — Project Context

> Hand-off / reference document. Read this first to resume work. It captures **what the app is, how the code is laid out, the user's taste and hard constraints, the Vivaldi filename rules, how to validate changes, and a backlog of ideas.**

---

## 1. What it is (one paragraph)

A **single self-contained HTML file** that turns an uploaded audio track into a **light, painterly "Le Quattro Stagioni" landscape**. Music drives a living scene: a luminous sky over reflecting water, a ribbon of light for each register of the orchestra, drifting particles, and season-specific life (butterflies, birds, falling leaves, snow). It is tuned for Vivaldi's *Four Seasons* and **reads the season from the audio file's name**, locking the whole palette and wildlife to Spring / Summer / Autumn / Winter. The user intends to **publish/host it**.

**Tech:** vanilla JavaScript + HTML5 **Canvas 2D** + **Web Audio API**. **No frameworks, no Three.js, no build step.** The only external load is Google Fonts (*Cormorant Garamond* + *Archivo*).

---

## 2. Files & locations

All deliverables live in `/mnt/user-data/outputs/`:

| File | Purpose |
|---|---|
| `orchestra-visualizer.html` | The entire app (HTML + CSS + JS in one file, ~35 KB). This is the thing to edit. |
| `README.md` | End-user instructions: run/host/customize/controls + filename→season rules + seasonal life. Keep in sync with the app. |
| `CONTEXT.md` | This document. |

**To publish:** the user just renames `orchestra-visualizer.html` → `index.html` and drops it on any static host (GitHub Pages, Netlify, etc.). No server needed.

> ⚠️ **Audio caveat (state this every time):** the in-chat preview iframe is sandboxed and **mutes audio**. The file must be **downloaded and opened in a browser (Chrome)** to actually hear sound and see it react.

---

## 3. Current feature set (the accepted build)

**Scene**
- DPR-aware full-window canvas; `horizonY = H * 0.7` (sky above, mirrored water below).
- Per-season palettes: 3-stop sky gradient, sun/glow color, two ribbon accents (warm `ribA` / cool `ribB`), water color, foliage color (`FG`), and a flower/accent color (`BLOOMC`).
- Smooth interpolation between seasons via `cur()` (floor/frac blend), so transitions are seamless.

**Audio → visuals mapping**
- **Stereo = wind.** L/R channels split via `ChannelSplitter` into two `AnalyserNode`s; `wind = (levelR − levelL)` leans the sun, bends ribbons/grass, and pushes particles. This is the "directioning."
- **Pitch = height.** 7 frequency **bands** → 7 flowing **ribbons**: basses sit low near the water, treble/air rides high in the sky. Each ribbon swells with its band's energy.
- Per-band **auto-leveling** (peak envelope) + **high-frequency tilt** (perceptual lift) so quiet high voices still register. Overall level pulses the sun's glow. Spectral **flux** triggers onset particles.

**Particles ("motes")**
- Two sprites: warm light + cool white. **Direction is season-coded:** spring/summer **rise** (pollen/petals/light), autumn/winter **fall** (leaves/snow). Winter uses the white sprite, slightly larger and slower (snow).
- **Ambient layer:** winter always has gentle drifting snow, spring always has floating petals — in both the playing and idle states — so those seasons read even in quiet passages.

**Seasonal life (creatures + foliage)** — all soft silhouettes, drawn **before** the reflection so they mirror in the water:
- **Spring 🦋** — butterflies with flapping wings wander on a sine path; flowering shoreline grass.
- **Summer 🕊️** — distant birds (gull silhouettes) glide across the sky on the wind; tall reeds/cattails.
- **Autumn 🍂** — leaves tumble, spin, and sway as they fall; golden drooping seed-heads.
- **Winter ❄️** — snow (via motes) over frosted, snow-capped reeds. (No flying creatures.)
- A **swaying foreground grass band** runs along the near shore every season, tinted to `cur().fg`, bending with wind+time. Drawn **after** the reflection (it's foreground, not mirrored).
- Target on-screen creature counts: spring 5 / summer 7 / autumn 6 / winter 0.

**Reflection**
- Mirrors the sky region into the water (`drawImage` vertical flip) + water sheen gradient + ripple streaks + horizon glow.

**UI (light theme: translucent white/blur panels, ink `#34323c`, warm accent `#b8783a`)**
- Hint/intro card explaining the filename→season feature.
- Controls: play/pause, seek + time, volume, sensitivity, **Track** (open file), **Tone** (panned test sweep, no file needed), **Seasons** (auto ↔ hold toggle), **Voices** (faint register labels), **Fullscreen**, **Save** (PNG).
- Keyboard: `Space` play, `F` fullscreen, `V` voices, `H` hide UI, `S` save, `→` nudge season.
- Drag-and-drop file load. UI auto-hides when idle. Canvas uses `preserveDrawingBuffer` so PNG save works.
- `#season` overlay shows the current season name; `#diag` shows audio state + L/R levels (debug aid).

---

## 4. Filename → season detection (Vivaldi-specific)

On load, `detectSeason(filename)` (lowercased) tries, in order:

1. **Season words**, English + Italian:
   - Spring: `spring`, `primavera`
   - Summer: `summer`, `estate`
   - Autumn: `autumn`, `fall`, `autunno`
   - Winter: `winter`, `inverno`
2. **RV catalogue numbers:** `269` → Spring, `315` → Summer, `293` → Autumn, `297` → Winter (matched only when prefixed by `rv`, to avoid false hits on random numbers).
3. **Concerto / movement numbers**, but **only** when the name also contains `vivaldi` / `stagion` / `four seasons` / `quattro`: matches `No. 1–4`, `Concerto 1–4`, `Op. 8 No. n`, `#n`, or roman `I–IV` → Spring/Summer/Autumn/Winter.

**Behavior:**
- **Detected** → lock the scene: `seasonAuto = false`, `seasonPhase = key`, Seasons button shows **"hold"**, status reads e.g. `Now playing · Autumn · L'autunno`.
- **Not detected** (e.g. a different composer) → `seasonAuto = true`, button shows **"auto"** (the season slowly cycles; user can flip to hold).

**Verified examples (all pass a smoke test):**
`Vivaldi - Spring.mp3`→spring · `Le Quattro Stagioni - LEstate.flac`→summer · `Vivaldi RV 293 Autumn.mp3`→autumn · `four seasons no. 4.mp3`→winter · `Vivaldi Op.8 No.2.m4a`→summer · `vivaldi concerto III.mp3`→autumn · `winter (linverno).mp3`→winter · `Beethoven Symphony 5.mp3`→**null (auto)** · `random song.mp3`→**null (auto)**.

---

## 5. Code map (inside the single IIFE in `orchestra-visualizer.html`)

Helpers: `TAU`, `clamp`, `lerp`, `hx(hex)→[r,g,b]`, `mixRGB(a,b,t)→[r,g,b]`, `rgba([r,g,b], a)→css`. **All color values are RGB arrays;** pass arrays into `rgba()`.

- **`SEASONS[]`** — 4 palette objects; `forEach` precomputes `_sky/_glow/_ribA/_ribB/_water`.
- **`FG[]`, `BLOOMC[]`** — foliage + accent colors per season (`.map(hx)`).
- **`seasonPhase` (float), `seasonAuto` (bool)**; **`cur()`** returns the interpolated `{sky, glow, ribA, ribB, water, fg, name}`; **`seasonNow()`** = rounded season index 0–3.
- **`SEASON_META[]` + `detectSeason()`** — see §4.
- **`resize()`** — sizes canvas, sets `horizonY`, calls **`buildGrass()`** (precomputes blade x/height/phase every 15px).
- **`BANDS[]`** (7) — freq ranges + smoothed energy + `eL/eR` + flow phases. Names are orchestral registers.
- **Motes:** `moteSprite` (warm), `moteSpriteCool` (white); `spawnMote(bandI, e)` (season-aware direction/size/sprite); `drawMotes(dt)` (additive `lighter`).
- **`clouds[]`** (4 soft drifting).
- **Creatures block:** `CR[]` pool (30), `pick()`, `spawnCreature()`, `drawButterfly()`, `drawBird()`, `drawLeaf()`, `updateCreatures(dt)` (maintains counts, updates+draws+recycles), `drawForeground(C)` (grass band + per-season tips).
- **Web Audio:** `ensureAudio()` (graph: `MediaElementSource → masterIn → volGain → destination`, plus `ChannelSplitter → analyserL/analyserR`); `analyse(dt)` (per-band eL/eR, level, `wind`, flux→motes, ambient motes, advances `seasonPhase` if auto; **idle branch** animates gently when no audio).
- **Draw order in `frame()`** (important — controls depth + what gets reflected):
  `drawSky → drawSun → drawClouds → drawRibbons → drawMotes → updateCreatures → drawReflection → drawForeground → drawVoices`
  (creatures **before** reflection ⇒ mirrored; grass **after** ⇒ foreground.)
- **`loadFile()`** — object URL, applies `detectSeason()`, resumes AudioContext, robust play/error/status messaging.
- UI wiring: buttons, sliders, keyboard, drag-drop, auto-hide, test sweep (panned oscillator), PNG save.

---

## 6. User preferences & HARD constraints (do not violate)

**Aesthetic (the user has rejected builds for breaking these):**
- **Light and beautiful.** No dark "space/cosmic" theme. (An earlier 3D black-hole/orchestra build was rejected.)
- **Cohesive & harmonious.** No garish rainbow colors, no blown-out neon glow blobs, nothing that looks "techy."
- **Calm but appealing.** Not hyper, not flashing, not "looks like a kid did it." Avoid strobing, screen-shake, shockwaves, jet bursts.
- **Elegant, painterly silhouettes.** Not cartoonish or clip-arty.
- Earlier the user also found an over-minimal calm version **too dull** — so: calm **and** rich/beautiful, not sterile.

**Priorities (in order):** outcome / creativity / **beauty first**; then stereo "directioning" and instrument mapping (secondary but still wanted).

**Must keep:**
- Filename → season **locking** (Vivaldi-specific).
- **Stereo = wind**, **pitch = height** mappings.
- **One self-contained file**, no external libs beyond fonts, hostable as `index.html`.

**Always tell the user:** download + open in **Chrome** for audio (preview mutes it).

---

## 7. Honest limitation to keep stating

True **per-instrument isolation from a stereo MP3 is not possible** — timbres overlap in frequency. What the app does is an **informed approximation**: accurate instrument frequency ranges, perceptual weighting, per-band auto-leveling, and onset detection. **Real per-instrument accuracy would require MIDI or stem files**, which is a fundamentally different input. Don't over-promise "exact instrument separation."

---

## 8. How to validate changes (no browser available here)

The environment can't run WebGL/Canvas/audio. Workflow each turn:
1. Edit `orchestra-visualizer.html`.
2. Extract the inline `<script>` with a small Python regex to `/tmp/check.js`.
3. `node --check /tmp/check.js` for syntax.
4. Optionally `node -e '...'` to **smoke-test pure logic** (e.g. `detectSeason` against sample filenames).
5. Confirm new function names / draw-order line are present (grep).
6. `present_files` the HTML + README.

Then rely on careful reading for visual correctness, and remind the user to verify in Chrome.

---

## 9. Backlog / ideas already floated (pick up here)

- **Summer storm:** build toward a hail/thunderstorm look when the music gets loud (Vivaldi's *L'estate* finale is a storm) — darkening sky, rain streaks, lightning flashes kept tasteful.
- **Spring thunderstorm-then-calm** moment (the *La primavera* middle).
- **Winter:** make it even stiller/paler; add a couple of **bare tree silhouettes** on the shore; optional cozy warm window glow far off.
- **Birds in a tighter V** formation; **soft birds in spring** too.
- **Autumn leaves settling** and floating on the water before fading.
- Per-season **palette/motion fine-tuning** on request.
- Possible **MIDI/stems input mode** for genuinely accurate per-instrument mapping (big feature, optional).
- Mobile/touch polish; a subtle title card with the movement name.

---

## 10. Quick-start prompt for a future session

> "Resume the Vivaldi *Four Seasons* visualizer. The app is `/mnt/user-data/outputs/orchestra-visualizer.html` — a single-file vanilla JS + Canvas 2D + Web Audio, light painterly seasonal landscape that reads the season from the filename. Read `CONTEXT.md` for architecture and the user's hard constraints (light, elegant, no garish/techy/kiddy effects; keep stereo=wind, pitch=height, filename→season locking, single hostable file). Validate edits with `node --check` on the extracted inline script, then `present_files`. Remind me to open it in Chrome for audio."

---

## 11. Session-2 additions — generative sound + focus tools

**New centerpiece: an in-app generative ambient engine** (`Atmosphere` object). It synthesizes calm, non-repeating, copyright-free music with the Web Audio API and **routes through `masterIn`**, i.e. through the *same* L/R analyser the visuals already read — so motion and sound are coherent *by construction* (the visuals react to music we generate). Voices: detuned saw **pads** → lowpass, sine **bass**, triangle **plucks** (random pan), a looping filtered-noise **air** bed, and a sine **chime** on phrase boundaries. A `setInterval(25ms)` look-ahead scheduler advances beats/bars; chords are built from in-scale stacked degrees (`deg(root,scale,i)`), so they're always consonant. **Season sets the musical key** (`SEASON_ROOT` MIDI roots D4/A3/G3/C3); **mood sets scale/tempo/density/brightness**. A synthesized convolver impulse (`makeImpulse`) adds reverb; `engineBus` + `verb` are created in `ensureAudio()`.

**Musical-coherence signals** (computed in `analyse()`, folded subtly into the draw code):
- `breath` — slow asymmetric envelope of energy → sun bloom, ribbon spread, glow lift (the "inhale/exhale").
- `pulse` — beat phase. Exact when the engine runs (`Atmosphere.lastBeatTime`/`beatDur`); from onset detection (`flux` vs adaptive `fluxAvg`) for uploaded audio. Drives water `ripples[]` and a small sun pulse.
- `centroid` — spectral centroid → tiny warm/white lift of the sun glow.
- `flowM` — `mood().flow * (1+breath*0.25)` multiplies ribbon phase advance (musical flow speed).
- Importantly, the `analyse()` idle guard now also checks `!Atmosphere.running`, so engine audio uses the live analysis path.

**Mood** (`MOODS[]`, `moodIdx`, `mood()`): Serene / Dreamy / Focus / Joyful / Melancholy. Each carries `{scale, bpm, density, bright, react, flow, wash, washA}`. `react` scales visual amplitude/bloom; `flow` scales motion speed; a low-alpha **`soft-light` color wash** at the end of `frame()` tints the mood. Cycued via the Mood button / `M`.

**Pomodoro** (`Pomo` object): 25/5 focus/break + 15-min long break every 4 cycles; timestamp-based countdown, start/pause/reset, cycle dots, `softChime()` + a `breath` bump at phase changes; auto-starts ambient on a focus block if nothing's playing. Panel `#pomo`, button / `P`.

**Study mode** (`setStudy`): hides chrome (`body.study`), forces **Focus** mood, starts ambient if idle, opens Pomodoro. **Esc** exits (also closes Pomodoro). Button `Study`.

**New keyboard:** `A` ambient · `M` mood · `P` pomodoro · `Esc` exit study/pomodoro (plus existing Space/F/V/H/S/→).

**New files:** `orchestra-visualizer-v1-backup.html` is the pre-session-2 version (the "current version is great" build) — safe to delete or keep as a fallback.

**Design rule honored:** all additions stay light, calm, and smoothed (springs/envelopes, low-alpha washes, slow tempos 52–72 BPM, gentle gains) — no garish/twitchy behavior. Validate as before: extract inline script → `node --check`; the music math can be smoke-tested in pure node (verified frequencies stay ~65–1568 Hz).

### Updated backlog
- **Summer storm** dynamics in the engine + visuals (Vivaldi's *L'estate*).
- Per-mood **palette tints** that interact with each season more deeply.
- **Tempo-tap / BPM sync** for uploaded tracks to get exact pulse like the engine has.
- Optional **MIDI/stems** input for true per-instrument accuracy.
- Save/restore **user presets** (mood + season + pomodoro lengths) — note: Artifacts can't use localStorage, but a standalone hosted file can.

---

## 12. Session-3 additions — sound-reactive water + cursor-following butterflies

- **Audio-reactive water.** In `drawReflection`, the previously time-only ripples are now driven by sound: `wAmp = 1.5 + level*9 + bassE*22 + breath*6` and `wSpd = 0.6 + level*1.5 + bassE*1.3`, where `bassE = (BANDS[0].e+BANDS[1].e)/2`. The mirrored reflection is drawn in ~20 horizontal **slices**, each shifted in x by `sin(sy*0.03 + t*wSpd + wind*1.5)*wAmp*depth` (more distortion nearer the viewer) — so the whole lake ripples with the music. The surface streak lines reuse `wAmp/wSpd` for matching wave size/speed. Beat ripples (from `pulse`) remain on top.
- **Pointer-following butterflies.** Globals `mx,my,pointerT` updated by a `pointermove` listener; `active = (now - pointerT) < 2500ms`. Spring butterflies (creature `type 0`) were converted from a fixed `baseY+sin` path to a **steering model**: each frame they seek a target — when active, an orbit point around `(mx,my)` offset by `(cos/sin)(wph)` so they swarm *around* the cursor rather than stacking on it; when idle, a slowly meandering wander point. Velocity is `lerp`-eased toward the target and speed-capped (170 active / 74 idle), `y` clamped to `[H*0.05, H*0.9]`. Wings still flap via `c.ph`. Butterflies remain spring-only by design (seasonal coherence); the app opens in spring so it's visible immediately.
- No new dependencies; same validation (`node --check`). Both effects honor the calm aesthetic (eased, bounded, near-glassy when quiet).

---

## 13. Session-4 additions — instrument-varying water + Focus panel (Pomodoro/Flowtime/tasks)

**Spectrum-driven water.** `drawReflection` now splits energy into `bassE=(B0+B1)/2`, `midE=(B2+B3)/2`, `trebE=(B4+B5+B6)/3`. Bass drives big swells (`wAmp`), mid adds speed (`wSpd`), and `chop=trebE` adds a **second high-spatial-frequency wave layer** to both the sliced reflection offset and the surface streaks: `+ sin(x*0.075*(1+chop) - t*(wSpd*2+chop*6))*fine*(...)`. Streak count raised 6→8, slices 20→24. Net effect: violins/flutes = fine fast chop, basses = slow rollers, quiet = glassy.

**Focus panel** (the old `#pomo` panel, rebuilt). Two tabs (`tabPomo`/`tabFlow`, `setTab`, `focusTab`) toggling `#viewPomo`/`#viewFlow`; switching pauses the other timer. The control button is relabeled **Focus** (still key `P`).
- `Pomo` (unchanged logic): 25/5 + 15-min long break every 4 cycles; countdown.
- `Flow` (new): count-**up** stopwatch via `cur()=elapsed+(running?(now-startAt)/1000:0)`; `start/pause`, `takeBreak()` logs a session (>30s) into `sessions`/`totalSec` then counts up a break, `reset()`. Auto-starts ambient on a focus block.
- **Tasks** (`tasks[]`, `renderTasks`, `addTask`): add via input (Enter or ＋), checkbox toggles `done` (strikethrough), hover-× removes, `taskCount` shows done/total. List grows; `#taskList` scrolls.
- **Persistence**: `saveState`/`loadState` use `localStorage` key `quattro.focus` wrapped in try/catch (persists when hosted, silently no-ops in the sandbox). Stores tasks, active tab, Flow sessions/total. Called once at init: `loadState(); renderTasks(); setTab(focusTab);`. `Pomo.show()` now also `setTab(focusTab)+renderTasks()`.

No new dependencies; `node --check` clean.
