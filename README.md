# Space Frontier

Space Frontier is a 2D tower defence / space fantasy project built with Godot and GDScript.

## Requirements

- Godot 4.7.2 stable (the MVP baseline defined by the Technical Design).

## Open the project

1. Clone this repository.
2. Start Godot 4.7.2.
3. In the Project Manager, select **Import**, choose the repository's `project.godot`, and open the project.

To open the editor from a terminal when the Godot executable is available on `PATH`:

```text
godot --editor --path .
```

To validate that the project loads without opening a window:

```text
godot --headless --editor --quit --path .
```

This foundation task intentionally provides no main scene or gameplay. Running the game becomes available when a later task adds the first runtime scene.

## AI contributors

Before modifying the project, read in this exact order:

1. `AGENTS.md`
2. `MASTER_AGENT_PROMPT.md`
3. `docs/Space_Frontier_Technical_Design_v0.1.md`
4. `planning/space_frontier_mvp_tasks_v0.1.json`
5. The assigned GitHub Issue
