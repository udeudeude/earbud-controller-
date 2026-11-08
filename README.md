# Earbud Controller Lab

A browser experiment for using the center button on **wired Apple earbuds** as a controller in iPhone Safari. [Open the experimental test app](https://udeudeude.github.io/earbud-controller-/) (where GitHub Pages is enabled).

The tested method keeps a nearly silent looping audio file playing and registers Media Session action handlers for media-button commands. No additional hardware, microphone permission, or paid services are needed.

## Confirmed experiment on iPhone (iOS 18.7, Safari)

| Physical sequence | Browser command | Observed result |
| --- | --- | --- |
| One center click | `pause` | Ten spaced singles were all detected |
| Double center click | `nexttrack` | Correctly identified in mixed-click test |
| Triple center click | `previoustrack` | Correctly identified in mixed-click test |
| Very rapid consecutive clicks | Can become grouped track commands | Physical click count **cannot be guaranteed** |
| Long press | Siri | Unsuitable as a game control |
| Volume + / − | No usable browser event | Not available as controls |

Commands were also received in the experiment with the screen asleep. This does **not** establish that a WebGL game can continue running while its tab is backgrounded.

## Experiment: count clicks using weighted media commands

**Proposed game mapping:** single click = **1 shot**, double click = **2 shots**, triple click = **3 shots**. This makes a grouped double or triple command useful instead of losing those presses to recenter/menu actions.

The webpage does not receive raw button-down and button-up events. It receives an iOS-interpreted media command, potentially after a multi-click recognition delay. When a click sequence is correctly recognized, compute:

```text
recognized_shots = pause_events + 2 * nexttrack_events + 3 * previoustrack_events
```

**Crucial distinction:** that is the number of **inferred physical presses**, not a guaranteed measured number. Pressing ten times as fast as possible is **not proven** to produce ten shots. Four or more rapid clicks may be grouped unpredictably, omitted, or handled by other media/system features. A `play` callback, media interruption, or spontaneous audio restart must **not** be blindly treated as another physical click.

### How to test the claim

1. Start listening with the lab. Run **10 spaced singles**, **10 rapid singles**, then **mixed clicks**.
2. For each experiment, record the number of physical clicks *you actually pressed*, not just browser callbacks.
3. Copy the event log and total the weighted commands as above. Compare the result with the physical count. Repeat at different speeds.
4. Record latency, unexpected `play`/`pause` events, Siri or call interruptions, and whether the audio carrier stopped.
5. Test again while game audio is playing, and after the phone returns from a locked/backgrounded state. iPhone behavior may vary by iOS version and earbuds.

### In-game firing rate is a separate constraint

Even if iOS reports a triple-click correctly, **three gunshots cannot necessarily occur simultaneously**. In [Dactyl Nightmare: Lucid Dream](https://github.com/udeudeude/Dactyl-Nightmare-Lucid-Dream), the gun currently fires no more than once every **1.8 seconds**. Three recognized shots can be queued and fired in sequence, taking at least about **3.6 seconds from the first shot** if the gun is initially ready.

The game now has optional `fire2` and `fire3` actions and an **all clicks fire (1 / 2 / 3)** preset on its controller settings (see its project branch / pull request). It keeps up to 12 pending shots, drops overflow, and cancels pending shots on exit or interruption. Those are application design decisions, **not** demonstrated iPhone button capabilities or recovered 1991 game mechanics.

**Conclusion:** weighting double and triple clicks is a useful improvement, but it does **not** turn the earbud remote into an unlimited-rate, lossless input device. The lab's rapid-click experiment is the proper way to establish actual reliability.

## Source and implementation

- Experiment: `index.html`, a nearly silent local WAV track and browser `navigator.mediaSession.setActionHandler`.
- Game integration: [Dactyl Nightmare Earbud Remote documentation](https://github.com/udeudeude/Dactyl-Nightmare-Lucid-Dream/blob/main/docs/EARBUD_REMOTE.md).
