---
title: Typing, writing, and reading speed as signals of cognitive state
brief: >-
  A processing idea. Use typing speed, writing speed, reading speed, and similar behaviour to
  interpret cognitive awareness. Be very careful: a parallel activity such as a phone call can
  make a value look low, and that must not cause a negative result.
date: 2026-11-20
status: promoted
promoted_to: docs/project.md, section 6.2
---

# Typing, writing, and reading speed as signals of cognitive state

This is an idea for additional processing, not a decision.

## The idea

Check typing speed, writing speed, reading speed, and similar behaviour, to interpret cognitive
awareness. This adds to the conversation evaluation in section 6.1 of the
[project document](../docs/project.md).

## The caution

Evaluating these things can be misleading. The person can do other things at the same time,
for example a phone call. A parallel activity must not cause a negative result.

## Possible approach

- Measure only in valid windows: continuous work in one app, no call or meeting active, no
  frequent app switches.
- A window that is not valid is "not measured". It is missing data, never a low value.
- Detect parallel activity from the other sources: call state on the phone, microphone in use on
  the computer, the meeting recorder, the phone screen.
- Compare each value only with my own values in the same context: the same device, app type,
  language, and keyboard.
- Use the median of many windows for each day, not single values.
- Keep pasted text, autocompletion, and code separate from typed prose.

## Connections

- Computer usage row and section 6.1 of the [project document](../docs/project.md).
- If this becomes part of the design, the computer collector needs microphone state and keyboard
  layout, and the phone app needs call state.
