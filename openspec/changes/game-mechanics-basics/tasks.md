## 1. Scene scaffold

- [x] 1.1 Replace the placeholder `MainScene` in `html/play.html` with `create()`/`update()` hooks, Arcade Physics enabled, and gravity configured.
- [x] 1.2 Define tuning constants at the top of the script (player speed, jump velocity, fire rate, enemy speed, spawn intervals, health max, power-up durations, world width).
- [x] 1.3 Set world bounds (e.g., 4000 × viewport height) and a background fill color.

## 2. Placeholder textures

- [x] 2.1 Generate a white-circle texture for the player via `Graphics.generateTexture`.
- [x] 2.2 Generate distinct textures for: enemy (red), player bullet (yellow), enemy bullet (orange, optional), health pickup (green), power-up pickup (blue).
- [x] 2.3 Create a dark-grey rectangle texture for ground and platforms.

## 3. Ground, platforms, and camera

- [x] 3.1 Build a static physics group containing the ground strip across the world and a few platforms at varied heights.
- [x] 3.2 Spawn the player sprite, give it a physics body, and add collision with the static group.
- [x] 3.3 Configure `cameras.main` to follow the player horizontally with lerp; clamp to world bounds.

## 4. Player controls

- [x] 4.1 Wire arrow keys / WASD for horizontal movement; set velocity each frame.
- [x] 4.2 Wire Space for jump; only allow when `body.blocked.down` (on ground).
- [x] 4.3 Track facing direction (`+1` / `-1`) based on last horizontal input for shooting.

## 5. Shooting and projectiles

- [x] 5.1 Create a projectile group with a fire-rate cooldown timer.
- [x] 5.2 Z / Ctrl spawns a bullet at the player's position moving horizontally in the facing direction.
- [x] 5.3 Destroy bullets that leave the world bounds.

## 6. Enemies

- [x] 6.1 Create an enemy group and a `time.addEvent` spawner that creates enemies just off the right viewport edge at valid ground/platform heights.
- [x] 6.2 In `update`, move each enemy toward the player's x position.
- [x] 6.3 Collide enemies with the static ground group.

## 7. Combat resolution

- [x] 7.1 Overlap player-bullets vs enemies → destroy both, increment score.
- [x] 7.2 Overlap enemies vs player → reduce health, set invulnerability timer, flash the player tint while invulnerable.
- [x] 7.3 When health reaches zero, switch the scene into a game-over state (pause spawners and gameplay).

## 8. Pickups and power-ups

- [x] 8.1 Add a pickup spawner with separate timers (or weighted random) for health vs. power-up.
- [x] 8.2 Overlap player vs pickup → apply effect, destroy pickup.
- [x] 8.3 Implement power-up effect (e.g., faster fire rate) with a duration timer that reverts stats on expiry.

## 9. HUD

- [x] 9.1 Create a HUD container with `setScrollFactor(0)`.
- [x] 9.2 Draw a health bar (Graphics) that updates whenever health changes.
- [x] 9.3 Add a score text object that updates on kill / on reset.
- [x] 9.4 Add a small colored indicator that appears while a power-up is active and hides on expiry.
- [x] 9.5 Add a game-over overlay with "Game Over", final score, and "Press Enter to restart" text; hide it during normal play.

## 10. Restart and polish

- [x] 10.1 On Enter in game-over state, call `scene.restart()` (or reset all state in-place) so health, score, enemies, pickups, and power-ups return to initial values.
- [x] 10.2 Manually verify by running `python3 -m http.server 8000` and opening `http://localhost:8000/html/play.html`: walk, jump, shoot, kill enemies, take damage, collect pickups, die, restart.
- [x] 10.3 Quick code pass: keep functions small, no console errors, no warnings in the browser devtools.
