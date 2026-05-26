## ADDED Requirements

### Requirement: Player shooting
The player SHALL fire projectiles horizontally in the direction they are facing while the shoot key is held, subject to a configured fire-rate cooldown.

#### Scenario: Single shot
- **WHEN** the player presses the shoot key (Z or Ctrl) once
- **THEN** one yellow projectile spawns at the player's position and travels horizontally in the player's facing direction

#### Scenario: Held-fire respects cooldown
- **WHEN** the player holds the shoot key
- **THEN** projectiles spawn at the configured fire-rate interval, not on every frame

#### Scenario: Projectile expires off-screen
- **WHEN** a player projectile travels beyond the visible camera bounds or world bounds
- **THEN** the projectile is destroyed and its physics body released

### Requirement: Enemy spawning and movement
Enemies SHALL spawn periodically off the right edge of the visible area and move leftward toward the player.

#### Scenario: Enemy spawn timer
- **WHEN** the game has been running and the spawn timer elapses
- **THEN** a red-circle enemy appears just beyond the right viewport edge at a valid ground/platform height

#### Scenario: Enemy walks toward player
- **WHEN** an enemy exists in the world
- **THEN** it moves horizontally toward the player's current x-position at the configured enemy speed

### Requirement: Projectile-enemy collision
A player projectile that overlaps an enemy SHALL damage or destroy the enemy and SHALL be consumed.

#### Scenario: Bullet kills enemy
- **WHEN** a player projectile overlaps an enemy
- **THEN** the enemy is destroyed, the score increases by the configured kill value, and the projectile is destroyed

### Requirement: Enemy-player damage
An enemy that contacts the player SHALL reduce the player's health by the configured contact-damage amount and trigger a brief invulnerability window to prevent multi-frame damage.

#### Scenario: Contact damage
- **WHEN** an enemy overlaps the player AND the player is not currently invulnerable
- **THEN** the player's health decreases by the contact-damage amount and the player becomes invulnerable for the configured duration

#### Scenario: Invulnerability prevents stacking
- **WHEN** an enemy overlaps the player while the player is invulnerable
- **THEN** no additional damage is applied

### Requirement: Pickups and power-ups
Pickups SHALL spawn periodically in the world; collecting one SHALL apply its effect to the player.

#### Scenario: Health pickup restores health
- **WHEN** the player overlaps a green health pickup
- **THEN** the player's health increases by the configured amount (capped at maximum) and the pickup is destroyed

#### Scenario: Power-up grants temporary effect
- **WHEN** the player overlaps a blue power-up pickup
- **THEN** the player's fire rate increases (or other configured effect applies) for the configured duration, and the pickup is destroyed

#### Scenario: Power-up expires
- **WHEN** the active power-up's duration elapses
- **THEN** the player's stats return to baseline

### Requirement: Game over and restart
When the player's health reaches zero the game SHALL end and offer a restart action.

#### Scenario: Health reaches zero
- **WHEN** the player's health drops to zero or below
- **THEN** the scene enters a game-over state, gameplay pauses, and a restart prompt is shown

#### Scenario: Restart
- **WHEN** the game-over state is active AND the player presses the restart key (e.g., Enter)
- **THEN** the scene resets to its initial state with full health, zero score, and the world re-initialized
