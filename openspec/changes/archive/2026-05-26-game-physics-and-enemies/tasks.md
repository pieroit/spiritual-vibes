## 1. Physics tuning

- [x] 1.1 In `CFG`, change `GRAVITY` from `1100` to `1900` and `JUMP_V` from `-540` to `-1150`.
- [x] 1.2 Verify in the browser: player jump arc clears the tallest platform (y-offset 320) with headroom, and falls feel snappy rather than floaty.

## 2. Surface registry

- [x] 2.1 In `buildWorld`, instead of (or alongside) `platformTops`, build `this.surfaces`: an array of `{x, y, w, left, right, top}` entries for the ground and every platform.
- [x] 2.2 Replace any remaining uses of `platformTops` with the equivalent lookup over `this.surfaces`.

## 3. Spawn timing (normal distribution)

- [x] 3.1 Add a `boxMuller(mu, sigma)` helper in the scene.
- [x] 3.2 Add `ENEMY_SPAWN_MU = 3000`, `ENEMY_SPAWN_SIGMA = 2000`, `ENEMY_SPAWN_MIN = 300`, `ENEMY_SPAWN_MAX = 8000` to `CFG`.
- [x] 3.3 Replace the looping `enemySpawner` with a self-rescheduling `scheduleNextEnemy()` that uses `time.delayedCall` with a per-call sampled delay clamped to `[MIN, MAX]`.
- [x] 3.4 Pause the spawner when `gameOver` becomes true (do not reschedule).
- [x] 3.5 Re-arm the spawner on `scene.restart()` via the normal `create()` path.

## 4. Spawn surface selection

- [x] 4.1 Implement `pickSpawnSurface()`: filter `this.surfaces` to entries entirely off-camera; prefer those whose `right >= cam.scrollX + cam.width` (ahead of player), fallback to all off-camera; return `null` if empty.
- [x] 4.2 `spawnEnemy()` calls `pickSpawnSurface()`, returns early if `null`, otherwise samples a uniform `x` inside the surface's extent and sets `y = surface.top - enemyRadius`.

## 5. Enemy archetypes

- [x] 5.1 Add color textures: keep `tex_enemy` for runner `0xd84545`; add `tex_patroller` `0xe6a23c` and `tex_shooter` `0x4a6fa5`. All circles, radius 18.
- [x] 5.2 Add `KIND_WEIGHTS_GROUND` and `KIND_WEIGHTS_PLATFORM` weight tables; implement `pickKind(surface)` that uses ground weights if the surface is the ground entry, platform weights otherwise.
- [x] 5.3 In `spawnEnemy()`, after picking surface and kind: create the enemy with the matching texture, `setData('kind', kind)`, `setData('hp', HP[kind])`, `setData('hpMax', HP[kind])`, `setData('surface', surface)`.
- [x] 5.4 Set behavior-specific physics: shooters get `setImmovable(true)` and `body.allowGravity` true (so they sit on the surface); patrollers get `setData('facing', ±1)` (random initial direction).

## 6. Per-archetype behavior in update

- [x] 6.1 In `updateEnemies()`, dispatch on `kind`:
  - Runner: existing chase behavior (velocityX toward player at `ENEMY_SPEED`).
  - Patroller: set velocityX = `facing × PATROLLER_SPEED` (70). If `(facing < 0 && x <= surface.left + r) || (facing > 0 && x >= surface.right - r)`, flip `facing`.
  - Shooter: set velocityX to 0; re-face player (`facing = player.x > x ? 1 : -1`); if `time.now >= nextFireAt` (stored on enemy data, init to `time.now + 1500`), spawn an enemy bullet and set `nextFireAt = time.now + 1500`.
- [x] 6.2 Add `SHOOTER_FIRE_MS = 1500`, `SHOOTER_BULLET_SPEED = 250`, `PATROLLER_SPEED = 70` to `CFG`.

## 7. Enemy bullets

- [x] 7.1 Add `tex_enemy_bullet` (orange circle `0xff8c3a`, radius 5).
- [x] 7.2 Add `this.enemyBullets = this.physics.add.group({ allowGravity: false })` in `setupGroups`.
- [x] 7.3 Implement `fireShooterBullet(shooter)`: spawn at `shooter.x + facing*22`, velocityX = `facing × SHOOTER_BULLET_SPEED`.
- [x] 7.4 Add `physics.add.overlap(this.enemyBullets, this.player, ...)` that applies the existing damage path (i-frames check, decrement health, set invulnUntil, update bar, game-over check). Destroy the bullet on overlap.
- [x] 7.5 In `cullBullets()`, also remove enemy bullets that leave world bounds.
- [x] 7.6 Enemy bullets do NOT collide with platforms — no `physics.add.collider` between them.

## 8. Enemy HP and damage

- [x] 8.1 Add `HP = { runner: 1, patroller: 2, shooter: 2 }` to `CFG`.
- [x] 8.2 Change the bullet-enemy overlap handler: decrement `enemy.data.values.hp` by 1; only when `hp <= 0` destroy the enemy and grant score. Always destroy the bullet.
- [x] 8.3 On the first damage tick where `hp < hpMax` and there's no `enemy.hpBar` yet, create the Graphics bar as a child of the scene (not the sprite); store on `enemy.hpBar`.
- [x] 8.4 In `update()`, iterate active enemies; for each one with a bar, redraw it at `(enemy.x - 12, enemy.y - radius - 6)` with width proportional to `hp / hpMax`.
- [x] 8.5 When destroying an enemy, also destroy its `hpBar` if present.

## 9. Kill-driven drops

- [x] 9.1 Delete the `pickupSpawner` and its callback registration.
- [x] 9.2 Add `DROP_RATE = { runner: 0.50, patroller: 0.70, shooter: 0.80 }` and `HEALTH_SHARE = { runner: 0.70, patroller: 0.50, shooter: 0.30 }` to `CFG`.
- [x] 9.3 On enemy death (in the bullet-enemy overlap, after `enemy.destroy()`), roll `Math.random() < DROP_RATE[kind]`; if pass, roll `Math.random() < HEALTH_SHARE[kind]` to decide kind; spawn pickup at the dead enemy's position.
- [x] 9.4 Refactor `spawnPickup(x, y, kind)` to accept explicit position and kind (no more random surface sampling).
- [x] 9.5 Set `allowGravity: false` on drops and do NOT add `physics.add.collider(this.pickups, this.platforms)` for them (drops stay in place).

## 10. Pickup despawn

- [x] 10.1 In `spawnPickup`, set `p.setData('expiresAt', this.time.now + 5000)`.
- [x] 10.2 Add `updatePickups()` called from `update()`: for each pickup, if `now >= expiresAt` destroy; else if `now >= expiresAt - 1000` toggle `setVisible` using a 100ms blink phase; else ensure `setVisible(true)`.

## 11. HUD: energy bar capability

- [x] 11.1 Confirm the energy bars rendered in step 8 are drawn on top of the world but below the HUD container (no depth conflict).
- [x] 11.2 Verify bars hide cleanly on `scene.restart()` (orphaned Graphics destroyed with their owning enemies).

## 12. Verification

- [x] 12.1 Run `python3 -m http.server 8000`, open `http://localhost:8000/html/play.html`.
- [x] 12.2 Confirm jump clears the tallest platform; falls feel weighty.
- [x] 12.3 Confirm enemies spawn on platforms as well as ground, with visible color distinction.
- [x] 12.4 Confirm shooters fire bullets that can damage the player; bullets can be jumped over.
- [x] 12.5 Confirm patrollers turn around at platform edges.
- [x] 12.6 Confirm energy bars appear after first hit on patrollers/shooters and shrink with damage.
- [x] 12.7 Confirm drops appear at the kill location and disappear at 5s with a final-second blink.
- [x] 12.8 Confirm no console errors or warnings.
