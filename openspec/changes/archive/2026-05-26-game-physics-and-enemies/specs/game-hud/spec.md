## ADDED Requirements

### Requirement: Enemy energy bars
Enemies with maximum HP greater than 1 SHALL display a small energy bar above their sprite once they have taken at least one point of damage. The bar SHALL reflect remaining HP as a fraction of maximum HP.

#### Scenario: Bar appears on first hit
- **WHEN** an enemy with HP maximum greater than 1 takes its first point of damage
- **THEN** an energy bar appears positioned just above the enemy sprite, showing remaining HP as a fraction of maximum

#### Scenario: Bar tracks enemy position
- **WHEN** an enemy with a visible energy bar moves
- **THEN** the energy bar follows the enemy each frame so it stays visually attached above it

#### Scenario: Bar shrinks with damage
- **WHEN** a damaged enemy takes additional damage and survives
- **THEN** the energy bar's filled portion shrinks proportionally to the new HP

#### Scenario: Bar is removed with enemy
- **WHEN** an enemy with an energy bar is destroyed
- **THEN** the energy bar is also removed from the world

#### Scenario: One-HP enemies show no bar
- **WHEN** an enemy with maximum HP of 1 is hit
- **THEN** no energy bar is drawn (the enemy is destroyed by the hit)
