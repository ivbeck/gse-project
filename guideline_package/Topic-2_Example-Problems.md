# Topic-2_Example-Problems.md

> **Example Problems & Solutions for Coding Guidelines**  
> _This document demonstrates the practical application of the Topic 2 Coding Guidelines through generic, real-world software engineering scenarios._

---

## Example 1: Securing a Legacy Data Serialization Layer
**Scenario:** A developer needs to update a legacy Python system that currently use `pickle` for storing user session data to disk. The goal is to migrate to a secure, modern format without introducing security vulnerabilities.

### Applying Guideline 5: Defensive Functional Prompting
Instead of asking for a specific library (which might lead the LLM to use deprecated methods like `yaml.load`), the developer describes the *requirement* and the *constraint*.

**Prompt:**
> "I need to replace a `pickle`-based serialization module with a modern, secure alternative in Python. The new implementation must prevent arbitrary code execution during deserialization and be suitable for production session data. Suggest a modern format (like JSON or a safe binary alternative) and provide the implementation for `save_session(data, path)` and `load_session(path)` functions."

**Guideline-Compliant Solution:**
The LLM selects `json` or a cryptographically signed alternative because it was prompted for *functionality* and *security constraints* rather than a library name.

---

## Example 2: Implementing a Complex String-Parsing Utility
**Scenario:** A requirement exists to parse a specialized log format: `[TIMESTAMP] (LEVEL) MESSAGE`. The timestamps are in ISO-8601, and the message may contain nested brackets.

### Applying Guideline 4: Atomic Task Decomposition
The developer breaks this down into two smaller prompts rather than one large "build the whole parser" request.

**Step 1 Prompt:**
> "Reason through the regex required to extract the `[TIMESTAMP]` and `(LEVEL)` components from string `[2026-04-10T12:00:00Z] (INFO) User logged in`. Show your work using Few-Shot Chain-of-Thought."

**Step 2 Prompt:**
> "Now, using the regex logic from step 1, write a Python function `parse_log_entry(line)` that returns a dictionary. Provide 3 diverse test cases including an empty message and a malformed timestamp."

### Applying Guideline 2: Interactive Test-Driven Validation
The developer includes the tests in the prompt to ensure the edge cases (like nested brackets in the message) are handled correctly.

---

## Example 3: Refactoring a Shared Database Utility
**Scenario:** A large project has a custom `DatabaseManager` class that wraps all SQL queries to prevent injection. A developer needs to add a `fetch_user_by_email` function that follows these internal patterns.

### Applying Guideline 1: Context-Aware Grounding
The developer provides a minimal context file (`AGENTS.md`) in the prompt.

**AGENTS.md Context Provided:**
```markdown
- Database: All queries must use the `QueryBuilder` interface in `src/db/utils.py`.
- Security: Never use f-strings for SQL queries; use bind parameters.
```

**Prompt:**
> "Following the patterns in our `AGENTS.md` context, implement a `fetch_user_by_email(email)` function. Use the `QueryBuilder` interface from `src.db.utils`."

**Guideline-Compliant Solution:**
The LLM uses `from src.db.utils import QueryBuilder` and follows the bind-parameter rule, preventing a "hallucination" where it might try to use `sqlite3` directly or insecure f-strings.

---

## Example 4: Debugging an Intermittent API Timeout
**Scenario:** A newly generated async function for fetching data from an external API is occasionally failing with a `TimeoutError`, but the error isn't being logged properly.

### Applying Guideline 3: Iterative Remediation
The developer uses a "Plan-Execute-Review" loop.

1.  **Draft:** LLM generates a basic async fetch.
2.  **Failure:** The code fails in the test environment.
3.  **Remediation:** The developer feeds the traceback back:
    > "The code failed with `asyncio.TimeoutError` at line 12. Update the implementation to include a retry loop (max 3 tries) and log the failure using our internal logger."

### Applying Guideline 2.3.4: Mandatory Self-Correction Cycles (Reviewer Turn)
After the fix, the developer prompts:
> "Act as a security and performance reviewer. Review the retry logic above for potential 'thundering herd' issues or resource leaks. Does it implement exponential backoff?"

---

## Example 5: Secure, library-agnostic serialization migration
- Guideline mapping: 5 (Defensive Prompting), 2.1.11 (quality tests)
- Scenario: Replace a Python app’s pickle-based session storage with a secure format.
- Prompt (teacher-facing):
  - Describe the goal and constraints (prevent deserialization code execution; production-ready; no deprecated libs).
  - Ask for a concrete implementation of save_session(data, path) and load_session(path) using a secure format (JSON or a safe binary