# MotionPlay — tablet camera → body tracking → games on the TV

Three plain web pages and no server code. (The TV page is built from the parts in `src/` by `src/build.sh`; you only need `tv.html`.)

| File | Runs on | Job |
|---|---|---|
| `tracker.html` | Samsung Tab A8 (Chrome) | Camera → MediaPipe pose tracking → filtering, calibration, gestures → sends motion to the TV |
| `tv.html` | TV browser, or a laptop plugged into the TV | Shows a pairing code, receives motion, runs the games |
| `index.html` | Either | Simple "which device is this?" chooser |

## Updating your GitHub copy

When you get a new version, open your `motionplay` repo → **Add file → Upload files**, drag in the new `tracker.html` and `tv.html` (they replace the old ones), and click **Commit changes**. Wait a minute, then reload the page on the TV **and** the tablet.

## How it works

```
Tablet camera ──► MediaPipe Pose (33 body points, on-device)
              ──► One Euro filter (steady when still, almost no lag when moving fast)
              ──► gestures: lane, jump, duck, hands-up + 2 hand cursors (with motion-blur coasting)
              ──► WebRTC data channel (PeerJS) over your Wi-Fi, every frame, timestamped
              ──► TV page: predicts the cursor slightly ahead to hide Wi-Fi delay ──► games
```

## The games

**Fruit Slash** – Your hands are glowing blades with trails. A quick swipe slices a fruit (12 kinds now: banana, strawberry, pineapple, grapes, dragon fruit and more).
- ❄ **Ice** slows time · ★ **Star** = 5 points · ♥ **Heart** gives back a life · ⚡ **Frenzy** = 7 seconds of a fruit storm with double points
- **Giant melon** takes 3 cuts for a big bonus, but costs a life if it escapes
- Slice 10 in a row without a miss for **×2**, 20 for ×3, 30 for ×4
- Every 45 seconds the scene changes: Dojo → Cherry Blossom Garden → Lantern Night Market, each with its own floating details
- Bombs and missed fruit each cost one of your 3 lives

**Lane Runner** – Stand still in the middle for the countdown. Step to change lanes, jump over red-and-white barriers, squat under yellow-and-black bars, dodge trains.
- Four worlds every 500 m: Sunny Meadow, Skyline City, Desert Sunset, Starry Night (with clouds, birds, trees, lamps, cacti…)
- **Checkpoint arches** every 250 m give +1 life (or bonus coins if you are on full lives)
- **Customize** (menu, game-over screen, tablet button, or key C): skin, hair colour, hair/hat style, jacket, trousers, gadget (headphones, scarf, cape, backpack, halo) and a trail (sparkles, fire, rainbow, hearts). Saved on the TV.

**Zombie Siege** – Your hands aim two blasters; hovering a crosshair over a zombie fires.
- Aim is steadier now: it filters your hand more when you slow down to aim, keeps a lock for a moment if your hand wobbles, and gently pulls the crosshair onto a zombie when you are close.
- From wave 2 some zombies **come in from the left and right** (red arrows warn you).
- New zombies: **Crawler** (small, fast), **Spitter** (stays back and lobs acid — shoot the acid ball down), **Armoured** (helmet: headshots are not double), **Bloater** (pops in a blast that hurts the zombies around it). Plus the walkers, runners, brutes and bosses from before.
- Raise both hands above your head for a grenade; hover a medkit to heal. The music gets busier as the waves do.

**Fit Zone** – a guided workout. Choose **Kids** or **Seniors**, and Quick (4 moves) / Normal (6) / Long (8).
- The TV checks the camera can see your whole body, calibrates, then shows a coach demonstrating each move. Your reps are counted from your body and shown in a big ring with stars, effects and a mascot.
- **Kids**: jumping jacks, high knees, big jumps, side dashes, frog squats, reaching, clapping, wipers, windmill arms, super sprint, flamingo balance.
- **Seniors**: always standing, no sitting, only a tiny toe bounce — marching, reach-ups, arm lifts, slow arm circles, gentle side bends, side steps, clapping, balance hold (with a "hold a chair" reminder). Longer rests, no time pressure, calmer music.
- If the camera cannot see your legs, leg moves are counted by how much you are moving (arm moves still work normally).

**Music & sounds** – every game has its own tune (menu, fruit, runner, zombie, kids workout, seniors workout), all made live in the browser. Slicing, jumping and clearing a wave have their own sound effects. **Browsers only allow sound after one touch or key press**, so the first click/OK-button press on the TV turns sound on. Press **N** (or the ♪ Music button on the tablet) to turn music off or on.

All games **pause automatically** when you step out of the camera's view, and resume with a 3-2-1 countdown.

## Testing without the tablet

On a laptop, open `tv.html` and press **K**. Then:
- **← →** change lanes, **↑** jumps, **↓** ducks.
- Use the mouse to slice or aim.
- **G** throws a grenade.
- **1/2/3/4** pick a game, **C** customize runner, **N** music on/off
- In Fit Zone: **R** counts a rep, **S** skips, **Q** quits, **P** pauses, **M** opens the menu, **F** goes full screen.
- **D** shows the TV's frame rate.

## Tuning (top of `tracker.html`, the `CFG` block)

| Setting | Change it if… |
|---|---|
| `laneOn` / `laneOff` | lane changes trigger too easily (raise) or need too big a step (lower) |
| `jumpOn`, `jumpVel` | small bounces count as jumps (raise) or real jumps are missed (lower) |
| `duckOn` | ducking needs too deep a squat (lower) |
| `reachX`, `reachTop`, `reachH` | hand cursors can't reach the screen edges (lower) or feel too twitchy (raise) |
| `handFilter.beta` | cursor lags behind fast swipes (raise) |
| `handFilter.minCutoff` | cursor wobbles when your hand is still (lower) |
| `handHoldMs` | hands vanish mid-swipe (raise) |
| **Accurate mode** checkbox | uses the bigger "full" model: steadier tracking, lower fps on the A8 |
| **Precise hands** checkbox | loads an extra finger-tracking model for steadier aim (slower; the model is fetched from Google at start, if it cannot load the game carries on normally) |
| **Hide preview** checkbox | stops drawing the camera on the tablet, giving a few extra fps |

The **Delay** number on the tablet shows how long motion takes to reach the TV. Under about 80 ms feels instant.

On the TV side, `SLICE_SPEED` (how fast a swipe must be) and `GUN_RATE` (seconds between shots) are near the top of `tv.html`.

## Adding your own game

Each game is one object in `tv.html` under `scenes`, with `enter()`, `update(dt)` and `draw()` methods. Read the player through the shared input helpers:

- `cursors()` → hand positions in screen pixels, with `hd.speed`
- `I.segments` → hand movements this frame, as line segments with speed
- `laneInput()`, `takeJump()`, `duckInput()`, `I.up` (both hands above head)
- `I.sk` → all 33 body points mapped to the screen
- Add `game:true` to get auto-pause for free

## Notes

- Libraries load from CDNs, so both devices need internet. The tablet needs Wi-Fi, but no SIM.
- If the TV's browser struggles, the page automatically lowers its internal resolution to keep the game smooth.

## Tuning the exercises

Exercise counting thresholds are in the `FITCFG` block at the top of the Fit Zone section of `tv.html` (search for `FITCFG`), in torso lengths. If a move is not being counted, lower its number a little; if it counts too easily, raise it. They were tested against simulated body poses, not real people, so expect to tweak them once.
