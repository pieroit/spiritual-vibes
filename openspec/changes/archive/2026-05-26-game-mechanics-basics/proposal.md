## Why

`play.html` is currently a "Coming soon" stub. To make the encyclical fun and approachable, we want a playful run-and-gun action game in the spirit of Metal Slug, with the Pope as the main character. This change establishes the foundational mechanics so we can iterate on art, levels, and themes later without rewriting the core game loop.

## What Changes

- Replace the "Coming soon" placeholder in `html/play.html` with a playable Phaser scene implementing core Metal-Slug-style mechanics.
- Implement horizontal side-scrolling action with a player character that can move, jump, and shoot.
- Add enemies, projectiles, pickups (power-ups / bonuses), and a simple HUD (health, score).
- Use **colored circles ("balls") as placeholder graphics** for every entity (player, enemies, bullets, pickups, ground/platforms). No sprite art in this change — art will be swapped in later changes.
- Keep the existing back-link and full-viewport canvas.

## Capabilities

### New Capabilities
- `game-core`: Player movement (run + jump), gravity, ground/platform collision, and camera that follows the player horizontally.
- `game-combat`: Player shooting, projectile lifecycle, enemy spawning, enemy AI (basic), collision/damage resolution, and pickup/power-up effects.
- `game-hud`: On-screen health bar, score counter, and current-weapon/power-up indicator.

### Modified Capabilities
<!-- None — first game change. -->

## Impact

- **Files**: `html/play.html` (rewritten game script section). No new files required; everything stays inline to honor the no-build-step convention.
- **Dependencies**: Continues to use Phaser 4 from CDN (already loaded).
- **Runtime**: Client-side only, no backend changes.
- **Future work**: Sprites, sound, level design, and encyclical-themed content will replace placeholder balls in later changes.
