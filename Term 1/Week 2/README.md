# Term 1 - Week 2: Loops & Functions

**Friday hackathon tool:** N8N  
**SDG:** SDG 3 - Good Health & Well-being

---

## 1. Homework & workshop assignments -> [`homework/`](homework/)

**What was the assignment?**
The Week 2 workshop ("Loops & Functions") focused on repetition, code reusability, modular programming, and critical evaluation of automated rules. The tasks were divided into three guided learning waves and one comprehensive programming assignment:

1. **Exercise 1: Clean Water Countdown (SDG 6)**: Implementing `while` loops to simulate water purification cycles, validating integer input with `while True:` and `break`, and controlling flow using `continue` to skip filter-cleaning cycles without causing infinite loops.
2. **Exercise 2: Vaccination Campaign Tracker (SDG 3)**: Utilizing `for` loops, the accumulator pattern, and `range()` to track daily cumulative vaccinations (250/day over 7 days), followed by a `while` loop to calculate days to reach an arbitrary target, and using `range(start, stop, step)` for mobile clinic schedules.
3. **Exercise 3: Health Risk Screener (BMI & Scope)**: Writing pure functions (`calculate_bmi` and `classify_bmi`) with `def` and `return`, demonstrating function-local scope (catching `NameError` outside the function), and writing an ethical reflection on the risks and accountability of automating decisions using flawed health metrics like BMI.
4. **Big Individual Assignment: Mood Tracker Bot**: Building an interactive 7-day mental health check-in bot requiring:
   - Input validation in a dedicated function (`get_valid_score(day)`) constraining entries to integers between 1 and 10.
   - A 7-day `for` loop that accumulates total mood, renders an ASCII bar chart (`'#' * score`), and tracks the best and hardest days without using lists (via comparison variables).
   - Modular helper functions (`calculate_average` and `give_feedback`) to classify weekly well-being.
   - An optional streak detector counting consecutive good days ($\ge 6$).
   - A critical reflection on the safety, limits, and liability of bots giving mental health advice.

**What did I hand in?**
- `homework/Week2_Workshop_Student.ipynb`: The fully completed Jupyter Notebook containing all working code implementations, edge-case validation, test runs, and written reflections for Exercises 1–3 and the Mood Tracker assignment.

**What did I find difficult, and how did I solve it?**
- **Tracking Extremes Without Data Structures:** The assignment strictly forbade lists for tracking the "best" and "hardest" days. I solved this by initializing sentinel variables outside the loop (`best_score = -1`, `hardest_score = 11`) and updating both the score and corresponding day index (`best_day`, `hardest_day`) using strict inequality (`>` and `<`) within the loop pass. This also established a consistent tie-breaking rule (preserving the earliest occurrence).
- **Infinite Loops with `continue`:** In Exercise 1 Step 3, placing the increment step after `continue` initially resulted in an infinite loop because the counter stopped advancing once it hit cycle 5. I resolved this by placing the counter update (`cycle += 1`) at the very top of the loop body before the condition check.
- **Robust Input Validation:** To prevent crashes from non-integer input (e.g., empty input, letters, or out-of-range floats) without relying on complex try/except blocks early on, I used `.isdigit()` combined with range boundary checks (`1 <= score <= 10`) inside an infinite `while True` loop that only terminates via `return`.

### Checklist
- [x] My workshop / homework files are in `homework/`
- [x] Everything runs without errors, or I explained what does not and why

---


## 2. Hackathon prototype -> [`hackathon/`](hackathon/)

**Project title:** MorningFlow: Context-Aware Well-Being Briefing for Students

**My pair partner:** Sandra Wandel

**Tool used:** N8N

**SDG addressed:** SDG 3 - Good Health & Well-being

**What problem does it solve, and for whom?**
This tool is built for hybrid-learning students who struggle with seasonal morning fatigue, screen lethargy, and aeroallergen sensitivity (pollen/dust). Students typically check weather, calendar, and health data across fragmented apps, often missing the limited daylight windows essential for circadian health because lectures get in the way, or airing out their rooms during peak particulate/pollen spikes. This tool is explicitly *not* for clinical asthma patients, individuals in acute mental health distress requiring professional therapy, or night-shift workers whose circadian rhythms run inverted to daylight cycles.

**What did you build?**
We built an autonomous, scheduled N8N workflow that fires every morning at 07:30 AM to pull real-time weather, UV indexes, and five distinct pollen metrics from Open-Meteo alongside the student's live HHS timetable (via RFC 5545 iCal). Custom JavaScript nodes parse and fuse this data to find open schedule slots, after which an integrated LangChain LLM synthesizes the conditions into a personalized, three-part morning briefing under 120 words. The briefing is pushed directly to the student's Discord with an actionable daylight window, an indoor/outdoor movement and ventilation recommendation, and a grounding mental health affirmation.

**Link to the live thing (if any):**
Exported workflow JSON: [`hackathon/MorningFlow.json`](hackathon/MorningFlow.json)

**How do I run it?**
1. In your N8N canvas, click **Workflows** -> **Import from File** and select `MorningFlow.json`.
2. Attach credentials:
   - Connect an **OpenAI API** credential to the `OpenAI Chat Model` node (or point to an alternative provider like OpenRouter).
   - Connect a **Discord Webhook** credential (or paste a Webhook URL) to the `Send Briefing to Discord` node.
3. In the `My Profile` node, adjust coordinates (default: The Hague) or health focus if desired. In `Fetch Schedule (iCal)`, paste your own timetable iCal link if replacing the HHS demo feed.
4. Click **Test workflow** to run it end-to-end, or toggle the workflow to **Active** to let the schedule trigger run at 07:30 AM daily.

**Who did what?**
- **Sandra Wandel:** Came up with the idea, defined the target student persona, thought about the ethical risks and mitigations, made the presentation, helped improve and test the N8N workflow.
- **Me:** Configured the LangChain `Chain LLM` and OpenAI nodes with prompt constraints, wired the Discord webhook integration.

**Ethical reflection - what are the risks of your tool? Who could it harm?**
Because this automation touches daily health habits, its greatest hazard is an AI hallucination or parsing error that provides bad physical advice-such as telling a pollen-sensitive student to do an intense outdoor workout during an unseasonably high birch pollen or PM2.5 spike, which could trigger an acute respiratory attack. If the workflow is late or goes down due to an API outage, the harm is limited to slight routine disruption because the workflow is purely informational and decoupled from critical wake-up alarms or academic deadlines. We would trust this automation for our own general habits, but automation must stop strictly at non-clinical, contextual lifestyle nudges (daylight timing, room ventilation, light exercise). It must never diagnose, prescribe, or replace medical advice for asthma or clinical depression. To mitigate risk, our prompt strictly forbids medical claims and bounds output to under 120 words; next iterations should add deterministic safety code that bypasses the LLM with an explicit warning banner if AQI or pollen exceeds moderate thresholds.

### Checklist
- [x] Prototype code (or export / workflow file) is in `hackathon/`
- [x] This week's slides are in `hackathon/`
- [x] The prototype actually runs, and I wrote down how to run it
- [x] Ethical reflection written above

---

## 3. Presentation -> [`presentation/`](presentation/)

*Only fill this in for the week your group was selected to present. You need at least **one** of these across the whole term.*

- [ ] My group presented in this week
- [x] Slides are in `presentation/`
- [ ] Proof of the live demo is in `presentation/` (recording, screenshots, or link)

**How did it go? What would I do differently next time?**

---

## 4. Reflection

**What is the most important thing I learned this week?**
The most important takeaway was understanding the conceptual shift from linear scripting to state-driven, modular architecture-both in pure Python and in automation tools like N8N. In Python, I learned how to manage execution flow deliberately: designing loops around strict termination conditions, avoiding side effects through local variable encapsulation (`def`/`return`), and separating input validation from data processing. 

During the hackathon, this exact principle bridged directly into building production-ready automated pipelines. I saw that relying entirely on an LLM to parse messy schedules or interpret environmental thresholds is brittle and unpredictable. The real architectural strength came from using deterministic programmatic logic (custom JavaScript nodes acting as pure functions to unfold RFC 5545 iCal data, offset timezones, and categorize AQI/pollen levels) *before* passing structured context to the generative model. Writing robust loops and pure functions is not just an introductory programming exercise; it is the essential scaffolding required to keep automated and AI-driven workflows deterministic, reliable, and debuggable.

**Where does this connect to "AI for Good"?**
This week connected directly to **SDG 3 (Good Health & Well-being)** by exposing the systemic danger of automating reductive metrics and unconstrained conversational health agents without deterministic guardrails.

In Exercise 3, automating the Body Mass Index (BMI) highlighted how algorithmic efficiency can scale historical bias. Because BMI reduces human health to a simplistic ratio ($kg/m^2$) without accounting for muscle density, bone structure, gender, or ethnic variability, automating clinical screening classifications with hard programmatic thresholds (`if/elif/else`) codifies false positives and systemic exclusions under a veneer of objective computation. Similarly, when building the Mood Tracker bot and the MorningFlow briefing tool, we confronted the legal and ethical liability of automated well-being advice. Probabilistic AI systems cannot perceive clinical distress, distinguish mild seasonal lethargy from severe depressive episodes, or understand acute respiratory vulnerability during high particulate spikes. 

Therefore, true "AI for Good" engineering requires implementing **hard non-AI tripwires**: deterministic boundary checks that proactively restrict the model's scope, prohibit medical diagnostics, and mandate transparent, human-in-the-loop escalation paths (such as automated crisis helpline disclaimers) whenever health-related metrics breach safe parameters.
