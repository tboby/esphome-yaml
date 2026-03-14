## Context

The AtomS3R + Echo Base voice assistant needs a simpler alarm model than a recurring daily alarm tied to fixed hour and minute fields. The new capability is a one-shot alarm scheduled from Home Assistant, persisted locally on the device, and executed locally using the device speaker and controls. The device currently receives its notion of current time from Home Assistant, and that source is acceptable for v1 even though alarms may be missed during time-unavailable periods such as reboots or network recovery.

This change crosses several concerns in the device configuration: persisted alarm state, time comparison, Home Assistant exposure, local audio playback, wake-word handling, and button behavior while ringing. The design must keep the user-facing model minimal while ensuring the device behaves predictably when alarms are replaced, dismissed, or missed.

## Goals / Non-Goals

**Goals:**
- Represent the alarm as a single one-shot scheduled timestamp with a simple runtime state model of `idle`, `armed`, or `ringing`.
- Let Home Assistant schedule, replace, and clear the one-shot alarm using an absolute datetime value that the device rounds down to minute precision.
- Persist the scheduled timestamp across reboot so the device can resume an upcoming alarm after reconnecting and time synchronization.
- Trigger alarm playback locally when the device's current local time reaches the scheduled minute.
- Allow local dismissal during ringing via the hardware button or any wake trigger, without starting normal voice assistant interaction.
- Stop ringing automatically after five minutes and return the device to `idle`.

**Non-Goals:**
- Recurring alarms, weekday masks, snooze, labels, volume controls, or natural-language alarm setup.
- Replay of alarms that were missed because valid time was unavailable or returned after the scheduled minute had passed.
- A full v1 display experience, beyond reserving documented state hooks for later UI work.
- Special handling for DST gaps such as the skipped hour during summer time transition.

## Decisions

### Use a nullable scheduled datetime plus enum state
The alarm will be modeled as one persisted scheduled datetime and one state enum with `idle`, `armed`, and `ringing`.

This matches one-shot semantics better than a recurring alarm model with separate enabled and time fields. A nullable datetime keeps the configuration surface small: a scheduled timestamp means the alarm is armed, and the absence of a timestamp means it is idle. The explicit runtime enum still captures whether the device is currently ringing.

Alternatives considered:
- Separate `enabled` flag and datetime fields: rejected because it creates invalid combinations such as enabled-without-target and complicates one-shot semantics.
- Daily hour/minute alarm fields: rejected because the user wants explicit one-shot scheduling that clears after firing.

### Treat Home Assistant time as the alarm clock source for v1
The device will compare the persisted target against the local time currently provided to the device by Home Assistant. The alarm subsystem will not add independent UTC conversion logic, RTC requirements, or missed-alarm backfill.

This keeps the implementation aligned with the current time source and the accepted failure model: if time is not available or skips past the target, the alarm may be missed. That trade-off is acceptable because the priority is surviving main server downtime at trigger time, not complete network or power isolation.

Alternatives considered:
- Add RTC-backed or SNTP-backed redundancy: deferred because it adds more system complexity than the current scope requires.
- Model everything in UTC and translate locally: rejected because the device can rely on the time semantics already delivered by Home Assistant.

### Round down to minute precision and ignore past timestamps
Incoming scheduled datetimes will be rounded down to the nearest minute. If the resulting target is already in the past relative to the current device time at the moment of setting, the device will ignore it.

Minute precision is enough for the intended alarm behavior, keeps scheduler checks simple, and avoids ambiguity about second-level trigger behavior. Ignoring past timestamps is more predictable than triggering immediately on stale input.

Alternatives considered:
- Preserve second precision: rejected as unnecessary complexity for a simple alarm.
- Trigger immediately when a past timestamp is received: rejected because it creates surprising behavior and weakens the meaning of a one-shot scheduled alarm.

### Clear the schedule on trigger and keep ringing as runtime state
When the target minute is reached, the device will transition to `ringing`, clear the persisted scheduled timestamp, and start local alarm playback. Dismissal or timeout will then move the state back to `idle`.

Clearing the schedule at trigger time prevents duplicate execution if time sync or scheduler evaluation runs again while the device is already ringing. It also ensures the alarm remains one-shot even across reconnects or transient runtime interruptions.

Alternatives considered:
- Keep the scheduled timestamp until dismissal: rejected because it makes duplicate trigger protection harder.

### Make ringing preemptive and dismissal-only
While the alarm is `ringing`, button presses, wake triggers, Home Assistant clears, and Home Assistant replacements will act only on alarm dismissal or replacement. Wake-trigger dismissal will not play wake confirmation audio and will not start a normal assistant turn.

This keeps the interaction model simple: a ringing alarm is an interrupting state that must be silenced first. It avoids accidental transitions into assistant listening or conversation flows while the user is only trying to stop the alarm.

Alternatives considered:
- Only accept an explicit `stop` wake word: rejected because any wake trigger is simpler and more forgiving.
- Let wake-trigger dismissal flow into a normal assistant interaction: rejected because it combines two separate intents and makes ringing behavior less predictable.

### Auto-stop after five minutes and clear missed alarms when they become known to be past
The device will stop ringing after five minutes if the user does not dismiss it. If the device later regains valid time and can determine that an armed alarm target is already in the past, it will clear that alarm without ringing.

This prevents infinite ringing and avoids keeping stale armed alarms around after reboot or network recovery. Because the design intentionally has no grace-period replay, any missed target is simply discarded once the device can detect that it has passed.

Alternatives considered:
- Ring indefinitely: rejected because it is operationally risky and unnecessary.
- Add a grace-period replay window after reboot or reconnect: rejected because the chosen semantics explicitly avoid backfilling missed alarms.

## Risks / Trade-offs

- [Alarm can be missed during time loss, reboot, or DST gap] -> Accept this behavior in v1 and document that the device only triggers when its current local time reaches the scheduled minute.
- [Home Assistant entity support may not map cleanly to nullable datetime plus enum state] -> Prefer that model conceptually, but allow an implementation-layer adapter if ESPHome entity constraints require a different wire representation.
- [Wake-trigger dismissal may occasionally fire from non-target speech or ambient audio] -> Limit wake-trigger handling to the `ringing` state and treat it strictly as dismissal, never as assistant activation.
- [Replacing an alarm while ringing changes active runtime behavior] -> Define replacement explicitly as stop-current-ringing-and-arm-new-target so the behavior is deterministic.
- [No v1 UI means limited local visibility into whether an alarm is armed] -> Keep UI requirements documented for a later revision and rely on Home Assistant as the source of truth for scheduling in v1.
