# Colodoro Timer

A minimalist, full-screen Pomodoro timer where **the background color is the timer**. When a session runs, the entire screen slowly fades from a start color to an end color, so you can gauge time at a glance without staring at numbers.

It's a single, self-contained `index.html` file — no build step, no dependencies, no backend. Perfect for GitHub Pages.

## Live idea

Pick a start color and an end color, choose a duration, and start. The screen begins at the start color and gradually shifts to the end color as time runs out. The controls stay hidden so the color has your full attention — you reveal them only when you need them.

## Features

- **Color-as-timer** — the background interpolates from the start color to the end color over the session.
- **Tabbed color picker** — one area with tabs for curated presets, three "opposite" strategies, a mixed mode, and your saved palettes.
- **Randomize the current tab** — a single Randomize button that regenerates options for whichever strategy tab you're on.
- **Save palettes** — store favorite color pairs with a **+** button; they're organized into paginated Saved tabs (6 per tab).
- **Quick durations** — one-tap 5 / 15 / 25 / 30 / 45 / 60 minute presets, plus a 1–60 min slider.
- **Gesture controls** — peek, pause, and reset with taps, holds, and swipes.
- **Full-screen mode** — a corner button hides the browser UI for a truly distraction-free display.
- **Auto-hiding cursor** — the mouse pointer disappears after a couple of idle seconds and returns on movement.
- **Never scrolls** — the panel automatically scales to fit any screen height.
- **Remembers your settings** — last duration, saved palettes, and selected color tab persist across reloads.

## How to use it

### Starting a session

1. Double-tap (or double-click) to reveal the control panel — it also opens automatically on load.
2. Pick colors and a duration.
3. Press **Start**. The overlay hides and the color begins shifting.

### Controls and gestures

| Action | Gesture | Notes |
|--------|---------|-------|
| **Peek at controls** | Single click / tap, or move the mouse | Shows the panel for ~5 seconds **without** pausing. |
| **Pause / resume** | Double-tap anywhere | Opening the panel this way pauses; dismissing it resumes. |
| **Hide panel** | Double-tap outside the panel | Resumes if it was paused. |
| **Reset (after finish)** | Long-press (~0.9s) | Only works once the timer is **Complete**. Resets and immediately starts a new session. |
| **Reset (after finish)** | Swipe / drag up | Same as above; works with mouse drag or touch. |
| **Reset (after finish)** | Scroll up | Mouse wheel or trackpad; accumulates a bit of scroll before firing. |
| **Dismiss overlay** | `Esc` key | Hides the panel. |

**Key behaviors:**

- **Peek vs. pause are separate.** A single click or mouse movement only *peeks* — it never pauses the timer. Pausing requires a deliberate double-tap.
- **When the timer finishes, nothing happens automatically.** The screen stays at the end color until you interact with it. Reset gestures (hold / swipe up / scroll up) are only active in this finished state.
- Reset gestures work whether the overlay is open or closed.

### Full-screen and cursor

- **Full-screen** — tap the expand icon in the top-right corner of the control panel to hide the browser chrome (address bar, tabs, etc.). Tap it again (or press `Esc`) to exit. The icon reflects the current state. This uses the browser's Fullscreen API, so the button is hidden on platforms that don't support it (e.g. iPhone Safari).
- **Auto-hiding cursor** — the mouse pointer hides after ~2.5 seconds of no movement, and reappears the moment you move, click, or scroll. Touch input is unaffected.

### Colors

Colors live in a single tabbed area. Pick a tab, tap a card to apply that pair, and the **Randomize colors** button regenerates the options for the tab you're currently viewing.

| Tab | What it shows | Randomize |
|-----|---------------|-----------|
| **Presets** | 6 curated opposite pairs (two rows) | Greyed out — curated, not randomizable |
| **Complement** | Generated pairs with opposite **hue** (±180°), e.g. Red ↔ Teal | Yes |
| **Light/Dark** | Generated near-neutral **brightness** opposites, e.g. White ↔ Black | Yes |
| **Mono** | Generated **same-hue** light↔dark pairs, e.g. Pale blue ↔ Navy | Yes |
| **Mixed** | Generated pairs randomly drawn from all three strategies above | Yes |
| **Saved** | Your saved palettes; tap **×** on a card to delete (up to 24) | Greyed out |

Other ways to set colors:

- **Manual pick** — use the two color inputs to set exact start and end colors.
- **Save a generated card** — tap the **+** button in the top-right corner of any generated option card. It saves the pair, shows a brief "Palette saved" toast, and the **+** disappears (it won't reappear for pairs you've already saved).
- **Save the active palette** — tap the **+** button beside the two color inputs to save whatever start/end colors are currently active. Hover it for a tooltip.

Saving never navigates you away from your current tab — you just get a confirmation toast.

**Paginated Saved tabs:** the Saved tab holds 6 palettes. Save a 7th and a **Saved 2** tab appears; a 13th adds **Saved 3**, and so on (up to 24 total). When you delete palettes and a page empties out, its extra tab is removed automatically.

The selected tab is highlighted and remembered between visits.

### Duration

- Drag the slider (1–60 minutes), or
- Tap a quick pick (5 / 15 / 25 / 30 / 45 / 60).

Your last-used duration is restored automatically on the next visit.

## How it works (technical)

### Color shift algorithm

The color is a direct function of how much of the session has elapsed — there is no separate animation timeline.

1. **Progress** `t` is computed each frame from the timer:
   `t = (totalMs - remainingMs) / totalMs`, giving `0` at the start and `1` at the end.
2. **Interpolation** blends the start and end colors channel-by-channel in RGB:
   `channel = start + (end - start) * t`
3. The result is applied to the background on every animation frame via `requestAnimationFrame`.
4. A CSS `transition: background-color` smooths the frame-to-frame updates.

Because `t` is linear in time, the shift is steady. Pausing freezes `t`; finishing pins it at `1` (the end color).

### Fit-to-screen (never scroll)

The page never scrolls. After the panel renders (or the window/content changes), its natural height is measured against the available viewport height, and a `transform: scale()` shrinks it just enough to fit. This keeps the whole control panel visible on any screen size.

### Persistence

All state is stored in the browser's `localStorage` (nothing is sent anywhere):

| Key | Stores |
|-----|--------|
| `colodoro:lastDuration` | Last-used duration in minutes |
| `colodoro:savedPresets` | Saved color palettes (JSON) |
| `colodoro:colorTab` | Selected color tab |

This data is per-browser and per-device; it doesn't sync across machines.

### Stack

- Plain HTML, CSS, and vanilla JavaScript in one file (`index.html`).
- No frameworks, no dependencies, no build tooling.
- Uses the Pointer Events API for unified mouse/touch gesture handling.
- Uses the Fullscreen API (with `webkit` fallback) for full-screen mode.
- `favicon.svg` provides a rainbow-tomato tab icon.

## Running locally

Since it's a static file, just open it — or serve it for the most accurate behavior:

```bash
# Option 1: open directly
open index.html

# Option 2: serve locally (recommended)
python3 -m http.server 8000
# then visit http://localhost:8000
```

## Deploying to GitHub Pages

1. Push this repository to GitHub.
2. Go to **Settings → Pages**.
3. Under **Source**, choose **Deploy from a branch**.
4. Select the `main` branch and the `/ (root)` folder, then **Save**.
5. Your site will be available at `https://<your-username>.github.io/colodorotimer/`.

## Files

| File | Purpose |
|------|---------|
| `index.html` | The entire app — markup, styles, and logic. |
| `favicon.svg` | Rainbow-tomato tab icon. |
| `README.md` | This document. |
