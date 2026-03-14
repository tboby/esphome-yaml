## Why

The AtomS3R + Echo Base voice assistant can already play local alarm audio, but the alarm behavior is not designed as a simple, one-shot capability that Home Assistant can schedule and the device can execute independently. We need a clearer alarm model so a user can arm a single alarm from Home Assistant and trust the device to ring locally even if the main server is unavailable at trigger time.

## What Changes

- Add a one-shot local alarm capability that stores a single absolute target timestamp on the device and executes locally when that minute is reached.
- Expose the alarm to Home Assistant as a schedulable one-shot alarm with a nullable scheduled datetime and a simple state model of `idle`, `armed`, or `ringing`.
- Define local dismissal behavior so a ringing alarm can be stopped by the hardware button, by any wake trigger while ringing, or by Home Assistant disabling or replacing the alarm.
- Define persistence, failure, and timeout behavior, including persisted armed alarms across reboot, ignoring past timestamps, auto-stopping ringing after five minutes, and clearing alarms that become known to be in the past.
- Reserve alarm UI states and presentation for a later version while documenting the intended future states now.

## Capabilities

### New Capabilities
- `one-shot-local-alarm`: Schedule, persist, trigger, dismiss, and clear a single one-shot alarm that rings locally on the device and is controlled from Home Assistant.

### Modified Capabilities

None.

## Impact

- Affects the AtomS3R + Echo Base voice assistant configuration and its local runtime state handling.
- Adds Home Assistant-facing alarm entities or equivalent exposed controls/state for scheduling and observing the one-shot alarm.
- Changes how local audio, wake triggers, and the front button interact while an alarm is ringing.
- Establishes alarm state semantics that later UI work can render on the 128x128 device display.
