# Topic-2_Guidelines.md

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
- **Blog posts:**
  - `[Blog Post 1] OpenAI Developers Promp Engineering`
  - `[Blog Post 2] OpenAI Developers GPT-5 prompting guide`

**Extracted Guidelines:**  
Format same as above.

---

**Guideline 2.2.1: Define Explicit Agent Roles for Coding Tasks**  
**Source:** OpenAI Developers Prompt Engineering Guide  
**Description:** Prompts should frame the model as a software engineering agent with clearly defined responsibilities and workflows.  
**Reasoning:** Explicit role definition improves task understanding and leads to more structured, reliable outputs.  
**Example:** "You are a software engineer responsible for implementing, testing, and validating a feature using the provided tools."  
**When to Apply:** When tasks involve multiple steps (e.g., coding, testing, debugging). When using tool-based workflows or agents. When consistency and reliability of outputs are important.  
**When to Avoid:** For simple, one-off questions or small code snippets. When flexibility or creative exploration is preferred over strict structure.

---

**Guideline 2.2.2: Enforce Structured Tool Usage with Examples**  
**Source:** OpenAI Developers Prompt Engineering Guide  
**Description:** Prompts should include explicit instructions and examples for how to use tools or function calls.  
**Reasoning:** Concrete examples reduce ambiguity and increase adherence to expected workflows.  
**Example:** Providing a sample `functions.run` call for executing code tasks.  
**When to Apply:** This guideline should be applied when tasks require interaction with tools, APIs, or function calls, especially in structured or automated workflows where correct tool usage is critical. It is particularly useful in production settings or agent-based systems where consistency, reproducibility, and reliability of execution are important.  
**When to Avoid:** This guideline should be avoided in simple tasks that do not involve tools or external function calls, or in exploratory scenarios where strict structure may limit flexibility. It may also be unnecessary when the task is purely conceptual and does not require execution or integration with external systems.

---

**Guideline 2.2.3: Require Testing and Validation of Generated Code**  
**Source:** OpenAI Developers Prompt Engineering Guide  
**Description:** The model should be instructed to validate outputs via tests (e.g., unit tests or execution).  
**Reasoning:** LLM-generated code may appear correct but fail functionally, testing ensures correctness.  
**Example:** "Run unit tests after implementation and verify all pass before finalizing."  
**When to Apply:** This guideline should be applied for any non-trivial coding task where correctness is important, especially in production, evaluation pipelines, or when generating algorithms and logic-heavy implementations. It is particularly useful in iterative workflows where outputs can be automatically tested and refined based on results.  
**When to Avoid:** This guideline may be unnecessary for very simple or illustrative code snippets where correctness can be easily verified by inspection, or in early prototyping stages where speed and exploration are prioritized over full validation. It may also be impractical when no testing environment or execution capability is available.  

---

**Guideline 2.2.4: Use Internal Rubrics for Self-Evaluation**  
**Source:** OpenAI Developers GPT-5 Prompting Guide  
**Description:** The model should internally construct a quality rubric (5–7 criteria) to evaluate its output.  
**Reasoning:** Self-evaluation improves output quality by enforcing structured reasoning and reflection.  
**Example:** Internally assessing criteria like correctness, simplicity, performance, and usability before responding.  
**When to Apply:** This guideline should be applied for complex or high-quality code generation tasks where multiple dimensions such as correctness, efficiency, and maintainability matter. It is especially useful in one-shot generation scenarios or when aiming for production-level outputs without extensive external feedback loops.  
**When to Avoid:** This guideline may be unnecessary for simple or low-stakes tasks where a quick response is sufficient, as the added internal reasoning can increase latency. It may also be less effective when strict external constraints or evaluation criteria are already provided explicitly in the prompt.

---

**Guideline 2.2.5: Encourage Iterative Refinement Until Quality is Met**  
**Source:** OpenAI Developers GPT-5 Prompting Guide  
**Description:** The model should refine its output iteratively until it meets high-quality standards.  
**Reasoning:** First outputs are often suboptimal, iteration improves correctness and completeness.  
**Example:** Rewriting a solution if it fails internal quality checks.  
**When to Apply:** This guideline should be applied in complex or high-stakes coding tasks where correctness, robustness, and completeness are critical, especially in scenarios without immediate external validation. It is particularly effective in workflows that allow multiple passes, such as agent-based systems or structured prompting setups with refinement loops.  
**When to Avoid:** This guideline may be unnecessary in simple tasks where the first response is likely sufficient, or in time-sensitive situations where latency must be minimized. It can also be inefficient when external evaluation mechanisms (e.g., tests or human review) already provide feedback for refinement.  

---

**Guideline 2.2.6: Enforce Structured Markdown Output for Code**  
**Source:** OpenAI Developers Prompt Engineering Guide  
**Description:** Outputs should follow strict Markdown conventions (code blocks, inline code, lists).  
**Reasoning:** Consistent formatting improves readability and usability of generated code.  
**Example:** Using fenced code blocks for implementations and backticks for file paths or functions.

---

**Guideline 2.2.7: Ensure Alignment with Existing Codebase Standards**  
**Source:** OpenAI Developers GPT-5 Prompting Guide  
**Description:** Generated code should follow the style, structure, and conventions of the existing codebase.  
**Reasoning:** Consistency ensures seamless integration and reduces refactoring effort.  
**Example:** Matching directory structure, naming conventions, and existing libraries.  
**When to Apply:** This guideline should be applied whenever code is generated for human consumption, documentation, or integration into workflows where readability and clarity are important. It is especially useful in collaborative environments, educational contexts, or when outputs are reused directly in development environments.  
**When to Avoid:** This guideline may be less necessary in purely machine-to-machine interactions or pipelines where formatting is stripped or irrelevant. It can also be excessive for very short responses or informal discussions where strict formatting does not add meaningful value.  

---

**Guideline 2.2.8: Promote Modular and Reusable Code Design**  
**Source:** OpenAI Developers Prompt Engineering Guide  
**Description:** Generated code should emphasize modularity and reuse of components.  
**Reasoning:** Modular design improves maintainability and scalability of generated solutions.  
**Example:** Extracting repeated UI logic into reusable components.  
**When to Apply:** This guideline should be applied in medium to large coding tasks, especially when building systems, applications, or features that may evolve over time. It is particularly useful in collaborative environments, frontend development, and projects where code reuse, maintainability, and scalability are important.  
**When to Avoid:** This guideline may be unnecessary for very small or one-off scripts where modularization would introduce unnecessary complexity. It can also be less suitable in rapid prototyping scenarios where speed is prioritized over long-term maintainability.  

---

**Guideline 2.2.9: Plan Before Generating Code (Structured Thinking)**  
**Source:** OpenAI Developers GPT-5 Prompting Guide  
**Description:** Prompts should encourage planning steps before producing the final solution.  
**Reasoning:** Structured planning leads to more coherent and complete implementations.  
**Example:** Defining requirements and approach before writing code.  
**When to Apply:** This guideline should be applied in complex or multi-step coding tasks where understanding requirements, edge cases, and system design is important. It is especially useful for algorithmic problems, system design tasks, and scenarios where correctness and completeness are critical.  
**When to Avoid:** This guideline may be unnecessary for simple or well-defined tasks where planning adds little value and increases response time. It can also be less suitable in time-constrained situations or when rapid iteration is preferred over detailed upfront reasoning.  

---

**Guideline 2.2.10: Define Project Structure and Separation of Concerns**  
**Source:** OpenAI Developers GPT-5 Prompting Guide  
**Description:** Prompts should specify directory layouts and separation between components (e.g., UI, logic, API).  
**Reasoning:** Clear structure enables better organization and integration into larger systems.  
**Example:** Separating `/components`, `/hooks`, `/lib`, and `/api` directories.  
**When to Apply:** This guideline should be applied when generating code for larger applications, multi-file projects, or systems that require clear organization and maintainability. It is particularly useful in frontend/backend projects, team environments, and when code needs to integrate into an existing codebase.  
**When to Avoid:** This guideline may be unnecessary for small scripts, single-file solutions, or quick prototypes where introducing a full project structure would add unnecessary complexity. It can also be excessive when the task scope does not require modular separation.  

---

**Guideline 2.2.11: Enforce UI/UX and Accessibility Best Practices (Frontend)**  
**Source:** OpenAI Developers GPT-5 Prompting Guide  
**Description:** Prompts should include constraints on typography, color usage, spacing, interaction states, and accessibility.  
**Reasoning:** Explicit UI/UX guidance ensures high-quality and usable frontend outputs.  
**Example:** Using consistent spacing, limited color palettes, and accessible components.  
**When to Apply:** This guideline should be applied when generating frontend interfaces, especially in user-facing applications where usability, consistency, and accessibility are critical. It is particularly important in production environments, design-sensitive projects, and when adhering to design systems or accessibility standards.  
**When to Avoid:** This guideline is unnecessary for backend-focused tasks or purely functional prototypes where UI quality is not a priority. It can also be excessive in early-stage prototyping where rapid iteration is more important than polished design.  

---

### 2.3 Guidelines from LLM Experimentation

**Models Used:**
- `[Gemma 4 26B A4B, GPT-5.3, GPT-5.4, Qwen3.6 Plus, Claude Opus 4.6]`

**Prompts Used:**

- `[What is the best way to use an LLM for coding. We are doing research on that in the Generative Software Engineering course. Be concise and always reason why.`

**Extracted Guidelines:**  
Format same as above.

---

**Guideline 2.3.1: Atomic Task Decomposition**  
**Source:** Qwen 3.6 Plus / Gemma 4 26B A4B
**Description:** Break complex feature requests into isolated, atomic, and testable units (functions or specific modules) before prompting.  
**Reasoning:** LLM performance degrades with task complexity and context length, smaller scopes reduce logical hallucinations and context window drift.  
**Example:** Instead of "Build a full web scraper," prompt for "A function that extracts all `<a>` tags from a BeautifulSoup object."  
**When to Apply:** This guideline should be applied for complex systems, multi-step features, or tasks that involve multiple components, especially when accuracy and correctness are critical. It is particularly effective when working with long prompts, limited context windows, or when combining LLM outputs into larger pipelines.  
**When to Avoid:** This guideline may be unnecessary for simple, well-defined tasks that can be solved in a single step without ambiguity. It can also be less efficient when the overhead of decomposition outweighs the benefits, such as in very small scripts or exploratory prototyping.  

---

**Guideline 2.3.2: Test-Driven Verification Loop**  
**Source:** Qwen 3.6 Plus / Gemma 4 26B A4B  
**Description:** Prompt the model to produce a comprehensive unit test suite before or alongside the logic, then feed any execution failures back into the model.  
**Reasoning:** Models optimize for surface-level plausibility, automated feedback loops provide a truth mechanism that can reduce bug injection by 40-60%.  
**Example:** "Generate a suite of unit tests in pytest that covers edge cases for this logic" followed by "Here is the traceback from the failed test, please fix the implementation."  
**When to Apply:** This guideline should be applied in scenarios where correctness is critical, such as production code, algorithmic problems, or complex logic with many edge cases. It is particularly effective when an execution environment is available to run tests and provide feedback, enabling iterative improvement loops.  
**When to Avoid:** This guideline may be less suitable when no execution or testing environment is available, or when tasks are too simple to justify the overhead of creating tests. It can also slow down rapid prototyping workflows where speed is prioritized over full correctness verification.  

---

**Guideline 2.3.3: Context Hydration via RAG**  
**Source:** GPT-5.4 / Claude Opus 4.6  
**Description:** Ground all generation in the local codebase by injecting relevant symbols, dependency graphs, and existing patterns (Retrieval-Augmented Generation).  
**Reasoning:** To maintain architectural consistency, the model must differentiate between generic training data and local project standards, grounding reduces context drift by ~30%.  
**Example:** Including specific error-handling wrappers or interface definitions from the current repository in the system prompt.  
**When to Apply:** This guideline should be applied when working within existing codebases, especially large or complex repositories with established conventions, dependencies, and architectural patterns. It is particularly useful for tasks involving integration, refactoring, or extending existing systems where consistency and correctness depend on project-specific context.  
**When to Avoid:** This guideline may be unnecessary for standalone tasks, greenfield projects, or simple scripts where no prior context exists. It can also introduce overhead if retrieving and injecting context is costly or if the retrieved information is noisy or irrelevant, potentially confusing the model.

---

**Guideline 2.3.4: Mandatory Self-Correction Cycles**  
**Source:** Claude Opus 4.6 / GPT-5.4
**Description:** Use a secondary "Reviewer" turn to force the model to critique its own code for security flaws, memory leaks, or SOLID violations.  
**Reasoning:** LLMs are empirically better at identifying errors in existing text than they are at avoiding them during initial generation.  
**Example:** "Review the code above for potential race conditions or security vulnerabilities before I finalize the pull request."  
**When to Apply:** This guideline should be applied in high-stakes or production-level coding tasks where robustness, security, and maintainability are critical. It is particularly useful in workflows that allow multiple interaction steps, such as code reviews, pull request preparation, or agent-based pipelines with separate generation and evaluation phases.  
**When to Avoid:** This guideline may be unnecessary for simple or low-risk tasks where the overhead of an additional review step outweighs the benefits. It can also be less effective in one-shot interactions where no iterative feedback loop is possible or when time constraints require immediate results.  

---

## 3. References

**Literature References:**  
[1] `[Full citation]`  
[2] `[Full citation]`

**Grey Literature References:**  
[1] [`OpenAI Developers Prompt engineering`](https://developers.openai.com/api/docs/guides/prompt-engineering?prompt-example=prompt#choosing-a-model)<br>
[2] [`OpenAI Developers GPT-5 prompting guide`](https://developers.openai.com/cookbook/examples/gpt-5/gpt-5_prompting_guide#agentic-workflow-predictability)

**LLM Prompts (Full Log):**  
[`[Log Link to Github]`](https://raw.githubusercontent.com/ivbeck/gse-project/refs/heads/main/guideline_package/Topic-2_Logs.md)
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
