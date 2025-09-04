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

```
	* modules/editor/ is the root editor shell (dock windows, inspectors).
	* It loads other editor modules (asset_tools, graph_tools, etc.).
	* When you run editor.exe, the main loads core, then editor, then game’s editor module if present.
	* So the editor boot sequence looks like:
	
	main() → load core → load reflect → load platform → load editor → load game’s Editor module.
		
```

## 5. More explanation

``` 

	* You always ship/run the same tiny host program (e.g. cobalt_runtime.exe).
	* It doesn’t know about modules at compile time (my earlier example hard-coded modules for clarity).
	* Instead, it just:
		* Reads a project descriptor (game.toml / manifest.json).
		* Loads all the modules listed there in dependency order

	* Your project doesn’t generate a “native” MyGame.exe — it always launches via the bootstrapper.

```

## Boot Sequence Examples

```
	Runtime (shipping game)

		main() [cobalt_runtime]
		↓
		load core (engine/modules/core)
		↓
		load reflect, platform, ecs, renderer, physics, …
		↓
		load Game.dll (projects/MyGame/Source/MyGame)
		↓
		enter main loop → tick modules

	Editor

		main() [cobalt_editor]
		↓
		load core (engine/modules/core)
		↓
		load reflect, platform, ecs, renderer, …
		↓
		load editor (modules/editor)
		↓
		load game editor module (projects/MyGame/Editor/MyGameEditor)
		↓
		enter editor main loop (panels, inspectors, asset tools)

```

## Minimal Sample code

```

** programs/cobalt_runtime/main.c **

#include "cobalt_platform.h"
#include "cobalt_modules.h"

int main(int argc, char** argv) {
    platform_init(argc, argv);

    module_manager_init();
    module_manager_load("core");
    module_manager_load("reflect");
    module_manager_load("platform");
    module_manager_load("ecs");
    module_manager_load("renderer");
    module_manager_load("physics");

    // load game root
    module_manager_load("MyGame");

    double dt;
    while (platform_pump(&dt)) {
        module_manager_tick(dt);
    }

    module_manager_shutdown();
    platform_shutdown();
    return 0;
}

** projects/MyGame/Source/MyGame/module.c **

	#include "cobalt_api.h"
	
	static const CobaltAPI* g_api;
	
	bool module_init(const ModuleInitParams* params) {
		g_api = params->api;
		g_api->log(1, "MyGame", "Game module initialized");
		return true;
	}
	
	void module_shutdown(void) {
		g_api->log(1, "MyGame", "Game module shutting down");
	}
	
	void module_tick(double dt) {
		// Game loop logic: update world, spawn systems, run ECS
	}

** Summary **

	* main() lives only in host programs (cobalt_runtime, cobalt_editor, etc.).
	* core is the root module of the engine — everything else depends on it.
	* The game root is just a module loaded last.
	* The editor root is just a module loaded when the editor host runs.	
	* The engine is not monolithic — it’s a collection of modules, the host decides which ones to load.

```

## Bootstrapper 

```

	1. Single Bootstrapper Executable (Most Common in Modular Engines)

	* It doesn’t know about modules at compile time (my earlier example hard-coded modules for clarity)	
	* Instead, it just:
		* Reads a project descriptor (game.toml / manifest.json).
		* Loads all the modules listed there in dependency order.
	
	* PRO: No recompilation needed for each project.
	* PRO: Clean separation: engine binary stays the same across all games.
	* PRO: Works well with hot-reloading (swap DLLs while the runtime stays alive).	
	* CON: Your project doesn’t generate a “native” MyGame.exe
	
	Directory: 
	
	programs/
	└─ cobalt_runtime/   				→ single reusable .exe
	
	projects/
	└─ MyGame/
		├─ game.toml     				→ declares entry module + dependencies
		└─ Binaries/Win64/MyGame.dll
	
	game.cproj

	{
		name = "MyGame"
		entry_module = "MyGame"
		modules = ["ecs", "renderer", "physics", "audio"]
	}
		
	int main(int argc, char** argv) 
	{
		platform_init(argc, argv);
		module_manager_init();

		ProjectManifest manifest = load_manifest("game.toml");
		for (module in manifest.modules)
			module_manager_load(module);

		module_manager_load(manifest.entry_module); // Game root

		loop();
	}
	
	* 


```

