# Punk Platformer — Game Design Document

## 1. Concept & Vision

**Genre:** 2D Side-scrolling Platformer

**Core Fantasy:** Играешь за панк-вайб подростка с ирокезом, который бегает по мрачному городу, собирает пивные банки, побеждает роботов и покупает крутые гитары. Визуальный стиль — неоновый панк 80-х с пиксель-арт эстетикой.

**Target Feel:** Быстрый, отзывчивый, с приятным feedback от прыжков и атак. Чувство контроля и прогресса.

---

## 2. Visual Style

### Art Direction
- **Пиксель-арт** в духе ретро-консолей (NES/SNES era)
- **Неоновая цветовая палитра:** чёрный фон, неоново-зелёный (#39ff14) и розовый (#ff4fd8) как акцентные цвета
- **Тёмный городской пейзаж** на фоне с параллакс-эффектом

### Color Palette
| Element | Color |
|---------|-------|
| Primary (Hair/FX) | #39ff14 (Neon Green) |
| Secondary (Accents) | #ff4fd8 (Hot Pink) |
| Background Dark | #0a0812, #12101a |
| Platform Brick | #8b4513, #a0522d |
| Ground | #2a2a2a, #3a3a3a |
| Enemy (Robot) | #cc2222, #881111 |
| Coin/Beer Can | #ffffff, #cc0000, #ffcc00 |
| Skin | #ddb892 |

### Resolution
- Internal resolution: 1920x1080 (scalable)
- Pixel-perfect rendering с `image-rendering: pixelated`

---

## 3. Player Character — The Punk

### Appearance
- **Size:** ~28x44 pixels (32x48 hitbox)
- **Body parts:** legs, boots, shirt (with arms), head, mohawk
- **Customizable colors:**
  - Hair: Neon Green, Hot Pink, Electric Blue, Purple, Orange
  - Shirt: Black, Red, Blue, Green, Purple
  - Pants: Dark Jeans, Black, Brown

### Animations
| State | Frames | Description |
|-------|--------|-------------|
| Idle | 1 | Breathing ( subtle body movement) |
| Run | 4 | Leg alternation, arm swing |
| Jump | 1 | Legs tucked |

### Controls
| Action | Keyboard | Gamepad |
|--------|----------|---------|
| Move Left | ← / A | Left Stick Left |
| Move Right | → / D | Left Stick Right |
| Jump | Space / ↑ / W | A Button |
| Attack | E | B Button |

### Physics Constants
```
GRAVITY = 0.5
JUMP_FORCE = -12
SPEED = 5
MAX_FALL_SPEED = 10
```

### Attack Mechanic
- **Cooldown:** 40 frames
- **Duration:** 20 frames
- **Visual:** Yellow rectangle slash effect
- **Hitbox:** 30px ahead of player

---

## 4. World & Level Design

### Level Structure
- **Total width:** ~2500 pixels
- **Tile size:** 32x32 pixels
- **Scroll type:** Horizontal camera follow (camera leads player ~33% from left)

### Platforms
1. **Ground** — Bottom row, сплошной
2. **Floating Platforms** — Brick-textured, variable sizes (4-5 tiles wide)

### Platform Layout (coordinates in tiles, Y from top)
```
Ground: y=480 (15 tiles from top)

Floating:
- x=200, y=350, width=5 tiles
- x=400, y=280, width=4 tiles
- x=600, y=320, width=5 tiles
- x=850, y=250, width=4 tiles
- x=1050, y=350, width=5 tiles
- x=1250, y=280, width=4 tiles
- x=1450, y=320, width=5 tiles
- x=1650, y=250, width=4 tiles
- x=1850, y=350, width=5 tiles
- x=2050, y=280, width=4 tiles
- x=2250, y=320, width=5 tiles
```

### Background Layers (Parallax)
1. **Stars** — Slowest (0.05x speed), sparse white dots
2. **Buildings** — 0.3x speed, dark silhouettes of varying heights

---

## 5. Collectibles — Beer Cans

### Appearance
- White/red cylinder (10x16 pixels)
- Red top/bottom bands
- Yellow label in middle

### Placement
16 positions along the level at y=400 (ground level)

### Behavior
- Static, no animation
- Collected on overlap (hitbox 24x24)
- **Score:** +10 points per can

### Feedback
- Sound: Two-tone beep (600Hz + 800Hz)
- Disappear immediately

---

## 6. Enemies — Angry Robots

### Appearance
- Red body with darker outline (28x28 pixels)
- White eyes with black pupils
- Yellow antenna with ball on top

### Behavior
- Patrol left-right within defined bounds
- Speed: 1.5 pixels/frame
- Reverse direction at patrol limits

### Spawn Positions
```
x=350, y=416, patrol 300-450
x=700, y=416, patrol 600-800
x=1000, y=416, patrol 900-1150
x=1400, y=416, patrol 1300-1550
x=1800, y=416, patrol 1700-1950
x=2200, y=416, patrol 2100-2350
```

### Collision with Player
| Condition | Result |
|-----------|--------|
| Player stomps from above (vy > 0, player bottom < enemy top) | Enemy dies, player bounces (+50 score) |
| Side/below collision | Player takes damage, 60 frames invincibility |

### Feedback
- Death sound: Two triangle beeps (420Hz + 480Hz)
- Disappear immediately

---

## 7. Goal — Victory Flag

### Appearance
- Gray pole (6x130 pixels)
- Neon green triangular flag

### Position
- x=2400, pole at y=350, flag extends to x+50

### Win Condition
- Player x + width > goalX (2400)
- Triggers win screen

---

## 8. Game States & Screens

### Main Menu Flow
```
[START SCREEN]
    ├── CREATE YOUR PUNK (character customization)
    ├── SHOP (guitar shop - buy/equip guitars)
    ├── PLAY (start game)
    └── SETTINGS (resolution)
```

### Start Screen Structure
- Title: "PUNK PLATFORMER"
- Subtitle: "CREATE YOUR PUNK"
- Character Preview (left)
- Customization: Hair / Shirt / Pants (right)
- Navigation: Arrow Up/Down selects option, Left/Right changes value
- Bottom buttons: SHOP | PLAY | SETTINGS
- Press Enter/Space to confirm selection

### Shop Screen
- Title: "GUITAR SHOP"
- Shows coin balance at top: "COINS: 150"
- Grid/List of all 10 guitars with:
  - Guitar preview sprite
  - Name and price (or "OWNED" / "EQUIPPED")
  - Buy button (if affordable and not owned)
  - Equip button (if owned)
- Back button returns to main menu

### Game States
- `start` — Main menu (character creator)
- `shop` — Guitar shop screen
- `settings` — Settings screen
- `playing` — Main game loop
- `win` — Victory screen
- `gameover` — Game over screen

### Game Over Screen
- Red "GAME OVER" text
- Final score display
- Coins earned this round added to total
- "Press Space to restart" → returns to main menu

### Win Screen
- Green "YOU WIN!" text
- Final score in gold
- Coins collected this round added to total (stored in localStorage)
- "Press Space to play again" → returns to main menu

### Coin Flow
1. Player collects beer cans during gameplay → coins added to session total
2. On win or game over → session coins added to persistent localStorage balance
3. In shop → persistent balance used for purchases
4. On new game → start with 0 session coins, persistent balance carries over

---

## 9. HUD (Heads-Up Display)

### Layout (top bar, semi-transparent black background)
```
[SCORE: 00000] [BEER: 0/16]                    [♥ ♥ ♥]
Left edge             Center                    Right edge
```

- **Score:** Neon green, 20px monospace, padded to 5 digits
- **Beer count:** Pink, shows collected/total
- **Lives:** 3 heart icons, red with lighter highlight, 25x22 pixels each

---

## 10. Audio Design

### Engine
- Web Audio API
- Simple oscillator-based beeps (square/triangle waves)

### Sound Effects
| Event | Frequencies | Duration | Wave |
|--------|-------------|----------|------|
| Jump | 300 + 400 Hz | 0.1s each | square |
| Coin | 600 + 800 Hz | 0.08s, 0.12s | square |
| Kill | 420 + 480 Hz | 0.08s, 0.1s | triangle |
| Win | 400, 500, 600, 800 Hz | 0.15s each | square |

### Background
- No background music (keep it simple)

---

## 11. Character Customization System

### Options
```
Hair Colors:
  - Neon Green (#39ff14)
  - Hot Pink (#ff4fd8)
  - Electric Blue (#00d4ff)
  - Purple (#9b4dff)
  - Orange (#ff6b35)

Shirt Colors:
  - Black (#1a1a1a)
  - Red (#cc3333)
  - Blue (#3355cc)
  - Green (#33aa55)
  - Purple (#8855aa)

Pants Colors:
  - Dark Jeans (#1a1a2e)
  - Black (#0a0a0a)
  - Brown (#4a3520)
```

### Selection UI
- Left/Right arrows around value name
- Color swatch preview
- Real-time character preview updates

---

## 12. Guitar Shop System

### Overview
Guitar shop is accessible from the main menu (before playing). Player spends collected coins to unlock guitars. Each guitar has unique visual appearance, damage value, and sound.

### Currency
- **Coins (Beer Cans)** — Collected during gameplay, persist across sessions via localStorage
- Starting balance: 0 coins (earned through gameplay)

### Shop UI
- Screen accessible from main menu (separate from character creator)
- List of all guitars with: preview image, name, price, owned status
- Click/select to purchase or equip
- "BACK" button returns to main menu
- Equipped guitar shown on character in game

### Guitar Inventory
| ID | Name | Price | Damage | Visual | Sound |
|----|------|-------|--------|--------|-------|
| 0 | Fists | 0 (default) | 1 | No guitar visible | - |
| 1 | Stratocaster | 50 | 1.5 | Single-cutaway body, sunburst finish | Clean "twang" |
| 2 | Les Paul | 100 | 2 | Single-cutaway, cherry burst | Warm "growl" |
| 3 | Telecaster | 75 | 1.8 | Double-cutaway, natural wood | Bright "twang" |
| 4 | Flying V | 150 | 2.5 | V-shaped, metallic paint | Heavy "distorted" |
| 5 | Firebird | 200 | 3 | Offset body, zebra stripes | Punchy "crunch" |
| 6 | Jaguar | 125 | 2 | Short scale, sparkle finish | Sparkly "shimmer" |
| 7 | Superstrat | 175 | 2.8 | Pointy shape, neon accents | Aggressive "bite" |
| 8 | Iceman | 250 | 3.5 | Random shape, flame decals | Deep "roar" |
| 9 | Explorer | 225 | 3.2 | Square body, white | Sharp "snap" |
| 10 | SG | 175 | 2.8 | Double cutaway, devil horns, cherry red | Thick "wail" |

### Guitar Visual Details
Each guitar is drawn as a distinct shape on the player's back:
- **Fists:** No guitar, punches deal base damage
- **Strat:** Curved top, 3 single-coil pickups → sunburst oval
- **Les Paul:** Rounded top, 2 humbuckers → cherry circle
- **Telecaster:** Angled top, 2 single-coils → tan rectangle
- **Flying V:** V-shape → purple triangle
- **Firebird:** Offset waist → zebra stripe pattern
- **Jaguar:** Small body, maple neck → sparkle dots
- **Superstrat:** Sharp angles, 2 humbuckers → green angular
- **Iceman:** Asymmetric → red with flame
- **Explorer:** Square headstock → white blocky

### Attack Animation with Guitar
When attacking, player swings their equipped guitar:
- **Hitbox:** 30-45px (scales with guitar size)
- **Damage:** Base damage × guitar multiplier
- **Sound:** Unique per guitar (see below)

### Guitar Sounds (Web Audio)
| Guitar | Attack Sound | Description |
|--------|--------------|-------------|
| Fists | 200Hz short beep | Quick punch |
| Strat | 523Hz + 659Hz (clean) | Bright chord |
| Les Paul | 392Hz + 523Hz (warm) | Full chord |
| Telecaster | 440Hz + 587Hz (bright) | Sharp stab |
| Flying V | 349Hz distorted | Heavy hit |
| Firebird | 294Hz punchy | Punch |
| Jaguar | 587Hz + 740Hz (sparkle) | Shimmer |
| Superstrat | 330Hz distorted | Aggressive |
| Iceman | 262Hz + 330Hz (deep) | Deep roar |
| Explorer | 494Hz + 622Hz (sharp) | Sharp snap |

### Purchase Flow
1. Player enters shop from main menu
2. List shows all guitars, owned items marked "OWNED" or "EQUIPPED"
3. If player has enough coins and guitar not owned → "BUY" button active
4. On purchase: deduct coins, mark as owned, equip automatically
5. If already owned → "EQUIP" button
6. Equipped guitar persists to gameplay

### Equipment UI
- In shop, currently equipped guitar highlighted with neon border
- Can switch between owned guitars freely (no cost)

---

## 14. Recommended Engine

**Godot 4.x** — Best choice for this project because:
1. Native 2D support with pixel-perfect rendering
2. Tilemap system perfect for platformer levels
3. Built-in animation state machines
4. Free and open-source
5. GDScript is easy to learn and similar to Python
6. Export to HTML5, Desktop, Mobile

**Alternative:** Unity (C#) or Defold (Lua)

---

## 15. Implementation Notes

### Player State Machine
```
States: idle, run, jump, attack
Transitions:
  - idle → run: vx != 0
  - run → idle: vx == 0
  - any → jump: onGround && jump_input
  - idle/run → attack: attack_input && cooldown <= 0
```

### Collision System
- AABB (Axis-Aligned Bounding Box) for all collisions
- Player collides with platforms (stops movement)
- Player collides with coins (triggers collection)
- Player collides with enemies (stomp or damage)
- Player collides with goal (triggers win)

### Camera System
- Follows player X with lerp (10% per frame)
- Clamped to level bounds (0 to LEVEL_W - INTERNAL_W)
- Vertical: Fixed (no vertical scroll)

### Save System
- High score persistence via localStorage
- Owned guitars saved locally
- Equipped guitar ID saved locally
- Coin balance saved locally (accumulated across playthroughs)

---

## 16. Scope for MVP

### Must Have
- [x] Start screen with character creation
- [x] Platformer movement (left/right/jump)
- [x] Collision with platforms
- [x] Collectible coins (beer cans)
- [x] Enemies that patrol
- [x] Stomp-to-kill mechanic
- [x] HUD (score, lives, beer count)
- [x] Win condition (reach flag)
- [x] Game over (fall or 0 lives)
- [x] Sound effects
- [x] Settings screen
- [x] Guitar shop with 10 guitars
- [x] Each guitar: unique visual, damage, sound
- [x] Coin persistence (localStorage)
- [x] Purchase and equip system

### Nice to Have (Future)
- [ ] Multiple levels
- [ ] Power-ups
- [ ] Moving platforms
- [ ] Background music
- [ ] High score table