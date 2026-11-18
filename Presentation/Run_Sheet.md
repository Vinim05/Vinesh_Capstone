# PolyPredict — Presentation Run Sheet and Rehearsal Log

**Project:** Prediction Market Sentiment Trading System (PolyPredict)  
**Team:** Vinesh Murikinati and Charlie Crocker  
**Presentation:** Fall 2026 Capstone Design Presentation  
**Planned speaking time:** 9 minutes 40 seconds (580 seconds; within the 10-minute limit)  
**Presenter and timekeeper:** Vinesh Murikinati  
**Other team member:** Charlie Crocker (not attending the presentation)  
**Recording plan:** Microsoft Teams; recording stored in OneDrive. Vinesh will post the presentation and video links to the team spreadsheet.

## 1. Slide Run Sheet

The planned start times below are **targets**, not measured rehearsal times. Since Vinesh is the only presenter, the hand-off lines are transitions between topics rather than hand-offs between speakers.

| Slide | Title | Speaker | Planned start | Planned duration | Hand-off / transition line |
|---|---|---|---|---|---|
| 1 | Title and Goals | Vinesh | 0:00 | 0:45 | "First, I'll introduce the team and how we've divided the project." |
| 2 | Meet the Team | Vinesh | 0:45 | 0:40 | "Now I'll explain what PolyPredict is designed to do." |
| 3 | Project Abstract | Vinesh | 1:25 | 1:10 | "With that goal in mind, these are the main user requirements." |
| 4 | User Stories | Vinesh | 2:35 | 0:50 | "Next, I'll show how the components work together." |
| 5 | System Architecture (D0) | Vinesh | 3:25 | 1:20 | "That design also has several constraints we needed to consider." |
| 6 | Major Constraints | Vinesh | 4:45 | 1:00 | "I'll now cover where we are in the project." |
| 7 | Current Project Status | Vinesh | 5:45 | 1:10 | "That leads into what we expect to finish this semester." |
| 8 | Fall 2026 Deliverables | Vinesh | 6:55 | 0:55 | "Next is how Charlie and I have divided the work." |
| 9 | Division of Work | Vinesh | 7:50 | 0:55 | "Finally, I'll explain what we plan to demonstrate at Expo." |
| 10 | Expected Expo Demo | Vinesh | 8:45 | 0:55 | "That concludes the design overview. Thank you, and I'm happy to take questions." |
| **Total** | | | | **9:40** | |

**Backup slides:** D1 and D2 detailed-design diagrams and any additional recording/timing information are reference material, not part of the timed presentation. If the submitted deck includes backup slides, do not present them during the main talk unless asked during questions.

## 2. Speaking and Support Roles

| Role | Person | Responsibility |
|---|---|---|
| Presenter | Vinesh Murikinati | Presents all ten main slides and answers questions. |
| Timekeeper | Vinesh Murikinati | Tracks elapsed time and moves to the next slide as needed. |
| Project teammate | Charlie Crocker | Owns the signal generator, storage, and dashboard; will not attend the presentation. |
| Recording and link submission | Vinesh Murikinati | Records in Microsoft Teams, stores the video in OneDrive, and posts both links in the team spreadsheet. |

**Full-group rehearsal requirement:** The November 18 run-through included Vinesh and the mentor, but not Charlie. A rehearsal involving both team members is therefore **still pending**, unless the instructor approves an exception to the full-group requirement.

## 3. Anticipated Questions

| Anticipated question | Owner | Short answer |
|---|---|---|
| Why use Polymarket data? | Vinesh | Event probability changes may help identify stocks or industries to study over time. |
| How does the system detect shifts? | Vinesh | It compares validated observations against defined probability and volume thresholds. |
| Why use a local Qwen model? | Vinesh | Local inference fits the $0 budget and lets us analyze event context without a paid AI service. |
| How are BUY, SELL, and HOLD decided? | Vinesh | BUY when AI analysis and stock checks support a simulated investment; HOLD the position during the fixed six-month tracking period; SELL at the end of six months. Signals that fail the entry rules do not open a position. |
| How do you evaluate long-term performance? | Vinesh | We save each simulated position's entry information and follow actual stock prices over subsequent weeks and months. |
| Why aren't there implementation results yet? | Vinesh | Fall 2026 focused on requirements and design; implementation and testing are planned for Spring 2027. |
| What will the Expo demo show? | Vinesh | It will show selecting an event, detecting a shift, identifying stocks, and starting simulated position tracking. Months-long outcomes will not be available immediately. |
| What happens if the demo fails? | Vinesh | Use a saved walkthrough or clearly labeled screenshots of the planned workflow. |
| Why is Charlie not presenting? | Vinesh | Charlie is unavailable for the presentation; our component ownership remains divided as shown on Slide 9. |

## 4. Timed Rehearsal Log

### Rehearsal 1 — November 18, 2026

| Field | Recorded result |
|---|---|
| Date | November 18, 2026 |
| Rehearsal type | Timed presentation run-through — reported completed |
| Total measured presentation time | **7:12 (432 seconds)** |
| Planned target | **9:40 (580 seconds)** |
| Difference | **2:28 (148 seconds) under target** |
| Presenter | Vinesh Murikinati |
| Timekeeper | Vinesh Murikinati |
| Other attendees | Mentor only ([Mentor name]); Charlie did not attend |
| Format | Not specified |
| Per-section / per-slide times | **Estimated allocation of the measured 7:12 total; individual times were not recorded** |
| Observations | Actual run-through was shorter than the planned 9:40 target. Specific section-level pacing issues were not provided. |
| Timing note | Individual times below are estimates distributed across slides; the 7:12 total is the only confirmed measurement. |

**Rehearsal status:** A **7:12** timed run-through with Vinesh and the mentor is documented. Charlie did not attend, so this was not a full-group rehearsal. The section and slide durations below are estimates, not measured results.

### Per-slide rehearsal timing — estimated breakdown

The durations below are **estimated allocations** of the confirmed 7:12 rehearsal total, based on the relative planned time per slide. They are **not actual stopwatch measurements**.

| Slide | Topic | Estimated rehearsal duration |
|---|---|---|
| 1 | Title and Goals | 0:33 |
| 2 | Meet the Team | 0:30 |
| 3 | Project Abstract | 0:52 |
| 4 | User Stories | 0:37 |
| 5 | System Architecture (D0) | 1:00 |
| 6 | Major Constraints | 0:45 |
| 7 | Current Project Status | 0:52 |
| 8 | Fall 2026 Deliverables | 0:41 |
| 9 | Division of Work | 0:41 |
| 10 | Expected Expo Demo | 0:41 |
| **Total** | | **7:12** |

### Per-section timing — estimated breakdown

| Section | Slides | Planned duration | Estimated rehearsal duration |
|---|---|---|---|
| Introduction and team | 1–2 | 1:25 | 1:03 |
| Abstract and user stories | 3–4 | 2:00 | 1:29 |
| Architecture and constraints | 5–6 | 2:20 | 1:45 |
| Progress and deliverables | 7–8 | 2:05 | 1:33 |
| Work division and Expo | 9–10 | 1:50 | 1:22 |
| **Total** | **1–10** | **9:40** | **7:12** |
