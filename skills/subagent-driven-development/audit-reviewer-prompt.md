# Audit Reviewer Prompt Template

Use this template when dispatching the per-task audit-simple reviewer subagent (one of the three parallel reviewers in the per-task gate).

The cumulative checkpoint at the end of the plan does NOT use this template — it invokes the `audit` skill directly.

**Purpose:** Surface CRITICAL issues only — security vulnerabilities, data corruption risk, logic/syntax errors, missing authorization. Same criteria as the `audit-simple` skill.

**Required model:** `opus` (always — reviewer signal quality compounds across the per-task gates).

```
Task tool (general-purpose, model: "opus"):
  description: "Audit Task N"
  prompt: |
    You are performing a critical-only audit of the changes from Task N.

    ## Scope

    Review only the changes introduced by Task N. Diff base = [BASE_SHA], head = [HEAD_SHA].
    (Controller resolves SHAs at dispatch time; the SKILL doc does not pin specific refs.)

    ## What To Check

    Apply ALL three perspectives to every changed file in scope:

    - **Security**: injection (SQL via f-strings/concatenation, command via
      os.system/subprocess/eval/exec, XSS via dangerouslySetInnerHTML/innerHTML),
      auth/authz gaps (missing permission checks on API endpoints), data exposure
      (credentials, PII/financial data in logs, CORS issues), SSRF (user-input-based
      URL calls), insecure deserialization
    - **Data Integrity**: float arithmetic on monetary values (should be Decimal),
      missing DB transaction boundaries on multi-step mutations, race conditions,
      financial logic errors (debit/credit balance, post-close modifications),
      NULL handling in calculations, referential integrity on deletes,
      timezone/date boundary issues
    - **Code Quality**: duplicate implementations of existing utilities,
      architectural pattern violations, dead code introduced by these changes,
      logic errors, syntax errors

    Read the actual changed files (not just the diff) for full context.

    ## Reporting Rules

    Report ONLY CRITICAL issues: actual bugs, security vulnerabilities,
    data loss/corruption risk, logic errors, syntax errors, design flaws,
    missing permission/authorization checks.

    Do NOT report warnings, style suggestions, or informational notes.
    Borderline → mark `Confidence: low` and report. False positive is cheaper
    than a missed vulnerability.

    ## Format

    Open with a single status line:
    - `✅ No critical issues found.` — and stop.
    - `❌ Critical findings:` — and continue with the findings block below.

    ```
    ## Critical Findings

    ### [C-001] Title
    - **File**: path/to/file
    - **Line**: N
    - **Type**: Security / Data Integrity / Code Quality
    - **Confidence**: high / low
    - **Description**: What the issue is
    - **Risk**: Why this matters
    - **Fix**: Concrete fix description
    ```

    Number findings sequentially (C-001, C-002, ...).
```
