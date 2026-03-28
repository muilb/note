# SDD - Step Pack 1

# Specific Procedures and Prompt Set (Copilot-only)

_(Version 01)_

## Table of Contents

- [0. How To Use This Prompt Pack (For New Users)](#0-how-to-use-this-prompt-pack-for-new-users)
  - [0-1. Basic Operation In VS Code (Copilot Agent)](#0-1-basic-operation-in-vs-code-copilot-agent)
  - [0-2. Memory-Less Operation Rules (Critical)](#0-2-memory-less-operation-rules-critical)
  - [0-3. Safety Rails Must Come First](#0-3-safety-rails-must-come-first)
  - [0-4. Evidence Must Be Stored In Files, Not Chat](#0-4-evidence-must-be-stored-in-files-not-chat)
- [1. Variables To Replace Before Use](#1-variables-to-replace-before-use)
- [2. Shared Prompts Used In Every Phase](#2-shared-prompts-used-in-every-phase)
  - [2-1. Session Warm-Up Prompt (Paste First)](#2-1-session-warm-up-prompt-paste-first)
  - [2-2. Approve-Plan Prompt](#2-2-approve-plan-prompt)
  - [2-3. Rework Prompt After Review](#2-3-rework-prompt-after-review)
- [3. Phase 0-A: Safety Rails + Storage Locations + Evidence](#3-phase-0-a-safety-rails--storage-locations--evidence)
  - [3-1. Deliverables For Phase 0-A](#3-1-deliverables-for-phase-0-a)
  - [3-2. Prompt For Phase 0-A](#3-2-prompt-for-phase-0-a)
  - [3-3. Completion Gates For Phase 0-A](#3-3-completion-gates-for-phase-0-a)
- [4. Phase 0-B: Common Base](#4-phase-0-b-common-base)
- [5. Phase 1: Spec Pack (With AC)](#5-phase-1-spec-pack-with-ac)
- [6. Phase 2: Ticket Context Setup](#6-phase-2-ticket-context-setup)
- [7. Phase 3: Implementation Plan + Impact Analysis](#7-phase-3-implementation-plan--impact-analysis)
- [8. Phase 4: Review Checklist + Self-Review Template](#8-phase-4-review-checklist--self-review-template)
- [9. Phase 5: Implementation -> Self-Review -> Human Review -> Fixes](#9-phase-5-implementation---self-review---human-review---fixes)
- [10. Phase 6: Test Plan + Test Implementation](#10-phase-6-test-plan--test-implementation)
- [11. Phase 7: Black-Box Test Specs + Test Data](#11-phase-7-black-box-test-specs--test-data)
- [12. Phase 8: Report](#12-phase-8-report)
- [13. Phase 9: Periodic Maintenance](#13-phase-9-periodic-maintenance)
- [14. Quick Formula For Success](#14-quick-formula-for-success)
- [15. Claude-Term -> Copilot-Term Quick Mapping](#15-claude-term---copilot-term-quick-mapping)

---

## 0. How To Use This Prompt Pack (For New Users)

This pack is designed for the current environment:

- VS Code + Copilot Agent only
- No GitHub repository automation
- No Copilot code review automation
- No persistent Copilot memory

### 0-1. Basic Operation In VS Code (Copilot Agent)

- Start each phase with a plan-first prompt.
- Do not let the agent edit files before you approve the plan.
- Keep edits small and reviewable.
- Reference required files explicitly in each session.

### 0-2. Memory-Less Operation Rules (Critical)

Because memory is disabled, every new session must re-provide core context:

- .github/copilot-instructions.md
- docs/changes/<TICKET>/spec-pack.md
- relevant docs/standards/* and docs/architecture/*

### 0-3. Safety Rails Must Come First

In this environment, safety rails are policy-driven and file-driven:

- .github/copilot-instructions.md is the operational constitution.
- Manual discipline replaces deny/allow enforcement automation.

### 0-4. Evidence Must Be Stored In Files, Not Chat

Chat output is temporary. Audit-ready evidence must be written to files under:

- docs/maintenance/phase0/
- docs/changes/<TICKET>/

---

## 1. Variables To Replace Before Use

Replace placeholders before pasting prompts:

- {{TICKET}} example: ABC-123
- {{FEATURE_NAME}} example: Invite user by email
- {{SCOPE_NOTE}} example: Backend + Frontend + E2E
- {{TIMEBOX}} example: 45 minutes to finalize this phase plan

Fixed deliverable locations:

- Phase 0 evidence: docs/maintenance/phase0/
- Ticket evidence: docs/changes/{{TICKET}}/
- Living docs: docs/architecture/, docs/standards/

---

## 2. Shared Prompts Used In Every Phase

### 2-1. Session Warm-Up Prompt (Paste First)

```text
You are an SDD assistant for this repository.
We are working on {{TICKET}} ({{FEATURE_NAME}}).

[Read first]
#.github/copilot-instructions.md
#docs/changes/{{TICKET}}/spec-pack.md
#docs/standards/
#docs/architecture/

[Absolute working rules]
- Show PLAN first (files to read, files to create/update, checkpoints, risks)
- Do NOT edit files until I approve the plan
- Keep changes small and reviewable
- No speculative implementation; unresolved points must go to Open Issues
- Deliverables must stay under docs/changes/{{TICKET}}/

[Session note]
- Scope: {{SCOPE_NOTE}}
- Timebox: {{TIMEBOX}}

Return plan first. Do not edit yet.
```

### 2-2. Approve-Plan Prompt

```text
I approve the plan.
Proceed step by step.
After each step, provide a short change summary and next check.
If spec or standards need updates, propose first and wait for approval.
```

### 2-3. Rework Prompt After Review

```text
Please rework based on the comments below.

Requirements:
1) Restate each comment in one line before editing.
2) List impacted files/spec/tests.
3) Apply minimal changes only.
4) Update self-review.md and report.md accordingly.

[Comments]
(paste here)
```

---

## 3. Phase 0-A: Safety Rails + Storage Locations + Evidence

### 3-1. Deliverables For Phase 0-A

Safety and governance:

- .github/copilot-instructions.md

Evidence pack:

- docs/maintenance/phase0/README.md
- docs/maintenance/phase0/phase0-plan.md
- docs/maintenance/phase0/phase0-execution-log.md
- docs/maintenance/phase0/phase0-decisions.md
- docs/maintenance/phase0/phase0-risk-register.md
- docs/maintenance/phase0/phase0-review.md

### 3-2. Prompt For Phase 0-A

```text
[Phase 0-A: Safety Rails + Storage + Evidence]

Goal:
- Establish operating safety rules for Copilot-only
- Establish document locations for SDD deliverables
- Produce phase0 evidence in files

Constraints:
- Do not modify application source code in this phase
- Do not read or expose secrets
- Do not propose destructive commands

Tasks:
1) Present PLAN first (read/create/update/checkpoints/risks)
2) Ensure .github/copilot-instructions.md exists and is usable
3) Ensure docs/maintenance/phase0/* evidence files exist
4) Fill evidence files with objective rationale and residual risks

Output:
- File list created/updated
- Safety rationale summary
- Phase 0-A completion judgment (Pass/Fail + reasons)

Return plan first. Do not edit yet.
```

### 3-3. Completion Gates For Phase 0-A

- [ ] Safety rules documented in .github/copilot-instructions.md
- [ ] Evidence files exist under docs/maintenance/phase0/
- [ ] Risks and mitigations documented
- [ ] Explicit pass/fail review captured

---

## 4. Phase 0-B: Common Base

```text
[Phase 0-B: Common Base]

Goal:
- Build reusable repository-level base for architecture and standards

Inputs:
#docs/architecture/
#docs/standards/

Tasks:
1) Present PLAN first
2) Create/update docs/architecture/overview.md and key-flows.md
3) Create/update docs/standards/coding.md, testing.md, security.md
4) Keep rules concise; details go to standards

Output:
- Updated common-base file list
- Assumptions and cautions for Phase 1/2

Return plan first. Do not edit yet.
```

Completion gates:

- [ ] Architecture common base exists
- [ ] Standards common base exists
- [ ] No ticket-specific deliverables mixed into common-base files

---

## 5. Phase 1: Spec Pack (With AC)

```text
[Phase 1: Spec Pack]

Inputs:
- ticket requirement sources
- design documents
- mockups

Tasks:
1) Present PLAN first
2) Create docs/changes/{{TICKET}}/spec-pack.md
3) Include mandatory sections: background, scope, AC, NFR, examples, open issues
4) AC must be testable and numbered (AC-1, AC-2, ...)
5) Add AC traceability table

Output:
- spec-pack.md
- readiness judgment (Yes/No for implementation start)
- prioritized open issues

Return plan first. Do not edit yet.
```

Completion gates:

- [ ] AC list is testable and numbered
- [ ] Open issues isolated
- [ ] AC traceability exists

---

## 6. Phase 2: Ticket Context Setup

```text
[Phase 2: Ticket Context Setup]

Inputs:
#docs/changes/{{TICKET}}/spec-pack.md
#docs/architecture/
#docs/standards/

Tasks:
1) Present PLAN first
2) Update ticket-specific architecture/convention notes only when needed
3) Create/seed ticket files:
   - impl-plan.md
   - self-review.md
   - report.md
4) Keep common-base updates minimal and justified

Output:
- Updated file list
- gap list for Phase 3+

Return plan first. Do not edit yet.
```

Completion gates:

- [ ] Ticket working files seeded
- [ ] Required context updates captured without scope bloat

---

## 7. Phase 3: Implementation Plan + Impact Analysis

```text
[Phase 3: Implementation Plan + Impact Analysis]

Inputs:
#docs/changes/{{TICKET}}/spec-pack.md
#docs/architecture/
#docs/standards/

Tasks:
1) Present PLAN first
2) Build impl-plan.md with atomic steps
3) Add impact analysis for API, DB, FE, config, logs, security
4) Add AC mapping table
5) Add rollback and verification strategy

Output:
- impl-plan.md
- pre-implementation checklist

Return plan first. Do not edit yet.
```

Completion gates:

- [ ] Atomic implementation steps
- [ ] Impact analysis complete
- [ ] AC mapping complete
- [ ] Rollback defined

---

## 8. Phase 4: Review Checklist + Self-Review Template

```text
[Phase 4: Review Checklist + Self-Review]

Inputs:
#docs/changes/{{TICKET}}/spec-pack.md
#docs/changes/{{TICKET}}/impl-plan.md
#docs/standards/

Tasks:
1) Present PLAN first
2) Create/update review checklist with severity tags
3) Create/update self-review template with checkbox sections
4) Map checklist items to AC

Output:
- review checklist
- self-review template

Return plan first. Do not edit yet.
```

Completion gates:

- [ ] Checklist covers security, compatibility, logging, testing
- [ ] AC mapping present
- [ ] Self-review format is actionable

---

## 9. Phase 5: Implementation -> Self-Review -> Human Review -> Fixes

Important for current context:

- No Codex review dependency
- Human review is mandatory
- Optional independent second-pass Copilot review may be used

```text
[Phase 5: Implementation + Self-Review + Human Review]

Inputs:
#docs/changes/{{TICKET}}/spec-pack.md
#docs/changes/{{TICKET}}/impl-plan.md
#docs/changes/{{TICKET}}/self-review.md
#docs/standards/

Tasks:
1) Present PLAN first
2) Implement only within approved scope
3) Keep changes small and report each step briefly
4) Run feasible checks/tests
5) Fill self-review with What/Why/Checks/Risk
6) Prepare handoff summary for human review

Constraints:
- no destructive commands
- no secrets in docs/logs
- no speculative scope expansion

Output:
- implementation summary
- updated self-review
- human-review handoff notes

Return plan first. Do not edit yet.
```

Completion gates:

- [ ] Self-review completed
- [ ] Human review performed
- [ ] Findings resolved or tracked with owner

---

## 10. Phase 6: Test Plan + Test Implementation

```text
[Phase 6: Test Plan + Test Implementation]

Inputs:
#docs/changes/{{TICKET}}/spec-pack.md
#docs/changes/{{TICKET}}/impl-plan.md
#docs/changes/{{TICKET}}/self-review.md

Tasks:
1) Present PLAN first
2) Create/update test plan mapped to AC
3) Implement UT/IT/E2E tests as appropriate
4) Record executed checks and results in artifacts and self-review/report sections

Output:
- test updates
- test results evidence

Return plan first. Do not edit yet.
```

Completion gates:

- [ ] AC-to-test mapping exists
- [ ] Test evidence captured
- [ ] Deferred tests documented with risk

---

## 11. Phase 7: Black-Box Test Specs + Test Data

```text
[Phase 7: Black-Box Specs + Test Data]

Inputs:
#docs/changes/{{TICKET}}/spec-pack.md
#docs/changes/{{TICKET}}/test results or notes

Tasks:
1) Present PLAN first
2) Create/update black-box testcases by AC
3) Create/update test-data definitions
4) Include normal/abnormal/boundary perspectives

Output:
- black-box testcases
- test-data references

Return plan first. Do not edit yet.
```

---

## 12. Phase 8: Report

```text
[Phase 8: Report]

Inputs:
#docs/changes/{{TICKET}}/spec-pack.md
#docs/changes/{{TICKET}}/impl-plan.md
#docs/changes/{{TICKET}}/self-review.md
#docs/changes/{{TICKET}}/test evidence

Tasks:
1) Present PLAN first
2) Create/update report.md with:
   - change summary
   - impact summary
   - review summary
   - test summary
   - residual risks
   - follow-up actions

Output:
- report.md

Return plan first. Do not edit yet.
```

Completion gates:

- [ ] Report has What/Why/Checks/Risk
- [ ] Evidence references are present
- [ ] Residual risk and next actions are explicit

---

## 13. Phase 9: Periodic Maintenance

```text
[Phase 9: Periodic Maintenance]

Inputs:
#.github/copilot-instructions.md
#docs/architecture/
#docs/standards/
#recent docs/changes/*/report.md

Tasks:
1) Present PLAN first
2) Identify outdated or conflicting rules/docs
3) Propose minimal updates with rationale
4) Apply only approved updates

Output:
- maintenance update summary
- updated docs list

Return plan first. Do not edit yet.
```

Completion gates:

- [ ] Outdated rules/docs reviewed
- [ ] Changes justified by evidence
- [ ] Rule bloat avoided

---

## 14. Quick Formula For Success

- spec-pack.md = source of truth
- impl-plan.md = how to execute safely
- self-review.md = quality checkpoint
- tests + artifacts = verification evidence
- report.md = handover and audit trail
- periodic maintenance = continuous stabilization

---

## 15. Claude-Term -> Copilot-Term Quick Mapping

| Claude-origin term | Copilot-only equivalent |
|---|---|
| .claude/CLAUDE.md | .github/copilot-instructions.md |
| .claude/settings.json deny/ask/allow | explicit policy + manual discipline |
| Codex review | self-review + human review (+ optional independent Copilot pass) |
| CI gates | manual quality gates |
| project memory auto-load | session warm-up prompt |

---

## Final Note

Use this pack as an operational guide.

For each phase:

1. paste session warm-up prompt
2. request plan first
3. approve plan
4. execute with small changes
5. record evidence in files
