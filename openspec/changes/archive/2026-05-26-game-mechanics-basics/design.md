## Context

`html/play.html` currently shows a "Coming soon" message inside an empty Phaser 4 scene. The site is a static, client-side-only project: no build step, no package manager, no backend. We want a Metal-Slug-inspired horizontal action game starring the Pope as the player character. This change focuses on the **mechanics layer** so we can later layer in sprite art, sound, levels, and encyclical-themed content without touching the core loop.

Constraints:
- No build pipeline — everything lives inline in `html/play.html` and loads Phaser 4 from CDN.
- Target audience is broad / low-scholarization adults — controls must be obvious and feel responsive.
- Renders into a full-viewport canvas with `Phaser.Scale.RESIZE`.

## Goals / Non-Goals

**Goals:**
- A single playable Phaser scene with horizontal scrolling, gravity, and platform collisions.
- Player character that can move left/right, jump, and shoot with keyboard input.
- Enemies that spawn, move, can be shot, and damage the player on contact.
- Pickups that grant a temporary power-up (e.g., faster fire rate) and/or restore health.
- HUD overlay showing health, score, and current power-up state.
- Game-over state with a restart action.
- Every visible entity rendered as a **colored circle** (Phaser `Graphics` or `Arc`) — no sprite assets.

**Non-Goals:**
- Final art, animation, sound, or music.
- Level design, multiple levels, or encyclical-tied content.
- Mobile/touch controls (keyboard only this round; we'll add touch later).
- Save/load, leaderboards, networking.
- Boss fights, complex enemy AI, or weapon-tree progression.

## Decisions

### Engine: Phaser 4 Arcade Physics
Already loaded via CDN; Arcade Physics is sufficient for AABB-style collisions and gravity in a 2D side-scroller. *Alternative considered:* hand-rolled physics — rejected because Phaser already handles overlap/collide cleanly and we want to stay focused on mechanics, not engine plumbing.

### Placeholder graphics as colored circles
Each entity type gets a distinct color so we can read the game without art:
- Player: white circle (radius ~20)
- Player bullet: yellow circle (radius ~5)
- Enemy: red circle (radius ~18)
- Enemy bullet: orange circle (radius ~5) — optional, only if enemies shoot
- Health pickup: green circle (radius ~12)
- Power-up pickup: blue circle (radius ~12)
- Ground/platforms: dark-grey rectangles (the one exception — circles don't tile as floor)

Rendered with `scene.add.graphics()` baked into a texture via `generateTexture` so sprites can use Arcade Physics bodies. *Alternative considered:* arcs without physics bodies — rejected, we want collision out of the box.

### Single scene, modular update steps
One `MainScene` with `create()` and `update()`, but internal logic split into small functions (`handlePlayerInput`, `updateEnemies`, `spawnPickup`, `updateHud`). Keeps it readable and prepares for splitting into multiple scenes later (menu, game-over).

### Input
Arrow keys / WASD for movement, Space to jump, Z (or Ctrl) to shoot. Keyboard polling each frame; no input remapping in this change.

### Camera
`cameras.main.startFollow(player, true, 0.1, 0)` with horizontal-only lerp. World bounds set wider than viewport (e.g., 4000×viewportHeight) so we get scrolling without infinite-world complexity.

### Spawning
Enemies and pickups spawn off-screen to the right on simple timers (`scene.time.addEvent`). Spawn rate scales mildly with score.

### HUD
A separate `this.add.container` fixed to the camera (`setScrollFactor(0)`) — text objects for score, a Graphics rectangle for the health bar, and a small icon (colored circle) for the active power-up.

## Risks / Trade-offs

- **[Risk] Placeholder visuals make playtesting feel unfun → Mitigation:** distinct colors + clear HUD feedback (numbers go up, health goes down) so the *mechanics* still feel satisfying.
- **[Risk] Single-file inline script grows large → Mitigation:** keep functions small and well-named; if it crosses ~600 lines we'll extract a `js/play.js` in a follow-up (still no build step).
- **[Risk] Arcade Physics not metal-slug-precise (no slopes, no fine pixel collision) → Mitigation:** acceptable for this MVP; revisit if level design later demands it.
- **[Risk] Keyboard-only excludes mobile users → Mitigation:** explicit non-goal; a touch-control change will follow.

## Open Questions

- Lives system vs. single-life-with-respawn? *Proposal:* single life, game-over screen with restart, simplest to implement.
- Should pickups stack or replace the active power-up? *Proposal:* replace, with a fresh duration timer.
