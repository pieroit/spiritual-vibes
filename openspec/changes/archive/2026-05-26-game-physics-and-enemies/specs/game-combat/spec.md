## MODIFIED Requirements

### Requirement: Enemy spawning and movement
Enemies SHALL spawn off-camera on any walkable surface (ground or platform), preferring surfaces ahead of the player when available. Spawn timing SHALL be sampled from a Normal distribution and clamped to a configured min/max delay. Enemies SHALL behave according to their archetype.

#### Scenario: Spawn delay is randomized
- **WHEN** the scheduler fires an enemy spawn
- **THEN** the next spawn is scheduled after a delay sampled from a Normal distribution with mean 3000ms and standard deviation 2000ms, clamped to [300ms, 8000ms]

#### Scenario: Enemy spawns off-camera
- **WHEN** an enemy spawn occurs
- **THEN** the enemy appears on a surface whose horizontal extent is entirely outside the camera's visible area at spawn time

#### Scenario: Spawn prefers surfaces ahead of the player
- **WHEN** at least one off-camera surface lies ahead of the player (to the right of the camera's right edge)
- **THEN** the spawn surface is chosen from those surfaces

#### Scenario: Spawn falls back when no ahead-surface is available
- **WHEN** no off-camera surface lies ahead of the player AND at least one off-camera surface exists elsewhere
- **THEN** the spawn surface is chosen from any off-camera surface

#### Scenario: Spawn skipped when no surface available
- **WHEN** no off-camera surface exists at all
- **THEN** no enemy is created for this tick and the scheduler reschedules normally

#### Scenario: Enemy spawns on platform
- **WHEN** the chosen spawn surface is a platform
- **THEN** the enemy is placed at a uniformly random x within the platform's horizontal extent, resting on its top surface

### Requirement: Projectile-enemy collision
A player projectile that overlaps an enemy SHALL reduce the enemy's hit points by 1 and SHALL be consumed. The enemy SHALL be destroyed only when its hit points reach zero.

#### Scenario: Bullet damages enemy
- **WHEN** a player projectile overlaps an enemy with HP > 1
- **THEN** the enemy's HP decreases by 1, the enemy remains in play, and the projectile is destroyed

#### Scenario: Bullet kills enemy
- **WHEN** a player projectile overlaps an enemy whose remaining HP is 1
- **THEN** the enemy is destroyed, the score increases by the configured kill value, and the projectile is destroyed

### Requirement: Pickups and power-ups
Pickups SHALL drop from defeated enemies at the enemy's last position. There is no periodic pickup spawner. Drop probability and the health-vs-powerup split SHALL be configured per enemy archetype. Drops SHALL stay in place where they spawn and SHALL disappear after a fixed lifetime.

#### Scenario: Drop on kill
- **WHEN** an enemy is destroyed
- **THEN** with the configured drop probability for that archetype, a pickup spawns at the enemy's last position; otherwise no pickup spawns

#### Scenario: Drop type chosen per archetype
- **WHEN** a pickup is rolled to drop from a defeated enemy
- **THEN** the kind (health vs power-up) is chosen using the configured health-share probability for that archetype

#### Scenario: Pickups stay in place
- **WHEN** a pickup spawns from a kill
- **THEN** the pickup remains at its spawn position without falling, and does not collide with platforms

#### Scenario: Health pickup restores health
- **WHEN** the player overlaps a green health pickup
- **THEN** the player's health increases by the configured amount (capped at maximum) and the pickup is destroyed

#### Scenario: Power-up grants temporary effect
- **WHEN** the player overlaps a blue power-up pickup
- **THEN** the player's fire rate increases for the configured duration, and the pickup is destroyed

#### Scenario: Power-up expires
- **WHEN** the active power-up's duration elapses
- **THEN** the player's stats return to baseline

#### Scenario: Pickup despawns after lifetime
- **WHEN** a pickup has existed in the world for 5 seconds without being collected
- **THEN** the pickup is destroyed and removed from play

#### Scenario: Pickup blinks before despawn
- **WHEN** a pickup is within 1 second of its despawn time
- **THEN** the pickup visibly blinks to warn the player

## ADDED Requirements

### Requirement: Enemy archetypes
The game SHALL support three enemy archetypes, each with distinct color, hit points, and behavior. Each enemy instance SHALL be tagged with its archetype kind on spawn.

#### Scenario: Runner exists
- **WHEN** the game spawns a `runner`
- **THEN** the enemy is rendered as a red circle, has 1 HP, and moves horizontally toward the player at the runner speed

#### Scenario: Patroller exists
- **WHEN** the game spawns a `patroller`
- **THEN** the enemy is rendered as an amber circle, has 2 HP, and walks back and forth along its spawn surface, reversing direction when it reaches an edge

#### Scenario: Shooter exists
- **WHEN** the game spawns a `shooter`
- **THEN** the enemy is rendered as a steel-blue circle, has 2 HP, remains stationary, faces the player, and fires a horizontal enemy bullet on a fixed cadence

#### Scenario: Surface flavor influences archetype mix
- **WHEN** an enemy spawns on the ground surface
- **THEN** the archetype is chosen using ground-weighted probabilities (runner-heavy)

- **WHEN** an enemy spawns on a platform surface
- **THEN** the archetype is chosen using platform-weighted probabilities (patroller-heavy)

### Requirement: Enemy bullets
Shooters SHALL fire enemy bullets that travel horizontally and damage the player on contact. Enemy bullets SHALL NOT collide with platforms.

#### Scenario: Shooter fires on cadence
- **WHEN** a shooter has existed for at least its fire interval since its last shot
- **THEN** the shooter spawns an orange bullet at its position traveling horizontally in its facing direction at the shooter bullet speed, and resets its fire timer

#### Scenario: Enemy bullet damages player
- **WHEN** an enemy bullet overlaps the player AND the player is not invulnerable
- **THEN** the player's health decreases by the contact-damage amount, the player becomes invulnerable for the configured duration, and the bullet is destroyed

#### Scenario: Enemy bullet ignored during i-frames
- **WHEN** an enemy bullet overlaps the player while the player is invulnerable
- **THEN** the bullet is destroyed and no damage is applied

#### Scenario: Enemy bullet passes through platforms
- **WHEN** an enemy bullet moves through a platform
- **THEN** the bullet is not deflected, stopped, or destroyed by the platform

#### Scenario: Enemy bullet expires off-world
- **WHEN** an enemy bullet leaves the world bounds
- **THEN** the bullet is destroyed
