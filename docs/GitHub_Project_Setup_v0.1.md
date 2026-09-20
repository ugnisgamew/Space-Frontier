# Space Frontier — GitHub Project Setup v0.1

**Status:** Approved setup baseline  
**Scope:** MVP project management and distributed AI development  
**Code implementation:** Out of scope for this setup

---

## 1. Purpose

This setup turns the Space Frontier repository into a controlled workspace for distributed human/AI development.

The repository must remain the source of truth for:

- merged code and public interfaces;
- architecture rules;
- AI contributor rules;
- task definitions and Definition of Done;
- Pull Requests and test evidence.

Google Sheets may be used as a PM dashboard, but it is not the source of truth for code or task completion.

---

## 2. Mandatory repository structure

```text
Space-Frontier/
├── AGENTS.md
├── MASTER_AGENT_PROMPT.md
├── README.md
│
├── .github/
│   ├── CODEOWNERS
│   ├── pull_request_template.md
│   └── ISSUE_TEMPLATE/
│       ├── agent_task.yml
│       ├── bug_report.yml
│       └── config.yml
│
├── docs/
│   ├── Space_Frontier_Technical_Design_v0.1.md
│   └── GitHub_Project_Setup_v0.1.md
│
├── planning/
│   ├── space_frontier_mvp_tasks_v0.1.json
│   ├── github_labels_v0.1.json
│   └── github_issue_seed_p0_v0.1.json
│
├── configs/
├── src/
├── scenes/
├── tests/
└── tools/
```

---

## 3. GitHub Project

Create one GitHub Project named:

**Space Frontier — MVP**

Use it for Issues and Pull Requests from the `Space-Frontier` repository.

### Project Status field

Configure the built-in `Status` field with exactly:

1. `Backlog`
2. `Ready`
3. `In Progress`
4. `Review`
5. `Blocked`
6. `Done`

### Custom fields

| Field | Type | Values / use |
|---|---|---|
| `Task ID` | Text | Stable identifier, e.g. `SF-COMBAT-008` |
| `Role` | Single select | `R1` … `R8` |
| `Priority` | Single select | `P0`, `P1`, `P2` |
| `Risk` | Single select | `Low`, `Medium`, `High` |

Do **not** duplicate Phase as a custom field. Phase is represented by GitHub Milestones.

### Recommended views

#### 1. `MVP Board`
- Layout: Board
- Group by: `Status`
- Main operational view.

#### 2. `Ready for Agents`
- Layout: Table
- Filter: `Status = Ready`
- Show: Task ID, Title, Role, Priority, Milestone, Assignee.

#### 3. `Critical Path`
- Layout: Table
- Filter: `Priority = P0`
- Sort: Milestone, Task ID.

#### 4. `By Role`
- Layout: Board
- Group by: `Role`
- Useful for workload/conflict review.

#### 5. `Blocked`
- Layout: Table
- Filter: `Status = Blocked`
- Every item here must have a written blocker in the Issue.

#### 6. `Review Queue`
- Layout: Table
- Filter: `Status = Review`
- Used by Lead Architect / QA before merge.

---

## 4. Project automations

Enable built-in GitHub Project workflows:

### Auto-add
Repository: `Space-Frontier`

Recommended filter:

```text
is:issue
```

This keeps all implementation Issues visible in the Project.

### Item added
When an Issue is added to the Project:

```text
Status = Backlog
```

The Lead changes it to `Ready` only after dependencies are satisfied.

### Closed Issue
When an Issue is closed:

```text
Status = Done
```

### Merged Pull Request
When a Pull Request is merged:

```text
Status = Done
```

Important: an implementation Issue should normally be closed by the merged PR using `Closes #<issue-number>`.

---

## 5. Milestones

Create exactly these milestones:

1. `P0 — Foundation`
2. `P1 — Headless Core`
3. `P2 — Vertical Slice`
4. `P3 — Content 1–5`
5. `P4 — Balance & Performance`
6. `P5 — MVP Release Candidate`

Do not set arbitrary due dates until the actual delivery cadence is agreed.

### Exit gates

| Milestone | Exit gate |
|---|---|
| P0 | Repository contracts, config schemas, navigation contract and QA foundation are stable |
| P1 | Deterministic headless battle works end-to-end |
| P2 | Level 1 is fully playable with HUD, build, upgrade, sell, targeting, pause and x1/x2/x3 |
| P3 | Levels 1–5 work; Level 5 ends with a unique boss |
| P4 | Balance and 500+ object performance targets pass |
| P5 | Acceptance suite passes; no blocker/critical defects; export smoke tests pass |

---

## 6. Labels

Use labels only for classification, not for workflow state.

Workflow state belongs in the GitHub Project `Status` field.

Create the labels from:

`planning/github_labels_v0.1.json`

Primary label groups:

- `type:*`
- `area:*`
- `priority:*`
- `role:*`
- `risk:*`

Avoid labels such as `in progress`, `done`, or `blocked`; that duplicates the Project Status field.

---

## 7. Issue policy

### One Issue = one Task ID

Correct:

```text
SF-COMBAT-008 — Poison / PoisonedEnemy integration
```

Incorrect:

```text
Finish combat system
```

Do not combine multiple Task IDs into one Issue unless Lead Architect explicitly approves it.

### Required Issue content

Every implementation Issue must contain:

- Task ID;
- role;
- milestone;
- priority;
- objective;
- dependencies;
- source documents;
- scope;
- out of scope;
- deliverable;
- Definition of Done;
- acceptance scenarios;
- permitted ownership area;
- expected tests.

Use `.github/ISSUE_TEMPLATE/agent_task.yml`.

---

## 8. Pull Request policy

### Branch naming

```text
agent/<role>/<task-id>-<short-name>
```

Example:

```text
agent/r3/SF-COMBAT-008-poison-status
```

### PR title

```text
[SF-COMBAT-008] Poison / PoisonedEnemy integration
```

### PR size

A PR should implement one Task ID.

Large opportunistic refactors are prohibited.

### PR body

Use `.github/pull_request_template.md`.

The PR must state:

- linked Issue;
- summary;
- changed contracts;
- tests and evidence;
- architecture compliance;
- risks;
- remaining TODO.

### Merge strategy

Preferred:

**Squash and merge**

The resulting commit message should retain the Task ID.

---

## 9. Main branch protection / ruleset

Create a branch ruleset:

**Name:** `Protect main`  
**Target:** default branch `main`

### Stage A — enable immediately

Enable:

- Require a Pull Request before merging.
- Block force pushes.
- Block branch deletion.
- Require conversation resolution before merging, if available.

Recommended initially:

- Required approving reviews: `0` or `1` depending on whether agents use separate GitHub identities.
- Do not require signed commits for MVP.
- Do not require status checks yet.

Reason: CI jobs do not exist until QA/DevOps implements them. Requiring nonexistent checks can block all merges.

### Stage B — enable after CI exists

After `SF-QA-003`, `SF-QA-004`, and later test jobs are merged:

Enable required status checks for the actual job names created by the repository.

Expected categories:

```text
config-validation
unit-tests
integration-tests
```

After performance automation exists, optionally add:

```text
performance-regression
```

Never configure a required check name before that check actually exists.

---

## 10. CODEOWNERS

For the MVP bootstrap, the repository owner is the fallback owner.

`CODEOWNERS` in this package uses:

```text
* @ugnisgamew
```

Do **not** enable "Require review from Code Owners" if the same GitHub account will also author all AI-generated PRs; GitHub approval rules can make that workflow inconvenient.

When agents receive separate GitHub identities or a human team is added, replace the fallback ownership with module-specific owners.

---

## 11. Source of truth hierarchy

Agents must resolve information in this order:

1. merged repository code and current public interfaces;
2. `AGENTS.md`;
3. `docs/Space_Frontier_Technical_Design_v0.1.md`;
4. accepted Architecture Decision Records, when introduced;
5. `planning/space_frontier_mvp_tasks_v0.1.json`;
6. assigned GitHub Issue;
7. chat instructions.

An agent must not silently invent a different architecture when these sources conflict.

---

## 12. Task lifecycle

```text
Backlog
   ↓
dependency check
   ↓
Ready
   ↓
agent assigned
   ↓
In Progress
   ↓
Pull Request
   ↓
Review
   ├── changes requested → In Progress
   ├── blocker found     → Blocked
   └── accepted
          ↓
       Merge
          ↓
        Done
```

### Who changes Status

| Status | Responsible |
|---|---|
| Backlog | Lead / PM |
| Ready | Lead Architect / PM |
| In Progress | assigned agent |
| Review | assigned agent when PR is ready |
| Blocked | agent or Lead, with explicit reason |
| Done | normally automation after Issue close / PR merge |

---

## 13. Distributed agent execution rule

Do **not** start eight agents at once on the first day.

The first goal is to stabilize repository structure and public contracts.

### Bootstrap Wave A

Run in parallel:

#### Agent R1
`SF-ARCH-001`  
Create Godot project/repository skeleton.

#### Agent R8
`SF-CONT-001`  
Create MVP content bible.

These tasks do not share runtime implementation.

### Wave B — after `SF-ARCH-001` is merged

Run:

#### R1
`SF-ARCH-002` — Public interfaces.

#### R7
`SF-QA-001` — Git/PR conventions and repository contribution rules.

#### R7 / QA agent
`SF-QA-004` — Unit-test harness.

Do not start R2/R3/R5 system implementation until `SF-ARCH-002` is merged.

### Wave C — after `SF-ARCH-002` is merged

Safe parallel set:

#### R2
`SF-BAL-001` — JSON conventions/versioning.

#### R5
`SF-NAV-001` — IPathProvider contract.

#### R1
`SF-ARCH-003` — BattleWorld/BattleState skeleton.

R1 then continues, preferably sequentially:

```text
SF-ARCH-003
→ SF-ARCH-004
→ SF-ARCH-005
```

This prevents multiple agents from simultaneously changing core event/command contracts.

### Wave D — after `SF-BAL-001`

R2 data tasks may be split by file ownership:

- `SF-BAL-002` — loader/validation;
- `SF-BAL-003` — Tower schema;
- `SF-BAL-004` — Enemy schema;
- `SF-BAL-005` — Status schema;
- `SF-BAL-007` — Level/Wave schema.

If more than one R2 agent works concurrently, each one must own separate schema/files and must not change shared conventions from `SF-BAL-001`.

### Concurrency limit

Recommended MVP bootstrap limit:

**maximum 4 active implementation agents at once.**

Increase only after P0 contracts are stable.

---

## 14. First issue set

The machine-readable first issue set is:

`planning/github_issue_seed_p0_v0.1.json`

Recommended initial ordering:

```text
1. SF-ARCH-001
2. SF-CONT-001

after SF-ARCH-001:
3. SF-ARCH-002
4. SF-QA-001
5. SF-QA-004

after SF-ARCH-002:
6. SF-BAL-001
7. SF-NAV-001
8. SF-ARCH-003

then:
9.  SF-ARCH-004
10. SF-ARCH-005

after SF-BAL-001:
11. SF-BAL-002
12. SF-BAL-003
13. SF-BAL-004
14. SF-BAL-005
15. SF-BAL-007
```

This ordering deliberately delays Combat implementation until the core/data contracts are stable.

---

## 15. When an agent receives a task

The instruction sent to an agent can be short because repository rules already exist.

Example:

```text
Open the Space Frontier repository.

Read, in order:
1. AGENTS.md
2. MASTER_AGENT_PROMPT.md
3. docs/Space_Frontier_Technical_Design_v0.1.md
4. planning/space_frontier_mvp_tasks_v0.1.json
5. GitHub Issue assigned to you

Execute only Task SF-BAL-003.

Verify dependencies before changing files.
Follow the Issue Definition of Done.
Create a dedicated branch and Pull Request.
Run all required tests and perform self-review.
Do not start another Task ID.
```

The GitHub Issue carries task-specific detail. The prompt should not duplicate the entire technical design.

---

## 16. Definition of "Project Setup complete"

GitHub Project Setup v0.1 is complete when:

- Project `Space Frontier — MVP` exists;
- Status options are configured;
- Role/Priority/Task ID/Risk fields exist;
- six views exist;
- six milestones exist;
- labels from the JSON catalog exist;
- Issue Forms are available;
- PR template is available;
- CODEOWNERS is committed;
- `Protect main` Stage A is enabled;
- task JSON is committed;
- first Issues are created;
- only tasks with satisfied dependencies are set to `Ready`.

At that point distributed agent implementation can begin.
