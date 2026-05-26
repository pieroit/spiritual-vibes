## Context

`html/play.html` runs a single Phaser 4 scene with Arcade Physics. The previous change established the core loop with one enemy type, a periodic pickup spawner, and a floaty jump (`gravity=1100`, `jump_v=-540`, apex ~132px). The tallest platform sits 320px above the floor and is unreachable in one jump. Combat reads as one-note: every enemy runs at the player from the right, one bullet kills them, pickups appear on a timer unrelated to play.

This change keeps the no-build constraint (all inline in `html/play.html`) and the colored-circles-as-placeholders convention. We are tuning feel and adding variety, not refactoring the architecture.

## Goals / Non-Goals

**Goals:**
- Jump feels weighty (Metal Slug arcade) and clears the tallest existing platform with headroom.
- Enemies vary in behavior, durability, and visual color so players can read threats at a glance.
- Spawn cadence is unpredictable but bounded.
- Reward is tied to combat (kills drop pickups) rather than a background timer.
- Pickups don't accumulate forever on the floor.

**Non-Goals:**
- Variable-height jump (tap vs hold). Arcade games commit to the arc.
- Sprite art or animation.
- New player abilities, weapons, or stats.
- Shooter aiming at player (Y-axis). Shooter fires horizontally — dodgeable by jumping.

## Decisions

### Jump tuning: g=1900, jump_v=-1150

Apex = `jump_v² / (2·g)` = `1150² / 3800` ≈ **348px**. Tallest platform is 320px above floor → clears with ~28px headroom. Airtime ≈ `2·|jump_v|/g` ≈ **1.21s**. Horizontal reach at `PLAYER_SPEED=230` is ~278px — still won't bridge the 400–500px gaps between platforms, which is intentional (player must use the ground as a highway, jumping up to platforms tactically rather than hopping across them).

*Alternative considered:* lower gravity to lengthen reach. Rejected — makes the jump floaty, the opposite of "arcade weight."

### Spawn timing: Normal(μ=3000, σ=2000), clamped [300, 8000]

We replace the fixed-interval `time.addEvent` with a self-rescheduling `delayedCall`. Each call:
1. Sample `delay = boxMuller(3000, 2000)`.
2. Clamp to `[300, 8000]`.
3. Schedule next spawn at `delay` ms; on fire, spawn one enemy and reschedule.

Box-Muller transform inline:
```
const u1 = Math.max(Math.random(), 1e-9);
const u2 = Math.random();
const z = Math.sqrt(-2 * Math.log(u1)) * Math.cos(2 * Math.PI * u2);
const delay = clamp(mu + sigma * z, MIN, MAX);
```

Clamp rationale: σ=2000 with μ=3000 puts ~7% of samples ≤0. The lower clamp prevents same-frame double spawns. The upper clamp prevents the rare 10s+ lull that feels like a bug.

### Spawn surface selection

We maintain a list of `surfaces`: ground + each platform, each with `{x, y, w}`. On spawn:
1. Filter to surfaces whose horizontal extent is **entirely off-camera**.
2. From that set, prefer surfaces where `surface.right < cam.scrollX` is false (i.e., ahead of or beyond the player). If non-empty, pick from those; otherwise fall back to all off-camera surfaces.
3. Pick uniformly from the chosen subset.
4. Enemy `x` is uniform random within the surface's horizontal extent; `y` is `surface.top - enemyRadius`.

*Alternative considered:* keep ground-only spawning. Rejected — platforms become safe havens, which kills the threat geometry.

### Enemy archetypes

A `kind` data field on each enemy sprite drives behavior in `update()`:

| Kind | Color | HP | Speed | Behavior |
|---|---|---|---|---|
| `runner` | `0xd84545` red | 1 | 95 | sets velocityX toward player each frame (current behavior) |
| `patroller` | `0xe6a23c` amber | 2 | 70 | walks at `facing × speed`; flips `facing` when at edge of its surface |
| `shooter` | `0x4a6fa5` steel-blue | 2 | 0 | stationary; every 1500ms, fires an enemy bullet at 250 px/s in current facing direction (re-faces player each shot) |

**Surface flavor (assignment weights):**
- Ground surface → 60% runner / 30% shooter / 10% patroller
- Platform surface → 60% patroller / 30% shooter / 10% runner

*Alternative considered:* pure random. Rejected — surface flavor makes the silhouette of a level readable ("pacing on the ledge, charging on the ground").

**Patroller edge detection:** at the start of each tick, if the patroller's current x is within `radius` of its surface's left/right edge AND it's facing that direction, flip facing. We attach the surface bounds via `setData('surface', {left, right})` at spawn.

**Charger telegraph** is no longer needed (Charger archetype dropped from this change).

### Enemy bullets

Shooters introduce a new `enemyBullets` physics group. Differences from player bullets:
- Texture: orange circle `0xff8c3a`, radius 5.
- No gravity.
- Speed 250 px/s (vs 700 for player) so they're dodgeable by jumping or sidestepping.
- Overlap with player → identical damage path as enemy contact (1 dmg, 900ms i-frames).
- Destroyed at world bounds.

We add a `physics.add.overlap(enemyBullets, this.player, ...)` mirroring the contact handler. Enemy bullets do **not** collide with platforms — they pass through, simplifying the model.

### Enemy energy bars

For each enemy with HP > 1, lazily create a small Graphics bar (24×3 px) above the sprite the **first time it takes damage**. Stored as `enemy.hpBar`. Each frame in `update()`, position it at `(enemy.x - 12, enemy.y - radius - 6)` and redraw width proportional to `enemy.hp / enemy.hpMax`. Destroyed with the enemy.

*Alternative considered:* always-on bars. Rejected — visual noise for 1-HP runners that never get to display one. Lazy reveal also signals "I hit it, it's not dead yet."

### Kill-driven drops

Replace `pickupSpawner` entirely. On bullet-enemy overlap, after the enemy's HP reaches 0:

```
if (Math.random() < DROP_RATE[kind]) {
  const isHealth = Math.random() < HEALTH_SHARE[kind];
  spawnPickup(enemy.x, enemy.y, isHealth ? 'health' : 'powerup');
}
```

Constants:
```
DROP_RATE     = { runner: 0.50, patroller: 0.70, shooter: 0.80 }
HEALTH_SHARE  = { runner: 0.70, patroller: 0.50, shooter: 0.30 }
```

Drops are created with `allowGravity: false` and do NOT collide with platforms — they stay floating at the death position (per user direction). The existing `pickups` group keeps its overlap-with-player handler unchanged.

### Pickup despawn

Each pickup gets `pickup.setData('expiresAt', this.time.now + 5000)`. In `update()`, iterate `pickups.getChildren()`:
- If `now >= expiresAt` → destroy.
- Else if `now >= expiresAt - 1000` → blink: `setVisible((Math.floor(now / 100) % 2) === 0)`.

## Risks / Trade-offs

- **Risk:** Higher gravity + higher jump_v changes a lot of feel at once. *Mitigation:* keep player horizontal speed and air control unchanged; only two constants move. Easy to retune.
- **Risk:** Normal-distribution spawn occasionally produces bursts (multiple ~300ms delays in a row). *Mitigation:* clamp prevents 0-delay; bursts are an arcade feature, not a bug. Re-evaluate after playtest if it feels unfair.
- **Risk:** Shooters introduce enemy bullets — a whole new damage path. *Mitigation:* mirror the existing contact-damage handler exactly; reuse i-frames.
- **Risk:** Lazy energy bars mean Runners never reveal one. *Mitigation:* by design; the bar is an information layer for "tough" enemies, not a universal HUD.
- **Risk:** Drops at the death position can land mid-air on platforms when a platform-enemy is killed. *Mitigation:* user explicitly chose "stays in place." Floating pickups read as a token, not a physics object.
- **Risk:** Off-camera-and-ahead-of-player surface filter can be empty if the player is near the world's right edge. *Mitigation:* fallback to all off-camera surfaces; if that's empty too, skip the spawn this tick.

## Open Questions

- Should the **player bullet** also damage enemy bullets (parry mechanic)? *Proposal:* no in this change; revisit if shooters feel oppressive.
- Should drops spawned mid-air drift slowly (light gravity, no platform collision) so they don't float weirdly above platforms? *Proposal:* keep them floating — user picked "stays in place" explicitly.
