# NITRO CIRCUIT

Night-city arcade racing in the browser. Procedural sports car, Cannon.js physics, drift nitro, 2 AI opponents, and a 3-lap circuit — no downloads, no assets, instant play.

## Play

Open `index.html` in a modern browser, or host the repo on GitHub Pages.

## Controls

### Desktop

| Action | Keys |
| --- | --- |
| Accelerate | `W` / Arrow Up |
| Brake / reverse | `S` / Arrow Down |
| Steer | `A` `D` / Arrow Left Right |
| Handbrake / drift | `Space` |
| Nitro | `Shift` |
| Pause | `Esc` / `P` |

### Mobile / touch

On-screen HUD: analog steer stick (bottom-left), GAS, BRAKE, DRIFT, NITRO (bottom-right). Pause with the `II` button. Keyboard and touch work at the same time, so touchscreen laptops are fully supported.

## Features

- 2 AI opponents (real Cannon.js rigid bodies, not scripted animation) — a blue aggressive racer (faster, tighter lines, less braking) and a green stable racer (smoother, brakes earlier into corners). Both steer via waypoint look-ahead along the track's own path data, with band-limited randomness for a human-like line, proper spawn spacing (no start-line pileup), corrected steering-direction math, accurate track-position initialization, spawn-grace-gated stuck-detection with automatic reverse-and-turn recovery, and full collisions with the player and each other
- Live race position (POS x/3) in the HUD, plus AI cars shown on the mini-map
- Procedural sports car (body, cabin, wheels, headlights, taillights)
- Asphalt circuit with curbs, barriers, sunset city skyline
- Dynamic lighting, headlights, drift smoke, speed FOV boost
- Arcade physics: accel, brake, reverse, handbrake drift, nitro impulse
- Zero-friction contact material with a manually damped lateral-grip model, so the car never sticks or jitters
- Low center of mass + active righting torque: no flipping during hard drifts
- Nitro regenerates over time and faster while drifting
- Speedometer, nitro bar, lap timer, live best lap, circular mini-map with car heading
- Start, pause, and finish overlays (glassmorphism)
- Auto-pause and input reset on tab hide / window blur (no sticky keys)
- Fixed-timestep sub-stepping decoupled from refresh rate (60 Hz, 144 Hz, variable)
- Pre-allocated vectors and quaternions; pooled drift smoke; resize-safe renderer
- Fullscreen responsive layout with iOS safe-area insets
- Optional engine tone generated with the Web Audio API (toggle in pause menu)

## Pause menu

- **RESUME** — continue the race
- **RESTART** — reset car, laps, and timer
- **SOUND** — toggle engine audio
- **TITLE** — return to the title screen

## AI behavior

- **Post-finish parking** — once an AI car completes its laps it peels off to its own shoulder of the track and coasts to a stop, instead of dead-stopping in the racing line where it could block a still-racing player.
- **Adaptive difficulty ("gets better every match")** — AI top speed, cornering aggression, and mistake rate are scaled by a skill multiplier saved in `localStorage` (`nitrocircuit.aiSkill`). Win, and the AI ups its game next race; get beaten badly, and it eases off — across sessions, not just within one.
- **Finish line now matches for player and AI, exactly** — AI cars spawn a few metres behind the start line for grid spacing, and their first crossing of it (just reaching the start, not a lap) was being miscounted as a completed lap. That off-by-one made them finish, park, and ghost after only 2 real laps while the player still needed 3. Fixed so only genuine full laps count; verified by measuring actual distance traveled per lap-increment (~627 units, matching the ~609-unit track length) instead of the old false first "lap" at 5.8 units.
- **Finished cars can never block you** — a parked AI becomes a "ghost" (collision response disabled) the instant it finishes, so even though it still visually coasts to the shoulder, the player can never be walled in by it on a later lap — verified by driving the player straight through a parked AI's exact position in the headless test.
- **Smarter recovery direction** — when the AI needs to back out of a mistake, it now turns the way it was already trying to (based on its last steering command) instead of a 50/50 random guess, so the visible reverse-and-recover moment is shorter and more decisive.
- **No-reverse / no-progress hardening** — the AI's own path-following logic can no longer command actual reverse at low speed (a prior loophole let a sharp turn + low speed trap it in a worsening backward spiral that a speed-only watchdog couldn't detect). A new direction-aware watchdog tracks real track progress and forces recovery within ~1.4s of any stall, reverse, or collision-spin, regardless of raw speed. Verified with a headless 3-lap physics simulation including deliberate player-AI collisions and a forced hard-reverse injection.

## Deployment (GitHub Pages)

1. Create a GitHub repository and push this project (keep `index.html` at the repo root).
2. GitHub: **Settings → Pages**.
3. Source: **Deploy from a branch**.
4. Branch: `main` (or `master`), folder: `/ (root)`.
5. Save. After a minute the game is live at `https://<user>.github.io/<repo>/`.

Local preview:

```bash
# Serve the folder (any static server)
npx --yes serve .
```

Then open the printed local URL.

## Tech stack

- **Three.js r128** — WebGL scene, lights, shadows, camera
- **Cannon.js 0.6.2** — rigid-body car physics
- **Vanilla ES6** — game loop, HUD, input, pause
- **Procedural canvas textures** — asphalt, curbs, building windows (no external images or models)

Zero npm dependencies in the game itself. CDN scripts only.

## Quality notes

- Works on desktop and mobile, portrait and landscape
- First interaction starts the race (title **RACE** button, or `Enter` / `W` / `Space`)
- Best lap stored in `localStorage` key `nitrocircuit.best`
- Physics verified headlessly with Cannon.js: 0-65 km/h in ~1 s, ~139 km/h top speed (163 km/h with nitro), zero roll during drifts
- GitHub Pages ready: single `index.html`, no build step
