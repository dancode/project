# architecture

## 1. Three root programs that bootstrap a executable.

programs/
 ├─ runtime/   				# Runs game builds (shipping or dev runtime)
 ├─ editor/    				# Runs the editor shell (with runtime loaded too)
 └─ server/    				# Optional dedicated server host.

	Each one has a main function that:
		- Sets up platform layer (window, input, file system).
		- Loads the module manager.
		- Loads the initial modules (core, reflect, etc.).
		- Hands control to a loop (while(running) { module_tick(dt); }).
	
## 2. Engine Root Module
	
	There is no engine.dll	
		- modules/core/ is the true root (logging, memory, jobs, assert).
		- other modules depend on core.
		- at startup the host always loads core first.
		
	Engine = the set of modules you load (core + ecs + renderer + …).
	No hardcoded dependency tree — just module.cproj specifying deps.
	
## 3. Game Root Module

## 4. Editor Root Module