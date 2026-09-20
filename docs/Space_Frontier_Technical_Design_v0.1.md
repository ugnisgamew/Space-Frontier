[Space_Frontier_Technical_Design_v0.1.md](https://github.com/user-attachments/files/32441221/Space_Frontier_Technical_Design_v0.1.md)
# Space Frontier — Technical Design v0.1

> **Repository source of truth:** этот файл является Markdown-версией Technical Design v0.1 для хранения в GitHub и чтения людьми/ИИ-агентами. Архитектурные требования и DoD ниже являются нормативными для MVP.

> **Статус —** Архитектурный baseline для MVP. Документ фиксирует контракты до начала массовой разработки.

| **Параметр**     | **Решение**                                                                     |
|------------------|---------------------------------------------------------------------------------|
| Движок           | Godot 4.7.2 stable                                                              |
| Язык             | GDScript                                                                        |
| Жанр             | 2D Tower Defence / Space Fantasy                                                |
| MVP              | Классические отдельные уровни, без монетизации                                  |
| Pathfinding      | Kingdom Rush style: фиксированные маршруты; интерфейс подготовлен под maze-mode |
| Скорость         | Pause + x1 / x2 / x3 всегда доступны                                            |
| Боссы            | Каждый 5-й уровень заканчивается уникальным боссом                              |
| Авторитет данных | JSON-конфиги + runtime validation                                               |

## Цель документа

Дать независимым исполнителям точные границы модулей, публичные интерфейсы, данные, события, команды, порядок жизни боя и проверяемый Definition of Done. Любая реализация, нарушающая эти контракты, считается архитектурным дефектом, даже если визуально работает.

## Ключевой принцип

> **Правило №1 —** Боевой домен не зависит от Node2D, Sprite2D, AnimationPlayer, Control и других визуальных классов Godot. View читает состояние и события домена, но не вычисляет исход боя.

## Содержание

1. Решения и границы MVP

2. Целевая архитектура и зависимости

3. Структура репозитория

4. Core interfaces

5. Data-driven JSON contracts

6. Combat systems

7. Pathfinding и maze-ready seam

8. BattleClock, Pause и x1/x2/x3

9. Levels, waves и boss cadence

10. Event contracts

11. Command contracts

12. Lifecycle одного боя

13. Balance simulator, QA и CI

14. Work packages и DoD для 8 агентов

15. MVP Definition of Done

16. Риски и архитектурные решения

17. Приложение A. Примеры JSON

18. Приложение B. Нормативные ссылки Godot

## Версионирование документа

| **Версия** | **Дата**   | **Содержание**                                                                                                            |
|------------|------------|---------------------------------------------------------------------------------------------------------------------------|
| 0.1        | 20.09.2026 | Первый baseline: MVP scope, combat/domain contracts, poison, boss cadence, fixed-path mode, pause/speed, 8 work packages. |

## 1. Решения и границы MVP

### 1.1. Что входит в MVP

- Классические отдельные уровни с началом, последовательностью волн, победой/поражением и экраном результата.

- Фиксированные дороги/сплайны (Kingdom Rush style). Башни не изменяют маршрут в MVP.

- Data-driven башни, враги, снаряды, статусы, уровни, волны, сложность и боссы.

- Типы урона: kinetic, plasma, laser, void, poison, true.

- Status Effects: freeze, burn, stun, armor_break, poisoned. Архитектура допускает новые эффекты без правки ядра.

- Pause всегда доступна; скорость боя переключается x1 → x2 → x3 → x1.

- Каждый 5-й уровень: финальная волна содержит уникального босса.

- Пул снарядов, VFX и damage numbers; target acquisition через spatial index.

- Headless battle simulation и smoke-тесты баланса в CI.

### 1.2. Что намеренно НЕ входит

- Монетизация, IAP, реклама, battle pass.

- Endless, roguelite, PvP, co-op.

- Свободное maze-building и динамическая перестройка маршрутов.

- Онлайн-бэкенд и серверная экономика.

- Сложная account/meta progression. Допускаются только локальные unlock-флаги, если нужны для теста уровней.

### 1.3. Архитектурный задел

> **Maze-ready —** BattleWorld зависит от IPathProvider, а не от Path2D напрямую. В MVP подключён SplinePathProvider. Будущий GridPathProvider/FlowFieldProvider должен подключаться заменой реализации без изменения EnemyMovementSystem.

## 2. Целевая архитектура и зависимости

### 2.1. Слои

```text
CONFIG / CONTENT
↓
CONFIG REPOSITORY + VALIDATION
↓
BATTLE DOMAIN (pure game rules)
↓ events / state snapshots
PRESENTATION + UI + AUDIO
META / LEVEL SELECT → BattleStartRequest → BATTLE → BattleResult
```

### 2.2. Разрешённые зависимости

| **Слой**      | **Может зависеть от**                           | **Не может зависеть от**                              |
|---------------|-------------------------------------------------|-------------------------------------------------------|
| Config        | DTO / schemas                                   | Scenes, UI, runtime nodes                             |
| Battle Domain | Config DTO, math, deterministic RNG, interfaces | Sprite2D, Control, AnimationPlayer, AudioStreamPlayer |
| Systems       | Models + domain services                        | Concrete views                                        |
| Presentation  | Read-only model snapshots + events              | Внутренности других views                             |
| UI            | Commands, read models                           | Прямые мутации TowerModel/EnemyModel                  |
| Tools/Tests   | Domain + configs                                | Production UI                                         |

### 2.3. Главные invariants

- Один и тот же seed + config snapshot + command stream должен давать тот же результат боя.

- Ни одна View не наносит урон и не меняет HP.

- Ни одна башня не содержит special-case проверки имени другой башни для синергии.

- Все коэффициенты баланса находятся в конфиге; константы в коде допустимы только как технические лимиты.

- События сообщают о свершившемся факте; команды выражают намерение игрока/системы.

- Pause и x1/x2/x3 не меняют математический результат боя.

## 3. Структура репозитория

```text
SpaceFrontier/
├── assets/
│ ├── sprites/ audio/ fonts/ shaders/ vfx/
├── configs/
│ ├── towers/ enemies/ projectiles/ statuses/ bosses/
│ ├── synergies/ difficulties/
│ └── levels/
├── scenes/
│ ├── battle/ towers/ enemies/ maps/ ui/
├── src/
│ ├── core/
│ ├── battle/
│ ├── models/
│ ├── systems/
│ ├── pathfinding/
│ ├── pooling/
│ ├── presentation/
│ ├── ui/
│ └── meta/
├── tests/
│ ├── unit/ integration/ balance/ regression/
├── tools/
│ ├── config_validator/ balance_simulator/
└── docs/
├── architecture/ balance/ adr/
```

### 3.1. Git rules

- main — только зелёная ветка; merge через Pull Request.

- feature/\<ticket\>-\<slug\>, fix/\<ticket\>-\<slug\>, balance/\<ticket\>-\<slug\>.

- Один PR — одна ответственность. Конфиг-баланс не смешивается с массовым рефакторингом.

- Любое изменение JSON schema требует migration note и обновления validator/tests.

- Generated/cache/import-файлы не коммитятся, кроме файлов, явно необходимых Godot-проекту.

- CODEOWNERS: core/contracts — Lead Architect; configs — Balance; UI — UI/UX; CI/tests — QA/DevOps.

## 4. Core interfaces

Интерфейсы ниже — нормативные контракты. Сигнатуры могут быть адаптированы к идиомам GDScript, но смысл и направление зависимостей сохраняются.

| **Интерфейс**        | **Контракт**                                                                                                            |
|----------------------|-------------------------------------------------------------------------------------------------------------------------|
| IConfigRepository    | get_tower(id), get_enemy(id), get_status(id), get_level(id), get_difficulty(id); immutable snapshots after BattleStart. |
| IBattleClock         | is_paused(), speed(), set_paused(bool), set_speed(1\|2\|3), advance(real_delta) -> simulation_steps.                   |
| IPathProvider        | get_path(path_id), sample(path_id, progress), distance_to_exit(path_id, progress), validate_level_paths(level).         |
| ITargetQuery         | query(position, radius, filter) -> candidate enemy ids.                                                                |
| ITargetStrategy      | select(candidates, tower, battle_state) -> enemy_id\|null.                                                             |
| IDamageResolver      | resolve(DamageRequest) -> DamageResult.                                                                                |
| IStatusSystem        | apply(StatusApplyRequest), remove(), tick(step_dt), query(entity_id).                                                   |
| IProjectilePool      | acquire(projectile_def_id), release(handle), active_count().                                                            |
| IEventBus            | publish(domain_event), subscribe(event_type, handler).                                                                  |
| ICommandBus          | dispatch(command) -> CommandResult.                                                                                    |
| IRandomSource        | next_float(), next_int(), fork(stream_name); seeded and deterministic.                                                  |
| IBattleResultBuilder | build(final_state, telemetry) -> BattleResult.                                                                         |

### 4.1. Основные runtime-модели

| **Model**       | **Минимальные поля**                                                                           |
|-----------------|------------------------------------------------------------------------------------------------|
| BattleState     | battle_id, level_id, seed, tick, phase, currency, lives, current_wave, entities, result        |
| TowerModel      | id, definition_id, position, level, cooldown, targeting_mode, buffs, total_damage              |
| EnemyModel      | id, definition_id, hp, max_hp, path_id, path_progress, speed_mod, resistances, statuses, alive |
| ProjectileModel | id, definition_id, source_id, target_id, position, velocity, lifetime, payload                 |
| StatusInstance  | instance_id, status_id, source_id, target_id, stacks, remaining, next_tick                     |
| WaveState       | index, started, spawn_cursor, alive_count, completed                                           |

## 5. Data-driven JSON contracts

### 5.1. Общие правила

- UTF-8 JSON. Поля snake_case. ID: lowercase snake_case, глобально стабильные.

- Каждый файл содержит schema_version.

- Неизвестное обязательное enum-значение = validation error; неизвестное optional поле = warning до миграции.

- Конфиги immutable во время боя: при старте создаётся ConfigSnapshot.

- Runtime не читает файлы посреди боя.

- Числовые диапазоны валидируются до запуска уровня.

### 5.2. TowerDefinition

```json
{
"schema_version": 1,
"id": "toxic_spire_t1",
"tags": ["tower", "poison", "dot"],
"economy": {"build_cost": 110, "sell_ratio": 0.7},
"attack": {
"base_damage": 10.0,
"damage_type": "poison",
"attacks_per_second": 1.2,
"range": 250.0,
"projectile_id": "toxic_orb",
"targeting_default": "first",
"status_on_hit": [{"status_id": "poisoned", "chance": 1.0}]
},
"upgrade_to": ["toxic_spire_t2"],
"synergy_tags": ["bio", "poison"]
}
```

> **DPS —** Не хранить как независимую ручную величину. Базовый single-target DPS вычисляется из base_damage × attacks_per_second и затем корректируется с учётом DoT/aoe/uptime аналитическими инструментами.

### 5.3. EnemyDefinition

```json
{
"schema_version": 1,
"id": "void_raider",
"max_hp": 260.0,
"base_speed": 72.0,
"armor": 12.0,
"resistances": {
"kinetic": 0.10, "plasma": 0.00, "laser": 0.20,
"void": 0.35, "poison": -0.15, "true": 0.00
},
"status_resistance": {"stun": 0.10, "freeze": 0.0, "poisoned": 0.0},
"kill_reward": 9,
"lives_damage": 1,
"tags": ["organic", "ground"]
}
```

`resistances.poison` отвечает за входящий poison damage. `status_resistance.poisoned` отдельно регулирует вероятность/длительность наложения статуса, если это понадобится конкретному врагу. Эти механики нельзя смешивать.

### 5.4. StatusDefinition

```json
{
"schema_version": 1,
"id": "poisoned",
"duration": 5.0,
"tick_interval": 1.0,
"stack_rule": "independent_stacks",
"max_stacks": 5,
"effects": [
{"kind": "damage_over_time", "damage_type": "poison", "amount_per_tick": 6.0}
]
}
```

Canonical status ID — `poisoned`. Никакого отдельного класса `PoisonedEnemy` в домене не создаётся: это EnemyModel + StatusInstance(poisoned). Визуальный слой может отображать состояние как угодно.

### 5.5. DifficultyDefinition

```json
{
"id": "medium",
"difficulty_multiplier": 1.0,
"scaling": {
"hp_exponent": 1.00,
"armor_exponent": 0.65,
"speed_exponent": 0.15,
"reward_exponent": 0.20
}
}
```

Формула: `scaled_value = base_value × difficulty_multiplier ^ exponent`. Это сохраняет одну глобальную ручку сложности, но не делает скорость врагов столь же агрессивной, как HP.

### 5.6. LevelDefinition / WaveDefinition

```json
{
"schema_version": 1,
"id": "sector_05",
"level_number": 5,
"start_currency": 450,
"start_lives": 20,
"map_scene": "res://scenes/maps/sector_05.tscn",
"path_provider": {"type": "spline", "path_ids": ["main"]},
"waves": [
{"id": "w01", "groups": [...]},
{"id": "w10", "groups": [...], "boss_id": "astral_devourer"}
]
}
```

Validator rule: если `level_number % 5 == 0`, последняя волна обязана иметь ровно один `boss_id`, а этот boss_id не должен использоваться как milestone-boss другого уровня кампании. Для остальных уровней boss_id в MVP запрещён, если явно не введено исключение новой версией дизайна.

### 5.7. BossDefinition

```json
{
"schema_version": 1,
"id": "astral_devourer",
"enemy_base": "boss_base",
"max_hp": 12000,
"base_speed": 36,
"armor": 45,
"resistances": {"kinetic":0.15,"plasma":0.25,"laser":0.10,"void":0.55,"poison":0.30,"true":0.0},
"phase_thresholds": [0.66, 0.33],
"abilities": ["void_pulse", "summon_shards"],
"tags": ["boss", "unique", "void"]
}
```

Boss abilities вызываются через BossAbilitySystem/ability definitions, а не через уникальный монолитный script на каждого босса. Допускается thin adapter для уникальной постановки/VFX, но урон и статусы проходят через общие системы.

### 5.8. Обязательные enum

| **Enum**           | **Значения MVP**                                                                           |
|--------------------|--------------------------------------------------------------------------------------------|
| damage_type        | kinetic, plasma, laser, void, poison, true                                                 |
| targeting_mode     | first, last, strongest, weakest                                                            |
| stack_rule         | refresh_duration, stack_duration, stack_magnitude, replace_if_stronger, independent_stacks |
| status kind        | damage_over_time, stat_modifier, stun, armor_modifier                                      |
| path_provider.type | spline (MVP); grid / flow_field reserved                                                   |
| battle_speed       | 1, 2, 3                                                                                    |

## 6. Combat systems

| **System**         | **Responsibility**                                              | **Запрещено**                  |
|--------------------|-----------------------------------------------------------------|--------------------------------|
| WaveSystem         | Спавн групп, интервалы, завершение волны, финальный boss spawn  | Расчёт damage                  |
| MovementSystem     | Продвижение EnemyModel по IPathProvider                         | Выбор цели башнями             |
| SpatialIndexSystem | Индекс enemy positions для быстрых range queries                | Решать targeting policy        |
| TargetingSystem    | Получить кандидатов и применить strategy                        | Наносить урон                  |
| AttackSystem       | Cooldown, attack readiness, формирование projectile/hit request | Менять HP напрямую             |
| ProjectileSystem   | Жизненный цикл projectile models, hit detection/arrival         | Спавнить VFX напрямую          |
| DamageSystem       | Единая формула damage/armor/resistance                          | Проигрывать анимации           |
| StatusSystem       | Apply/stack/tick/remove statuses                                | Hardcode tower/enemy names     |
| DeathSystem        | Смерть, reward request, EnemyKilledEvent                        | Менять UI                      |
| EconomySystem      | Build/upgrade/sell/reward, проверка валюты                      | Знать о кнопках UI             |
| BossAbilitySystem  | Phases/abilities по definitions                                 | Обходить Damage/Status systems |
| Juice/View         | VFX, shake, damage numbers, animations                          | Влиять на исход боя            |

### 6.1. Damage pipeline

```text
DamageRequest
→ outgoing modifiers
→ armor penetration
→ armor mitigation
→ damage-type resistance (including poison)
→ received modifiers
→ clamp / true-damage rule
→ HP mutation
→ DamageResult + EnemyDamagedEvent
```

В MVP формула armor должна быть одна и покрыта unit tests. Конкретную математическую кривую (линейная, hyperbolic и т.п.) Balance Engineer фиксирует отдельным ADR до наполнения контента.

### 6.2. Object Pooling

- Пули/ракеты/магические орбы: ProjectileModel — домен; ProjectileView — pooled visual.

- Взрывы, hit flashes, death effects, damage numbers — отдельные pools.

- Pool обязан иметь prewarm, max_capacity, acquire/release counters и debug overlay.

- В production-loop запрещено создавать/удалять сотни визуальных nodes в секунду через instantiate/queue_free без профилирования и исключения.

## 7. Pathfinding и seam для будущего maze-mode

### 7.1. MVP: SplinePathProvider

- Уровень содержит один или несколько Path2D/Curve2D маршрутов.

- EnemyModel хранит path_id + path_progress, а не NodePath к сцене.

- MovementSystem использует IPathProvider.sample(); графика интерполирует между simulation snapshots.

- Path validation выполняется при загрузке уровня: start/end, длина \> 0, уникальные path_id.

### 7.2. Будущее: GridPathProvider / FlowFieldProvider

```text
IPathProvider
├── SplinePathProvider ← MVP
├── GridPathProvider ← future maze
└── FlowFieldPathProvider ← future high-density maze
```

EnemyMovementSystem не меняется. Будущий placement validation сможет временно блокировать grid cell и отклонять постройку, если spawn→exit недостижим.

> **Не реализовывать сейчас —** A\* reroute, flow fields, dynamic terrain mutation и build-block validation не входят в MVP. Реализовать только интерфейс, enum type и contract tests, гарантирующие возможность замены provider.

## 8. BattleClock, Pause и x1/x2/x3

### 8.1. Почему отдельный BattleClock

Глобальный Engine.time_scale удобен, но для воспроизводимой headless-симуляции и тестов лучше собственный fixed-step clock. BattleClock масштабирует количество симуляционных шагов, а UI/меню остаются на real time.

```gdscript
SIMULATION_HZ = 30
FIXED_DT = 1.0 / 30.0
accumulator += real_delta * battle_speed
while accumulator >= FIXED_DT and steps < MAX_STEPS_PER_FRAME:
battle_world.tick(FIXED_DT)
accumulator -= FIXED_DT
# battle_speed ∈ {1,2,3}
# pause → no BattleWorld.tick()
```

### 8.2. UX-контракт

- Pause button видима и доступна на протяжении всего активного боя, кроме уже завершённого result state.

- Speed button всегда показывает текущее значение: x1 / x2 / x3.

- Tap/click циклически: x1→x2→x3→x1. После паузы выбранная скорость сохраняется.

- Pause останавливает wave timers, movement, attacks, projectiles, statuses и boss abilities.

- Pause overlay и кнопки Resume / Restart / Exit обрабатывают input при остановленном бою.

- UI tween/hover может продолжать работать в real time; он не меняет combat state.

### 8.3. Godot integration

SceneTree.paused может применяться как дополнительный runtime guard для gameplay-node представлений; pause UI должен иметь process mode, позволяющий обработку во время паузы. Сам BattleWorld всё равно контролируется BattleClock, что делает headless и runtime поведение единообразным.

## 9. Levels, waves и boss cadence

### 9.1. Правило milestone boss

```text
if level_number % 5 == 0:
assert final_wave.boss_id != null
assert exactly_one_boss(final_wave)
assert boss_id_is_unique_for_campaign_milestone()
else:
assert final_wave.boss_id == null # MVP rule
```

### 9.2. Boss uniqueness

- Уникальность означает отдельный boss definition + отличимая механика/phase pattern, а не только новый sprite и +HP.

- Босс не должен быть обычным enemy с 20× HP. Минимум один уникальный ability/phase mechanic.

- Boss reward, intro/outro VFX и UI могут отличаться, но combat damage/status идут через общие pipelines.

- Level 5, 10, 15... формируют milestone. Для минимального content-MVP рекомендуется минимум 5 уровней, чтобы проверить весь цикл и boss cadence.

## 10. Event contracts

Все события immutable, имеют `event_id`, `battle_id`, `tick`, `type`, `payload`. Event — факт после успешного изменения доменного состояния.

| **Event**            | **Payload**                                            |
|----------------------|--------------------------------------------------------|
| BattleStarted        | level_id, difficulty_id, seed                          |
| BattlePaused         | paused, selected_speed                                 |
| BattleSpeedChanged   | old_speed, new_speed                                   |
| WaveStarted          | wave_index, wave_id                                    |
| WaveCompleted        | wave_index, duration_ticks                             |
| EnemySpawned         | enemy_id, definition_id, path_id                       |
| TowerBuilt           | tower_id, definition_id, position, cost                |
| TowerUpgraded        | tower_id, from_definition_id, to_definition_id, cost   |
| TowerSold            | tower_id, refund                                       |
| TargetingModeChanged | tower_id, old_mode, new_mode                           |
| ProjectileSpawned    | projectile_id, source_id, target_id, definition_id     |
| ProjectileHit        | projectile_id, target_id, position                     |
| EnemyDamaged         | enemy_id, source_id, damage_type, raw, final, hp_after |
| StatusApplied        | target_id, status_id, source_id, stacks, duration      |
| StatusTicked         | target_id, status_id, amount, damage_type              |
| StatusRemoved        | target_id, status_id, reason                           |
| EnemyKilled          | enemy_id, killer_id, reward                            |
| EnemyLeaked          | enemy_id, lives_damage, lives_after                    |
| BossPhaseChanged     | boss_id, phase_index, hp_ratio                         |
| BossKilled           | boss_id, level_id                                      |
| BattleWon            | level_id, lives, currency, duration_ticks              |
| BattleLost           | level_id, reason, wave_index                           |

### 10.1. Event ordering rules

- Для одного tick события публикуются в стабильном порядке по системному pipeline и entity_id.

- EnemyKilled всегда следует после финального EnemyDamaged/StatusTicked, вызвавшего HP ≤ 0.

- Reward начисляется после death validation; UI получает уже итоговое currency state.

- BattleWon публикуется только когда финальная волна исчерпана и нет живых обязательных enemies/boss.

## 11. Command contracts

Command — намерение. Command handler валидирует состояние, может отказать и только затем меняет домен.

| **Command**      | **Payload / validation**                    |
|------------------|---------------------------------------------|
| StartBattle      | level_id, difficulty_id, seed?              |
| PauseBattle      | paused                                      |
| SetBattleSpeed   | speed: 1\|2\|3                              |
| BuildTower       | tower_definition_id, build_slot_id/position |
| UpgradeTower     | tower_id, target_definition_id              |
| SellTower        | tower_id                                    |
| SetTargetingMode | tower_id, mode                              |
| StartWaveEarly   | optional; only if level policy permits      |
| RestartBattle    | same level/difficulty; seed policy explicit |
| ExitBattle       | destination                                 |

### 11.1. CommandResult

```json
{
"accepted": false,
"error_code": "INSUFFICIENT_CURRENCY",
"message_key": "ui.error.insufficient_currency",
"details": {"required": 120, "available": 95}
}
```

UI не должен сам вычислять окончательную валидность покупки; он может предварительно подсветить состояние, но authoritative result возвращает Economy/Command handler.

## 12. Lifecycle одного боя

1. Level Select создаёт StartBattle(level_id, difficulty_id, seed).

2. ConfigRepository валидирует и создаёт immutable ConfigSnapshot.

3. BattleFactory создаёт BattleState, seeded RNG streams, BattleClock, systems и выбранный IPathProvider.

4. Map scene загружается Presentation-слоем; Battle Domain не хранит scene references.

5. Prewarm pools по level content manifest.

6. Battle phase = READY; UI показывает HUD, Pause и x1.

7. WaveSystem запускает первую волну по level policy.

8. Каждый simulation tick: commands → spawn → movement → spatial index → targeting → attack → projectiles → damage → status → death/economy → boss abilities → wave completion → events.

9. Presentation читает snapshots/events и обновляет sprites, health bars, damage numbers, VFX и audio.

10. Игрок может в любой момент вызвать PauseBattle или SetBattleSpeed(1/2/3).

11. При lives ≤ 0 → BattleLost. При завершении финальной волны и отсутствии обязательных enemies → BattleWon.

12. На каждом 5-м уровне победа невозможна до смерти milestone boss финальной волны.

13. BattleResultBuilder формирует результат и telemetry summary.

14. BattleWorld dispose: unsubscribe handlers, return pooled views, release map, assert zero leaked runtime handles.

### 12.1. Fixed-step order

```text
1 Commands
2 Spawn
3 Movement
4 Spatial index refresh
5 Target acquisition
6 Attack cooldown/fire
7 Projectile advance/hits
8 Damage resolution
9 Status ticks/modifiers
10 Death + rewards + leaks
11 Boss phase/ability transitions
12 Wave completion / battle completion
13 Publish ordered event batch
```

## 13. Balance simulator, QA и CI

### 13.1. Headless simulation

Тот же BattleWorld запускается без scenes. Simulator подаёт scripted command stream / bot placement strategy и собирает метрики.

| **Метрика**                       | **Назначение**                                           |
|-----------------------------------|----------------------------------------------------------|
| win_rate                          | Сравнение difficulty/levels при фиксированных стратегиях |
| leaks / lives_remaining           | Реальная опасность волны                                 |
| damage_by_tower                   | Доминирующие/бесполезные башни                           |
| damage_per_100_cost               | Экономическая эффективность                              |
| status_uptime                     | Сила freeze/poison/stun                                  |
| enemy_time_to_exit / time_to_kill | Темп и choke points                                      |
| currency_curve                    | Снежный ком экономики                                    |
| boss_phase_duration               | Проверка boss pacing                                     |
| active_projectiles_peak           | Performance budget                                       |
| determinism_hash                  | Регрессия deterministic outcome                          |

### 13.2. CI pipeline

```text
PR
→ GDScript/static checks
→ JSON schema validation
→ unit tests
→ domain integration tests
→ deterministic replay test
→ balance smoke simulations
→ boss cadence validation
→ headless startup test
→ merge allowed
```

### 13.3. Performance budgets MVP

| **Budget**                              | **Цель**                                                                                    |
|-----------------------------------------|---------------------------------------------------------------------------------------------|
| Active enemies                          | 500 без логических ошибок; performance target тестируется на target devices                 |
| Active pooled projectile/effect handles | 500+ одновременно                                                                           |
| Simulation                              | 30 fixed ticks/s; x3 = до 90 sim ticks/s эквивалента                                        |
| Frame                                   | 60 FPS target на reference desktop; mobile budget фиксируется после выбора reference device |
| Allocations                             | No unbounded per-tick object churn in hot combat loops                                      |
| Pool leaks                              | 0 active handles после dispose battle                                                       |

## 14. Work packages и Definition of Done для 8 агентов

> **Оркестрация —** Каждый work package имеет owner, входные контракты и DoD. Агент не меняет соседний публичный контракт без ADR + review Lead Architect.

### A — Lead Architect

#### Задачи

- Создать BattleWorld/State/Factory, EventBus, CommandBus, ConfigSnapshot, dependency rules.

- Зафиксировать ADR: fixed-step, deterministic RNG, error policy, lifecycle/dispose.

- Сделать contract tests для интерфейсов.

#### Definition of Done

- Battle Domain запускается headless без scenes.

- Запрещённые зависимости проверены code review/static grep.

- Одинаковый seed+commands даёт одинаковый determinism hash 100 прогонов.

- Все публичные интерфейсы документированы и имеют минимум happy-path + error-path tests.

### B — Balance Engineer

#### Задачи

- Создать JSON schemas и validator.

- Формализовать damage/armor/difficulty formulas.

- Создать стартовые tower/enemy/status/difficulty/level definitions.

- Сделать balance simulator metrics.

#### Definition of Done

- Невалидный enum/range/reference блокирует CI.

- DPS derived, не дублируется ручным полем.

- Poison damage + poisoned status + resistance покрыты тестами.

- Milestone boss rule валидируется для level 5/10/... .

- Simulator создаёт reproducible report из seed.

### C — Combat Engineer

#### Задачи

- Movement, Targeting, Attack, Damage, Status, Death, Economy systems.

- Target strategies first/last/strongest/weakest.

- Boss ability integration через общие pipelines.

#### Definition of Done

- Нет UI/scene references в systems.

- Все damage types включая poison проходят единый resolver.

- Статусы корректно stack/expire при x1/x2/x3.

- Build/upgrade/sell authoritative и возвращают CommandResult.

- Unit coverage критических формул и edge cases.

### D — Performance Engineer

#### Задачи

- Spatial index, projectile runtime, pools, debug counters.

- Профилирование 500 enemies / 500+ pooled effects.

- Оптимизация hot loops без изменения domain results.

#### Definition of Done

- Нет full scan 100 towers×500 enemies каждый frame без доказанного профиля.

- Pools prewarm/reuse/reset корректно.

- Battle dispose оставляет 0 pooled active handles.

- Performance benchmark воспроизводим и сохранён в docs/perf.

### E — Navigation Engineer

#### Задачи

- IPathProvider + SplinePathProvider.

- Level path validation.

- Contract stub/tests для future grid provider.

#### Definition of Done

- MVP enemies проходят Path2D curves без scene coupling в model.

- Некорректный path config отклоняется до старта.

- MovementSystem тестируется с fake IPathProvider.

- Grid/flow-field код не реализован сверх seam/stub.

### F — UI/UX Lead

#### Задачи

- Adaptive HUD, tower shop, context menu, damage numbers, health bars.

- Pause + speed x1/x2/x3 всегда доступны.

- Targeting menu: First/Last/Strong/Weak.

- Game Juice по domain events.

#### Definition of Done

- UI отправляет commands, не мутирует models напрямую.

- Pause overlay остаётся интерактивным.

- Состояние скорости визуально однозначно.

- Layout проходит desktop + типовые mobile aspect ratios.

- Отключение VFX не меняет battle result/determinism hash.

### G — QA/DevOps

#### Задачи

- GitHub Actions, schema checks, tests, deterministic replay, smoke sim.

- Regression fixtures, PR template, branch protection.

- Crash/leak/error logs policy.

#### Definition of Done

- main защищена required checks.

- Красный schema/test/sim блокирует merge.

- Есть one-command local test runner.

- Сформирован baseline regression suite для Level 1 и milestone Level 5.

- CI artifact содержит test/balance summary.

### H — Meta/Content Designer

#### Задачи

- Определить progression только для навигации по MVP-уровням.

- Создать level/wave/boss content specs.

- Проверить pacing и milestone boss uniqueness.

#### Definition of Done

- Каждый уровень имеет objective, wave curve и economy budget.

- Каждый 5-й level имеет уникального boss + mechanic brief.

- Контент не требует hardcoded правок core.

- Level 1→5 образуют завершённый тестовый difficulty ramp.

## 15. Общий MVP Definition of Done

- Минимум один полностью проходимый набор классических уровней; рекомендованный baseline — 5 уровней для проверки milestone boss.

- Все башни/враги/волны/уровни/статусы загружаются из JSON; core не знает конкретных ID контента.

- Есть минимум одна poison-башня, применяющая Poison damage и статус poisoned.

- Есть минимум один enemy с положительной poison resistance и один с уязвимостью к poison.

- Freeze, burn, stun, armor_break и poisoned работают через общий StatusSystem.

- Pause работает в любой момент активного боя. x1/x2/x3 работают без рассинхронизации timers/statuses/projectiles.

- На Level 5 финальная волна завершается уникальным boss encounter.

- Targeting: first, last, strongest, weakest переключается через context menu.

- Object pooling покрывает projectiles/VFX/damage numbers.

- Headless simulator использует тот же BattleWorld, что runtime.

- CI валидирует JSON, tests, deterministic replay, milestone boss rule и smoke simulation.

- После завершения/рестарта боя нет подписок, nodes или pooled handles, удерживающих предыдущий BattleWorld.

- Добавление новой обычной башни/врага/статуса не требует изменения существующего core-кода, если используются поддерживаемые primitives.

### 15.1. Acceptance scenarios

| **ID** | **Сценарий**                                | **Ожидание**                                                                            |
|--------|---------------------------------------------|-----------------------------------------------------------------------------------------|
| AC-01  | Пауза во время полёта 100 снарядов          | После resume позиции/таймеры продолжаются без скачка; UI работал на паузе.              |
| AC-02  | Переключение x1→x3 при 5 активных DoT       | Количество logical ticks соответствует BattleClock; итог deterministic.                 |
| AC-03  | Poison tower атакует resistant enemy        | Damage уменьшен resistance; статус применяется по отдельному status-resistance правилу. |
| AC-04  | Level 5 final wave без boss_id              | Config validation FAIL до запуска.                                                      |
| AC-05  | Новый tower JSON с существующими primitives | Появляется в игре без правок Damage/Targeting/Status core.                              |
| AC-06  | VFX layer полностью отключён                | Battle result совпадает с включённым VFX при том же seed/commands.                      |
| AC-07  | Restart battle                              | Новый BattleWorld, старые pools/subscriptions очищены.                                  |
| AC-08  | 500 enemies / 500+ effects stress scene     | Нет функциональных ошибок; performance metrics записаны.                                |

## 16. Риски и архитектурные решения

| **Риск**                                               | **Митигирование**                                                                      |
|--------------------------------------------------------|----------------------------------------------------------------------------------------|
| Слишком ранний “чистый ECS” усложнит Godot-разработку  | Hybrid: domain models/systems + SceneTree presentation.                                |
| Engine.time_scale создаст расхождения runtime/headless | BattleClock fixed-step — authoritative; Engine pause/time only auxiliary.              |
| Hardcoded уникальные boss scripts размоют core         | Ability definitions + shared systems; unique adapter только при необходимости.         |
| Слишком много damage/status enums                      | MVP фиксирует небольшой набор; расширение через schema migration.                      |
| JSON refs ломаются при rename                          | Stable IDs + validator + reference tests; ID не равен display name.                    |
| Poison смешает damage resistance и status immunity     | Два независимых поля: resistances.poison и status_resistance.poisoned.                 |
| x3 перегрузит кадр                                     | Fixed-step accumulator + MAX_STEPS_PER_FRAME + telemetry; UI сохраняет responsiveness. |
| Будущий maze-mode потребует переписать movement        | IPathProvider seam + fake provider tests с первого дня.                                |

### 16.1. ADR backlog до начала production-content

1. ADR-001: Armor mitigation formula.

2. ADR-002: Status stacking precedence и cap rules.

3. ADR-003: Spatial index cell size / update cadence.

4. ADR-004: Projectile hit model: hitscan vs homing vs ballistic primitives.

5. ADR-005: Boss ability primitive catalog.

6. ADR-006: Difficulty presets для MVP (минимум Normal; Easy/Hard optional).

7. ADR-007: Deterministic RNG stream naming.

8. ADR-008: Level 1–5 content budget и reference device performance target.

## Приложение A. Примеры JSON

### A.1. Poison projectile

```json
{
"schema_version": 1,
"id": "toxic_orb",
"movement": {"type": "homing", "speed": 420.0},
"lifetime": 2.5,
"impact": {"radius": 0.0},
"pool": {"prewarm": 64, "max_capacity": 512}
}
```

### A.2. Wave group

```json
{
"enemy_id": "void_raider",
"count": 12,
"spawn_interval": 0.7,
"start_delay": 1.5,
"path_id": "main"
}
```

### A.3. Synergy definition

```json
{
"id": "bio_catalyst",
"requires_nearby_tags": ["bio", "poison"],
"radius": 220.0,
"modifiers": [
{"stat": "poison_duration", "operation": "multiply", "value": 1.15}
]
}
```

### A.4. Event envelope

```json
{
"event_id": 18422,
"battle_id": "b_20260920_001",
"tick": 912,
"type": "EnemyDamaged",
"payload": {
"enemy_id": 403, "source_id": 71,
"damage_type": "poison",
"raw": 10.0, "final": 8.5, "hp_after": 44.0
}
}
```

## Приложение B. Нормативные ссылки Godot

Актуальность проверена на 20.09.2026. Для MVP baseline используется Godot 4.7.2 stable; 4.8 остаётся development branch на эту дату.

| **Источник**                  | **URL**                                                                    |
|-------------------------------|----------------------------------------------------------------------------|
| Godot 4.7.2 stable release    | https://godotengine.org/article/maintenance-release-godot-4-7-2/           |
| Godot release archive         | https://godotengine.org/download/archive/                                  |
| Pausing games / process modes | https://docs.godotengine.org/en/4.7/tutorials/scripting/pausing_games.html |
| Engine.time_scale reference   | https://docs.godotengine.org/en/4.7/classes/class_engine.html              |
| Godot 4.7 documentation root  | https://docs.godotengine.org/en/4.7/                                       |

### Финальное архитектурное правило

> **Gate —** Новый gameplay-код принимается только если его можно протестировать без графики, его параметры вынесены в данные, а public contract не требует знания конкретной башни/врага. Исключения оформляются ADR.
