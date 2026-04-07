# Topic-Coding_Guidelines.md

> **Template for Student Guideline Packages**  
> _Fill in the bracketed sections `[...]` with your team's curated content._

---

## Team Information

**Team Name:** `Team 2`  
**Topic:** `Coding`  
**Date:** `12.04.2026`  
**Authors:** `Iven Beck, Petar Malamov, Denis Maxheimer, Richard Plummer`

---

## 1. Unified Guidelines

> **Note:** These are the merged, refined guidelines that your team recommends to the class. Each guideline should be actionable, specific, and usable during real SE/coding tasks.

### Guideline 1: `[Title]`

**Description:**  
What should developers do (i.e., state it clearly and concretely)?

**Reasoning:**  
Explain _why_ this guideline is important. Reference:

- Relevant literature readings
- Grey literature (blogs, documentation, etc.)
- LLM experimentation insights

**Example:**  
Provide a (simple) illustrative example (code snippet, pseudo-code, or description).

**When to Apply:**  
Describe the conditions where this guideline is most effective.

**When to Avoid:**  
Describe edge cases or situations where this guideline may not work well.

---

### Guideline N: `[Title]`

(Repeat the same structure for each guideline.)

---

## 2. Raw Guidelines (Source Documents)

> **Note:** Include the original guidelines from each of the three sources before merging. This shows your curation process.

### 2.1 Guidelines from Literature Readings

**Readings Assigned:**

- Ziyao Zhang, Chong Wang, Yanlin Wang, Ensheng Shi, Yuchi Ma, Wanjun
  Zhong, Jiachi Chen, Mingzhi Mao, and Zibin Zheng. 2025. LLM
  Hallucinations in Practical Code Generation: Phenomena, Mechanism, and
  Mitigation. Proc. ACM Softw. Eng. 2, ISSTA, Article ISSTA022 (July
  2025), 23 pages. https://doi.org/10.1145/3728894

  -DONE

- S. Fakhoury, A. Naik, G. Sakkas, S. Chakraborty and S. K. Lahiri,
  "LLM-Based Test-Driven Interactive Code Generation: User Study and
  Empirical Evaluation," in IEEE Transactions on Software Engineering,
  vol. 50, no. 9, pp. 2254-2268, Sept. 2024, doi:
  10.1109/TSE.2024.3428972. https://arxiv.org/abs/2404.10100

  -DONE

- Noble Saji Mathews and Meiyappan Nagappan. 2024. Test-Driven
  Development and LLM-based Code Generation. In Proceedings of the 39th
  IEEE/ACM International Conference on Automated Software Engineering (ASE
  '24). Association for Computing Machinery, New York, NY, USA, 1583–1594.
  https://doi.org/10.1145/3691620.3695527

- Evaluating AGENTS.md: Are Repository-Level Context Files Helpful for
  Coding Agents?. https://arxiv.org/abs/2602.11988

- Can LLMs Keep Up with Library Changes? An Exploratory Study on
  LLM-Generated Code, Xiangrong Lin (Zhejiang University, China), Jiakun
  Liu (Harbin Institute of Technology, China), Lingfeng Bao (Zhejiang
  University, China)

- Sapkota, Ranjan, Konstantinos I. Roumeliotis, and Manoj Karkee. "Vibe
  coding vs. agentic coding: Fundamentals and practical implications of
  agentic ai." arXiv preprint arXiv:2505.19443 (2025).
  https://arxiv.org/abs/2505.19443

- Schulhoff, Sander, et al. "The prompt report: A systematic survey of
  prompt engineering techniques." arXiv preprint arXiv:2406.06608 (2024).
  https://arxiv.org/abs/2406.06608

**Extracted Guidelines:**  
For each relevant guideline from readings:

**Guideline 2.1.1: Integration of Repository-Wide Context Information**  
**Source:** Zhang et al. (2025)
**Description:** Code generation using LLMs should move beyond creating isolated functions and incorporate specific project context, including user-defined dependencies, attributes, and non-code resources like configuration files.
**Reasoning:** In practice, over 70% of functions are not standalone but depend on other entities within the project. Since LLMs often lack access to the full repository, they tend to invent internal APIs or functions (Dependency Conflicts), leading to runtime errors.  
**Example:** Without context awareness, a model incorrectly generated the function call generate_default_schema(), while the function actually defined in the project was generate_default_observer_schema_dict() (figure 9 of paper).

---

**Guideline 2.1.2: Prioritization of Non-Functional Security Requirements**  
**Source:** Zhang et al. (2025)
**Description:** Generated code must be specifically checked for non-functional requirements such as security to prevent the adoption of unsafe coding patterns from the models' training data.
**Reasoning:** LLMs often reflect problematic patterns from their training corpora. They may favor insecure APIs if they are widespread in public repositories, leading to vulnerabilities like SQL injection or insecure deserialization.
**Example:** Zhang et al. (2025) show that models frequently generate the insecure yaml.load() function instead of the safer alternative yaml.safe_load(), as the insecure pattern appears in hundreds of thousands of lines of code in the training data.

---

**Guideline 2.1.3: Comprehensive Validation of Functional Requirements**  
**Source:** Zhang et al. (2025)
**Description:** Developers must rigorously verify that generated code aligns with all functional requirements detailed in natural language inputs to prevent the introduction of wrong or missing functionalities.
**Reasoning:** Functional Requirement Violation is the most prevalent form of hallucination (36.66%) across all studied models, often stemming from a limited capacity to accurately capture and interpret complex user intentions or multi-step logic.
**Example:** In one instance, a model failed to incorporate a requirement for handling LocalTime based on a specific timezone, resulting in incomplete code that passed syntax checks but failed to meet the task's actual functional needs (figure 3).

---

**Guideline 2.1.4: Utilization of Automated Verification for Conflict Detection**  
**Source:** Zhang et al. (2025)
**Description:** Developers should use static analysis and dynamic test execution to systematically identify and locate identifiable hallucinations, specifically those categorized as Dependency Conflicts, Environment Conflicts, and API Knowledge Conflicts.
**Reasoning:** These types of hallucinations are relatively easy to recognize because they trigger detectable issues such as undefined variables, incorrect API methods, runtime errors, or test failures, unlike more elusive functional or security violations.
**Example:** A Dependency Conflict, such as a model generating the non-existent function generate_default_schema() instead of the project-defined generate_default_observer_schema_dict(), can be quickly caught by static analysis tools flagging undefined methods.

---

**Guideline 2.1.5: Identification and Mitigation of silent Hallucinations**  
**Source:** Zhang et al. (2025)
**Description:** Developers must implement specialized security audits and comprehensive functional reviews to detect 'silent' hallucinations, specifically incomplete functionality and security vulnerabilities, which standard verification tools often miss.
**Reasoning:** These hallucinations are uniquely dangerous because they can likely pass static checks and all functional test cases, potentially leading to the deployment of unreliable or insecure systems in production. Furthermore, these issues often arise from fundamental limitations in the model's training corpora and intention understanding capacity, meaning current self-feedback localization methods only detect them to a certain extent.
**Example:** See Guideline 2.1.2 example.

---

**Guideline 2.1.6: Intent Disambiguation via Interactive Testing Source**  
**Source:** Fakhoury et al. (2024)
**Description:** Developers should use automatically generated unit tests (TiCoder) as an interactive mechanism to formalize and clarify ambiguous natural language intent before finalizing code selection.
**Reasoning:** Natural language is inherently informal and ambiguous, which makes it nearly difficult to automatically verify if a generated code snippet truly satisfies a user's intent. Without a checkable specification, users often accept "plausibly correct" code that contains subtle logical bugs.
**Example:** For a function description requiring lowercase letters joined by an underscore, an LLM might generate code that incorrectly accepts "aa_bb_cc" when the user only intended for two sequences. By presenting the user with a test like text_lowercase_underscore("aa_bb_cc") == True?, the system allows the user to clarify their intent (by answering "No"), thereby formalizing the requirement.

---

**Guideline 2.1.7: Reduction of Developer's Cognitive Load**  
**Source:** Fakhoury et al. (2024)
**Description:** To improve developer performance, AI assistants should execute candidate code against user-validated tests to prune incorrect suggestions and present a reduced, ranked list of candidates.
**Reasoning:** Reviewing a large list of AI-generated code suggestions is a high-effort task that increases cognitive load and the likelihood of human error. Studies show that developers spend more time reviewing code than writing it; pruning suggestions based on actual execution behavior significantly reduces mental demand and stress.
**Example:** In a user study, participants using the TICODER workflow (which prunes code suggestions based on test feedback) reported significantly lower cognitive load (28.00–29.52) compared to those using a standard interface (45.46) where they had to manually scan all suggestions.

---

**Guideline 2.1.8: Human-in-the-Loop Validation of AI-Generated Tests Source**  
**Source:** Fakhoury et al. (2024)
**Description:** Developers must critically verify the correctness of AI-generated tests before using them as a filtering mechanism, as incorrect tests can lead to the pruning of valid code.
**Reasoning:** LLMs are prone to generating incorrect tests or test outputs, particularly for complex edge cases.
**Example:** In a study task involving a recursive binary search, 50% of participants incorrectly identified the expected output of an AI-generated test when the output was hidden, highlighting that humans are prone to making mistakes on edge cases that will then incorrectly prune correct code suggestions.

---

### 2.2 Guidelines from Grey Literature / Practitioner Sources

**Sources Explored:**

- `[Blog post 1]`
- `[Documentation 1]`
- `[Tool guide 1]`
- `[Community discussion 1]`

**Extracted Guidelines:**  
Format same as above.

---

### 2.3 Guidelines from LLM Experimentation

**Models Used:**

- `[e.g., GPT-5.2, Claude 4.5 Sonnet, DeepSeek Coder, GitHub Copilot (Ask vs Agent etc.)]`

**Prompts Used:**

- `[Prompt 1]`
- `[Prompt 2]`
- `[Prompt 3]`

**Extracted Guidelines:**  
Format same as above.

---

## 3. References

**Literature References:**  
[1] `[Full citation]`  
[2] `[Full citation]`

**Grey Literature References:**  
[1] `[Blog post title and URL]`  
[2] `[Documentation title and URL]`

**LLM Prompts (Full Log):**  
See Appendix A or provide a link to a separate file with full prompt-response logs.

---

## 4. Appendix (Optional)

- **A. Full Prompt Logs:** Link to detailed LLM interaction logs
- **B. Decision Matrix:** How you decided which guidelines to merge
- **C. Conflicts Resolved:** Examples of contradictory guidelines and how you resolved them

---

## Instructions for Use

1. **Replace all `[...]` placeholders** with your team's specific content
2. **Number guidelines consecutively** (Guideline 1, Guideline 2, etc.)
3. **Cite sources properly** using academic citation style (e.g., APA, ACM)
4. **Include concrete examples** - code or textual snippets (depending on SE task) are highly recommended
5. **Be specific about applicability** - when does this guideline work vs. fail?
6. **Submit as `Topic-XX_Guidelines.md`** where `XX` is your topic number

---

## Grading Criteria (for your reference)

- ✅ **Clarity:** Guidelines are specific and actionable
- ✅ **Evidence:** Each guideline is supported by reasoning and examples
- ✅ **Curation:** Shows thoughtful merging of multiple sources
- ✅ **Practicality:** Examples are relevant to real development tasks
- ✅ **Transparency:** Raw guidelines from all three sources are included

---

_Template version: 1.0 | Last updated: 24 February 2026_
