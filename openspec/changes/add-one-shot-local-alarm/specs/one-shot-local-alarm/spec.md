## ADDED Requirements

### Requirement: Device stores a single one-shot local alarm
The device SHALL expose a single one-shot alarm that is controlled from Home Assistant using one nullable scheduled datetime and an alarm runtime state of `idle`, `armed`, or `ringing`. The scheduled datetime SHALL be persisted across reboot.

#### Scenario: Scheduling a future alarm
- **WHEN** Home Assistant sets a scheduled datetime that is in the future
- **THEN** the device SHALL round the scheduled datetime down to the nearest minute
- **AND** the device SHALL persist that rounded scheduled datetime
- **AND** the device SHALL set the alarm state to `armed`

#### Scenario: Clearing the alarm
- **WHEN** Home Assistant clears the scheduled datetime
- **THEN** the device SHALL clear any persisted scheduled datetime
- **AND** the device SHALL set the alarm state to `idle`

#### Scenario: Restoring an upcoming alarm after reboot
- **WHEN** the device boots with a persisted scheduled datetime that is still in the future once valid time is available
- **THEN** the device SHALL restore the alarm state to `armed`

### Requirement: Device ignores invalid scheduled datetimes
The device SHALL ignore any scheduled datetime that resolves to a time in the past.

#### Scenario: Home Assistant sets a past datetime
- **WHEN** Home Assistant sets a scheduled datetime that is in the past relative to the device's current valid local time
- **THEN** the device SHALL ignore the requested alarm
- **AND** the device SHALL NOT enter the `armed` state for that request

### Requirement: Device triggers the one-shot alarm locally at the scheduled minute
The device SHALL compare the persisted scheduled datetime against the device's current local time and SHALL trigger local alarm playback when the scheduled minute is reached. When the alarm triggers, it SHALL become a one-shot runtime event and SHALL no longer remain scheduled.

#### Scenario: Scheduled minute is reached
- **WHEN** the device has a persisted scheduled datetime and its current valid local time reaches that same minute
- **THEN** the device SHALL clear the persisted scheduled datetime
- **AND** the device SHALL set the alarm state to `ringing`
- **AND** the device SHALL start local alarm playback

#### Scenario: Alarm becomes known to be missed
- **WHEN** the device has an `armed` alarm and later determines from valid local time that the scheduled datetime is already in the past
- **THEN** the device SHALL clear the persisted scheduled datetime
- **AND** the device SHALL set the alarm state to `idle`
- **AND** the device SHALL NOT start alarm playback

### Requirement: Device supports local dismissal while ringing
While the alarm state is `ringing`, the device SHALL treat button presses and wake triggers as alarm dismissal actions only.

#### Scenario: Button dismisses ringing alarm
- **WHEN** the alarm state is `ringing` and the user presses the hardware button
- **THEN** the device SHALL stop local alarm playback
- **AND** the device SHALL set the alarm state to `idle`

#### Scenario: Wake trigger dismisses ringing alarm
- **WHEN** the alarm state is `ringing` and the device detects a wake trigger
- **THEN** the device SHALL stop local alarm playback
- **AND** the device SHALL set the alarm state to `idle`
- **AND** the device SHALL NOT play a wake confirmation sound
- **AND** the device SHALL NOT start a normal voice assistant interaction

### Requirement: Home Assistant can stop or replace an active alarm
Home Assistant SHALL be able to stop a ringing alarm by clearing the scheduled datetime and SHALL be able to replace any existing alarm by setting a new scheduled datetime.

#### Scenario: Home Assistant clears a ringing alarm
- **WHEN** the alarm state is `ringing` and Home Assistant clears the scheduled datetime
- **THEN** the device SHALL stop local alarm playback
- **AND** the device SHALL set the alarm state to `idle`

#### Scenario: Home Assistant replaces an armed alarm
- **WHEN** the device already has an `armed` alarm and Home Assistant sets a different future scheduled datetime
- **THEN** the device SHALL discard the previous scheduled datetime
- **AND** the device SHALL persist the new rounded scheduled datetime
- **AND** the device SHALL remain in the `armed` state

#### Scenario: Home Assistant replaces a ringing alarm
- **WHEN** the alarm state is `ringing` and Home Assistant sets a different future scheduled datetime
- **THEN** the device SHALL stop local alarm playback
- **AND** the device SHALL persist the new rounded scheduled datetime
- **AND** the device SHALL set the alarm state to `armed`

### Requirement: Ringing alarm stops automatically after five minutes
The device SHALL stop a ringing alarm automatically after five minutes if it is not dismissed earlier.

#### Scenario: Ringing alarm times out
- **WHEN** the alarm has been in the `ringing` state for five minutes without dismissal
- **THEN** the device SHALL stop local alarm playback
- **AND** the device SHALL set the alarm state to `idle`
