# Space Frontier — AGENTS.md

## Purpose
This file is the operating contract for human and AI contributors implementing the Space Frontier MVP.

## Mandatory reading order
1. `docs/Space_Frontier_Technical_Design_v0.1.md`
2. `AGENTS.md`
3. `planning/space_frontier_mvp_tasks_v0.1.json`
4. Assigned GitHub Issue / Task ID
5. Existing public interfaces touched by the task

## Source of truth
- Code and merged interfaces: Git repository.
- Task state: GitHub Issue/Project, mirrored in `planning/space_frontier_mvp_tasks_v0.1.json`.
- Architecture: Technical Design + accepted Architecture Decision Records.
- Game balance: files under `configs/`. Never hardcode a balance number if it belongs in config.

## Branch rule
`agent/<role>/<task-id>-<short-name>`

Example:
`agent/r3/SF-COMBAT-008-poison-status`

## Isolation rules
- Work only inside the assigned role ownership unless the task explicitly requires a public interface change.
- A public contract change requires an RFC/issue and Lead Architect approval.
- UI/Presentation must not become the source of gameplay state.
- Combat/domain logic must remain executable headlessly.
- MVP navigation is fixed-path only. Do not implement maze/A*; preserve only the `IPathProvider` seam.
- No monetization systems in MVP.
- Pause and x1/x2/x3 use BattleClock semantics and must preserve deterministic results.
- Every level divisible by 5 must end with a unique configured boss.
- Poison is a first-class status/damage interaction with enemy poison resistance.

## Definition of Done for every task
A task is not Done until:
- implementation is merged;
- assigned DoD is satisfied;
- tests/config validation pass;
- no new architectural coupling is introduced;
- PR URL and test evidence are recorded;
- documentation/contracts are updated if behavior changed.

## PR handoff format
### Summary
What was implemented.

### Contracts changed
List interfaces/events/commands/config schemas changed, or `None`.

### Tests
Exact tests/commands and result.

### Risks
Known limitations or performance risks.

### Remaining TODO
Only items outside this task's scope.

## Forbidden
- Expanding MVP scope without an issue.
- Large opportunistic refactors in another role's module.
- Hardcoding tower/enemy/wave balance in GDScript.
- Using View nodes as gameplay truth.
- Marking a task Done without evidence.
