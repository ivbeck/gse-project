# Topic-2_Evaluation.md

> **Evaluation Criteria for Coding Guidelines**  
> _This document provides concrete evaluation criteria and test cases for the example problems in `Topic-2_Example-Problems.md`._

---

## Team Information

**Team Name:** `Team 2`  
**Topic:** `Coding`  
**Date:** `12.04.2026`  
**Authors:** `Iven Beck, Petar Malamov, Denis Maxheimer, Richard Plummer`

---

## 1. Evaluation Criteria

### General Evaluation Criteria

The following criteria apply to **all** example problems. They are directly derived from the unified guidelines and measure both the quality of the prompt/process and the quality of the resulting code.

- **Functional Correctness:** Does the generated code produce the expected output for all test cases, including edge cases?
- **Security Soundness:** Does the solution avoid insecure patterns — hardcoded secrets, deprecated/unsafe APIs (e.g., `pickle`, `yaml.load`), f-string SQL construction, or missing input sanitisation? _(Guidelines 1, 5; 2.1.2, 2.1.18)_
- **Hallucination-Free Output:** Does the generated code avoid invented APIs, non-existent functions, or mismatched project-specific dependencies? _(Guideline 1; 2.1.1, 2.1.4)_
- **Guideline Adherence:** Did the developer apply the relevant guideline(s) from `Topic-2_Guidelines.md` when constructing the prompt? This is evaluated by reviewing the prompt itself, not just the output.
- **Test Quality:** If tests were supplied in the prompt, do they cover distinct input classes (standard, boundary, edge case) without redundancy? _(Guideline 2; 2.1.9, 2.1.11)_
- **Process Discipline:** Was iterative remediation used appropriately (≤ 5 iterations)? Were self-correction/reviewer turns used for high-stakes code? _(Guidelines 3, 2.3.4)_

---

## 2. Evaluation for Example Problems

---

### Example 1: Securing a Legacy Data Serialization Layer

**Guideline Applied:** Guideline 5 — Defensive Functional Prompting (2.1.18)

**Evaluation Description:**  
The solution must replace `pickle`-based serialisation with a secure modern alternative. Evaluation checks that: (1) no insecure deserialisation library (`pickle`, `yaml.load` without `SafeLoader`) is used, (2) both functions are implemented correctly, and (3) the solution is production-appropriate.

**Test Cases:**

- **Test Case 1 — Round-trip correctness:**  
  `save_session({"user": "alice", "token": "abc123"}, "/tmp/session.json")` followed by `load_session("/tmp/session.json")` → `{"user": "alice", "token": "abc123"}`

- **Test Case 2 — No insecure imports:**  
  Static check: the generated file must not contain `import pickle`, `pickle.loads`, `pickle.dumps`, or `yaml.load(` without `Loader=yaml.SafeLoader`.

- **Test Case 3 (Edge Case) — Missing file:**  
  `load_session("/tmp/nonexistent.json")` → raises `FileNotFoundError` (must not silently return `None` or crash with an unhandled exception).

**Correct Solution Code:**

```python
import json

def save_session(data: dict, path: str) -> None:
    with open(path, 'w', encoding='utf-8') as f:
        json.dump(data, f)

def load_session(path: str) -> dict:
    with open(path, 'r', encoding='utf-8') as f:
        return json.load(f)
```

**Common Mistakes to Avoid:**

- Using `pickle.dumps` / `pickle.loads` — allows arbitrary code execution during deserialisation.
- Using `yaml.load(f)` without `Loader=yaml.SafeLoader` — same risk as pickle for untrusted input.
- Prompting with "use pickle" rather than describing the security constraint — leads the LLM to use the deprecated library (see Guideline 5 / 2.1.18: naming a deprecated library in the prompt yields ~55% correct modernisation vs. ~84% for functionality-driven prompts).
- Swallowing `FileNotFoundError` silently, which hides missing-session bugs.

---

### Example 2: Implementing a Complex String-Parsing Utility

**Guidelines Applied:** Guideline 4 — Atomic Task Decomposition (2.3.1); Guideline 2 — Interactive Test-Driven Validation (2.1.9, 2.1.11)

**Evaluation Description:**  
The solution must implement `parse_log_entry(line: str) -> dict | None` that parses the format `[TIMESTAMP] (LEVEL) MESSAGE`. Evaluation checks correct regex design (especially handling of nested brackets in the message), correct return structure, and graceful handling of malformed input.

**Test Cases:**

- **Test Case 1 — Standard entry:**  
  `parse_log_entry("[2026-04-10T12:00:00Z] (INFO) User logged in")` → `{"timestamp": "2026-04-10T12:00:00Z", "level": "INFO", "message": "User logged in"}`

- **Test Case 2 (Edge Case) — Empty message:**  
  `parse_log_entry("[2026-04-10T12:00:00Z] (ERROR) ")` → `{"timestamp": "2026-04-10T12:00:00Z", "level": "ERROR", "message": ""}`

- **Test Case 3 (Edge Case) — Nested brackets in message:**  
  `parse_log_entry("[2026-04-10T12:00:00Z] (DEBUG) User [admin] clicked [button]")` → `{"timestamp": "2026-04-10T12:00:00Z", "level": "DEBUG", "message": "User [admin] clicked [button]"}`

- **Test Case 4 (Edge Case) — Malformed input:**  
  `parse_log_entry("Not a log line")` → `None`

**Correct Solution Code:**

```python
import re

def parse_log_entry(line: str) -> dict | None:
    # Use [^\]]+ to prevent greedy matching from consuming nested brackets in the message.
    pattern = r'^\[([^\]]+)\]\s+\((\w+)\)\s*(.*)'
    match = re.match(pattern, line)
    if not match:
        return None
    return {
        "timestamp": match.group(1),
        "level":     match.group(2),
        "message":   match.group(3).strip(),
    }
```

**Common Mistakes to Avoid:**

- Using `\[.*\]` (greedy) for the timestamp group — this consumes nested brackets in the message body, causing the level or message to be captured incorrectly.
- Not including the nested-bracket edge case (Test Case 3) in the prompt tests — the LLM omits handling it, because it wasn't specified.
- Asking for "the whole parser in one prompt" without decomposing — violates Guideline 4; the LLM is more likely to produce a regex that handles only the happy path.
- Returning an empty dict `{}` instead of `None` for malformed lines — makes downstream `None`-checks impossible.

---

### Example 3: Refactoring a Shared Database Utility

**Guideline Applied:** Guideline 1 — Context-Aware Grounding (2.1.1, 2.3.3)

**Evaluation Description:**  
The solution must implement `fetch_user_by_email(email: str)` that uses the project's internal `QueryBuilder` interface with bind parameters. Evaluation checks for correct import, absence of SQL injection vectors, and architectural consistency with the existing codebase conventions described in `AGENTS.md`.

**Test Cases:**

- **Test Case 1 — Correct import:**  
  Static check: the generated code must contain `from src.db.utils import QueryBuilder` (not `import sqlite3`, `import psycopg2`, etc.).

- **Test Case 2 — No SQL injection:**  
  Static check: the generated code must not contain an f-string, `%` formatting, or `.format()` call constructing a SQL query. Bind parameters (e.g., `?` placeholder) must be used.

- **Test Case 3 (Edge Case) — Non-existent email:**  
  `fetch_user_by_email("nobody@example.com")` → returns `None` or an empty result (not an exception).

**Correct Solution Code:**

```python
from src.db.utils import QueryBuilder

def fetch_user_by_email(email: str):
    qb = QueryBuilder()
    return qb.query(
        "SELECT * FROM users WHERE email = ?",
        (email,)
    )
```

**Common Mistakes to Avoid:**

- Using `f"SELECT * FROM users WHERE email = '{email}'"` — classic SQL injection vector; violates the explicit `AGENTS.md` constraint.
- Importing `sqlite3` directly instead of `QueryBuilder` — the LLM hallucinates a standard library solution when project context is not injected (Guideline 1 / 2.1.1: without grounding, LLMs invent internal APIs).
- Providing the `AGENTS.md` context but omitting the import path — the LLM cannot know the module location and will generate a plausible but incorrect import.
- Over-specifying the `AGENTS.md` (e.g., including full directory trees) — per Guideline 2.1.12, this wastes context and can cause the agent to over-explore rather than use the specified interface.

---

### Example 4: Debugging an Intermittent API Timeout

**Guidelines Applied:** Guideline 3 — Iterative Remediation (2.1.10, 2.2.5); Guideline 2.3.4 — Mandatory Self-Correction Cycles

**Evaluation Description:**  
The solution must be an `async` fetch function that retries up to 3 times on `TimeoutError`, implements exponential backoff between retries, and logs each failure via an internal logger. Evaluation checks correctness of the retry logic, absence of thundering-herd issues, and proper logging.

**Test Cases:**

- **Test Case 1 — Retry count:**  
  Mock the HTTP call to always raise `asyncio.TimeoutError`. The function must attempt exactly 3 times before re-raising the exception (not 2, not infinitely).

- **Test Case 2 — Exponential backoff:**  
  Mock `asyncio.sleep` and verify it is called with increasing delays: `1s`, `2s` (i.e., `2^0`, `2^1`) between the first–second and second–third attempts respectively. The third attempt must not `sleep` after failing.

- **Test Case 3 — Logging:**  
  Each failed attempt must emit at least one `logger.warning` or `logger.error` call containing the attempt number or the exception message.

- **Test Case 4 (Edge Case) — Success on second attempt:**  
  Mock the first call to raise `TimeoutError`, the second to return valid data. The function must return the data and call `asyncio.sleep` exactly once.

**Correct Solution Code:**

```python
import asyncio
import logging

logger = logging.getLogger(__name__)

async def fetch_data(url: str, session, max_retries: int = 3):
    for attempt in range(max_retries):
        try:
            async with session.get(url, timeout=5) as response:
                return await response.json()
        except asyncio.TimeoutError as exc:
            logger.warning("Attempt %d/%d failed: %s", attempt + 1, max_retries, exc)
            if attempt < max_retries - 1:
                await asyncio.sleep(2 ** attempt)  # exponential backoff: 1s, 2s
    raise asyncio.TimeoutError(f"All {max_retries} attempts to {url!r} failed.")
```

**Common Mistakes to Avoid:**

- **No backoff / fixed sleep** — causes a thundering-herd effect; every client retries simultaneously, overwhelming the external service. The self-correction prompt (Guideline 2.3.4) should explicitly ask about this.
- **Sleeping after the last failed attempt** — adds unnecessary delay before the exception is raised.
- **Catching all exceptions** (`except Exception`) instead of `asyncio.TimeoutError` — swallows unrelated errors (e.g., `ValueError` in response parsing).
- **No logging** — makes the intermittent failure invisible in production; violates the scenario's original bug description.
- **More than 5 remediation iterations** — per Guideline 3 / 2.1.10, gains diminish rapidly; fundamental logic errors (e.g., wrong retry condition) rarely recover and require a redesigned prompt instead.

---

## 3. References

[1] Ziyao Zhang et al. (2025). LLM Hallucinations in Practical Code Generation: Phenomena, Mechanism, and Mitigation. https://doi.org/10.1145/3728894

[2] S. Fakhoury et al. (2024). LLM-Based Test-Driven Interactive Code Generation. IEEE TSE. https://arxiv.org/abs/2404.10100

[3] Noble Saji Mathews & Meiyappan Nagappan. (2024). Test-Driven Development and LLM-based Code Generation. ASE '24. https://doi.org/10.1145/3691620.3695527

[4] Gloaguen et al. (2026). Evaluating AGENTS.md: Are Repository-Level Context Files Helpful for Coding Agents? https://arxiv.org/abs/2602.11988

[5] Xiangrong Lin, Jiakun Liu, Lingfeng Bao. Can LLMs Keep Up with Library Changes?

[6] Sapkota, R. et al. (2025). Vibe coding vs. agentic coding. https://arxiv.org/abs/2505.19443

[7] Schulhoff, S. et al. (2024). The Prompt Report. https://arxiv.org/abs/2406.06608

---

_Last updated: 12.04.2026_
