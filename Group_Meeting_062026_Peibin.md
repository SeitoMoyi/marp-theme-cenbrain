---
marp: true
theme: cenbrain
paginate: true
size: 16:9
---

# Graphic Abstract / System Overview

| | Project 1: Movement Decoding | Project 2: Speech Decoding |
|---|---|---|
| **Goal** | 4-direction movement imagery → robot control | Freeform speech imagery → TTS output |
| **Pipeline** | sEEG → Preprocessing → Bandpower → SVM → ZMQ → Robot | sEEG → Offline Decoder → Word Prediction → TTS → Audio |
| **Status** | SVM decoder integrated, ZMQ API validated | Demo client built, end-to-end validated |

**Shared Infrastructure:** Natus EEG ← ZMQ Acquisition ← Pipeline Framework ← sEEG Data

---

# Key Findings & Challenges

## Key Findings / Problems

**Project 1 — Movement Decoding for Robotic Control**
- Analyzer migrated: Jupyter → config-driven package (14 presets)
- Server: SVM decoder and ZMQ robot API integrated
- GUI monitor: real-time phase tracking & confidence bars
- **Problems**: Small dataset (78 trials), baseline drift, multi-session unknown

**Project 2 — Freeform Speech Neuroprosthesis**
- Demo client built & validated end-to-end pipeline
- Real-time decoded word display + TTS utterance
- **Problems**: Limited dataset, latency


---

# Project Comparison — Work Done, Status & Outputs

| Project | Work Done (Last Month) | Current State | Outputs |
|---|---|---|---|
| **P1: Movement Decoding<br> for Robotic Control** | • Jupyter framework<br>• 2 axis ML decoder only | • Package migrated<br>• 14 experiment presets<br>• ZMQ multi-sink output API | **Demo:** Robot movement video<br>**Test:**: robot test |
| **P2: Freeform Speech<br> Neuroprosthesis** | • Demonstration client built | • More features to <br> demonstration client<br>• Real-time word display + TTS | **Code:** Decoding client<br> |


---

# Timeline, Milestones & Expected Outcomes

| Period | Project 1 — Movement Decoding | Project 2 — Speech Decoding |
|---|---|---|
| **Weeks 1–2** *(done)* | ✓ Package migration & evaluation — config-driven analyzer, best model identified | ✓ Hospital visit & demo client — sEEG data, end-to-end validation |
| **Weeks 3-5** *(current)* | Live robotic control test, latency benchmarking | Dataset expansion, additional hospital sessions |
| **Weeks 5-7** | Deep learning baseline and integration | Real-time pipeline integration |
| **Weeks 8–10** | Multi-session generalization, baseline adaptation | Latency optimization, second clinical demo |
