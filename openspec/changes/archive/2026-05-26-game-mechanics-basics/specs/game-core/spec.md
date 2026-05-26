## ADDED Requirements

### Requirement: Horizontal scrolling world
The game SHALL present a side-scrolling 2D world wider than the viewport, where the camera follows the player horizontally as they move right or left.

#### Scenario: Camera follows player rightward
- **WHEN** the player moves right past the horizontal center of the viewport
- **THEN** the camera scrolls right so the player stays roughly centered, until the world's right edge is reached

#### Scenario: World bounds clamp the player
- **WHEN** the player attempts to move past the left or right edge of the world
- **THEN** the player position is clamped at the world bound and cannot exit it

### Requirement: Player movement
The player character SHALL move left and right under keyboard control with constant horizontal speed while a movement key is held.

#### Scenario: Move right
- **WHEN** the player presses the Right arrow or D key
- **THEN** the player sprite translates rightward at the configured run speed until the key is released

#### Scenario: Move left
- **WHEN** the player presses the Left arrow or A key
- **THEN** the player sprite translates leftward at the configured run speed until the key is released

#### Scenario: Idle
- **WHEN** no movement key is pressed
- **THEN** the player's horizontal velocity is zero

### Requirement: Gravity and jumping
The player SHALL be affected by gravity and SHALL be able to jump only while standing on a platform or ground.

#### Scenario: Falling under gravity
- **WHEN** the player is not in contact with a platform
- **THEN** the player accelerates downward until landing on a platform

#### Scenario: Jump from ground
- **WHEN** the player is on the ground or a platform AND the Space key is pressed
- **THEN** the player receives an upward velocity impulse and leaves the ground

#### Scenario: No double jump
- **WHEN** the player is airborne AND the Space key is pressed
- **THEN** no additional upward impulse is applied

### Requirement: Ground and platform collision
The world SHALL contain a continuous ground plane and zero or more elevated platforms; the player and enemies SHALL collide with them as solid surfaces.

#### Scenario: Player lands on platform
- **WHEN** the player falls onto a platform
- **THEN** the player's downward velocity stops and the player rests on the platform's top surface

### Requirement: Placeholder visuals
All game entities in this change SHALL be rendered as solid colored shapes (circles for actors and pickups, rectangles for ground/platforms), with no sprite assets.

#### Scenario: Player appears as a white circle
- **WHEN** the scene starts
- **THEN** the player is drawn as a white filled circle with a clearly visible radius

#### Scenario: Each entity type has a distinct color
- **WHEN** multiple entity types are on screen
- **THEN** the player, enemies, bullets, and pickups are each drawn in their assigned distinct colors
