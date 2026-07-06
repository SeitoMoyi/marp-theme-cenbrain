---
marp: true
theme: cenbrain
paginate: true
size: 16:9
title: Protocol-aware Acquisition Infrastructure for the rteeg Speech Client
---

# Protocol-aware Acquisition Infrastructure for the rteeg Speech Client

**Short-term progress report**  
Peibin / rteeg speech project  
July 2026

---

# Summary

- Recent work focuses on the **rteeg-speech-client**, not a new neural decoder.
- The contribution is a more **protocol-aware acquisition front end** for neural speech decoding experiments.
- Main improvements:
  - YAML task controls
  - sentence preview and character-level execution
  - separated live/final decoding display
  - queued marker delivery
  - auxiliary audio and screenshot capture
<!-- - Evidence boundary: implementation evidence only; no accuracy, clinical, or formal latency claims yet. -->

---

# Motivation

Neural speech decoding systems depend on more than model inference.

- Visual stimuli must be repeatable.
- Events must be labeled.
- Online outputs must be visible during collection.
- Auxiliary behavioral traces should be alignable with neural data.

**Positioning:** this stage prepares the acquisition interface so later decoding data can be collected in a controlled, observable, and auditable way.

---

# System Context

![width:920px](figures/architecture.svg)

---
<!-- _class: two-cols -->

# Client Role in the Pipeline

<div class="columns">
<div class="col-left">

The speech client is the participant-facing and operator-facing layer.

| Responsibility | Purpose |
|---|---|
| Load task files | define repeatable prompts and timing |
| Present stimuli | control what the participant sees |
| Send markers | label session, sentence, word, or character events |
| Receive decoding messages | expose online feedback |
| Capture audio/screenshots | provide auxiliary audit streams |

</div>
<div class="col-right">

<div class="figure-placeholder">
<div class="figure-title">Acquisition Flow</div>
<div class="mini-flow">
<span>Task YAML</span><span class="arrow">→</span><span>Preview UI</span><span class="arrow">→</span><span>Marker Queue</span><span class="arrow">→</span><span>Server Recording</span><span class="arrow">→</span><span>Decoder Feedback</span>
</div>
<div class="figure-subtitle">Suggested file: <code>figures/acquisition-flow.svg</code></div>
</div>

</div>
</div>

---
<!-- _class: two-cols -->

# Execution Layers

<div class="columns">
<div class="col-left">

| Layer | Main responsibility | Research relevance |
|---|---|---|
| Vue renderer | task state, stimulus presentation, preview overlays, decoding display | participant interaction and visible feedback |
| Electron main process | session discovery, ZeroMQ communication, audio streaming, screenshot capture, IPC routing | communication with recording and processing pipeline |

</div>
<div class="col-right">

<div class="figure-placeholder blue">
<div class="figure-title">Renderer / Main Process</div>
<div class="mini-flow">
<span>Vue Renderer</span><span class="arrow">⇄</span><span>IPC Bridge</span><span class="arrow">⇄</span><span>Electron Main</span><span class="arrow">⇄</span><span>ZeroMQ</span>
</div>
<div class="figure-subtitle">Suggested file: <code>figures/client-layer-separation.svg</code></div>
</div>

</div>
</div>

---
<!-- _class: two-cols -->

# Progress 1: YAML Task Configuration

<div class="columns">
<div class="col-left">

Implemented task definition fields include:

- `preview_time`
- `show_preview_text`
- `record_by_character`

These controls make task timing and collection granularity explicit in configuration.

**Research value:** repeatable stimulus and timing definitions for speech experiments.

</div>
<div class="col-right">

<div class="figure-placeholder">
<div class="figure-title">Task Schema → Runtime</div>
<div class="mock-panel"><strong>YAML</strong><br><code>preview_time</code><br><code>show_preview_text</code><br><code>record_by_character</code></div>
<div style="color:#FB9231; font-size:28px; margin:8px;">↓</div>
<div class="mock-panel"><strong>Runtime object</strong><br>preview state · execution units · marker schedule</div>
<div class="figure-subtitle">Suggested file: <code>figures/yaml-task-schema-flow.svg</code></div>
</div>

</div>
</div>

---
<!-- _class: two-cols -->

# Progress 2: Sentence Preview and Character-level Execution

<div class="columns">
<div class="col-left">

The experiment core can now:

- show a sentence preview with countdown;
- hide preview text while preserving timing;
- split a phrase into character units;
- emit session-, sentence-, and word-level markers during execution.

**Research value:** controlled presentation and fine-grained event labels.

</div>
<div class="col-right">

<div class="figure-placeholder">
<div class="figure-title">Preview Timeline</div>
<div class="timeline-labels"><span>Preview</span><span>Sentence</span><span>Units</span><span>Done</span></div>
<div class="timeline-bar"></div>
<div class="timeline-labels"><span>show/hide text</span><span>start marker</span><span>word/char labels</span><span>final marker</span></div>
<div class="figure-subtitle">Suggested file: <code>figures/preview-character-timeline.svg</code></div>
</div>

</div>
</div>

---
<!-- _class: two-cols -->

# Progress 3: Online Decoding Feedback Display

<div class="columns">
<div class="col-left">

The client separates:

- **live partial decoding** during the current trial;
- **final sentence history** after sentence-level completion.

Main-process topic routing distinguishes partial, word-level, sentence-word, and final sentence decoding messages.

**Research value:** online feedback becomes more observable without claiming it is accurate.

</div>
<div class="col-right">

<div class="figure-placeholder blue">
<div class="figure-title">Decoding Display Mockup</div>
<div class="mock-panel"><strong>Live partial decoding</strong><br><code>... real-time words ...</code></div>
<div class="mock-panel"><strong>Finalized sentence history</strong><br><code>sentence_result[]</code></div>
<div class="figure-subtitle">Suggested file: <code>figures/decoding-display-mockup.png</code></div>
</div>

</div>
</div>

---
<!-- _class: two-cols -->

# Progress 4: Marker Reliability

<div class="columns">
<div class="col-left">

The control client serializes marker sending through a promise queue.

The main process also queues markers generated before the control client is ready and flushes them after initialization.

**Problem addressed:** rapid or premature marker calls can conflict with ZeroMQ request/reply ordering.

**Allowed claim:** the client reduces control-message ordering risk.  
**Not yet claimed:** millisecond synchronization precision.

</div>
<div class="col-right">

<div class="figure-placeholder blue">
<div class="figure-title">Marker Queue Sequence</div>
<div class="mini-flow">
<span>UI marker</span><span class="arrow">→</span><span>Pending buffer</span><span class="arrow">→</span><span>Promise queue</span><span class="arrow">→</span><span>ZMQ control</span><span class="arrow">→</span><span>Server log</span>
</div>
<div class="figure-subtitle">Suggested file: <code>figures/marker-queue-sequence.svg</code></div>
</div>

</div>
</div>

---
<!-- _class: two-cols -->

# Progress 5: Auxiliary Capture Streams

<div class="columns">
<div class="col-left">

The client can collect auxiliary behavioral evidence:

| Stream | Current role | Future validation need |
|---|---|---|
| Audio | mono microphone packets to server-side audio channel | acoustic latency and packet timing |
| Screenshots | timestamped frames with callback information | frame timing against server-side records |

</div>
<div class="col-right">

<div class="figure-placeholder">
<div class="figure-title">Multi-stream Alignment</div>
<div class="mock-panel">Neural data track</div>
<div class="mock-panel">Marker track · audio packets · screenshot frames</div>
<div class="mock-panel">Online decoding messages</div>
<div class="figure-subtitle">Suggested file: <code>figures/multistream-alignment-placeholder.svg</code></div>
</div>

</div>
</div>

---

# Main Progress Areas

| Progress area | Local evidence | Research value |
|---|---|---|
| Task configuration | README, schema, `configUtils.ts` | repeatable stimulus/timing definitions |
| Preview and character mode | `expcore.ts`, `ExperimentView.vue` | controlled presentation and fine-grained labels |
| Online feedback | `index.ts`, `DecodingDisplay.vue` | observable live/final decoding streams |
| Marker queue | `netutils.ts`, `index.ts` | reduced control-message ordering risk |
| Auxiliary streams | `audio.ts`, `screenshot.ts` | behavioral evidence for later audit |

---

# Research Relevance

A later decoder can only be interpreted if the acquisition record shows:

1. which stimulus was shown;
2. when the sentence, word, or character unit began;
3. what online output was visible;
4. which auxiliary behavioral streams were recorded;
5. how markers and session metadata relate to recorded neural data.

This work contributes **infrastructure readiness** before decoder-performance evaluation.

---

# Claim Boundary

| Allowed claim | Claim to avoid |
|---|---|
| The client implements protocol-aware task and marker infrastructure. | The client improves neural decoding accuracy. |
| The online display separates live and final sentence outputs. | The system is a complete clinical speech neuroprosthesis. |
| Queued marker sending reduces ordering risk in the control path. | Marker timing has been validated to millisecond precision. |
| Audio and screenshot streams can support later audit. | Auxiliary streams are already synchronized with neural data. |

---

# Current Limitations

The current materials do **not** include:

- neural decoding accuracy;
- patient or participant outcomes;
- formal marker latency measurements;
- clinical validation;
- validated acoustic or screenshot timing;
- complete end-to-end integration testing with server-side recording.

Therefore, claims should remain implementation-centered.

---
<!-- _class: two-cols -->

# Validation Plan

<div class="columns">
<div class="col-left">

Near-term validation should convert mechanisms into measured evidence.

1. End-to-end marker consistency checks
2. Marker/audio/screenshot latency measurements
3. UI rehearsal for preview and character-level tasks
4. YAML workflow documentation cleanup
5. Integration tests with server-side recording
6. Failure-mode documentation

</div>
<div class="col-right">

<div class="figure-placeholder blue">
<div class="figure-title">Validation Roadmap</div>
<div class="mini-flow">
<span>Implementation evidence</span><span class="arrow">→</span><span>Session rehearsal</span><span class="arrow">→</span><span>Timing benchmark</span><span class="arrow">→</span><span>Server integration</span><span class="arrow">→</span><span>Acquisition-quality evidence</span>
</div>
<div class="figure-subtitle">Suggested file: <code>figures/validation-roadmap.svg</code></div>
</div>

</div>
</div>

---

# Conclusion

The short-term contribution is a more **protocol-aware rteeg speech client** for neural speech decoding experiments.

It integrates:

- task configurability;
- stimulus preview;
- online feedback display;
- marker queueing;
- auxiliary audio and screenshot capture.

Its value is **controlled future data collection**. Its honest boundary is that decoding performance and synchronization precision still require empirical validation.

---

# Context References

- Metzger et al., *Nature*, 2023 — high-performance speech neuroprosthesis context
- Willett et al., *Nature*, 2023 — real-time communication and recording pipeline context
- Card et al., *NEJM*, 2024 — clinical speech BCI context
- Moses et al., *NEJM*, 2021 — speech neuroprosthesis background
- Makin et al., *Nature Neuroscience*, 2020 — neural decoding background

<!-- Replace with a project-specific bibliography slide or QR/link when final references are available. -->
