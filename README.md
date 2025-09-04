# project
A C11 game engine


## High-Level Project Directory Map 

Here is the top-level directory structure. Each component is explained in detail in the following sections.

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
	
	There is no engine.dll	
		- modules/core/ is the true root (logging, memory, jobs, assert).
		- other modules depend on core.
		- at startup the host always loads core first.
		
	Engine = the set of modules you load (core + ecs + renderer + …).
	No hardcoded dependency tree — just module.cproj specifying deps.
	

/project root/				# contains the compiled 
├─── bin/                	# Build output (binaries)
├─── build/                	# Build output (intermediate files)
├─── config/  				# Base engine config (.ini files, defaults).
├─── assets/   				# Raw assets (before converted to engine assets)
├─── content/   			# Engine assets (starter maps, icons, shaders, etc).
├─── docs/                 	# Offline documentation
├─── external/             	# Third-party engine libraries and dependencies
├─── saved/             	# Engine generated files (logs, autosaves, crash dumps).
└─── source/             	# Source for this module (project).

/tools/
├─── reflect_tool/			# Prebuild header tool (scanner + codegen)
├─── asset_tool/			# Asset compiler (importers, processors, packer)
├─── package_tool/			# Prepare release builds with required packaged data.
├─── shader_tool/			# Shader compiler.
└─── reload_tool/			# Fast C compilation for scripts/hot reload.

/tests/
└─── test_category/			# Engine + tool tests (unit/soak/perf)

/runtime/					# Rumtime source (no game/editor coupling)
└─── source/               	// ALL engine, editor, and game source code	
 	 ├─── include/          // The headers for the engine api's.
 	 ├─── runtime/          // Loader.
	 └─── tests/          	// Unit/integration tests.

/modules/					# Standard engine modules communicate through C api.
├─── core/					# Foundation (containers, memory, jobs, math)
├─── reflect/				# Reflection runtime & type registry.
├─── platform/				# OS + platform API wrappers (windowing, threads, fs, time)
├─── math/					# Mostly static includes.
├─── ecs/					# Entity/component storage + systems.
├─── rhi/					# Render hardware interface (backends as submodules)
├─── render/				# Frame graph, passes, materials.
├─── animation/				# Pose system, state machines (runtime only)
├─── audio/					# Audio processing, mixer, spatialization.
├─── physics/				# Rigid body simulation.
├─── assets/				# Asset loaders for various file formats.
├─── net/					# Low-level net, replication hooks
├─── input/					# Devices, mapping, action state
├─── scripting/				# C-script runtime ABI + runtime integratio.
├─── editor/				# Editor shell, panels, asset importers
└─── <feature>/        		# Any optional feature pack (AI, particles, foliage…)

/modules/<Name>				# The module structure
├─── include/<Name>.h 		# Public headers (C API) declared in module config.
├─── source/               	# Implementation
├─── editor/				# (optional) editor-specific code.
├─── assets/				# (optional) raw assets converted into engine content assets.
├─── content/				# (optional) shaders + default assets.
└─── module.cproj/			# configuration (name, dependency, version, flags)

/game/
└─── source/               	# game framework module (re-used game components)

/editor/					# An executable that uses engine as a library.
├─── source/
│	 ├─── editor_main/		# application start. 
│	 ├─── editor_modules/	# editor-only modules (inspector, scene view, editor ui)
├─── ui/					# UI assets/layouts
├─── app/					# Editor shell (docking, projects, cmdlets)
├─── inspectors/			# Property editors, details panels,
├─── graph/					# Node-based editors (materials, anim)
├─── world/					# Level/scene editors, gizmos, nav bake
└─── plugin/				# modular extention for editor

/templates/
	└─── <Example>/			# starter projects to demonstrate engine.

/projects/             		# Game project data (assets, scripts, configs)
     └─── <GameName>/
		  ├─── config/		# set internal variables from configuration.
          ├─── assets/		# unprocessed raw assets.
		  ├─── content/		# processed game engine format assets.		  
          ├─── scripts/		# compiled game script code
		  ├─── editor/		# editor tooling for this game project only.
		  ├─── source/		# compiled game module code		  
		  └<GameName>.cproj	# Project manifest (modules, targets, cook rules)

/package/
	└─── <GameName>/		# The output deliverable game from project -> package.


/plugins/          			# Modular extentions for (engine, editor, game features).
├─── CMakeLists.txt			# 
└─── sample_plugin/			# Each plugin has its own directory

     ├─── game/             // The player-facing Game application


/assetc compile pipeline/	# We sync (raw, engine, and packed assets) for hot reload.
├─── assets/              	# Build output (binaries)
├─── content/ 				# Build output (intermediate files)
└─── package  				# Base engine config (.ini files, defaults).

```

## Detailed Engine Directory Breakdown

```
/bin/

	The executable output path (editor.exe, game.exe) and dynamic library (.dll, .so)
	This has per-platform build outputs.
	
/build/

	The destination for all compiled artifacts. 
	
	# Generated by the build system and can be deleted.
	# It is excluded from version control.
	
/cmake/

	# Toolchains, Find*.cmake, presets.
	# We need an easy way for users to add new modules Cmake can find!
	
/docs/

	# Design docs, ADRs, module guides
	
/extern/

	All third-party libraries so they are self contained.
	
	# We check dependencies directly into our repository.
	# This means no reliance on package managers.
	# Compiles consistently on any machine.
	# in separate root directory to filter our reflections scanning.

	/external/cglm/ 	A highly optimized math library for C.
	/external/glfw/ 	Windowing and input library.
	/external/stb/ 		Single-header public domain libraries (image loading, etc.).
	/external/tcc/ 		The Tiny C Compiler, used for our C scripting backend.

/engine/ 			Contains all the custom code for the engine and its applications.
/engine/core/		Minimal, platform-agnostic, zero dependencys engine module.

	The heart of the engine: lean and dependency-free

	data structures	: Optimized data structures (dynamic arrays, hash tables)
	foundation		: A single header for basic types (u8, f32), macros, and assertions.
	
	reflection_api 	: Reflection capabilities.
	memory_api		: Custom memory allocators (arena, pool).
	file_api		: Abstracted file I/O interfaces.
	module_api		: The module loading and management system (pluggable architecture).	
	job_system_api	:  
	logging_api		:  
	
	* A lean core ensures that the engine's foundation is stable and fast. 
	* Dependency-free, can compile quickly and use it as a stable base for all other systems. 
	* The module system is the lynchpin; it allows the rest of the engine to be built as isolated units.

/source/include/

	* public engine SDK headers (stable API)

/source/modules/	

	Each module directory contains a config file to build the module.
	
	{
		name = "renderer"
		type = "runtime"          # runtime | editor | both
		version = "1.0.0"
		deps = ["rhi", "reflect"]
		public_headers = ["include/renderer.h"]
	}

	* Conains the standard high level engine features.
	* Each subdirectory is a self-contained module that exposes its functionality through a C API.
	* These modules can depend on /core/ but should not, as a rule, depend on each other directly.
	* They communicate through the core systems or well-defined interfaces.	
	* This is the essence of a modular engine. The renderer doesn't know what physics is.
	* The scripting system only knows about functions and data types exposed via reflection.	
	* Development can work independently, developers only include modules they need.
	
	./renderer/		: Handles all graphics. 
	
	* Exposes an API like renderer_api->draw_mesh(...)
	* It can be swapped out (e.g., from Vulkan to DirectX).
	
	./physics/ 		: The physics simulation.	
	./audio/ 		: Sound playback and processing.	
	./ecs/ 			: The entity-component-system for managing game objects.	
	./script/ 		: The C scripting host. 
	
	It loads, compiles (using TCC), and manages the lifecycle of script files. 
	It uses the reflection data to bind engine functions to the script.
			
/source/plugins/

	* Similar to modules, but easily shareable and optional.
	* For third-party additions or features not considered "standard" for the engine.

	./plugin_steam/ 			: Steamworks integration.
	./plugin_procedural_mesh/ 	: A tool for generating procedural geometry.	
	
	* A formal plugins directory encourages a healthy ecosystem. 
	* A game developer can easily drop a new plugin into this folder.
	* The module system in /core/ will recognize and load it without engine source modification.
	
/source/tools/

	* Purpose: Command-line utilities that support the development pipeline.
	* The most important is the reflection tool.
	
	/reflection_tool/ 			
	
	* A C program that parses specially commented C header files (.h) across the engine.
	* Generates .c files containing reflection data.
	* This data includes information about structs, their members, functions, and enums.

	/asset_tool/
	/package_tool/

	Reasoning: This pre-build step is the "magic" that enables the rest of the engine to be so powerful.
	The generated reflection data is used by:

	1. The Editor: To automatically create property editors for any reflected C struct.
	2. The Serializer: To save and load component data to/from disk.
	3. The Scripting System: To expose engine functions and data types to C scripts without manual binding.

/source/editor/

	* The source code for the editor application itself. 
	* The editor is just another application that uses the engine's core and modules.

	main.c			: The entry point for the editor.
	editor_ui.c		: All the UI code (e.g., using a library like Dear ImGui). 
					: This UI is built by inspecting the reflection data.

	gizmos.c		: Code for drawing and interacting with 3D manipulation gizmos in the viewport.

	* The editor is not the engine; it is a consumer of the engine. 
	* This approach forces us to develop clean, public-facing APIs for our modules, 
	* As the editor has to use them just like a game would.
	
/source/game/

	* The source code for playable game runtime -- This is what you ship to players.

	game.c 			: Logic for managing the game state.
	main.c			: The entry point for the game. 
	
	Initializes the core, loads the necessary modules and game scripts, and starts the main game loop.
	
	* We build two primary executables: editor.exe and game.exe. 
	* The game executable is highly optimized and does not include any editor-specific code or modules, resulting in a smaller, faster final product for players.

/projects/
	
	* This directory lives outside the /source/ tree and contains the actual game project data.
	* The engine's code should remain completely separate from the game's data.
	
	./demo_game/assets/		: All game assets—models, textures, sounds.
	./demo_game/scripts/	: The user-written C script files. These are loaded and hot-reloaded by the scripting_system module at .runtime.
	./demo_game/config/		: Project configuration files, input bindings, etc.

	* Decoupling the engine source from project data is paramount. 
	* It means you can update the engine source code without impacting your game data. 
	* It also means multiple game projects can share the same engine installation. 
	* The engine treats project files as pure data to be loaded and manipulated.

```