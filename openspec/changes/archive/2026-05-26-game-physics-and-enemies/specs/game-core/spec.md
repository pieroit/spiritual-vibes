## MODIFIED Requirements

### Requirement: Gravity and jumping
The player SHALL be affected by gravity and SHALL be able to jump only while standing on a platform or ground. Gravity and jump impulse SHALL be tuned so the player's single-jump apex is high enough to land on every platform in the world from the surface beneath it.

#### Scenario: Falling under gravity
- **WHEN** the player is not in contact with a platform
- **THEN** the player accelerates downward until landing on a platform

#### Scenario: Jump from ground
- **WHEN** the player is on the ground or a platform AND the Space key is pressed
- **THEN** the player receives an upward velocity impulse and leaves the ground

#### Scenario: No double jump
- **WHEN** the player is airborne AND the Space key is pressed
- **THEN** no additional upward impulse is applied

#### Scenario: Jump reaches every platform
- **WHEN** the player jumps from directly under any platform in the world
- **THEN** the apex of the jump is at or above the platform's top surface, so the player can land on it
