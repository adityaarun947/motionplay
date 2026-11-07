# MotionPlay — tablet camera → body tracking → games on the TV

Three plain web pages, no build step, no server code.

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

**Fruit Slash** – Your hands are glowing blades with trails. A quick swipe through a fruit slices it. Slicing 3 or more in one flurry gives a combo bonus.
- ❄ **Ice fruit** slows time for 4 seconds.
- ★ **Star fruit** is worth 5 points.
- Bombs and missed fruit each cost one of your 3 lives.

**Lane Runner** – Stand still in the middle for the countdown, which calibrates the game to your body.
- **Step** left or right to change lanes.
- **Jump** over the low red-and-white barriers.
- **Squat** under the yellow-and-black bars.
- Change lanes to get around the blue trains, and collect coins.

**Zombie Siege** – Your hands aim two blasters. Hovering a crosshair over a zombie fires automatically.
- **Normal zombies take 5 hits.** Headshots count double.
- Zombies come in waves, and each wave is bigger. Runners appear from wave 3, brutes from wave 4, and a boss every 5th wave.
- Zombies that reach the fence bite it, and your health drops.
- **Raise both hands above your head** and hold them there for half a second to throw a grenade once the meter is full.
- A medkit appears after each wave. Hover over it to heal.

All games **pause automatically** when you step out of the camera's view, and resume with a 3-2-1 countdown when you're back.

## Testing without the tablet

On a laptop, open `tv.html` and press **K**. Then:
- **← →** change lanes, **↑** jumps, **↓** ducks.
- Use the mouse to slice or aim.
- **G** throws a grenade.
- **1/2/3** pick a game, **P** pauses, **M** opens the menu, **F** goes full screen.
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
