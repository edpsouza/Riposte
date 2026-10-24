# PongPing: Game Concept

*A living design document. It describes the game as it is built today, and is updated whenever a
feature lands.*

**Last updated:** 2026-10-24

---

## 1. Pitch

**A fast, timed take on the classic paddle game. Climb a ladder of rivals, grab upgrades, beat the
boss.**

PongPing is a short-session, single-player arcade game. Each run is a ladder of three one-minute
matches against AI rivals with their own personalities and tricks. Win and you pick an upgrade
that lasts the rest of the run; lose once and the run is over. It's built to feel great to play:
every hit freezes, shakes, sparks and sings, and it plays as well with a finger on a phone as with
a keyboard on a desktop.

| | |
| --- | --- |
| **Genre** | Arcade sports, roguelite run structure |
| **Players** | 1, against the computer |
| **Session** | A run is three matches of one minute each, about 4 to 6 minutes with menus |
| **Platforms** | macOS (v0.1 pre-release), iOS (in testing), Windows (planned) |
| **Engine** | Godot 4.7, GDScript, Compatibility renderer |
| **Tone** | Bright, cozy and playful, with competitive bite |

## 2. Design pillars

1. **Every hit feels good.** Freeze frames, screen shake, sparks, a pitch-climbing hit sound and
   slow motion turn a simple rally into something punchy.
2. **Short and replayable.** One-minute matches and short runs. Losing ends the run, so each match
   matters, but a new run is one tap away.
3. **Rivals with character.** Each opponent is the same AI with different tuning and one signature
   trick, plus a face and something to say, so each match plays and feels different.
4. **Plays anywhere.** Keyboard or touch, any screen shape, at the same pace everywhere.

## 3. Core loop

```
Title ─▶ Opponent intro ─▶ One-minute match ─▶ Result
                ▲                                  │
                │                     won ┌────────┴────────┐ lost
                │                         ▼                 ▼
                └──── Pick 1 of 3 upgrades           Run over ─▶ Title
                          (beat the boss: Champion! ─▶ Title)
```

1. **Title**: Play or Settings.
2. **Intro card**: the next opponent's portrait, name, trick and an opening line.
3. **Match**: one minute; most points wins. A tie at the buzzer goes to sudden death.
4. **Result**: the score and the opponent's parting words.
5. **Win**: choose one of three upgrades, then face the next opponent. Beat the boss and you're
   Champion. **Lose**: the run ends, and upgrades are lost.

## 4. The match

### Court and rules

- The player defends the **right** side, the AI the **left**. A ball through a side scores for
  the other side.
- Matches last **60 seconds**. The clock turns red and pulses for the final 10 seconds.
- **Sudden death**: tied at 0:00, play continues and the timer reads "Sudden Death" until the next
  goal decides it.
- The court is at least **1280 × 800** units and grows to fill wider or taller screens (see
  [Screens and platforms](#9-screens-and-platforms)).

### Serve and ball

- After each point the ball pops in at the center and serves after **1 s**, at a random 45°
  diagonal, **420** units/s on each axis.
- Walls bounce the ball off the top and bottom.
- **Paddle angle control**: where the ball hits the paddle sets the return angle, from straight
  (center) to **60°** (edges).
- **Rally speed-up**: each paddle hit speeds the ball up **4%**, capped at **1.5×** the serve
  speed. A new serve resets it.
- **Match pressure**: over the minute, the ball's speed ramps up to **1.6×** and the AI's paddle
  speed from **360** to **620**.
- **Slow motion**: when a ball is within **140** units of a paddle it isn't lined up with, the game
  eases down to **30%** speed until the danger passes.

### The charged smash

- Hold **Space** (or a second finger) before the ball arrives to charge. Full charge takes
  **0.6 s**, and the paddle slows to **45%** of its speed at full charge.
- A charged hit sends the ball up to **1.6×** faster and curves it back against its bounce angle,
  so edge hits bend the most and center hits fly straight. The curve fades over time, and a wall
  bounce cancels it.
- A smashed ball turns orange until the next hit. Smashes hit harder: a longer freeze, a bigger
  shake and extra orange sparks.

### Pace on any screen

Movement scales with the court's size: horizontal speeds with its width, vertical with its height.
A crossing takes the same time on a 1280 × 800 desktop and a 2.16:1 iPhone, and distance-based
triggers (slow motion, feints, goal-line warnings) fire at the same moment in a rally.

## 5. Opponents

All three share one AI. It reacts after a delay, aims its return away from the player's paddle,
misses by more the less accurate it is, and drifts back to the center when the ball heads away.
Each rival tunes it differently and adds one trick.

| | Juke | Echo | Mason (boss) |
| --- | --- | --- | --- |
| **Portrait** | A winking fox | A mirrored, two-tone face | A cracked stone block |
| **Color** | Orange | Lavender | Stone grey |
| **Trick** | Feints | Mimics | Wall |
| **Reaction delay** | 0.2 s | 0.15 s | none |
| **Speed** | 0.9× | 1.0× | 2000 against normal shots, 300 against smashes |
| **Aim accuracy** | 60% | 85% | 95% |
| **Return speed** | normal | normal | 0.8× (never below serve speed) |
| **Theme music** | *Chickens In The Meadow* | *Strange Worlds* | *Forgotten Biomes* |

- **Juke, the Feint.** On 70% of returns it lines up for the opposite angle, then switches 220
  units out. The late switch sometimes costs it the ball.
- **Echo, the Mimic.** Returns the ball at the angle the player just used, and smashes back with
  the same charge when the player smashes. On serves there's nothing to copy, so it returns
  straight.
- **Mason, the Wall (final boss).** Reaches every normal shot, but its returns slow the ball. A
  smash catches it flat-footed, and Split Ball's second ball can slip past it.

### Voice

Each rival has an intro line, three taunts (one shown when they score, never the same twice in a
row), a line for beating you and one for losing.

| | Intro | Taunts | Wins | Loses |
| --- | --- | --- | --- | --- |
| **Juke** | "Watch the paddle. Or don't." | "Left. No, right." · "Eyes on me, not the ball." · "Fooled you!" | "Too easy to read." | "Okay, okay. You saw through me." |
| **Echo** | "Show me what you've got. I'll show it back." | "Nice shot. Mine now." · "Everything you do, I do." · "Copy that." | "You beat yourself." | "I can't copy that one." |
| **Mason** | "Nothing gets past me." | "Take your time." · "Slow and certain." · "Like hitting a wall, huh?" | "Patience wins." | "Cracked. Well played." |

## 6. Upgrades

After each win (except the boss), the player picks one of three random upgrades they don't own.
Each can be owned once per run; the Goal Shield can be offered again once it breaks. All upgrades
are lost when the run ends. Held upgrades show as icons under the player's score.

| Upgrade | Effect |
| --- | --- |
| **Split Ball** | Your first hit each rally splits the ball in two, 20° apart. Both balls score, either way; the rally ends when the last one scores. |
| **Curving Shots** | Every return you hit curves, even without a smash. |
| **Bigger Paddle** | Your paddle is 30% taller. |
| **Goal Shield** | A glowing barrier on your goal line blocks one goal, then breaks. |
| **Magnet Pull** | Balls on your half bend toward the center of your paddle. |
| **Quick Feet** | Your paddle moves 25% faster. |
| **Quick Charge** | Smashes charge twice as fast. |

## 7. Game feel ("juice")

| Moment | Feedback |
| --- | --- |
| **Paddle hit** | 0.05 s freeze frame, small screen shake, paddle glow, a spark burst that grows with the rally, and a hit sound that climbs 5% in pitch per hit (up to 2×) |
| **Smash** | Up to 0.1 s freeze, a bigger shake, orange sparks, an orange ball |
| **Wall bounce** | Small green sparks |
| **Goal** | Big shake, sparks from the goal line, the score pops, the goal line flares (gold when you score, red when you're scored on), and the goal sound plays (lower when you're scored on) |
| **Ball nearing a goal** | The goal line lights up in a soft patch at the ball's height, brighter the closer it gets |
| **About to be missed** | Slow motion, with the music bending down in pitch |
| **Ball** | A short trail, a glow, a highlight, and a springy pop-in on each serve |
| **Opponent scores** | A taunt in a speech bubble under their portrait |
| **Win** | Confetti; twice as much for beating the boss |

The game runs on its own clock, so freeze frames and slow motion slow the match while the effects
keep moving.

## 8. Screens and flow

| Screen | Contents | Ways out |
| --- | --- | --- |
| **Title** | Bobbing title, the controls, **Play**, **Settings** | Play (or Space), Settings, Q to quit (desktop) |
| **Settings** | Music and Effects volume sliders with percentages, **Mute All Sound** | **Back** or Esc returns to the screen that opened it |
| **Intro** | "Opponent n of 3" or "Final Boss", portrait, name, trick, opening line | Tap or Space |
| **Match HUD** | Both scores, the opponent's portrait and name, the timer (also the pause button), held upgrades, taunt bubble | Tap the timer or Esc to pause |
| **Pause** | **Resume**, **Settings**, **Quit Match** | Resume (or Esc / Space); Quit Match asks "Quit this match?" with **Keep Playing** / **Quit Match** |
| **Upgrade** | Three cards: icon, name, description; the selected card lifts | Tap a card twice, or ← → and Space |
| **Result** | "You Win!", "Run Over" or "Champion!", the opponent's parting words, the score, how it went | Tap or Space |

Screens fade in and out over the court, and the HUD fades in as the title or a card fades out.
Taps on intro and result cards are ignored for 0.4 s after they appear, so a late tap meant for
the match can't skip them.

## 9. Screens and platforms

- **Any shape**: the project stretches in "expand" mode. The court is at least 1280 × 800 and
  grows to fill the screen, and the HUD and menus re-anchor to it.
- **Notches**: on phones the court keeps its sides inside the safe area, while its colors extend
  to the screen's edges. The full height is used, since the home indicator is just an overlay.
- **Orientation**: landscape, either way up.
- **Pausing**: the game pauses when it loses focus or goes to the background.

| Platform | Status |
| --- | --- |
| **macOS** | v0.1 pre-release, a universal app; ad-hoc signed, not yet notarized |
| **iOS** | Exported and played on an iPhone 12 Pro Max; personal team signing |
| **Windows** | Planned, once it has been tested |

## 10. Controls

| Action | Keyboard | Touch |
| --- | --- | --- |
| Move | ↑ ↓ or W S | Drag anywhere; the paddle follows your finger's height, no faster than the keys allow |
| Charge a smash | Hold Space | Hold a second finger down |
| Pause | Esc, or click the timer | Tap the timer (shows ❚❚ on touch screens) |
| Play, continue | Space | Buttons, or tap intro and result cards |
| Choose an upgrade | ← → and Space | Tap a card, then tap it again |
| Back | Esc | Back button |
| Quit the game | Q on menu screens | (the OS) |

## 11. Audio

The music and effects come from the *Cozy Tunes* / *Cozy FX* pack by Pizza Doggy. Its license
allows them in the game but not as downloadable files, so they stay out of the repo; without them
the game uses a generated hit sound and plays no music.

| Use | Asset |
| --- | --- |
| Menus (title, upgrades, results) | *Golden Gleam* |
| Juke | *Chickens In The Meadow* |
| Echo | *Strange Worlds* |
| Mason | *Forgotten Biomes* |
| Paddle hit | `cozy_fx_27` |
| Goal | `cozy_fx_26` |

- Themes crossfade over **1.2 s** and loop. Settings keeps the music of the screen that opened it.
- The music dips **10 dB** while paused, and bends down to **0.85×** pitch in full slow motion.
- Mix: music sits **4 dB** under its mastered level, and the master bus adds **6 dB** with a hard
  limiter at **−1 dB**. A recorded match measures about −24 LUFS.
- **Settings**: Music and Effects volume, and Mute, remembered between sessions.

## 12. Visual direction

- **Palette**: court green `#26B99A` and dark green `#14A085`, light green `#81CCB8`, yellow
  `#F3D55B` for the ball and highlights, orange `#FF8C69` for smashes, red `#E64646` for warnings.
- **Type**: Poppins Bold throughout.
- **Shapes**: soft and rounded: pill-shaped paddles and buttons, drop shadows, glows.
- **Drawn in code**: the court, ball, paddles, sparks, opponent portraits and upgrade icons are all
  drawn procedurally, so there are no art files to manage.
- **UI**: one shared theme: yellow pill buttons for main actions, outlined ones for the rest,
  cards with soft shadows.
- **Icon**: the court in the game's colors, both paddles, and a smashed ball flying off the right
  paddle with sparks and an orange comet tail.

## 13. Ideas and open questions

Not built yet, for discussion:

- More rivals, or a longer ladder with a mid-boss.
- Ways to stand out between runs: a best-run record, unlockable rivals or upgrades, a daily run.
- Haptic feedback on phones for hits and goals.
- Volume of individual sounds, or a separate setting for the hit sound's pitch climb.
- A tutorial or attract mode for first-time players.
- A Windows release, once it has been tested.
- App Store distribution: needs a paid Apple developer account.

## 14. Changelog

| Date | Change |
| --- | --- |
| 2026-10-24 | First public pre-release, **v0.1**, for macOS. |
| 2026-10-24 | Concept document created, covering everything built so far: the core game, the opponent ladder, upgrades, touch controls, screen fitting and pace, goal lines, soundtrack, title and settings screens, and the pause menu with Quit Match. |
