---
name: feedlab-security-review
description: Audit the current codebase for security vulnerabilities (broken access control, injection, auth and session flaws, secrets, data exposure, insecure configuration, vulnerable dependencies) and register the confirmed ones as security issues in a FeedLab project via its MCP tools. Use when the user asks for a security review, security audit, vulnerability scan, or pentest-style review of their code and wants the findings in FeedLab.
---

# Security review into FeedLab issues

You audit code for security vulnerabilities, prove each one is real, and register the confirmed findings as **security issues** in a FeedLab project through the `feedlab` MCP tools. This is a defensive review of the user's own code: report what an attacker could do and how to fix it. Never exploit anything against a live system, and never exfiltrate data or secrets.

If the `feedlab` tools (`list_projects`, `list_issues`, `create_issues`, …) are not available, stop and tell the user to create a token in FeedLab under Settings → API Tokens and connect with:

```bash
claude mcp add --transport http feedlab https://feedlab.cloud/api/mcp --header "Authorization: Bearer <token>"
```

## Steps

### 1. Scope

Confirm what to audit: the whole codebase, a named area ("payments", "the public API"), or the current branch (`git diff` against the base branch). Note the stack (framework, database, auth provider, hosting) because it decides where protections live.

### 2. Connect

Call `list_projects` and pick the project matching this repo (ask if ambiguous). Call `list_issues` with `status: "open"` so you don't re-report known problems.

### 3. Map the attack surface

Before hunting, list every way input and identity enter the system, and cite the files:
- **Entry points**: routes, API handlers, server actions, webhooks, background jobs, file uploads, public forms, anything reachable without signing in.
- **Trust boundaries**: where the code decides who the caller is and what they may touch (session checks, middleware, row-level security, API tokens, service/admin credentials).
- **Sensitive assets**: credentials, personal data, payments, files, admin functions.

### 4. Hunt

Work through each class against the map:
- **Broken access control** (most common, check first): endpoints or actions missing an auth check; checks on the wrong resource; trusting client-supplied IDs, roles, or cookies (IDOR); admin/service credentials used without first checking the caller; database policies that are missing or too broad.
- **Injection**: SQL or query-builder strings built from input; HTML/email/template output without escaping (XSS); shell commands; path traversal in file names; unvalidated redirects.
- **Authentication and sessions**: weak or missing verification, tokens that never expire, tokens stored in plain text, password reset and invite flows that can be replayed or hijacked, missing rate limits on login and public forms.
- **Secrets**: keys or passwords committed to the repo, exposed to the browser (e.g. `NEXT_PUBLIC_*` holding a secret), or written to logs.
- **Data exposure**: API responses or queries returning more fields or rows than the caller should see; public storage buckets holding private files; verbose errors leaking internals.
- **Server-side request forgery and webhooks**: fetching user-supplied URLs, unsigned or unverified incoming webhooks, payment callbacks trusted without verifying with the provider.
- **Configuration**: missing security headers, permissive CORS, debug modes, default credentials.
- **Dependencies**: run the ecosystem's audit (`npm audit`, `pip-audit`, …) if available; report only vulnerabilities that are reachable from this code.

### 5. Prove it

For every candidate, before it goes in the report:
- Write the concrete attack: who the attacker is (anonymous, signed-in user of another workspace, member with a low role), the exact request or steps, and what they gain.
- Check every layer that could stop it: middleware, database constraints and row-level security, framework defaults, the auth provider. If any layer blocks it, it's not a finding (at most a low-severity hardening note, and only if the user wants those).
- Drop anything you can't make concrete. False alarms cost trust.

### 6. Plan

Show the user the findings before registering anything: a table of title, severity, the attacker, and location, grouped by severity. Mention anything skipped as already reported. Wait for approval and apply their edits.

For **critical** findings, suggest the user fix them before registering, or register them with care: issues are visible to everyone in the workspace.

### 7. Register

Call `create_issues` with `category: "security"` for every finding, at most 25 per call, with `review_name` set (e.g. "Security review – API routes"). Report the counts by severity and anything in `skipped_duplicates`, and tell the user the issues are in the project's feedback, tagged `code-review` and `security`.

When asked to fix them later, fix, then call `update_issue` with `status: "completed"` and a `note` naming the commit or PR.

## Issue rules

- **Title**: the vulnerability and what it allows. "Any signed-in user can read other workspaces' invoices", not "IDOR in invoices API".
- **Description**: the attack in 2–4 sentences: attacker, steps, result. Write it so a developer who didn't do the review can confirm it.
- **Locations**: every file and line involved, including where the missing check should go.
- **Impact**: what data or capability is exposed, and to whom.
- **Recommendation**: the specific fix (e.g. "check membership of the invoice's workspace before returning it"), plus defence in depth where it matters (a database policy as well as the app check).
- **Severity**:
  - `critical`: exploitable by an anonymous or any signed-in user to take over accounts, read or change other customers' data, or run code.
  - `high`: exploitable with some precondition (a member role, a victim's click) with serious impact.
  - `medium`: limited impact or hard to exploit, but real.
  - `low`: hardening; no demonstrated exploit.
- **Type**: `bug`, or `feature` for a missing control (e.g. no rate limiting on login).
- **Never include secrets, tokens, passwords, or personal data** in an issue, even ones you found. Say where they are ("a live API key is committed in config/prod.ts line 12") and recommend rotating them.
- Don't include working exploit code. Describe the steps; that's enough to fix it.
