# Term 1 - Week 1: Python Basics & Flow Control

**Friday hackathon tool:** Bolt.new (or Lovable)  
**SDG:** SDG 4 - Quality Education

---

## 1. Homework & workshop assignments -> [`homework/`](homework/)

**What was the assignment?**
The assignment covered core Python fundamentals and algorithmic decision-making across three structured learning waves, culminating in a standalone carbon footprint assessment tool:
1. **Wave 1 – Python as a Calculator:** Practicing arithmetic operators (`+`, `-`, `*`, `/`), understanding floor division (`//`), modulo (`%` for odd/even checks), exponentiation (`**`), operator precedence, and calculating real-world flight CO2 emissions.
2. **Wave 2 – Variables and Types:** Assigning variables using Python naming conventions (snake_case), working with the four fundamental data types (`str`, `int`, `float`, `bool`), explicit type casting, managing string concatenation, and taking dynamic input via `input()` to calculate personal AI energy consumption (kWh).
3. **Wave 3 – Making Decisions:** Evaluating boolean expressions (`and`, `or`, `not`), distinguishing assignment (`=`) from equality comparison (`==`), and structuring conditional logic with `if` / `elif` / `else` blocks to classify loan risk based on income and age, paired with an ethical reflection on automated bias.
4. **Big Individual Assignment – "Green or Not?":** Synthesizing all concepts into an interactive carbon footprint calculator that prompts the user for annual flight counts, driving distances, and dietary habits, computes total kg of CO2, classifies the footprint against a sustainable budget benchmark (2500 kg threshold), and prints a formatted diagnostic report.

**What did I hand in?**
- [`homework/Week1_Workshop.ipynb`](homework/Week1_Workshop.ipynb): The complete Jupyter notebook containing all code cells, outputs, and written markdown answers for Waves 1–3 and the "Green or Not?" calculator.

**What did I find difficult, and how did I solve it?**
- **Shadowing Built-in Identifiers:** In Section 2.2, reassigning `max = 15` overwrote Python’s native `max()` function, throwing a `TypeError: 'int' object is not callable` when trying to find the highest value in a list. I solved it by learning that built-in names must never be used as variable identifiers, commenting out the reassignment, and restarting the kernel to restore the namespace.
- **Data Type Casting from `input()`:** `input()` invariably captures user responses as strings (`str`). During both the AI energy calculation and the "Green or Not?" task, attempting direct arithmetic or comparisons led to string duplication (e.g., `'1258' + '1258'` evaluating to `'12581258'`). I solved this by explicitly casting numerical inputs using `int()` and parsing yes/no text into a boolean using `.lower() == 'yes'`.
- **String Formatting & Concatenation:** Concatenating numeric results into print statements required wrapping variables in `str()` and handling floating-point decimals. I solved this by applying `round()` to numeric totals before casting them into output strings.

### Checklist
- [x] My workshop / homework files are in `homework/`
- [x] Everything runs without errors, or I explained what does not and why
---


## 2. Hackathon prototype -> [`hackathon/`](hackathon/)

**Project title:** Knowledge Runner

**My pair partner:** Domilė Žukaitė

**Tool used:** Bolt.new

**SDG addressed:** SDG 4: Quality Education (Target 4.4 – relevant cognitive skills for academic and vocational success; Target 4.7 – accessible, self-directed educational technology that removes the barrier of expensive private tutoring).

**What problem does it solve, and for whom?**
It solves the Active Recall Deficit and digital attention fragmentation for high school/college students (aged 15–24) and independent self-learners revising technical, fact-heavy topics (e.g., AI Ethics, organic chemistry, vocabulary). 

84% of college students rely primarily on passive re-reading and text highlighting (Kornell & Bjork), which creates an illusion of competence ("familiarity") rather than true retrieval strength. However, scientific evidence shows that active retrieval practice boosts long-term conceptual retention by over 400% compared to repeated study (Karpicke & Roediger, *Science*, 2008). Compounding this, average digital screen focus before task switching has dropped to just 47 seconds (Gloria Mark, 2023). Existing tools fail because manual flashcard authoring (e.g., Anki) requires high upfront friction, and static quiz decks run out of content.

*Who is excluded:* 
- Children under 10 years old (the game mechanic requires rapid reading comprehension within split seconds).
- Students writing long-form essays or synthesis papers (rapid multiple-choice retrieval is unfit for multi-paragraph nuanced argumentation).

**What did you build?**
Knowledge Runner is a 3D endless runner game driven by generative AI. The user inputs any study topic, and the system dynamically generates curriculum-aligned multiple-choice questions streamed into a 3-lane obstacle course. The player steers their avatar left, center, or right at high speed to run through the correct answer gate under time pressure, earning streak multipliers and extra lives, followed by an end-of-run diagnostic review screen that contrasts chosen answers with correct answers and provides full conceptual explanations.

**Link to the live thing (if any):**  
Deployed live at: [https://knowledgerunner.alexwr.cc](https://knowledgerunner.alexwr.cc)

**How do I run it?**
1. Open [https://knowledgerunner.alexwr.cc](https://knowledgerunner.alexwr.cc) in any modern desktop or mobile browser (or clone [the repo](https://github.com/alex24106429/Knowledge-Runner) and serve locally via `npx serve .`).
2. On the title screen, enter any subject into the **"Quiz Topic"** field (e.g., `AI Ethics`, `French Vocab`, or `Organic Chemistry`).
3. Click **START**.
4. Play using the keyboard (`Left`/`Right` Arrow Keys, `A`/`D`) or tap the bottom HUD lane cards on touchscreen/mobile to steer into the gate showing the correct answer.
5. Once your lives run out, scroll through the **RUN COMPLETE** summary to review question-by-question explanations, or click **RUN AGAIN** / **NEW TOPIC**.

**Who did what?**

**Both:**
- Came up with ideas and target audience
- Planned the features and learning experience
- Researched the problem and user needs
- Tested the game and made improvements

**Domilė Žukaitė**:
- Worked on the research and presentation
- Came up with the game concept and content
- Helped prompting Bolt.new
- Tested the game and provided feedback

**Alex Wrobel**:
- Worked on the technical development and coding using Bolt.new
- Implemented AI features
- Fixed technical issues

**Ethical reflection - what are the risks of your tool? Who could it harm?**
- **Hallucinated Answers:** The core ethical risk is an LLM generating factually inaccurate questions or answers. Under rapid, visuomotor conditioning, students could subconsciously memorize false facts. We mitigated this by setting the LLM temperature to a low `0.7`, applying strict system prompts requiring verifiable academic consensus, and providing detailed post-run answer explanations so users can evaluate reasoning; our next roadmap step is RAG (Retrieval-Augmented Generation) grounded strictly in uploaded textbook PDFs.
- **Cognitive Overload & Exclusion:** High-speed visual gameplay risks inducing anxiety or discriminating against students with slower processing speeds or visual impairments. We addressed this by enforcing a strict 1–3 word length limit per answer gate and displaying high-contrast, static HUD cards at the bottom of the screen to minimize visual track distortion.
- **Cultural & Model Bias:** Generative models often reflect Western-centric perspectives or treat disputed opinions as absolute facts. Rather than displaying an arbitrary, unexplained "Wrong!" mark, the game-over review screen provides full contextual reasoning for every answer to encourage critical reflection rather than dogmatic rote memorization.
- **Scope & Privacy Safeguards:** To prevent harm regarding surveillance and data exploitation, the app intentionally does not have user accounts, tracking, or persistent database storage; all session state is ephemeral and resets upon refresh.

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
The most critical takeaway was understanding how foundational programmatic control structures-explicit data typing, input sanitization, and deterministic conditional branching (`if`/`elif`/`else`)-serve as the indispensable bedrock for building robust, ethical software. Bridging fundamental Python mechanics (such as preventing type coercion bugs from `input()` and avoiding namespace collisions) with rapid AI prototyping in Bolt.new demonstrated that AI systems and LLM outputs cannot function reliably in a vacuum; they require rigorous logical scaffolding, strict boundary checks, and defensive validation to transform raw, probabilistic generative outputs into structured, safe user experiences.

**Where does this connect to "AI for Good"?**
This week directly connected to AI for Good across two vital dimensions: **Algorithmic Fairness (Social Ethics)** and **Computational Environmental Cost (Sustainability)**. 

1. In the workshop’s loan risk assignment, writing automated decision trees highlighted how easily human biases and structural inequalities can be codified into "objective" automated rules-underscoring that developers must actively audit threshold logic to prevent algorithmic discrimination against marginalized groups. 
2. In the AI energy consumption and "Green or Not?" carbon accounting tasks, quantifying kilowatt-hours and CO₂ equivalents made the invisible environmental footprint of digital technologies tangible. As we deploy LLM-powered applications (like our SDG 4 *Knowledge Runner* prototype), we must practice computational frugality-minimizing unnecessary token inference, mitigating hallucination risks through low-temperature grounding, and ensuring technology democratizes quality education without exacerbating ecological degradation or socioeconomic exclusion.