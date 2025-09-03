# project
A C11 game engine


** High-Level Project Directory Map ** 

Here is the top-level directory structure. Each component is explained in detail in the following sections.

/engine/
├─── build/                	// Build output (binaries, intermediate files)
├─── docs/                 	// Engine documentation
├─── external/             	// Third-party libraries and dependencies
├─── source/               	// ALL engine, editor, and game source code
│    ├─── core/             // The heart of the engine: lean and dependency-free
│    ├─── modules/          // Standard engine modules (renderer, physics, etc.)
│    ├─── plugins/          // Optional, shareable plugins
│    ├─── tools/            // Engine-related tools (reflection generator, etc.)
│    ├─── editor/           // The Game Editor application
│    └─── game/             // The player-facing Game application
└─── projects/             	// Game project data (assets, scripts, configs)
     └─── demo_game/
          ├─── assets/		// unprocessed raw assets.
		  ├─── content/		// processed game engine format assets.
          ├─── scripts/		// compiled game script code
          └─── config/		// 
		  └─── source/		// compiled game module code

