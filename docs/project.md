# crsphy-lifelog, project document

Version 0.1, 2026-11-18. Status: design, nothing is built yet.

## 1. Purpose

Record one person (me) from every source that is possible, and use AI to find what to change
so that I think and feel better.

The target is cognitive ability: memory, clear thinking, and focus. Cognition is not isolated
from health, so the project includes health also. Simply, all together.

Examples of what the project must find:

- A food that has a bad effect on me, maybe some days after I eat it.
- The real effect of the vitamins and supplements that I take.
- Habits that change my sleep, for example long video sessions on the phone at night.

These problems are not visible in daily life, because nobody can keep track of all of them.
When the system shows what needs attention, it already helps me to focus better, learn better,
and be more productive.

## 2. Why this project exists

This project comes from [crsphy](https://github.com/akbayt/crsphy). That project trains a model
to predict the next state of an organism, to recover hidden mechanisms. It starts with
simulated plants, and it needs a long path before it can help a person.

This project takes the other route. The reason is **not** that the data of one person is
sufficient to discover deep relations alone. The reason is the large AI models:

- They extract very complex information from raw data.
- They have a world model that is better than many experts in each isolated area. No doctor
  has seen as much data as a large model has seen.
- They can do research in a short time. No person can.

Thus the model does not need to learn medicine from my data. It already knows it. My data is
the evidence that selects between the possible causes for me. This needs much less data than
the crsphy approach.

The expectation is a quicker, practical result for one person, at a fraction of the work.

## 3. Principles

1. **Collect everything that is possible.** Raw data is recorded as it is.
2. **Raw data does not change.** It is stored one time. All processing starts again from it.
3. **Processing is in components.** Each component has a defined input, output, operation, and
   version. The operation is a fixed formula, a machine learning model, a classical statistical
   method, or a language model that reads meaning.
4. **Derived data, not raw data, answers questions.** An agent that reads raw data for each
   question solves the same problem again each time, and gives a different answer each time.
   Example: if the raw data is the position of a particle and the question is about its
   velocity, calculate the velocity one time as a component, and store it.
5. **The pipeline is open.** The raw data stays available. Thus the agent can add a new
   operation, or correct an operation and run it again, at any time.
6. **Local models first.** Local models process the raw data. A remote large model gets only
   data without personal details.
7. **A pattern is a hypothesis until a test confirms it.** A large model writes a convincing
   explanation also when the evidence is weak. The agent is the researcher. My data and small
   experiments are the judge.
8. **One question from start to end before the next source.** This keeps the work on results,
   not on infrastructure.

## 4. The repository is the system

There is no separate server and no separate chat product. I open a coding agent in this
repository. The same agent:

- develops the collectors and the processing code,
- processes the data,
- answers my questions about the data,
- and, in the discussion of the results, improves the processing and the collection.

The repository is the source of truth for the full operation. It knows where the data is, how to
fetch it, and how to process it. A new, better model improves the full system, because the
repository can be used with any model.

### 4.1 What is in the repository

- The code of the collectors and the importers.
- **Skills**, one for each process. A skill says when and how to do a process.
- **Scripts** that the skills call. A script does the calculation. Fixed formulas are always in
  scripts, so the result does not change when the model changes.
- The component definitions and their versions.
- The research log and the hypothesis list.

### 4.2 What is not in the repository

- Raw data, audio, photos, and derived data. They are in object storage, see section 5.2.

Thus the repository can become public later without a data leak.

### 4.3 When the processes run

- **The collectors run always.** They are small programs that do not need the agent.
- **The processing runs when the computer is on**, from a schedule or when I open the agent. If
  the computer was off, the processing starts when it wakes up.
- A **processing record** shows which raw files each component completed. Thus the processing
  continues where it stopped, and it does no work two times.

### 4.4 The limit for a cloud agent

A cloud model sees all that the agent reads. Thus:

- The cloud agent reads only derived data, through one query tool.
- Only local models read raw data.
- Local models remove personal details before data goes to a remote model.

## 5. The pipeline

```
sources ──► collectors ──► raw store ──► components ──► derived data ──► research agent
            (record only)  (unchanged)   (versioned)                     reports, tests
```

### 5.1 Devices

I use the system on more than one computer and on the phone. The data of each device must stay
separate.

- Each installed collector creates a random **device ID** at its first start, and keeps it.
- Each event carries the device ID, a device name ("work laptop"), the platform, and the
  collector version.
- Each event stores the time in UTC and the local time zone.
- Only the components combine the data of different devices.

### 5.2 Storage and transport

The storage and the computer that runs the models are separate. Thus each device can reach the
data, and the model computer can change.

```
phone ──┐
laptop 1 ┼─► encrypt ─► object storage (R2) ─► model computer
laptop 2 ┘   on device   raw/ imports/ derived/   local cache, decrypts, processes
```

1. **Each device writes to its own local files first,** one file for each day. Audio and photos
   are separate files, and an event points to each file. Recording continues without a network.
2. **Each device encrypts the files before upload.** The key stays on my devices. The storage
   provider cannot read the data.
3. **Each device uploads to object storage** over HTTPS. Proposal: Cloudflare R2, because it has
   no fee to download data, and the model computer downloads often.
4. **The model computer downloads what it needs,** decrypts it into a local cache, runs the
   components, and uploads the encrypted derived data.
5. **Raw objects are only added, never changed or deleted.** Each device has its own access key.

Object layout:

```
raw/<device ID>/<source>/<date>/      events, audio, photos
imports/<source>/<date>/              Takeout, Samsung Health, Timeline exports
derived/                              output of the components
```

The repository points to the storage with settings: the bucket, the access key, and the
location of the encryption key. No setting with a secret is committed.

**Backup.** Object storage is not a backup. A second copy of `raw/` and `imports/` is kept in a
different place, for example an external disk, and updated each week.

### 5.3 Event format

Each event has: start, end, device ID, source, signal, value, unit, confidence.

### 5.4 Component types

| Type | Operation | Example |
|---|---|---|
| Measurement | A fixed formula in a script | screen time after 22:00 |
| AI interpretation | A model reads meaning | presence in a meeting |
| Daily state | Combines components into one record for each day | the daily state in section 7 |

For each AI result, the system records the model and the prompt. A change of the model can
change the scores. Without this record, a change of the model looks like a change in me. When
the model changes, it runs again on the old data.

## 6. Sources and derived parameters

Phase marks: **1** initial version, **2** next, **3** end state.
Parameter marks: *(AI)* is an AI interpretation and needs a reference. *(vendor)* is a vendor
value that can change without notice.

| Source | Phase | How | Raw data | Derived parameters |
|---|---|---|---|---|
| Phone usage | 1 | Phone app | App in front, screen on and off, unlocks, notifications, call state | screen time per day, screen time after 22:00, last screen-off before sleep, night pickups, time per app category, video time at night |
| Computer usage | 1 | Computer collector | Active app and window, keyboard and mouse activity, idle periods, microphone in use, keyboard layout, quantity of visible text | active time, deep-work minutes, longest focus block, switch rate, typing speed, correction rate, writing speed, pauses in writing, reading speed, work start and end, activity type *(AI)* |
| Watch | 1 | Samsung Health, Health Connect, phone app | Heart rate, sleep sessions, steps, exercise, blood oxygen, skin temperature | sleep duration, sleep start and end, sleep regularity, resting heart rate, steps, exercise minutes, sleep stages *(vendor)*, heart rate variability *(vendor)* |
| Meetings | 1 | Meeting recorder, local transcription | Audio of all speakers, transcript with speaker and time, context: meeting type, language, number of speakers, my role | speech rate, pause length, filler rate, talk share, conversation evaluation *(AI)*, change across 15-minute segments |
| Food and intakes | 1 | Phone app, photo and one tap | Meal photo with time, taps for caffeine, alcohol, supplements, water | foods and ingredients *(AI)*, rough portion *(AI)*, meal times, eating window, caffeine after 14:00, alcohol units, supplements taken |
| Self reports | 1 | Phone app, one tap | Symptoms (headache, stomach problems, low energy), mood, energy, "Was I sharp?" after a meeting | symptom days, mood level, energy level, reference for the AI scores |
| Face photo | 1 | Phone app, 1 each day | One photo of my face each day, same time, same place, same light | tired look *(AI)*, dark areas below the eyes *(AI)*, swelling of the face *(AI)*, skin colour *(AI)*, skin condition *(AI)* |
| History exports | 1 | Importers, one time, then each month | Google Takeout (YouTube history, location history), Samsung Health export, calendar | night video time in past years, sleep baseline in past years, meeting load per day, time away from home |
| Blood test | 1 | Laboratory, manual entry, 2 each year | Laboratory values, for example vitamin D, B12, ferritin, glucose, thyroid | values out of range, change from the last test, reference for supplements |
| Short cognitive tests | 2 | Phone app, 2 minutes, random times | Reaction times, memory task results | attention score, memory span, time-of-day curve, check of the conversation evaluation |
| Room | 2 | My small devices, sensor bridge | Carbon dioxide, temperature, humidity, light | bedroom air at night, room temperature in sleep, morning light, evening light |
| Supplement experiments | 2 | Phone app selects random on and off periods | Schedule, taps that confirm each intake | effect of each supplement on the daily state, confidence of the effect |
| Voice, all day | 2 | Phone or small microphone, local processing | Continuous audio, processed in the night | speech amount, social time, mood *(AI)*, stress signs *(AI)* |
| Video stream | 3 | Head camera, local processing | Continuous video of the day | activities *(AI)*, eating and drinking events *(AI)*, food without a photo *(AI)*, environment *(AI)* |
| Blood sugar | 3 | Skin sensor | Glucose values, day and night | response to each meal, variation per day, night level |
| Brain signals | 3 | Headband, night first | Electrical brain activity | sleep stages measured at the head, deep sleep minutes, drowsiness *(AI)* |

### 6.1 Conversation evaluation

A state-of-the-art language model reads each conversation and evaluates my cognitive state in
it. It is a full diagnosis of the conversation, not a count of a few fixed signals. Am I
present? Do I keep track of the earlier points? Do I understand what is asked, and answer it?

To make the evaluation stable:

- A fixed list of dimensions. The model gives a score and quotes as evidence for each one.
- A fixed model version and prompt. A sample is scored two times to measure consistency.
- The meeting context is stored, so the analysis compares similar meetings.
- The audio features (pauses, speech rate) are added, because a transcript does not have them.
- One tap after a meeting, "Was I sharp?", and the short cognitive tests, are the references.

### 6.2 Typing, writing, and reading speed

Typing speed, writing speed, and reading speed during normal work show cognitive awareness.
They add to the conversation evaluation, and they come from the computer collector.

| Parameter | How it is calculated |
|---|---|
| typing speed | Keys per minute in continuous typing |
| correction rate | Share of delete keys in typing |
| writing speed | Words per minute of new text in writing apps |
| pauses in writing | Length of pauses before sentences and inside words |
| reading speed | Visible text divided by the time on it, in long text |

These values can be misleading. I can do other things at the same time, for example a phone
call. A parallel activity must not cause a negative result. Thus:

1. **Only valid windows are measured.** A window is valid when the work is continuous in one
   app, no call or meeting is active, and the app switches are few.
2. **A window that is not valid is "not measured".** It is missing data, never a low value.
3. **Parallel activity is detected from the other sources:** call state on the phone,
   microphone in use on the computer, the meeting recorder, and the phone screen.
4. **A value is compared only with my own values in the same context:** the same device, app
   type, language, and keyboard.
5. **The daily value is the median of many windows,** not a single value.
6. **Pasted text, autocompletion, and code are separate** from typed prose.

## 7. The daily state

One record for each day. The research agent looks for links from the inputs to the state, with
a delay of 0 to 72 hours.

| Inputs of the day | State of the day |
|---|---|
| foods | sleep duration |
| meal times | sleep regularity |
| caffeine | deep-work minutes |
| alcohol | switch rate |
| | typing, writing, and reading speed |
| supplements | conversation evaluation *(AI)* |
| screen time at night | mood |
| exercise | energy |
| meeting load | symptoms |
| daylight | face signs *(AI)* |
| room air | resting heart rate |

## 8. The research agent

This is the centre of the system. The agent uses the knowledge of a large model on my data. The
pipeline does not limit it.

The loop of the agent:

1. **Observe.** It reads the daily state and finds what is unusual.
2. **Propose causes.** It uses its knowledge to list possible causes, also causes that I did not
   think of.
3. **Investigate.** It queries the derived data. If a parameter does not exist, it designs a new
   component, and the component runs on the raw data.
4. **Check the processing.** If the data looks inconsistent, it asks for a new run of the
   refinement step, with a corrected operation.
5. **Report.** It writes the finding with its evidence, and proposes a test.

**Components are the memory of the agent.** Each new operation that the agent designs becomes a
stored component with a version. The library of parameters grows, and the same question gets
the same answer the next time.

**The research log** records each question that the agent examined, also those with no result.
The agent does not repeat work, and I can see how many ideas it tried before it found one.

**The hypothesis list** gives each pattern a status: candidate, in test, confirmed, or rejected.
A rejected pattern does not return as a new finding.

**Protection of the components.** The agent that looks for patterns can also edit the code.
Thus each component change is a commit with a reason, and a small fixed data sample must give
the expected result after the change.

## 9. What I get

| Output | Phase | Content |
|---|---|---|
| Weekly report | 1 | What was unusual, the candidate causes with their evidence, one proposed experiment |
| Hypothesis list | 1 | Each pattern with its status |
| Collector monitor | 1 | A warning when a collector stops or a source has a gap |
| Research agent | 1 | Questions and answers from the derived data |
| Experiments | 2 | Random on and off periods for one change, for example a supplement or a screen limit at night |

## 10. Phases

**Phase 1, initial version.**

1. Start the collectors that cost no effort: phone, computer, watch, meetings.
2. Import the history: Google Takeout, Samsung Health export, calendar.
3. Do a blood test.
4. Build the phone app for food photos, the face photo, and one-tap entries.
5. Build the daily state and the weekly report.
6. Answer the first question from start to end:
   **Does phone use at night change my sleep, and my focus on the next day?**
   I already believe the answer. Thus it tests the full pipeline on a known pattern.

**Phase 2, next.** Short cognitive tests, room sensors, supplement experiments, voice all day.

**Phase 3, end state.** Video stream, blood sugar, brain signals. Each measurable signal of my
body and my day is recorded, and the research agent examines it without a fixed limit.

## 11. Risks

| Risk | Control |
|---|---|
| False patterns from many signals and one person | Hypothesis list, research log, small random experiments |
| A convincing explanation with weak evidence | Each finding shows its evidence; the agent proposes a test |
| AI scores change with the model | Record the model and the prompt; run again on old data |
| Vendor values change without notice | Keep the raw signal; mark vendor values |
| A collector stops without notice | Collector monitor |
| Raw data is lost | Backup of the raw data; it cannot be collected again |
| Changes in light make the face photo useless | Same time, place, and light each day |
| A parallel activity makes a speed look low | Valid windows only; a window that is not valid is missing data |

## 12. Open decisions

1. The phone system. This document assumes Android, because of the Samsung watch.
2. The systems of the two computers.
3. The computer that runs the local models, and its power.
4. The final list of the daily state.
5. Which operation uses a local model, and which uses a remote model.
6. The confirmation of the first question.

## References

- Design page with all sources and parameters:
  <https://claude.ai/artifact/4kvfZuk6FL3iBWXefqCjho> (private)
- Origin of the idea: [crsphy](https://github.com/akbayt/crsphy), notes
  `2026-08-19-no-training-in-context-pattern-finding.md`,
  `2026-08-22-consumer-wearable-data-assessment.md`,
  `2026-11-13-project-review.md`, `2026-11-15-pivot-to-self-tracking-one-person.md`.
