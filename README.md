# quattro. — Le Quattro Stagioni

> A living, painterly landscape that paints itself from music. Built for Vivaldi's *Four Seasons*.

**[Live Demo →](https://blackjacket1403.github.io/Study_The_Classical_Way/)**

---

## What it does

Upload an audio file — ideally a movement of Vivaldi's *Four Seasons* — and watch the scene transform. The landscape reads the season from the filename and locks the palette, wildlife, and atmosphere to Spring, Summer, Autumn, or Winter.

- **Ribbons of light** glow for each register of the orchestra — basses near the water, violins high in the sky
- **Stereo = wind** — the left/right balance leans the sun, bends grass, and pushes particles
- **Seasonal wildlife** — butterflies follow your cursor in spring, birds glide in summer, leaves tumble and settle on the water in autumn, snow drifts in winter
- **Summer storms** — loud passages darken the sky with rain and tasteful lightning
- **Winter trees** — bare, elegant silhouettes sway on the shoreline
- **Audio-reactive water** — bass = slow swells, treble = fine chop, quiet = glassy mirror

No frameworks. No build step. A single HTML file — vanilla JavaScript + Canvas 2D + Web Audio API.

---

## Features

| Feature | Description |
|---|---|
| 🎵 **Audio Upload** | Drag-and-drop or click to load any audio file |
| 🎤 **Microphone Input** | Play Vivaldi on a real instrument and see it visualized live |
| 🎼 **Generative Ambient** | Built-in ambient engine synthesizes calm, copyright-free music |
| 🍅 **Pomodoro Timer** | 25/5 focus cycles with chime transitions |
| ⏱️ **Flowtime** | Count-up focus timer with session tracking |
| ✅ **Task List** | Add, check, and persist tasks via localStorage |
| 📖 **Study Mode** | Hides chrome, auto-starts ambient, opens focus tools |
| 🎨 **5 Moods** | Serene · Dreamy · Focus · Joyful · Melancholy |
| 📸 **PNG Save** | Capture the canvas as a high-res image |

---

## Filename → Season Detection

Name your file to auto-lock the season. The app detects:

| Season | Keywords | RV Number |
|---|---|---|
| 🌸 Spring | `spring`, `primavera`, `printemps`, `frühling` | `RV 269` |
| ☀️ Summer | `summer`, `estate`, `été`, `sommer` | `RV 315` |
| 🍂 Autumn | `autumn`, `fall`, `autunno`, `automne`, `herbst` | `RV 293` |
| ❄️ Winter | `winter`, `inverno`, `hiver` | `RV 297` |

Also detects concerto/movement numbers (`No. 1–4`, `mvt. 1–4`, Roman numerals `I–IV`) when the filename contains `vivaldi`, `stagioni`, `four seasons`, or `Op. 8`.

**Examples:** `Vivaldi - Spring.mp3` → 🌸 · `été vivaldi.flac` → ☀️ · `RV 293.mp3` → 🍂 · `four seasons no. 4.mp3` → ❄️

If no season is detected, the landscape cycles through all four automatically.

---

## Keyboard Shortcuts

| Key | Action |
|---|---|
| `Space` | Play / Pause |
| `L` | Load a track |
| `F` | Fullscreen |
| `V` | Toggle voice labels |
| `H` | Hide / show UI |
| `S` | Save PNG |
| `A` | Toggle ambient engine |
| `M` | Cycle moods |
| `P` | Focus panel (Pomodoro/Flowtime) |
| `→` | Nudge season forward |
| `1–4` | Jump to Spring/Summer/Autumn/Winter |
| `Esc` | Exit study mode / close focus panel |

---

## Run locally

Just open `index.html` in **Chrome** (or any modern browser). No server needed.

```bash
# Or serve it:
npx serve .
```

> ⚠️ Audio requires user interaction to start (browser policy). Click a button or press Space first.

---

## Deploy to GitHub Pages

1. Push this repo to GitHub
2. Go to **Settings → Pages**
3. Set source to **main branch**, root `/`
4. Your site will be live at `https://<username>.github.io/Study_The_Classical_Way/`

---

## Tech

- Vanilla JavaScript (no frameworks, no dependencies)
- HTML5 Canvas 2D
- Web Audio API (FFT analysis, stereo splitting, generative synthesis)
- Google Fonts (Cormorant Garamond + Archivo)

---

## Credits

A landscape by **blackjacket1403**

Inspired by Antonio Vivaldi's *Le Quattro Stagioni* (The Four Seasons), Op. 8, 1725.
