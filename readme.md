# microbe-wars

A single-file, zero-dependency browser game built on the HTML5 Canvas API. Loosely based on real PCR diagnostic results (MicrogenDX semen panel, March 2026) showing a 20-year chronic *Enterococcus faecalis* infection in prostatic tissue.

Click to fire Fosfomycin. Try not to lose.

**[Play it →](https://YOUR_USERNAME.github.io/microbe-wars)**

---

## How it works

### Architecture

One HTML file. No build step, no bundler, no dependencies. Everything — styles, game logic, rendering — is inline.

```
microbe_war.html
├── <style>        # Layout + HUD styles
├── <canvas>       # The battlefield
└── <script>       # All game logic (~380 lines vanilla JS)
```

### Rendering

Pure Canvas 2D API — no WebGL, no frameworks. Each frame:

1. `drawBackground()` — clears and repaints the tissue environment
2. `fosfos.forEach(update + draw)` — moves projectiles, draws trails
3. Collision detection — fosfos vs microbe bounding volumes
4. `microbes.forEach(update + draw)` — moves and draws all organisms
5. `explosions` + `speechBubbles` — particle effects and floating text

Runs at requestAnimationFrame cadence (~60fps). Global `t` counter drives all animations via `sin(t * freq + phase)` offsets so nothing needs a timestamp.

### Entities

**`Microbe` class** — three subtypes sharing one class:

| type | shape | HP | movement |
|---|---|---|---|
| `efaecalis` | sphere (gram+ cocci) | 4 | slow drift, bounded |
| `ecoli` | rod, rotates | 2 | faster, tumbles |
| `ehormaechei` | longer rod, rotates | 2 | medium, tumbles |

Collision detection is per-type:
- Spheres: euclidean distance vs radius
- Rods: point rotated into local space, then AABB check

Each microbe has a `phase` offset for independent wobble/drift, a `speechTimer` for random quote popups, and a `hitFlash` counter that triggers a `brightness(2.5)` CSS filter for the hit frames.

**`Fosfo` class** — Fosfomycin projectiles. Spawned on click, aimed at the nearest living microbe with some jitter. Stores a 10-frame position trail rendered as fading circles. Destroyed on microbe contact or leaving canvas bounds.

### Spawning & difficulty

`spawnWave()` staggers 8 microbes with `setTimeout` at 120ms intervals. Auto-respawn fires every 480 frames if *E. faecalis* count drops below 6 — representing biofilm-mediated regrowth. Intentionally hard to fully clear.

### Interactions

| action | result |
|---|---|
| Click on microbe | Direct hit, no projectile spawned |
| Click empty space | Spawns `Fosfo` homing toward nearest microbe |
| Fosfo reaches microbe | `microbe.hit()`, fosfo removed, small explosion |
| Microbe HP → 0 | 18-particle burst, kill counter increments |

### Visual style

3D illusion via layered circles: base fill → dark stroke → highlight ellipse (upper-left) → shadow arc (lower-right, low opacity). Rods get a specular ellipse at the leading cap. Helmets on *E. faecalis* are two filled ellipses. Flags on rods are three-point filled paths.

Speech bubbles are `roundRect` fills with `globalAlpha` fade tied to `speechTimer`.

---

## Scientific basis

The organisms, proportions, and resistance genes are pulled from an actual NGS + qPCR semen panel:

| Organism | Load | Resistance |
|---|---|---|
| *Enterococcus faecalis* | 80% (>10⁷ copies/mL) | ermB, tetM, qnr |
| *Enterobacter hormaechei* | 13% | CTX-M, qnr |
| *Escherichia coli* | 6% (1.68×10⁶ copies/mL) | CTX-M |

*E. faecalis* gets 4HP because it genuinely is harder to eradicate — gram-positive cell wall, intrinsic fluoroquinolone resistance via *ermB*, and an exceptional capacity for biofilm formation in prostatic ductal tissue. Standard urine cultures miss it. The only thing that hit it was a 12-week Fosfomycin course targeting MurA (cell wall synthesis enzyme) — hence the projectile label.

*E. hormaechei* is clinically notable for being rare in this context. Finding it in two partners with identical resistance genes is what confirmed transmission.

---

## Run locally

```bash
open microbe_war.html        # macOS
xdg-open microbe_war.html    # Linux
start microbe_war.html       # Windows
```

Or serve it:

```bash
npx serve .
python3 -m http.server 8080
```

## Deploy to GitHub Pages

```bash
gh repo create microbe-wars --public --source=. --push
# then: Settings → Pages → Deploy from branch → main → / (root)
```

Live at `https://YOUR_USERNAME.github.io/microbe-wars` within ~60 seconds.

---

## License

MIT. Do whatever you want with it. If you're a doctor who dismissed a patient's chronic infection for years, maybe just play the game and reflect.