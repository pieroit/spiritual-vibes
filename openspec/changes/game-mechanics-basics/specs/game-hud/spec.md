## ADDED Requirements

### Requirement: Health display
The HUD SHALL display the player's current health relative to the maximum, fixed to the camera so it remains on-screen regardless of scroll position.

#### Scenario: Health bar reflects damage
- **WHEN** the player takes damage
- **THEN** the on-screen health bar shrinks proportionally to the new health value within the same frame or the next

#### Scenario: Health bar reflects healing
- **WHEN** the player collects a health pickup
- **THEN** the on-screen health bar grows proportionally, capped at the maximum

### Requirement: Score display
The HUD SHALL display the current score as a numeric counter fixed to the camera.

#### Scenario: Score increments on kill
- **WHEN** an enemy is destroyed by the player
- **THEN** the displayed score increases by the configured kill value

#### Scenario: Score resets on restart
- **WHEN** the game restarts after a game over
- **THEN** the displayed score returns to zero

### Requirement: Power-up indicator
The HUD SHALL display the currently active power-up (if any) and provide a visual cue of its remaining duration.

#### Scenario: Power-up icon appears
- **WHEN** the player collects a power-up
- **THEN** a colored indicator (matching the power-up's color) appears on the HUD

#### Scenario: Indicator disappears when expired
- **WHEN** the power-up's duration elapses
- **THEN** the indicator is removed from the HUD

### Requirement: Game-over overlay
When the game enters game-over state, the HUD SHALL show an overlay with the final score and restart instructions.

#### Scenario: Show game over
- **WHEN** the game-over state is entered
- **THEN** an overlay appears showing "Game Over", the final score, and a prompt indicating how to restart

#### Scenario: Hide game over
- **WHEN** the game restarts
- **THEN** the overlay is removed and normal HUD elements resume
