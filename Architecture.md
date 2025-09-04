# Architecture

## 1. Three root programs that bootstrap a executable.

```
programs/
 ├─ runtime/   				# Runs game builds (shipping or dev runtime)
 ├─ editor/    				# Runs the editor shell (with runtime loaded too)
 └─ server/    				# Optional dedicated server host.

	Each one has a main function that:
		- Sets up platform layer (window, input, file system).
		- Loads the module manager.
		- Loads the initial modules (core, reflect, etc.).
		- Hands control to a loop (while(running) { module_tick(dt); }).

```

## 2. Engine Root Module

```
	
	There is no engine.dll	
		- modules/core/ is the true root (logging, memory, jobs, assert).
		- other modules depend on core.
		- at startup the host always loads core first.
		
	Engine = the set of modules you load (core + ecs + renderer + …).
	No hardcoded dependency tree — just module.cproj specifying deps.

```
	
## 3. Game Root Module

```

	Each project has a root runtime module in projects/<Game>/source/<GameName>/.
	It’s treated exactly like an engine module — it exports module_init, module_shutdown, module_tick.

	This becomes the main gameplay entry point.

	So Game.dll is just another module, but the host runtime ensures it gets loaded last.
	(so it can bind to all engine modules).
		

```

## 4. Editor Root Module

	modules/editor/ is the root editor shell (dock windows, inspectors).

	* It loads other editor modules (asset_tools, graph_tools, etc.).

	* When you run editor.exe, the main loads core, then editor, then game’s editor module if present.

	* So editor boot sequence looks like:
	
	So editor boot sequence looks like:
	
	main() → load core → load reflect → load platform → load editor → load game’s Editor module.
		

## Boot Sequence Examples

```
	main() [cobalt_runtime]
	↓
	load core (engine/modules/core)
	↓
	load reflect, platform, ecs, renderer, physics, …
	↓
	load Game.dll (projects/MyGame/Source/MyGame)
	↓
	enter main loop → tick modules

```