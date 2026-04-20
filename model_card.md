# BugHound Mini Model Card (Reflection)

Fill this out after you run BugHound in **both** modes (Heuristic and Gemini).

---

## 1) What is this system?

**Name:** BugHound  
**Purpose:** Analyze a Python snippet, propose a fix, and run reliability checks before suggesting whether the fix should be auto-applied.

**Intended users:** Students learning agentic workflows and AI reliability concepts.

---

## 2) How does it work?

5 steps: plan, analyze, act, test, reflect. Heuristics or Gemini find issues, a fix gets proposed, the risk assessor scores it, and the agent decides auto-fix or human review.

---

## 3) Inputs and outputs

Tested short functions with try/except, prints, and TODOs. Detected bare excepts, print statements, and TODOs. Fixes swapped print for logging.info and tightened exception handling. Risk report gives a score, level, and should_autofix.

---

## 4) Reliability and safety rules

Rule 1 - New imports: flags when the fix adds import lines. New deps can fail or cause side effects. FP: safe stdlib import gets flagged. FN: misses swapping one import for another.

Rule 2 - Return removed: flags when original had return but fix does not. Dropping a return silently changes output to None. FP: “return” in a comment triggers it. FN: misses when return value just changes.

---

## 5) Observed failure modes

1. LLM returned an empty issues list for a file with a bare except and TODO, so the agent skipped the fix and said nothing was wrong.

2. A print-only file got should_autofix True even though the fix added import logging, which is a real behavioral change worth reviewing.

---

## 6) Heuristic vs Gemini comparison

Only tested heuristic mode. Heuristics caught bare except, prints, and TODOs reliably. Gemini would likely catch subtler things like logic errors. Risk scorer works the same either way since it only looks at fix structure.

---

## 7) Human-in-the-loop decision

If the fix adds a new import, block autofix and require human review. Trigger lives in the risk assessor. Message: “This fix adds a new import. Please review before applying.”

---

## 8) Improvement idea

The print check is a substring match so it fires on strings and comments containing “print(“ by accident. Using Python's ast module instead would catch only real calls, no new dependencies needed.
