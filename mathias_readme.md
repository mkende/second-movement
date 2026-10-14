# Personal build (`master` branch)

This branch is the firmware I flash on my own watch (Sensor Watch Pro board,
custom LCD). It is `upstream/main` (joeycastillo/second-movement) plus a linear
stack of feature commits that are not merged upstream yet, followed by a single
"Personal movement config." commit.

It is rebuilt from scratch (not merged) whenever upstream moves, so expect it to
be force-pushed.

## Remotes

- **upstream**: https://github.com/joeycastillo/second-movement, the reference
  firmware.
- **origin**: https://github.com/mkende/second-movement, my fork. Its `main`
  mirrors `upstream/main` and the feature branches below are based on it.
- **alesgenova**: https://github.com/alesgenova/second-movement, source of the
  alarm/stock clock work. Its `rtc-counter32` branch (that we used to depend on)
  is merged upstream as PR #65.

## Branches stacked on top of upstream, in order

My own feature branches are kept squashed into a single commit whose message
describes the current state of the feature (fixes are folded into it rather
than added as new commits), unless there is a reason not to. Branches taken
from other people (alesgenova) keep their original commits and authors.

- **`countdown_autostart`**: automatically starts the countdown once a duration
  was set by tapping the watch. No upstream PR yet.
- **`countdown_indicator_fix`**: follow-up to the merged PR #113, the repeating
  countdown also shows the looped arrows (not the bell) on the custom LCD when
  re-entering the face. No upstream PR yet.
- **`goto_clock`**: `MOVEMENT_JUMP_TO_CLOCK_AFTER_INTERACTION`, pressing Mode
  after having used Light or Alarm on a face goes back to the clock instead of
  the next face. No upstream PR yet.
- **`strong_timeout`**: `MOVEMENT_TIMEOUT_GOES_TO_CLOCK`, faces showing useful
  but static information defer their timeout to the default handler, which goes
  back to the clock when this option is true. Upstream PR #106 (open).
- **`quick_settings`**: `MOVEMENT_HIDDEN_FACE_INDEX`, a set of "hidden" faces
  skipped by Mode and reached by long pressing Alarm on the clock. Upstream
  PR #104 (open).
- **`stock_alarm_improvements`**: alesgenova's `stock-alarm-improvements`
  (upstream PR #84, open): full GW-M5610U style `alarm_face` (4 alarms, snooze,
  hourly chime) and `stock_clock_face` delegating the chime to it. Plus my fix
  for the snooze alarm not wrapping around midnight.
- **`wakeup_any_button`**: alesgenova's `wakeup-any-button` (no upstream PR):
  any button wakes the watch from low energy sleep (not only Alarm), and that
  press is also handled by the current face. It keeps the button interrupts
  enabled during sleep, which may cost a little standby current.

Commits only present on `master`:

- **Also support the "hidden" watch faces in the `stock_clock_face`**: depends on
  both `quick_settings` and `stock_alarm_improvements`.
- **Personal movement config**: face list, `movement_config.h` defaults and the
  countdown default duration (7 min) and tap window (4 s).

## Face layout

- **Primary** (Mode cycles): `stock_clock_face`, `countdown_face`, `alarm_face`,
  `fast_stopwatch_face`, `world_clock_face`.
- **Secondary** (long press Mode on the clock): `tide_face`, `totp_face`,
  `moon_phase_face`, `sunrise_sunset_face`, `temperature_display_face`,
  `voltage_face`.
- **Hidden** (long press Alarm on the clock): `set_time_face`, `settings_face`,
  `finetune_face`, `nanosec_face`.

The TOTP secrets live in `watch-faces/complication/totp_face.secrets.h`, which
is git-ignored and must be kept locally.

## Building

```
make BOARD=sensorwatch_pro DISPLAY=custom
emmake make BOARD=sensorwatch_pro DISPLAY=custom   # simulator, in build-sim/
python3 -m http.server -d build-sim                # then open localhost:8000/firmware.html
```

## Rebuilding after an upstream update

1. `git fetch --all --prune` and fast-forward `main` (and `origin/main`) to
   `upstream/main`.
2. Drop the branches that were merged upstream (`git cherry upstream/main <branch>`
   shows no `+` line) from this file and from the stack.
3. Rebase each remaining feature branch on `upstream/main`, then force-push it
   (this updates its upstream PR).
4. Recreate `master` from `upstream/main` and cherry-pick the branches above in
   order, then the `master`-only commits.
