# Endless Runner - Project Brief
**Godot 4 · 2D · Portrait · Google Play**

Senast uppdaterad: 2026-08-02

---

## 1. Spelkoncept

Ett hybrid-casual endless runner med uppgraderingar, power-ups, skins och achievements.
Målet är att publicera på Google Play Store.

### Core Loop
- Spelaren springer automatiskt framåt
- 3 filer (lanes)
- Swipe-kontroller (standard):
  - Swipe vänster/höger → byt fil
  - Swipe upp → hoppa
  - Swipe ner → rulla/slide
- Möjlighet att byta till knappstyrning i Options senare
- Hastigheten ökar i tydliga nivåer (inte kontinuerligt)

### Nivåsystem
- Varje nivå ≈ 1 minut
- Sömlös övergång mellan nivåer (bara "Level X" feedback)
- Paus efter nivå 5, 10 och 15
  - Under pausen kan spelaren se poäng, mynt och uppgraderingar
- Efter paus: 10 sekunders upprampning (hastigheten ökar gradvis)
- Från nivå 15 och framåt: Maxhastighet. Därefter bara score-jakt.

### Power-ups (temporära)
1. Magnet – drar in mynt
2. Sköld – tål 1 träff
3. Dubbla mynt – 2x mynt under en tid
4. Extra liv – som power-up (inte permanent uppgradering från start)

### Meta / Progression
- Enkla uppgraderingar (köps med mynt)
- 3 skins från start (1 standard + 2 upplåsbara) för att testa systemet
- Achievements
- Tre olika scoreboards:
  1. **Raw** – Inga extra liv tillåtna
  2. **Session** – Bara mynt och extra liv från den aktuella sessionen
  3. **Absolute** – Allt tillåtet (inklusive köpta extra liv)

### Analytics & Automation (viktigt)
- Inbyggd loggning av:
  - Sessionlängd
  - Var spelaren dör
  - När folk slutar spela (dag 1, 3, 7, 30)
  - Power-up användning
  - Ad-visningar och avhopp
  - Vad som leder till köp
- Automatiska tester efter iterationer
- AI-spelare som kan köra runs och rapportera buggar
- Målet är att Grok Build ska kunna analysera loggar och föreslå balansändringar

---

## 2. Teknisk Stack

- **Motor:** Godot 4
- **Språk:** GDScript
- **Grafik:** 2D
- **Orientation:** Portrait
- **Export:** Android App Bundle (AAB) till Google Play

### Rekommenderad mappstruktur
```
res://
├── scenes/
│   ├── main/
│   ├── player/
│   ├── obstacles/
│   ├── ui/
│   ├── powerups/
│   └── menus/
├── scripts/
│   ├── player/
│   ├── managers/
│   ├── ui/
│   └── systems/
├── autoload/
│   ├── GameManager.gd
│   ├── SaveManager.gd
│   ├── AnalyticsManager.gd
│   ├── AudioManager.gd
│   └── AchievementManager.gd
├── assets/
├── data/               # JSON för upgrades, achievements, levels
└── tests/
```

### Planerade Autoloads
| Autoload              | Ansvar                                      |
|-----------------------|---------------------------------------------|
| GameManager           | Spelstate, nivå, hastighet, paus            |
| SaveManager           | Mynt, upgrades, skins, highscores           |
| AnalyticsManager      | Sessioner, dödsfall, ads, drop-off          |
| AchievementManager    | Låsa upp och spara achievements             |
| AudioManager          | Ljud och musik                              |

---

## 3. Byggfaser

### Fas 1 – Core Loop (aktuell)
1. Player (rörelse, swipe, hoppa, rulla, 3 filer)
2. Grundläggande hinder + spawning
3. Kamera
4. Enkel poäng + mynt
5. Död → restart

### Fas 2 – Nivåsystem
- LevelManager
- Paus vid 5/10/15
- Upprampning efter paus
- Maxhastighet från nivå 15

### Fas 3 – Power-ups & Meta
- Power-ups
- Enkla uppgraderingar
- 3 skins
- Save/Load

### Fas 4 – UI & Scoreboards
- Meny, pause, death-screen
- Tre scoreboards

### Fas 5 – Achievements + Analytics
- Achievement-system
- Full loggning

### Fas 6 – Automation & Polish
- Automatiska tester
- AI-spelare
- AdMob + grundläggande IAP

---

## 4. Fas 1 – Detaljerad teknisk specifikation

### Player
**Scen:** `scenes/player/Player.tscn`

**Noder:**
- CharacterBody2D (root) – namn: Player
  - CollisionShape2D
  - AnimatedSprite2D (eller Sprite2D temporärt)
  - Area2D (för power-ups senare) + CollisionShape2D

**Script:** `scripts/player/Player.gd`

**Viktiga variabler:**
- `lane: int = 1` (0 = vänster, 1 = mitten, 2 = höger)
- `lane_positions: Array[float]`
- `is_jumping: bool`
- `is_rolling: bool`
- `speed: float` (styrs senare från GameManager)
- `jump_force`
- `roll_duration`

**Funktioner:**
- `change_lane(direction: int)`
- `jump()`
- `roll()`
- Swipe-detektering via InputEventScreenDrag

**Signaler:**
- `died`
- `coin_collected`

### Hinder
Tre typer från start:
- ObstacleLow (måste hoppa)
- ObstacleHigh (måste rulla)
- ObstacleFull (måste byta fil)

Gemensamt script: `Obstacle.gd`
- Rör sig mot spelaren (eller världen scrollar)
- Försvinner när den är bakom kameran
- Signal `hit_player` vid kollision

### Spawning
`ObstacleSpawner` (Node2D)
- Timer-baserad spawning
- Väljer fil + typ av hinder
- Enkel object pooling redan från start

### Kamera
Camera2D som child till Player (enklast i början)

### Poäng & Mynt
Hanteras i GameManager:
- current_score
- current_coins
- distance_traveled

### Död & Restart
När Player skickar `died`:
1. Stoppa spawning
2. Visa enkel DeathScreen
3. "Spela igen" → resetta eller ladda om scenen

### Byggordning inom Fas 1
1. Player + swipe + 3 filer
2. Mark/väg (scrolling background)
3. Ett hinder + kollision
4. Spawner
5. Mynt
6. Poängräkning
7. Död + restart

---

## 5. Designbeslut som är låsta

- 3 filer från start (2/4 som framtida game modes)
- Godot 4 + 2D + Portrait
- Swipe som standardkontroll
- Hoppa + rulla + filbyte från start
- 3 power-ups + Extra liv som power-up
- Nivåer på ca 1 minut
- Paus vid 5/10/15 + 10s upprampning
- Max hastighet från nivå 15
- 3 skins från start
- Tre scoreboards (Raw / Session / Absolute)
- Achievements ingår
- Stark fokus på analytics och automation

---

## 6. Framtida önskemål
- Game modes med 2 och 4 filer
- Jämförelse med andra populära spel via data
- Grok Build ska kunna läsa loggar och föreslå förbättringar
- Så mycket automatisering som möjligt (tester, balansering, etc.)
