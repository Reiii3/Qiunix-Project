# Qiunix: Apollo Plugin Update
> **v10120-290926-S | 26-09-2026** | The Qiunix plugin update version 10120 focuses on tweak optimization and the refinement of the Apps and Game Compiler system introduced in this version. It also includes the addition of several features for Qiunix premium users, as well as optimizations for the system daemon and the CLI (`Command Line Interface`) in this plugin.

## What's New
- Added the Game Compiler feature, now available in the Game Manager.
- Refined the Game Compiler function.
- Refined the Compiler System within the Daemon Engine.
- Added DNS Changer and Multitask Mode functions for Daily Mode, which are executed without needing to trigger the main daily mode function.
- Fixed a game force-close bug caused by `game_boost` executing the compiler function while the game was running.
- Fixed the `save_thermal_mode` function, which previously failed to execute, resulting in no changes being applied.
- Added a new tweak: **Telemetry AdService**. This tweak functions to optimize background jobs and Google AdService software. This is not a game optimization tweak, but rather an overall device optimization. (**`PREMIUM USER`**)
- Added ADPF CPU & GPU hints, which are automatically calculated by the daemon based on device conditions. This tweak serves to optimize CPU and GPU usage. (**`PREMIUM USER`**)
- And other minor updates.

> With this update, we hope the plugin system we built can run smoothly and optimally, and we also hope this plugin can fully optimize your device's system. `@Reiieja`