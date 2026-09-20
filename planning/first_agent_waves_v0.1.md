# Space Frontier — First Agent Waves v0.1

This file is an operational launch sequence. It does not replace the task JSON.

## Wave A — bootstrap

Run concurrently:

| Role | Task | Start condition |
|---|---|---|
| R1 | `SF-ARCH-001` | Immediately |
| R8 | `SF-CONT-001` | Immediately |

**Gate:** `SF-ARCH-001` merged.

## Wave B — repository contracts

After the gate:

| Role | Task | Notes |
|---|---|---|
| R1 | `SF-ARCH-002` | Highest priority; establishes public contracts |
| R7 | `SF-QA-001` | Repository/PR conventions |
| R7 | `SF-QA-004` | Test harness foundation |

**Gate:** `SF-ARCH-002` merged.

## Wave C — first safe parallel system work

| Role | Task | Notes |
|---|---|---|
| R2 | `SF-BAL-001` | JSON conventions/versioning |
| R5 | `SF-NAV-001` | PathProvider contract |
| R1 | `SF-ARCH-003` | BattleWorld/BattleState skeleton |

R1 continues sequentially:

`SF-ARCH-003 → SF-ARCH-004 → SF-ARCH-005`

Do not parallelize those three across multiple architects during bootstrap.

## Wave D — data schemas

After `SF-BAL-001` is merged:

- `SF-BAL-002`
- `SF-BAL-003`
- `SF-BAL-004`
- `SF-BAL-005`
- `SF-BAL-007`

These may run in parallel only if each agent has isolated file ownership and does not modify the conventions established by `SF-BAL-001`.

## Concurrency rule

During P0, target no more than **4 active implementation agents** at one time.

Do not start Combat runtime tasks until public core/data contracts required by those tasks are merged.
