# Term 1 - Week 3: Lists & Dictionaries

**Friday hackathon tool:** Claude API / OpenAI API  
**SDG:** SDG 10 - Reduced Inequalities

---

## 1. Homework & workshop assignments -> [`homework/`](homework/)

**What was the assignment?**
The Week 3 workshop covered ordered collections (**lists**) and key-value mapping (**dictionaries**), demonstrating how data representation directly shapes computational scalability and societal fairness:

1. **Wave 1 – Lists & Inventory Management (Exercise 1):** Practicing zero-based and negative indexing (`supplies[-1]`), measuring length (`len()`), mutating lists via `.append('lentils')`, `.insert(1, 'cooking oil')`, `.remove('pasta')`, in-place sorting (`.sort()`), and checking item membership using the `in` operator combined with `if/else` branching.
2. **Wave 2 – Analysing List Data (Exercise 2 - Wage Gap Analyzer):** Computing cohort salary means using `sum(xs) / len(xs)`, calculating gender wage gap percentages (`(avg_men - avg_women) / avg_men * 100`) rounded to one decimal, concatenating lists with `+`, extracting global extremes using `min()` and `max()`, and filtering entries below the grand mean using a `for` loop with conditional evaluation.
3. **Wave 3 – Dictionaries & Key-Value Lookups (Exercise 3 - City Services Map):** Mapping municipal districts of The Hague to public health facility counts. Contrasting direct indexing (`services['Laak']`) with safe `.get('Ypenburg', 0)` to handle missing keys without runtime crashes (`KeyError`), iterating over `.items()`, identifying the most underserved district using the comparison variable pattern, validating user input with `in`, and writing a critical reflection on how administrative omissions (e.g., omitting Ypenburg's 30,000 citizens) lead to systemic budget neglect.
4. **Big Individual Assignment – Food Bank Manager:** Developing a fully interactive, menu-driven CLI inventory management system:
   - Wrapped in a `while True` loop featuring options 1–5 with strict input validation.
   - Dynamic bar chart visualization via `show_stock(inventory)` using string repetition (`'#' * qty`).
   - Donation intake using the counting pattern: `inventory[item] = inventory.get(item, 0) + qty`.
   - Package distribution that decrements every available item, skips exhausted stock using `continue`, and flags critical shortages (`< 5` units).
   - Diagnostic reporting computing total stock (`sum(inventory.values())`), identifying the lowest-stocked commodity, compiling an urgent shortage list, tracking recipient names (`families_served`), and terminating via `break`.
   - Ethical reflection on distributive justice rules (first-come, first-served vs. acute need).
5. **Wrap-up & API Integration (Exercise 4):** Installing the Anthropic SDK, abstracting client requests into a reusable `ask(question)` function returning string outputs, iterating over a list of food bank domain questions, caching prompt-response pairs into an `answers` dictionary, and analyzing model hallucination vs. factual grounding when querying unindexed districts.

**What did I hand in?**
- [`homework/Week3_Workshop_Student.ipynb`](homework/Week3_Workshop_Student.ipynb): The fully completed script containing all exercise solutions, data analysis logic, Food Bank Manager CLI, API helper functions, and written reflections.

**What did I find difficult, and how did I solve it?**
- **Handling `KeyError` vs. Safe Lookups:** In Exercise 3, directly accessing a missing district (`services['Ypenburg']`) raised an unhandled `KeyError`. I resolved this by integrating `.get(key, fallback)` to provide default fallbacks and employing `if key in dict:` checks before attempting direct lookups.
- **Case Sensitivity and Dynamic Input Normalization:** In the Food Bank Manager, disparate casing (e.g., typing `'Rice'` versus `'rice'`) initially created duplicate dictionary entries. I resolved this by sanitizing all string inputs using `.strip().lower()` prior to key lookups or updates.
- **Inventory Mutability During Distribution:** In menu option 3, decrementing values while iterating over `inventory.items()` required careful state handling. By applying `continue` on items with zero stock, I prevented negative inventory states and triggered dynamic low-stock warnings (`qty < 5`) immediately following deductions.
- **The Counting Pattern:** Learning to replace complex multi-line `if/else` key initialization blocks with the idiomatic pattern `inventory[item] = inventory.get(item, 0) + qty` significantly streamlined donation processing and dictionary mutations.

### Checklist
- [x] My workshop / homework files are in `homework/`
- [x] Everything runs without errors, or I explained what does not and why

---

## 2. Hackathon prototype -> [`hackathon/`](hackathon/)

**Project title:** Cognita — Equal Access Through Autonomous Desktop Assistance

**My pair partner:** Arbër Deda

**Tool used:** Claude API / OpenAI API

**SDG addressed:** SDG 10 - Reduced Inequalities (Target 10.2: Empower and promote the social, economic, and political inclusion of all, irrespective of disability; Target 10.3: Ensure equal opportunities and reduce inequalities of outcome by eliminating discriminatory physical/digital barriers).

**What problem does it solve, and for whom?**
Cognita is built for citizens with severe physical motor impairments and neurodegenerative conditions (such as Multiple Sclerosis, ALS, Parkinson's disease, and ataxia) who are cognitively capable of making decisions but face insurmountable physical exhaustion and fine-motor barriers when navigating complex, fragmented municipal and state portals (e.g., Dutch WMO municipal care applications, UWV WIA disability claims, and SVB PGB personal health budget declarations).

*The Evidence & Scale:*
- **Digital Exclusion:** 2.1 million people in the Netherlands live with moderate to severe physical disabilities (CBS, 2023).
- **Procedural Inequality:** Over 50% face insurmountable physical barriers when navigating fragmented, multi-step public service portals (Ieder(in), 2023).
- **Unclaimed Entitlements:** Up to 20% of eligible citizens abandon complex social applications due to digital exhaustion (Algemene Rekenkamer, 2023).
- **Fine-Motor Penalty:** Individuals with MS or ataxia spend 4× to 10× longer clicking form fields and repeatedly mis-click tiny UI hitboxes (WHO Digital Health Report).

*Who is excluded:*
- Individuals seeking fully unattended, autonomous submissions without human oversight (Cognita enforces human-in-the-loop verification).
- Individuals without established legal entitlements or verified personal records (Cognita navigates and executes forms; it does not fabricate eligibility).

**What did you build?**
Cognita is an autonomous vision-language desktop agent wrapped in a PyQt6 interface. Operating directly at the native OS level across browsers, desktop software, and PDF forms, it converts a single user intent (typed or spoken) into complete, multi-step form executions. The agent continuously captures the screen, normalizes coordinates to a `[0, 1000]` grid, injects verified user identity data, and queries multimodal vision LLMs (Anthropic Claude, OpenAI, Gemini, or DeepSeek) to autonomously click, type, scroll, and tab through multi-page forms. It provides real-time transparency via a live Agent Trace log, a dynamic "What the agent sees" screenshot preview, and audio task completion announcements using local (Supertonic-3) or cloud (ElevenLabs, OpenAI) Text-to-Speech.

**Link to the live thing (if any):**
- Full source code and configuration files: https://github.com/alex24106429/Cognita
- End-to-end demo video: [`presentation/cognita-demo.mp4`](presentation/cognita-demo.mp4) (Demonstrating autonomous completion of a multi-step municipal assistance form for an applicant with SPMS).

**How do I run it?**
1. Ensure Python 3.10+ is installed, then install project dependencies:
   ```bash
   pip install PyQt6 pyautogui openai requests sounddevice numpy python-dotenv
   # Optional: pip install supertonic pywhispercpp
   ```
2. Navigate to the project root and launch the application:
   ```bash
   python src/main.py
   ```
3. Click **Configure API** to select your provider (Anthropic, OpenAI, OpenRouter, Gemini, DeepSeek, or Local) and enter your API key.
4. Click **User Data** to view, edit, or import verified profile details (or click **⚡ Load Example Template** to test with the verified Dutch applicant profile `Marloes van den Berg`).
5. Enter your goal in the Task bar (e.g., *"Fill in the municipal WMO assistance form based on my data"*) and click **Run Agent**.
6. The agent will minimize the window, inspect the desktop via screenshots, and execute the form fields step-by-step. To halt execution at any millisecond, use the **STOP** button or fling the mouse cursor to any screen corner to trigger the PyAutoGUI fail-safe.

**Who did what?**

**Both:**
- Defined the target user persona, architectural pipeline, and SDG 10 alignment.
- Structured the multimodal prompt strategies and tested visual grounding accuracy on mock Dutch administrative portals.
- Collaborated on the PyQt6 GUI layout, fail-safe integration, and Friday hackathon presentation.

**Alex Wrobel:**
- Implemented the core autonomous execution loop, prompt construction, and normalized coordinate translation.
- Built the tool execution engine, supporting `mouse_click`, `click_and_type`, `mouse_scroll`, `type_text`, and `press_hotkey`.
- Integrated multi-provider LLM clients (Anthropic Messages API, OpenAI Chat Completions, Gemini, DeepSeek) and the remote target daemon bridge.
- Implemented the audio synthesis engine supporting local Supertonic ONNX and cloud TTS (OpenAI, ElevenLabs).

**Arbër Deda:**
- Conducted policy and legislative research on Dutch administrative exclusion (CBS, Ieder(in), Algemene Rekenkamer data on WMO, UWV WIA, and SVB PGB).
- Designed the User Data Vault architecture and built the tabbed management dialog (`vault_setup_dialog.py`, `config.py`), including sample profile data schemas.
- Designed and authored the complete presentation slide deck.
- Recorded, edited, and narrated the end-to-end video demonstration of the agent filling out a live municipal form.
- Formulated the ethical reflection framework, focusing on data sovereignty and risk mitigation.

**Ethical reflection - what are the risks of your tool? Who could it harm?**
- **Primary Risk:** The model hallucinating critical personal, legal, or financial figures on official government forms (such as generating an incorrect BSN, misrepresenting medical limitations, or reporting false income).
- **Consequence:** Rejection of social benefits, prolonged delays in obtaining vital assistive devices (e.g., electric wheelchairs, home barrier removals), potential accusations of administrative fraud, and severe financial distress for vulnerable individuals.
- **Three Core Mitigations:**
  1. *Deterministic Context Injection:* The vision LLM is strictly grounded via a local User Data Vault injected directly into the system prompt. The model is explicitly barred from fabricating personal records; if data is missing, it must stop and request clarification.
  2. *Human Oversight & Fail-safes:* Cognita operates strictly under human supervision. The user monitors progress through the live Agent Trace and the "What the agent sees" preview. The agent does not click final "Submit" buttons automatically. Immediate physical abort is guaranteed via PyAutoGUI's hardwired corner fail-safe.
  3. *Data Sovereignty & Local Privacy:* User profile data, medical diagnoses, and identity documents remain strictly local on the client's machine. No personal records are stored on intermediary external agent servers or commercial databases.

### Checklist
- [x] Prototype code (or export / workflow file) is in `hackathon/`
- [x] This week's slides are in `hackathon/`
- [x] The prototype actually runs, and I wrote down how to run it
- [x] Ethical reflection written above

---

## 3. Presentation -> [`presentation/`](presentation/)

- [x] My group presented in this week
- [x] Slides are in `presentation/`
- [x] Proof of the live demo is in `presentation/` (recording, screenshots, or link)

**How did it go? What would I do differently next time?**
The presentation was very well received. The panel and our peers engaged strongly with the problem definition: bridging the physical execution gap for cognitively capable citizens facing severe fine-motor exhaustion. The live slide presentation clearly articulated why simpler alternatives fail (browser auto-fillers cannot navigate multi-step flows, and static RPA breaks when CSS classes change) and highlighted how a vision-language agent adapts dynamically across applications. The recorded demo video clearly showed the agent reading fields and populating complex Dutch municipal forms using verified data from the local vault.

*What we would do differently next time:*
While our mouse-in-corner abort fail-safe provides an effective technical kill-switch, individuals with severe motor tremors cannot reliably fling the mouse to a screen corner in an emergency. In future iterations, we would implement a prominent hardware-switch hotkey (e.g., pressing Spacebar or a dedicated accessibility switch) or voice-activated abort word via continuous local Whisper monitoring to ensure the kill-switch is accessible to our target demographic.

---

## 4. Reflection

**What is the most important thing I learned this week?**
The most critical technical takeaway was understanding the role of structured data representations—lists, dictionaries, and JSON schemas—as the essential interface between deterministic software and probabilistic AI models. Throughout the workshop and hackathon, every AI interaction was governed by these structures: conversational histories are lists of message dictionaries (`[{"role": "user", "content": ...}]`), tool capabilities are defined through nested JSON schema dictionaries, and structured outputs are parsed back into key-value pairs. 

Without rigid data structures to sanitize, filter, and validate inputs, an autonomous agent quickly becomes unpredictable. Learning how to manage dictionary states, employ defensive lookups (`.get()`), and validate local schemas demonstrated that reliable AI systems do not emerge from prompt engineering alone, but from the deterministic programmatic scaffolding that constrains and directs the model.

**Where does this connect to "AI for Good"?**
This week connected directly to **SDG 10 (Reduced Inequalities)** and the fundamental ethics of data representation. As demonstrated in Exercise 3, whoever is omitted from a dataset ceases to exist for the software consuming it; leaving Ypenburg out of a service directory mirrors how real municipal algorithmic resource allocations invisibly disenfranchise uncounted communities.

In Cognita, we applied this lesson to administrative accessibility. Digitized state bureaucracy systematically discriminates against individuals with motor disabilities by demanding continuous, fine-grained physical dexterity across hostile web interfaces. By deploying a vision-language agent grounded entirely in a locally controlled, sovereign data vault, we remove the physical barrier of digital interaction while preserving human autonomy and dignity. AI for Good is realized when automation actively neutralizes structural inequalities without requiring vulnerable communities to sacrifice privacy or control.