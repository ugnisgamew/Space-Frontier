# Space Frontier — MASTER_AGENT_PROMPT

## Purpose

This document defines the mandatory execution protocol for any AI agent working on Space Frontier.

An agent must execute only the assigned Task ID and must not independently expand project scope.

---

## 1. Mandatory reading

Before making any change, read:

1. `AGENTS.md`
2. `docs/Space_Frontier_Technical_Design_v0.1.md`
3. `planning/space_frontier_mvp_tasks_v0.1.json`
4. the assigned GitHub Issue / Task ID
5. the current merged repository code
6. all public interfaces directly related to the assigned task

Do not start implementation before reading these sources.

---

## 2. Source of truth

Resolve conflicts in this order:

1. current merged repository code and public interfaces;
2. `AGENTS.md`;
3. `docs/Space_Frontier_Technical_Design_v0.1.md`;
4. accepted architecture decisions;
5. `planning/space_frontier_mvp_tasks_v0.1.json`;
6. assigned GitHub Issue;
7. chat instructions.

If two higher-level sources conflict, do not invent a solution.

Report:

`ARCHITECTURE CONFLICT`

and explain the conflict.

---

## 3. Assigned task

You will receive one Task ID, for example:

`SF-COMBAT-008`

Execute only that Task ID.

Before implementation:

- verify its dependencies;
- read its Definition of Done;
- read its acceptance scenarios;
- identify the smallest set of files that must change;
- check whether the task requires any public contract change.

Do not begin another Task ID after completing the assigned one.

---

## 4. Scope control

Implement only what is required by:

- the assigned Issue;
- its Definition of Done;
- Technical Design;
- existing public contracts.

Do not:

- add unrelated features;
- perform large opportunistic refactors;
- implement future systems unless explicitly required;
- change architecture merely because another approach seems preferable;
- expand MVP scope.

If additional work is necessary but outside the assigned Task ID, report it under:

`REMAINING TODO`

Do not implement it automatically.

---

## 5. Architecture rules

The following rules are mandatory.

### Domain separation

Gameplay/domain logic must not depend on:

- UI;
- sprites;
- animations;
- presentation nodes;
- visual effects.

Presentation may observe domain Events but must not become the source of gameplay state.

### Data-driven balance

Do not hardcode gameplay balance values in GDScript when they belong in configuration.

Examples:

- tower damage;
- attack speed;
- range;
- costs;
- enemy HP;
- speed;
- armor;
- resistances;
- status values;
- wave composition;
- difficulty values.

### Headless compatibility

Combat/domain systems must remain executable without graphics where required by the Technical Design.

### Navigation

For MVP:

- implement fixed-path / Kingdom Rush navigation only;
- do not implement maze gameplay;
- do not implement AStar gameplay;
- preserve the architectural seam for future `GridPathProvider`.

### Time

Battle simulation must follow `BattleClock`.

Pause and x1/x2/x3 must preserve deterministic simulation results where applicable.

### Poison

Poison is a first-class combat/status mechanic.

Enemy configuration must support poison resistance.

Do not create a separate hardcoded `PoisonedEnemy` enemy class when a normal EnemyModel with a poison status is sufficient.

### Boss rule

Every level whose number is divisible by 5 must end with a unique configured boss.

---

## 6. Public contract changes

Do not silently change:

- public interfaces;
- Events;
- Commands;
- JSON schemas;
- shared data structures.

If the assigned task explicitly requires a public contract change:

1. make the smallest necessary change;
2. document it;
3. update tests;
4. report it under `CONTRACTS CHANGED`.

If the change is not required by the task, stop and report it as an architecture issue instead.

---

## 7. Git workflow

Use a dedicated branch.

Format:

`agent/<role>/<task-id>-<short-name>`

Example:

`agent/r3/SF-COMBAT-008-poison-status`

One Task ID should normally equal one Pull Request.

Do not combine unrelated Task IDs.

Preferred PR title:

`[TASK-ID] Task title`

Example:

`[SF-COMBAT-008] Poison / PoisonedEnemy integration`

Do not merge your own PR unless explicitly authorized.

---

## 8. Implementation quality

Code must be:

- readable;
- modular;
- testable;
- deterministic where required;
- free from unnecessary coupling;
- free from duplicated gameplay logic;
- consistent with existing project conventions.

Avoid unnecessary singleton dependencies and hidden side effects.

Comments should explain non-obvious intent, not obvious syntax.

---

## 9. Tests

Every implementation task must include the tests required by its Issue.

Depending on the task, verify:

- normal behaviour;
- edge cases;
- invalid input;
- pause/resume;
- x1/x2/x3;
- deterministic behaviour;
- config validation;
- headless execution;
- performance constraints.

Do not modify correct production behaviour merely to make an incorrect test pass.

If an existing test appears wrong, report the conflict.

---

## 10. Definition of Done

A task is complete only when:

- implementation is finished;
- Issue Definition of Done is satisfied;
- required tests pass;
- existing tests are not broken;
- configuration validation passes where applicable;
- architecture boundaries remain valid;
- documentation is updated when contracts change;
- no undocumented task-related TODO remains;
- self-review has been completed.

A task must not be marked Done merely because code was written.

---

## 11. Mandatory self-review

Before reporting completion, inspect your own work as a senior reviewer.

Check for:

- architecture violations;
- unnecessary coupling;
- duplicated logic;
- hardcoded balance values;
- deterministic simulation problems;
- BattleClock errors;
- invalid status stacking;
- performance regressions;
- missing edge cases;
- insufficient tests;
- scope creep;
- accidental changes to another role's module.

Fix discovered problems before final handoff.

---

## 12. Pull Request requirements

If GitHub access is available, create a Pull Request.

The PR must:

- reference the Task ID;
- link/close the assigned Issue;
- use the repository PR template;
- contain test evidence;
- state whether public contracts changed;
- state known risks;
- contain no unrelated changes.

Do not merge the PR yourself unless explicitly instructed.

---

## 13. Final report format

After completing the task, return the following report.

### TASK

`[TASK_ID] — [TASK_NAME]`

### STATUS

One of:

- `DONE`
- `PARTIALLY DONE`
- `BLOCKED`

### SUMMARY

Brief description of what was implemented.

### FILES CHANGED

List all created or modified files.

### IMPLEMENTATION

Describe the important technical decisions.

### CONTRACTS CHANGED

List changed:

- interfaces;
- Events;
- Commands;
- JSON schemas;
- shared data structures.

If none:

`None`

### TESTS

List exact tests/checks executed and their results.

### DEFINITION OF DONE

For every DoD condition:

`PASS` or `FAIL`

### ARCHITECTURE REVIEW

State whether the implementation complies with:

- `AGENTS.md`;
- Technical Design;
- existing public contracts.

### RISKS

Known technical risks.

If none:

`None`

### BLOCKERS

If none:

`None`

### REMAINING TODO

Only work outside the assigned Task ID.

If none:

`None`

### BRANCH

Branch name used.

### PULL REQUEST

PR URL if created.

Otherwise:

`Not created`

### RECOMMENDED NEXT TASK

State the Task ID that becomes logically available after this task.

Do not begin it automatically.

---

## 14. Forbidden actions

An agent must not:

- start another Task ID without authorization;
- modify MVP scope;
- implement monetization;
- implement maze/AStar gameplay for MVP;
- hardcode data-driven balance values;
- make Presentation the source of gameplay truth;
- bypass dependencies;
- mark work complete without test evidence;
- merge its own PR without authorization;
- silently redesign project architecture.

---

## 15. Core principle

The agent is an executor of an approved engineering task.

The agent may propose improvements, but must not independently redefine Space Frontier architecture, scope, task dependencies, or product requirements.
