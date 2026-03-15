## 1. Alarm state and Home Assistant exposure

- [x] 1.1 Add persisted alarm storage for a nullable scheduled datetime and a runtime enum state of `idle`, `armed`, or `ringing`
- [x] 1.2 Expose Home Assistant-facing controls and state for scheduling, clearing, and observing the one-shot alarm
- [x] 1.3 Implement replacement semantics so a newly scheduled alarm overwrites any existing armed alarm and replaces a ringing alarm cleanly

## 2. Scheduling and trigger logic

- [x] 2.1 Implement scheduling logic that rounds incoming datetimes down to the nearest minute and ignores alarms set in the past
- [x] 2.2 Add a simple NTP time source alongside Home Assistant time and treat any valid local time source as sufficient for scheduling
- [x] 2.3 Implement minute-based trigger evaluation that starts local ringing when current device time reaches the scheduled minute
- [x] 2.4 Implement boot and resynchronization handling so future alarms are restored and already-missed alarms are cleared without replay

## 3. Ringing and dismissal behavior

- [x] 3.1 Implement local ringing behavior that clears the scheduled alarm on trigger and auto-stops after five minutes
- [x] 3.2 Make the hardware button dismiss the alarm only while the device is in the `ringing` state
- [x] 3.3 Make wake triggers dismiss the alarm only while ringing, without playing wake confirmation audio or starting a normal assistant interaction
- [x] 3.4 Make Home Assistant clears stop a ringing alarm and return the device to `idle`

## 4. Validation and follow-up

- [x] 4.1 Validate the one-shot alarm flow for schedule, replace, trigger, dismiss, timeout, and clear scenarios
- [x] 4.2 Validate time-source fallback behavior when Home Assistant time is unavailable and NTP provides valid time
- [x] 4.3 Validate reboot behavior for persisted future alarms and missed alarms after time becomes valid again
- [x] 4.4 Document or stub the deferred UI states for `idle`, `armed`, and `ringing` so later display work aligns with the alarm model
