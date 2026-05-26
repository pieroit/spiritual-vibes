## Why

The initial game mechanics ship with floaty jumps that can't reach the taller platforms, a single enemy archetype that always trickles in from the right, and a periodic pickup spawner detached from combat. The result reads as a tech demo rather than an arcade game. This change tightens the feel toward Metal Slug: heavier physics with a taller jump arc, a small roster of enemy archetypes with distinct behaviors and energy bars, drops tied to kills, and randomized spawn cadence.

## What Changes

- **Jump physics**: increase gravity and jump impulse so the player feels heavier *and* clears the tallest platform in a single arc. Fixed-arc (no variable jump), full air control preserved.
- **Spawn timing**: enemy spawn delay is sampled from a Normal distribution (μ=3000ms, σ=2000ms), clamped to [300ms, 8000ms], rescheduled per spawn.
- **Spawn surfaces**: enemies spawn off-camera on any surface (ground or platform), preferring surfaces ahead of the player.
- **Enemy archetypes**: three types replace the single enemy.
  - **Runner** (red) — chases the player horizontally. 1 HP.
  - **Patroller** (amber) — walks back and forth on its surface, turning at edges. 2 HP.
  - **Shooter** (steel-blue) — stands still, faces the player, fires slow horizontal bullets on a 1.5s cadence. 2 HP.
- **Enemy energy bars**: small bar above the enemy, revealed on first damage taken.
- **Kill-driven drops**: remove the periodic pickup spawner. Pickups drop at the kill location with type-specific drop rate and health/powerup split. Drops stay in place (no gravity).
- **Pickup despawn**: pickups disappear 5s after spawning, blinking in the final 1s.

## Capabilities

### Modified Capabilities
- `game-core`: jump impulse and gravity retuned.
- `game-combat`: enemy spawning (timing, surfaces, archetypes), enemy HP and behaviors, enemy bullets, kill-driven drops, pickup despawn.
- `game-hud`: enemy energy bars added.

## Impact

- **Files**: `html/play.html` only (script section). No new files; inline per project convention.
- **Dependencies**: none.
- **Runtime**: client-side; no asset additions (still colored circles).
- **Future work**: sprites will replace the recolored circles; further archetypes (Charger, Jumper) deferred.

## Non-goals

- No sprite art, animation, or sound.
- No new player abilities (variable jump, dash, etc.).
- No mobile/touch input.
- No level design or progression.
