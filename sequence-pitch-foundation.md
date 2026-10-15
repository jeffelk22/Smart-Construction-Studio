# SEQUENCE

## A verifiable control system for a construction project's final-delivery risk

### The one-line pitch

**SEQUENCE forecasts whether a construction project will miss its final completion date by detecting the promises in the schedule that are already breaking on site.**

A schedule promises that a design will be ready, an approval will arrive, a material will be delivered and a crew will complete work at a certain rate. The final date moves when these promises fail, often first recorded in diaries, chats, submittal/MAF records and snag lists rather than in the schedule itself.

---

## What this project is — and is not

### It is

- a weekly, project-level early-warning system for the **final contractual / practical-completion date**;
- an evidence trail from local events to schedule risk;
- a tool for a project manager or planner deciding what to escalate this week;
- a feasibility prototype tested on public, traceable schedule history and real but incomplete site records.

### It is not

- an autonomous schedule writer;
- a promise to know every delay cause or replace a planner;
- a generic machine-learning model trained on every type of project;
- a tool that identifies individuals to blame;
- a black-box date invented by an LLM.

---

## The studio pillars

| Studio question | SEQUENCE answer | Evidence for the pitch |
|---|---|---|
| **Where is the problem?** | Stage 3: on-site execution. The final date is reviewed weekly, but evidence of its movement is scattered across programme updates, site records, approvals and messages. | Public schedule-history records plus the UpSite document map; no invented UpSite final outcome. |
| **What is the real problem?** | A project team discovers that the final date is at risk after local slippages have already accumulated. The schedule holds the plan; the earliest warning signs live elsewhere. | Dated examples: late delivery, pending MAF, redesign/recalculation, failed inspection, open snag, blocked predecessor. |
| **Who experiences it?** | Primary user: project manager / planning manager during weekly control. Evidence contributors: site manager, subcontractors, document controller. Affected: client, contractor and eventual occupants. | One interview with a planner or site manager: “When did you know the date had moved?” |
| **What data exists?** | MS Project baseline and updates, task dependencies, actual progress, site diaries, meeting minutes, project chat, MAF/submittal register, test sheets, snag list and remaining-works lists. | Public forecast/actual history for outcome evaluation; UpSite records for extraction and audit. |
| **What can AI generate?** | A dated completion range, risk level, the most influential evidence and questions to investigate. | A time-bounded replay at each public reporting date; no future data exposed. |
| **Why AI, not only a rule?** | The schedule calculation and integrity checks are rules. AI extracts and classifies dated disruption events from unstructured diaries, chats and PDFs, then proposes links to activities for a human to verify. | Benchmark the schedule-risk layer on public outcomes; audit the evidence-extraction layer separately on UpSite. |
| **Where are the limits?** | Missing records, ambiguous language, unrecorded site events and a poor baseline constrain the forecast. It reports a range and confidence, not a guaranteed date. | Measure error in days, early-warning lead time, false alarms and traceability. |
| **Who carries responsibility?** | The project manager owns the decision. SEQUENCE supplies cited evidence; it does not instruct teams to accelerate, reallocate labour or change contract obligations. | Read-only data, source citation and human sign-off in the interface. |

---

## The problem, expressed properly

Do **not** pitch this as: “Construction projects are delayed for many reasons, so we predict delays with AI.” That is too broad and indistinguishable from existing platforms.

Pitch it as:

> Construction teams already have a schedule. What they lack is a reliable, early connection between the small problems reported on site today and the final date they will move tomorrow.

The literature supports the category of problem: reviews repeatedly list poor planning, material shortages or slow delivery, approval delays, coordination failures and design changes among common construction-delay causes. The project will not claim those causes are equally important on every site. It will only test which are extractable from the available UpSite records. [A review identifying 149 delay causes across 97 studies](https://www.sciencedirect.com/org/science/article/abs/pii/S1753837819000627)

---

## Causal map: macro problem to buildable prototype

```text
FINAL COMPLETION DATE MISSED
            │
            ├── 1. baseline plan is not plausible
            │     ├── missing / weak dependencies
            │     ├── unrealistic durations or calendars
            │     └── critical work has little or no usable float
            │
            ├── 2. design and approval assumptions break
            │     ├── solution recalculation or design change
            │     ├── MAF / submittal pending too long
            │     └── late decision or unclear drawing
            │
            ├── 3. procurement assumptions break
            │     ├── material delivery is late
            │     └── substitute or re-approval is needed
            │
            └── 4. execution assumptions break
                  ├── failed test / quality rework / open snags
                  ├── missing labour, access or predecessor work
                  └── slow progress relative to plan
            ↓
ATOMIC MVP: each week, identify documented events that threaten
unfinished critical-path work, then update the project-end-date range.
```

The team should start with only the event categories that are actually recoverable from the UpSite data. A broad causal map is not a broad first prototype.

---

## The system

```text
MS Project schedule     Site diaries / minutes     Chats / MAFs / snags
        │                         │                         │
        └──── schedule facts ─────┼── dated site events ────┘
                                  ↓
                    1. schedule integrity check
                    2. event extraction and classification
                    3. activity / dependency linking
                    4. completion-date range calculation
                                  ↓
               weekly risk briefing for final project date
                                  ↓
              planner verifies evidence and decides response
```

### Agent boundaries

| Component | Job | Must not do |
|---|---|---|
| Schedule parser | Read dates, dependencies, float, progress and revisions from MS Project exports. | Infer facts absent from the schedule. |
| Schedule-integrity checker | Identify weak logic, hard constraints, missing predecessors and suspicious durations. | Call a weak baseline “the cause” of every delay. |
| Document agent | Extract a dated event, category, affected trade/activity, source quote and confidence from a diary, message or MAF. | Convert speculation into a fact or assign blame. |
| Forecast engine | Use remaining work, schedule status and verified events to produce P50/P80-style end-date range. | Produce a single guaranteed completion date. |
| Explanation agent | Explain which evidence moved the forecast and show links to sources. | Calculate the forecast itself. |

---

## What the user sees

> **Project completion forecast — as of 16 October 2024**
>
> Baseline completion: 30 November 2024  
> Current forecast: 12–22 December 2024  
> Confidence: medium — schedule data complete; procurement status partly missing.
>
> **Why the range moved this week**
>
> 1. A MAF associated with a critical predecessor has been open 18 days beyond its planned approval window.  
> 2. A diary records a late material delivery affecting two remaining activities.  
> 3. Three inspection failures created rework on a work package with four days of remaining float.
>
> **Before acting:** confirm the delivery date and whether the MAF actually blocks the predecessor.

This is deliberately different from an alert saying “project delayed.” It says what changed, what evidence supports it, and what still needs an engineer or PM to confirm.

---

## The crucial technical argument

The numerical forecast must never be generated by a language model.

### Non-AI baseline

Start with a transparent forecast based on:

- the most recent schedule finish date;
- actual vs planned progress;
- remaining duration of unfinished critical / near-critical work;
- weekly completion rate / burn-down of remaining activities or snags.

### Added AI feature

The document agent extracts structured, dated events from messy project records:

```text
{ date, event_type, affected_package, possible_activity, source, quote,
  confidence, stated_status }
```

In a future partner pilot, the forecast engine can test whether verified events improve the forecast over a schedule-only baseline. The present prototype does not make that performance claim from UpSite: it lacks the continuous, traceable history required to test it fairly.

### The claim you can honestly make

> “SEQUENCE makes final-date risk verifiable: it tests date-risk forecasting where public outcomes exist, and it makes the scattered evidence from a real site auditable before a planner acts.”

The two parts are deliberately distinct. The public benchmark evaluates forecast quality; UpSite evaluates whether the evidence extraction is accurate and useful.

---

## Positioning in the market

Do not hide the fact that this market already exists.

- **nPlan** offers AI schedule-risk forecasting and describes models trained on 750,000 past projects, alongside a schedule-integrity checker. [nPlan product overview](https://www.nplan.io/products-overview)
- **ALICE** offers AI-based schedule optimisation and scenario exploration for construction planning. [ALICE Technologies](https://www.alicetechnologies.com/home)

Therefore, do not claim that SEQUENCE beats those platforms or is the first to forecast construction risk. Your position is more specific:

> SEQUENCE is an evidence-first project-control layer for a contractor or project team whose critical signals sit in their own schedule, diaries, approvals and chat — and whose forecast must be traceable to those records.

This is a feasibility prototype, not a general machine-learning product. That honesty is a strength.

---

## How we test it honestly

### 1. Public forecast benchmark — NYC Capital Projects Dashboard

[NYC’s public Schedule History and Variance dataset](https://data.cityofnewyork.us/City-Government/Capital-Projects-Dashboard-Citywide-Schedule-Histo/95tx-snak/about_data) records historical estimated and actual project completion dates, reporting periods, phases and revisions. It has 22.5k schedule snapshots and is updated three times per year.

1. Choose projects with more than one historical snapshot and a later actual completion date.
2. At each historical reporting date, expose the system only to rows available then.
3. Compare a transparent baseline — the agency’s current forecast — with a simple SEQUENCE risk rule from prior revision trend, phase and variance.
4. Keep projects separate between development and test; never put the past of the same project in both.
5. Measure error in days, median error, warning lead time between reporting periods and false alarms for a pre-set delay threshold.

The dataset’s stated reason for a forecast change is **not** used to predict that same change. It is a retrospective check on the explanation, otherwise we would leak the answer into the input.

### 2. UpSite evidence audit — not a final-date benchmark

Use the MPP, dated chat, MAF register and remaining-works lists to create 20–30 anonymised evidence cards. A human marks a small truth set as relevant / irrelevant and confirms the category. Measure:

| Measure | Meaning |
|---|---|
| Evidence precision | Is a cited event genuinely relevant and dated correctly? |
| Category accuracy | Did the agent distinguish an approval, material, design, access, quality or coordination event? |
| Traceability | Can a planner open the source and challenge the interpretation? |
| Abstention quality | Does the agent say “needs confirmation” instead of inventing an activity link? |

No error-in-days or causal-delay result is reported for UpSite. The project does not have a complete, traceable outcome history for it.

---

## 10-slide outline

1. **SEQUENCE** — “When will this project really finish?”
2. **The problem** — schedule date vs physical reality; a small site event can move a final contractual date.
3. **The broken-promise map** — planning, approvals/design, procurement, execution.
4. **The moment and the user** — weekly project-control meeting; PM/planner must decide what to escalate.
5. **Two kinds of evidence** — public forecast/actual history for validation; MPP, chats, MAFs and snags for real-site evidence extraction.
6. **The product** — a date range, confidence and evidence — not a black-box prediction.
7. **Why AI / why not only AI** — rules calculate; language model structures unstructured evidence; human verifies.
8. **How it works** — agent harness and strict boundaries.
9. **The test** — public temporal replay against actual completion dates, plus an audited UpSite evidence-extraction test.
10. **Limits and value** — no guarantees, no blame; earlier, traceable escalation for the final project date.

---

## Two-minute spoken pitch

> Construction projects rarely miss their final date because nobody made a schedule. They miss it because the schedule contains promises that slowly stop being true: a material will arrive, an approval will be issued, a solution will not change, a crew will progress at the planned rate. The first signs of those failures appear in site diaries, MAFs, chats and snag lists — not in the final completion date.
>
> Our project is SEQUENCE, an early-warning system for the whole project. Every week, it asks one question: based on the programme and what has actually happened on site so far, can this project still meet its final completion date?
>
> It reads the MS Project programme to understand activities, dependencies and remaining work. It reads diaries, messages and approval records to identify dated disruption events: late deliveries, pending approvals, failed tests, design rework or blocked predecessors. A transparent forecast engine then produces a completion-date range, confidence level and the evidence that moved it.
>
> The AI does not invent the date. The schedule calculations remain explicit. AI is used where the team currently loses time: turning scattered, unstructured project records into traceable signals. The project manager remains responsible for checking whether an event really threatens the critical path and for deciding what to do.
>
> We will not pretend that incomplete UpSite records can prove a final-date prediction. Instead, we test the date-risk forecast on public project histories that contain both successive forecasts and actual completion dates. Separately, we test the hard real-world part on UpSite: can the agent read incomplete chats, approvals and remaining-work lists, turn them into cited evidence, and say clearly when a planner must verify the link? We are not claiming a universal delay predictor. We are building a verifiable way to see a project’s final-date risk earlier, and to see why.

---

## Likely jury questions

| Question | Answer |
|---|---|
| Why not just update MS Project properly? | MS Project holds the programme but not all the earliest evidence: diaries, approval records and project chat. SEQUENCE does not replace update discipline; it makes those records usable in forecast review. |
| Why AI? | Rules already calculate schedule status. AI is useful for extracting comparable, dated events from unstructured records at a scale a planner cannot review weekly. |
| How can one project train a predictor? | It cannot. UpSite does not train the predictor. Public, traceable project histories are used for the first benchmark; UpSite tests evidence extraction. A partner pilot with complete updates is required before we claim evidence improves forecasting. |
| How do you know an event affects the final date? | The system presents a candidate link to unfinished critical or near-critical work. The PM validates it; unconfirmed links remain hypotheses. |
| What happens when it is wrong? | A false warning costs investigation time; a false reassurance is more dangerous. The UI shows a range, evidence and data gaps, and never auto-changes the programme or assigns blame. |

