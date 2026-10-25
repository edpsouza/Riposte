# Riposte: Game Concept

*A living design document. It describes the game as it is built today, and is updated whenever a
feature lands.*

**Last updated:** 2026-10-24


---

## 1. Pitch

**A fast, timed take on the classic paddle game. Fight through five regions of rivals, grab
upgrades, beat their bosses.**

Riposte is a single-player arcade game. Each run crosses five regions, each with three rivals
and a boss, for twenty one-minute matches. Every opponent in a region shares its ability, and each
boss hides a super ability it unleashes mid-match. Win and you pick an upgrade that lasts the rest
of the run; lose once and the run is over.

| | |
| --- | --- |
| **Genre** | Arcade sports, roguelite run structure |
| **Players** | 1, against the computer |
| **Session** | A run is twenty matches of one minute each, about 25 to 30 minutes with menus |
| **Platforms** | macOS (v0.1 pre-release) and iOS (tested), Windows (export preset) |
| **Engine** | Godot 4.7, GDScript, Compatibility renderer |
| **Tone** | Bright, cozy and playful, with competitive bite |

## 2. Design pillars

1. **Every hit feels good.** Freeze frames, screen shake, sparks, a pitch-climbing hit sound and
   slow motion turn a simple rally into something punchy.
2. **Short and replayable.** One-minute matches and short runs. Losing ends the run, so each match
   matters, but a new run is one tap away.
3. **Rivals with character.** Each opponent is the same AI with different tuning and abilities
   from their region's family, plus a portrait and something to say, so each match plays and
   feels different, and each region has its own flavor.
4. **Plays anywhere.** Keyboard or touch, any screen shape, at the same pace everywhere.

## 3. Core loop

```
Title ─▶ Opponent intro ─▶ One-minute match ─▶ Result
                ▲                                  │
                │                     won ┌────────┴────────┐ lost
                │                         ▼                 ▼
                └──── Pick 1 of 3 upgrades           Run over ─▶ Title
                          (beat the final boss: Champion! ─▶ Title)
```

1. **Title**: Play or Settings.
2. **Intro card**: the run so far, the next opponent's portrait, name, ability and an opening
   line, and their region's creed.
3. **Match**: one minute; most points wins. A tie at the buzzer goes to sudden death.
4. **Result**: the score and the opponent's parting words.
5. **Win**: choose one of three upgrades, then face the next opponent. Beating a region's boss
   clears the region; beat the last one and you're Champion. **Lose**: the run ends, and upgrades are lost.

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

## 5. Regions and opponents

A run crosses five regions in order. Each has three rivals and a boss, one ability they all
share, a creed that sums it up, a palette, its own music, and weather drifting over the court. The
portraits come from the *Boss Portraits* pack, framed by the game in pixel-art cards whose color
climbs with each region: common grey, rare blue, epic purple, and legendary gold for the boss.

| | Frozen Tundra | Sea | Oasis | Volcano | Graveyard |
| --- | --- | --- | --- | --- | --- |
| **Creed** | The cold slows you down. | The tide never runs straight. | Nothing is what it seems. | Everything here burns faster. | The dead take what's yours. |
| **Ability** | Frostbite | Sidewinder | Mirage | Heatwave | Reap |
| **Rivals** | Icicle Duelist, Glacier Knight, Snow Priestess | Ghost Captain, Tidecaller Priestess, Leviathan Warden | Storm Caller, Scimitar Champion, Caravan Master | Magma Wyrmling, Forge Priestess, Wyrm Oracle | Scythe Duelist, Ossuary Golem, Relic Merchant |
| **Boss** | Ice Elemental | Pirate Captain | Bandit Chief | Red Dragon | Lich King (final) |
| **Hidden super** | Deep Freeze | Broadside | Smoke Bomb | Inferno | Undying |
| **Weather** | Snow | Rising bubbles | Blowing sand | Rising embers | Ash |
| **Music** | *Frostbound Path* | *The Silent Lake* | *Arcane Whispers* | *March of Iron* | *Chant of the Fallen* |
| **Boss music** | *Dawn of Blades* | *Banners in the Wind* | *Riders of the Storm* | *Legends of the Flame* | *The Broken Crown* |

All twenty share one AI. It reacts after a delay, aims its return away from the player's paddle,
misses by more the less accurate it is, and drifts back to the center when the ball heads away.
Its **anticipation** sets how well it reads the ball: at 0 it chases where the ball is, at 1 it
heads straight for where it will arrive, bounces included. Each opponent tunes these differently;
bosses also smash some returns.

### Abilities

| Ability | Where | What it does | The counter |
| --- | --- | --- | --- |
| **Frostbite** | Frozen Tundra | Its returns are icy blue. Catch one and your paddle is chilled: **55%** speed for **1.2 s**. | Smash it: a charged hit shatters the ice. *Warm Gloves.* |
| **Sidewinder** | Sea | Its returns weave **40** units either side of their path, like waves. | Read the swell, not the line. *Sea Legs.* |
| **Mirage** | Oasis | Each return fans into three balls, **10°** apart, and only one is real. The decoys shimmer, cast no shadow, and vanish before the middle (45% across). | Spot the shimmer. *True Sight* turns them red. |
| **Heatwave** | Volcano | Its returns leave at **0.7×** speed and heat up to **1.4×** as they reach you, glowing orange to red. | Be in place early. *Heat Shield* caps them at normal. |
| **Reap** | Graveyard | Each goal it scores shortens your paddle by **10%**, up to **3** times. | Score back to restore it. *Soul Anchor* limits it to once. |

### Hidden supers

Each boss's intro card shows its region's ability and hints that it "hides something worse". The
super is unleashed the first time a goal puts you ahead, or with **30 s** left: a freeze frame, a
big shake, a burst of sparks, a line from the boss, and a banner naming the super. It lasts the
rest of the match.

| Boss | Super | What it does |
| --- | --- | --- |
| **Ice Elemental** | Deep Freeze | Every **3rd** return glows with a ring of ice. As it crosses the middle, your paddle freezes solid for **0.45 s** (you can still charge). |
| **Pirate Captain** | Broadside | Every **4th** return fires an iron cannonball alongside it, **20°** apart, at 85% speed. It scores like a ball, but can only be blocked, not returned: blocking breaks it apart. |
| **Bandit Chief** | Smoke Bomb | Its returns vanish in a puff of smoke between **30%** and **65%** of the way across, and reappear. |
| **Red Dragon** | Inferno | Every other return is a full-power smash, on top of Heatwave. |
| **Lich King** | Undying | A bone guard rises on its goal line, blocks a goal, and rises again **18 s** after breaking. |

### Tuning and balance

Every match was simulated against a test player standing in for a decent human: it reacts after 0.18 s, heads for where the ball will arrive, slightly off, aims
its returns and smashes a third of them, falls for a Mirage decoy half the time (never with True
Sight), and can't see a ball in smoke. It holds as many random upgrades as a run would have by
then, up to the cap of 3. A script nudged each opponent's stats toward a target win rate over several rounds of
simulations, then each was confirmed over 150 to 300 matches (within about ±4%).

The targets climb in a **sawtooth**: within a region each rival is harder than the last and the
boss is the peak; the next region opens a little easier than that boss, as a breather, but harder
than the previous region opened. Across the run, region openers fall from 92% to 75% and bosses
from 69% to 53%, while the player gains an upgrade after every win.

| | Reaction | Speed | Aim | Anticipation | Smashes | Test player wins | Target |
| --- | --- | --- | --- | --- | --- | --- | --- |
| **Icicle Duelist** | 0.19 s | 0.92× | 61% | 0.00 |  | 97% | 95% |
| **Glacier Knight** | 0.18 s | 0.96× | 66% | 0.20 |  | 89% | 88% |
| **Snow Priestess** | 0.16 s | 0.99× | 70% | 0.28 |  | 85% | 82% |
| **Ice Elemental** | 0.17 s | 0.97× | 68% | 0.25 | 15% | 79% | 72% |
| **Ghost Captain** | 0.16 s | 1.00× | 72% | 0.33 |  | 90% | 85% |
| **Tidecaller Priestess** | 0.15 s | 1.00× | 72% | 0.36 |  | 90% | 80% |
| **Leviathan Warden** | 0.15 s | 1.02× | 75% | 0.44 |  | 74% | 75% |
| **Pirate Captain** | 0.17 s | 0.95× | 65% | 0.25 | 15% | 60% | 66% |
| **Storm Caller** | 0.20 s | 0.88× | 55% | 0.12 |  | 84% | 80% |
| **Scimitar Champion** | 0.19 s | 0.88× | 55% | 0.07 |  | 81% | 75% |
| **Caravan Master** | 0.17 s | 0.93× | 63% | 0.13 |  | 63% | 70% |
| **Bandit Chief** | 0.19 s | 0.87× | 54% | 0.00 | 15% | 66% | 62% |
| **Magma Wyrmling** | 0.11 s | 1.12× | 70% | 0.34 |  | 77% | 76% |
| **Forge Priestess** | 0.09 s | 1.14× | 71% | 0.36 |  | 65% | 71% |
| **Wyrm Oracle** | 0.09 s | 1.14× | 71% | 0.36 |  | 71% | 66% |
| **Red Dragon** | 0.12 s | 1.08× | 66% | 0.22 | 15% | 61% | 57% |
| **Scythe Duelist** | 0.10 s | 1.12× | 68% | 0.27 |  | 81% | 72% |
| **Ossuary Golem** | 0.10 s | 1.14× | 70% | 0.35 |  | 69% | 66% |
| **Relic Merchant** | 0.09 s | 1.15× | 71% | 0.38 |  | 57% | 60% |
| **Lich King** | 0.10 s | 1.10× | 66% | 0.22 | 15% | 45% | 50% |

Anticipation turned out to be the lever that matters: without it, even a near-perfect AI couldn't
stop well-aimed angled shots. Before the 3-upgrade cap, a late run held a dozen upgrades, and even
maxed-out opponents lost 80–95% of matches to them; the cap is what made the late game balanceable.

**Abilities and supers**, measured on and off in real-run conditions (test player's wins): the
supers each cost 17–27 points (Deep Freeze 90%→73%, Broadside 93%→66%, Smoke Bomb 89%→65%, Inferno
80%→60%, Undying 74%→53%), so they bite without deciding a match. Of the region abilities, Mirage
costs 29 points and the rest barely register against the test player (Frostbite −3, Heatwave −4,
Sidewinder and Reap within noise), partly because it re-reads the ball every frame and doesn't feel
pressure; the Oasis rivals are tuned weaker to balance Mirage out. Worth checking in playtests.

### Serves

The opponent can't concede a point before you've hit the ball. A ball you haven't touched since
it was served is met right away, aimed near the middle of the opponent's paddle, at up to 1000
units/s, and in simulation it never got past. As a guarantee, an untouched ball that did would
bounce off the opponent's goal line instead of scoring. You can still be scored on from a serve.

### Portraits

Each opponent has a framed pixel-art portrait from the *Boss Portraits* pack (itch.io). The
frame's rarity follows the run: **common** for a region's first rival, **rare** for its second,
**legendary** for its boss. The pack can't be redistributed, so the files stay out of the repo,
like the audio; without them each opponent gets a card in their color with their initial.

### Voice

Each opponent has an intro line, three taunts (one shown when they score, never the same twice in
a row), a line for beating you and one for losing, and each boss a line for unleashing its super.

## 6. Upgrades

After each win (except the final boss), the player picks one of three random upgrades they don't
own. They can **hold at most 3** at once (the Goal Shield counts while it's up). With all three
slots full, a pick leads to a second step, **Make Room**: choose one of the held upgrades to swap
out for the new one, or **Keep Yours** (also ESC) to pass on it. The cap keeps a long run
balanceable: holding a dozen upgrades at once made late matches unwinnable for any fair opponent,
so instead each slot is a real decision, like swapping in a region's counter before its boss.
Each can be owned once per run; the Goal Shield can be offered again once it breaks. All upgrades are lost when the run ends. Held
upgrades show as icons under the player's score. Five of them each counter one region's ability,
so a pick can be a bet on what's coming.

| Upgrade | Effect |
| --- | --- |
| **Split Ball** | Your first hit each rally splits the ball in two, 20° apart. Both balls score, either way; the rally ends when the last one scores. |
| **Curving Shots** | Every return you hit bends further from a straight line, even without a smash, so it's harder to read. |
| **Bigger Paddle** | Your paddle is 30% taller. |
| **Goal Shield** | A glowing barrier on your goal line blocks one goal, then breaks. |
| **Magnet Pull** | A ball reaching your paddle up to 40 units past either end snaps onto that end. (It used to bend balls toward the paddle's middle mid-flight, which fought your own movement and killed your aim: it measured as a big handicap.) |
| **Quick Feet** | Your paddle moves 15% faster. |
| **Quick Charge** | Smashes charge twice as fast. |
| **Warm Gloves** | Chills and freezes last half as long (Frostbite, Deep Freeze). |
| **Sea Legs** | Weaving returns weave half as much (Sidewinder). |
| **True Sight** | Mirage decoys glow red, and Smoke Bomb can't hide the ball below half visible. |
| **Heat Shield** | Heatwave returns never speed up past normal. |
| **Soul Anchor** | Reap can only shorten your paddle once. |
| **Heavy Hitter** | Your returns leave 10% faster. |
| **Head Start** | Begin every match 1–0 up. |
| **Focus** | Slow motion kicks in from twice as far. |
| **Underdog** | While you're behind, your paddle is 30% faster. |

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
| **Win** | Confetti; more for a region's boss, and twice as much for the final boss |

The game runs on its own clock, so freeze frames and slow motion slow the match while the effects
keep moving.

## 8. Screens and flow

| Screen | Contents | Ways out |
| --- | --- | --- |
| **Title** | Bobbing title, the controls, **Play**, **Settings** | Play (or Space), Settings, Q to quit (desktop) |
| **Settings** | Music and Effects volume sliders with percentages, **Mute All Sound** | **Back** or Esc returns to the screen that opened it |
| **Intro** | The run's ladder (every rival's portrait under their region, beaten ones ticked, the next highlighted), "Region · Opponent n of 3", "Region Boss" or "Final Boss", portrait, name, ability, the region's creed, opening line | Tap or Space |
| **Match HUD** | Both scores, the opponent's portrait and name, the timer (also the pause button), held upgrades, taunt bubble | Tap the timer or Esc to pause |
| **Pause** | **Resume**, **Settings**, **Quit Match** | Resume (or Esc / Space); Quit Match asks "Quit this match?" with **Keep Playing** / **Quit Match** |
| **Upgrade** | Three cards: icon, name, description; the selected card lifts | Tap a card twice, or ← → and Space |
| **Result** | "You Win!", "Sea Cleared!" (after a region's boss), "Run Over" or "Champion!", the opponent's parting words, the score, how it went (a loss says in which region, and which opponent of 20) | Tap or Space |

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
| **Windows** | Export preset only, untested in Godot |

## 10. Controls

| Action | Keyboard | Touch |
| --- | --- | --- |
| Move | ↑ ↓ or W S | Drag anywhere; the paddle follows your finger's height, no faster than the keys allow |
| Charge a smash | Hold Space | Hold a second finger down |
| Pause | Esc, or click the timer | Tap the timer (shows ❚❚ on touch screens) |
| Choose a menu button | Hover it, or ↑ ↓ ← → / W S A D; the selected button lifts, with pointers either side | (no selection: a button shows it's pressed while your finger is on it) |
| Play, continue | Space (presses the selected button, if any) | Buttons, or tap intro and result cards |
| Choose an upgrade | ← → and Space | Tap a card, then tap it again |
| Back | Esc | Back button |
| Quit the game | Q on menu screens | (the OS) |

## 11. Audio

The music is the *Pixel Fantasy 30 Tracks Music Pack* by AlkaKrab (its seamless `Loops`), and the
effects come from *Cozy FX* by Pizza Doggy. Both licenses allow them in the game but not as
downloadable files, so they stay out of the repo; without them the game uses a generated hit sound
and plays no music. The tracks were chosen by their titles, and are easy to swap.

| Use | Asset |
| --- | --- |
| Menus (title, upgrades, results) | *The Old Tavern* |
| Each region, and its boss | See [Regions and opponents](#5-regions-and-opponents) |
| Paddle hit | `cozy_fx_27` |
| Goal | `cozy_fx_26` |

- A boss's intro card and match play its own theme instead of its region's.
- **Level matching**: the tracks are mastered between −13.8 and −20.6 LUFS, so each is turned to
  **−21 LUFS** using its measured loudness before the mix below.
- 27 tracks and the pack's three short *Fx* stingers are still unused: room for more regions,
  bosses, or win and loss jingles.

- Themes crossfade over **1.2 s** and loop. Settings keeps the music of the screen that opened it.
- The music dips **10 dB** while paused, and bends down to **0.85×** pitch in full slow motion.
- Mix: music sits **4 dB** under −21 LUFS, and the master bus adds **6 dB** with a hard
  limiter at **−1 dB**. A recorded match measures about −24 LUFS.
- **Settings**: Music and Effects volume, and Mute, through dedicated audio buses, remembered between sessions.

## 12. Visual direction

- **Palette**: fixed colors for play: yellow `#F3D55B` for the ball, orange `#FF8C69` for smashes,
  red `#E64646` for warnings. (The original greens remain in the icon.)
- **Region palettes**: the court, HUD, cards, buttons and panels take on a region's palette (court,
  opponent's half, soft and accent colors). The title screen wears the first
  region's, the Frozen Tundra's icy blues with its snow, so a run starts where the title left off.
  A boss's court is 20% darker. The palette eases over
  about a second, so entering a new region visibly shifts the arena on its first intro card. The
  ball, sparks, smashes and goal flashes keep their fixed colors so gameplay reads the same
  everywhere.
- **Type**: one pixel font, *Diary of an 8-bit mage* (Chequered Ink), for all text, always at
  multiples of 8 px (16 for small print, 24 for body, 32 and up for prompts, names and titles) so
  its pixels stay crisp. It's licensed for use in the game only, so it's not in the repo, and
  Poppins Bold stands in without it.
- **Shapes**: on the court, soft and rounded: pill-shaped paddles, drop shadows, glows. The UI is pixel art (see UI).
- **Drawn in code**: the court, ball, paddles, sparks, weather and upgrade icons are all drawn
  procedurally. The opponent portraits are pixel art from the *Boss Portraits* pack (see
  [Portraits](#portraits)).
- **Abilities**: ice blue `#9EDBFF` for Frostbite and Deep Freeze (an icy ball, a frosted paddle);
  decoys shimmer; the Undying guard is a barrier in the opponent's color on their goal line.
- **UI**: one pixel-art look everywhere, from the *Complete UI Essential Pack* by Crusenho (CC BY
  4.0): its Paper theme's torn-parchment buttons and frames, drawn at 2× with
  crisp pixels and 9-sliced to any size. They're turned to greys as they load and colored by the
  current palette: main buttons in the accent, secondary ones in the softer shade, and panels,
  upgrade cards and the HUD timer in a deep shade of the court, so each region recolors the same
  UI. Slider handles, the taunt
  bubble and outlines are square pixel art too.
- **Region grounds**: each region's court is a ground texture from the *Pixel Art Tiles* pack by
  Marauder Games, tiled at 2× in its own colors: snow, water, desert sand, dark magma rock with
  glowing cracks, and violet graveyard stone. Each is dimmed (more on the opponent's half, so the
  two-tone court stays), built into an 8 × 8 tile patch (mixing several of the pack's variants, matched
  to the same average color, like the Tundra's six snows) under broad, soft light and shade, and softened toward
  its average color, so the small 32 px tile doesn't show
  as a repeating pattern and the white paddles and yellow ball stand out. The title shows the
  first region's snow.
- **Pointer**: a pixel-art arrow from the same pack, on desktops.
- **Icon**: pixel art on a 64 px grid, scaled up crisply: the Frozen Tundra's icy court in a gold
  legendary card frame, a ball sent back off the player's paddle with an orange smash trail and a
  white slash arcing round the paddle, the riposte. Drawn in code.

## 13. Ideas and open questions

Not built yet, for discussion:

- More regions from the portrait pack (Sunken, Verdant, Emberwing, Irondeep, Skyvault, Gloomcap,
  Moonveil), or a random pick of regions per run.
- Nine matches is long for one sitting: consider shorter matches early on, or saving progress at
  each region.
- With 8 upgrade picks and 7 upgrades, a full run ends up owning almost everything. More upgrades,
  or picks only after bosses.
- Playtest and tune the abilities; the numbers above are first guesses.
- Ways to stand out between runs: a best-run record, unlockable rivals or upgrades, a daily run.
- Haptic feedback on phones for hits and goals.
- Volume of individual sounds, or a separate setting for the hit sound's pitch climb.
- A tutorial or attract mode for first-time players.
- Test the Windows export, and decide whether to ship it.
- App Store distribution: needs a paid Apple developer account.

## 14. Changelog

| Date | Change |
| --- | --- |
| 2026-10-25 | **Fair powers.** Every upgrade, ability and super was measured in simulation. Upgrades are capped at 3 held, with a Make Room step to swap; Magnet Pull now snaps near-misses onto the paddle and Curving Shots bend shots outward (both used to hurt the player); Quick Feet and Heavy Hitter trimmed. Mirage decoys vanish before the middle and Broadside fires every 4th return, more slowly, as both could decide a match alone. |
| 2026-10-24 | **Riposte.** The game's working title is now Riposte, with a new pixel-art icon in the current look. On 2026-10-25 the app's bundle identifier and the repositories were renamed too. |
| 2026-10-24 | **Balance and fair serves.** A simulated test player measures every match; the rivals and abilities were tuned to a steady climb (97% down to 49% for the final boss, see Tuning and balance). Mirage decoys now vanish past the middle. The opponent can no longer concede a ball you haven't hit. Button text is centered on the paper buttons. |
| 2026-10-24 | **Five regions, balanced.** Twenty opponents tuned by simulation to a sawtooth climb (region openers 92% down to 75% for a decent player, bosses 69% down to 53%), with a new anticipation stat for how well each reads the ball. Every region's ground mixes tile variants, matched in tone, under broad, soft shading so it doesn't look tiled. |
| 2026-10-24 | **Region grounds.** Each region's court is now a dimmed, softened ground texture from the *Pixel Art Tiles* pack (snow, water, sand, magma, graveyard stone) instead of tinted paper. |
| 2026-10-24 | **Title in the first region's colors.** The title and its settings wear the Frozen Tundra's palette and snow instead of the original greens. Secondary buttons now use the softer shade, so main buttons stand out in every palette. |
| 2026-10-24 | **Menu selection.** On desktops, hovering a button or the arrow keys select it (it lifts, with pixel pointers either side) and Space presses it; touch screens show only the pressed state. |
| 2026-10-24 | **Paper court.** The court is tinted paper too, keeping the darker opponent half and each region's colors. |
| 2026-10-24 | **Paper.** The shared UI now uses the pack's Paper buttons and frames instead of Wood buttons and Flat frames, still recolored by the palette. |
| 2026-10-24 | **One look.** The per-region UI skins and the mix of fonts were inconsistent, so all text is now the pixel font, and every screen shares one pixel-art UI (Wood buttons, Flat frames) recolored by the region's palette. Sliders, the HUD timer and the taunt bubble became pixel art too. |
| 2026-10-24 | **Fantasy dressing.** New soundtrack from the *Pixel Fantasy* pack (a tavern tune for menus, a theme per region and per boss), level-matched. Pixel-art UI skins per region (Wood, Stone, Paper, Mystic Wood) for buttons, panels and upgrade cards; a pixel display font; a pixel-art pointer. |
| 2026-10-24 | **Region palettes.** The court and UI recolor to each region's palette (Frosthold blue, Duneveil terracotta, Ashen violet), darker against a boss, easing between them; the title keeps the greens. Ashen's ash is now light, to show on its darker court. |
| 2026-10-24 | **Regions.** The run is now three regions (Frosthold, Duneveil, Ashen) of two rivals and a boss each, nine matches in all, replacing Juke, Echo and Mason. Six abilities in three families (Frostbite, Deep Freeze, Sidewinder, Mirage, Reap, Undying) replace the old tricks; bosses combine their region's two and smash. Pixel-art portraits from the *Boss Portraits* pack replace the drawn faces. The intro card shows the run's ladder and the region's creed, each region has its own music and weather, and beating a region's boss clears it. |
| 2026-10-24 | First public pre-release, **v0.1**, for macOS. |
| 2026-10-24 | Concept document created, covering everything built so far: the core game, the opponent ladder, upgrades, touch controls, screen fitting and pace, goal lines, soundtrack, title and settings screens, and the pause menu with Quit Match. |
