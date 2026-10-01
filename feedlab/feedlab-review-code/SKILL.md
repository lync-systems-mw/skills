---
name: feedlab-review-code
description: Review the current codebase for deficiencies (bugs, security holes, performance, reliability, accessibility, maintainability) and register the confirmed ones as issues in a FeedLab project via its MCP tools. Use when the user asks to review, audit, or scan code and log/report/register the problems in FeedLab, or to update FeedLab issues after fixing them.
---

# Review code into FeedLab issues

You review code, confirm each problem is real, and register the findings as **issues** in a FeedLab project through the `feedlab` MCP tools. The issues land in the team's triage board, so every one must be worth a teammate's time: real, located, and actionable.

If the `feedlab` tools (`list_projects`, `list_issues`, `create_issues`, …) are not available, stop and tell the user to create a token in FeedLab under Settings → API Tokens and connect with:

```bash
claude mcp add --transport http feedlab https://feedlab.cloud/api/mcp --header "Authorization: Bearer <token>"
```

## Steps

### 1. Scope

Pin down what to review:
- **Named area** ("the payments module"): that code and what it calls.
- **Recent changes** ("review my branch"): files touched by `git diff` against the base branch.
- **Whole codebase**: list the main areas you find, confirm the list with the user, then review one area at a time.

Ask about focus if the user gave one (e.g. "security only").

### 2. Connect

Call `list_projects` and pick the project matching this repo (name or website URL; ask if ambiguous). Call `list_issues` with `status: "open"` to know what's already reported, including earlier reviews (`source: "code_review"`).

### 3. Review

Read the code in scope and look for:
- **Security**: missing auth or permission checks, injection (SQL, HTML, command), secrets in code, unsafe defaults, data exposed to the wrong user.
- **Bugs**: wrong logic, unhandled errors or nulls, race conditions, off-by-one, broken edge cases (empty, limits, time zones).
- **Reliability**: missing retries or timeouts, silent failures, partial writes without transactions.
- **Performance**: N+1 queries, sequential calls that could run in parallel, unbounded queries or loops, heavy work on hot paths.
- **Accessibility and UX**: unlabeled controls, keyboard traps, low contrast, confusing or missing states.
- **Maintainability**: dead code, duplicated logic that has already drifted, missing validation at boundaries. Only when it causes or invites real bugs.

### 4. Verify

For every candidate, confirm it before it goes in the report:
- Trace the actual code path. Name the input or sequence of events that triggers it.
- Check it isn't already handled elsewhere (middleware, database constraints, row-level security, a wrapper).
- Drop it if you can't make a concrete case. A short list of real problems beats a long list of maybes.

### 5. Plan

Show the user the findings before registering anything: a table of title, severity, category, and location, grouped by severity, plus anything you skipped because it's already an open issue. Wait for approval, and apply their edits.

### 6. Register

Call `create_issues` with the approved findings, at most 25 per call, related findings together. Pass a `review_name` (e.g. "Auth review" or the branch name); the team gets one notification per call.

Done when every approved finding is created. Report the counts by severity and anything returned in `skipped_duplicates`, and tell the user the issues are in the project's feedback, tagged `code-review`.

### 7. Follow up (when asked to fix them)

When you fix a registered issue, call `update_issue` with `status: "completed"` and a `note` naming the commit or PR. Use `in_progress` while you're working on it.

## Issue rules

- **Title**: the problem, specific and scannable. "Password reset token never expires", not "Security issue in auth".
- **Description**: what is wrong and how it shows up, in 2–4 sentences a teammate can follow without re-reviewing the code.
- **Locations**: repo-relative file paths with line numbers, every place that needs to change.
- **Impact**: who is affected and how (users locked out, data leaked, page slow for large projects).
- **Recommendation**: the fix, concrete enough to act on.
- **Severity**:
  - `critical`: exploitable now, or causes data loss or a breach.
  - `high`: likely to break a core flow or affect many users.
  - `medium`: real, but limited in reach or with a workaround.
  - `low`: minor; worth fixing when nearby.
- **Category**: one of `security`, `bug`, `performance`, `reliability`, `accessibility`, `ux`, `maintainability`, `other`.
- **Type**: `bug` for defects, `feature` for a missing capability (e.g. no rate limiting at all).
- Never paste secrets, tokens, or personal data into an issue, even if you find them in the code; say where they are instead.
