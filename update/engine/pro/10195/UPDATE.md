# Change Log | v10195-UFX-S-PRO

## Additions

- Helper `get_max_freq` and `get_min_freq`: reads frequencies from `cpufreq/policy*`, with a fallback to cpu1.
- New variables: `THERMAL_CAP`, `GAME_STATE_FILE`, `NET_MODE_FILE`, `REAPER_LOCK_FILE`.
- `save_game_state` and `restore_game_state`: saves and restores the original states for location, auto time, and auto timezone.
- `stop_adpf_cpu_hint`: stops the ADPF loop and clears `debug.hwui.target_*` props upon exiting a game.
- Periodic thermal re-evaluation (`thermal_tick`, every 4 loops) so the override adapts to the actual temperature during a session.
- Restored animation scale to 1.0 in `daily_feature_daemon`.
- Restored `cmd greezer enable true` and `cmd miui.downscale disable-downscale false` in `cmd_daily_loop`.
- Saved and restored the original `preferred_network_mode` (previously forced to 9 constantly).
- Locked PID for `background_reaper` to prevent overlapping runs.
- Skipped `background_reaper` during High multitask mode in games.
- Cleared `enable_gpu_debug_layers`, `gpu_debug_app`, `gpu_debug_layers`, and `debug.vulkan.layers` in the `game_render` restore branch.

## Fixes

- `safe_thermal_override`: runs `cmd thermalservice reset` before reading the status, ensuring `mStatus` doesn't return its own overriden value.
- `quality_core`: uses `safe_thermal_override 2` (previously applied a raw `override-status 2` without checking temperatures).
- `perf_core`, `balance_core`, `quality_core`: saves caps to `THERMAL_CAP`.
- ADPF (`start_adpf_cpu_hint`):
  - Only executed when entering a game, instead of every loop.
  - Added a zero-division guard (`delta_total`).
  - Lower bound limit for `cpu_percent` is set to 30.
  - `target_gpu_time_percent` uses a fixed value of 56, no longer relying on CPU load.
- `boost_pkg`: `service call activity 51 i32 0` now only runs in Low multitask mode.
- `restore_boost_pkg`: `service call activity 51` now uses `i32 -1` (previously 5).
- Cluster frequencies in all three profiles: now uses `get_min_freq` / `get_max_freq` instead of cpu1 (little cluster).
- `network_adjustment_restore`: restores `preferred_network_mode` to its original value.
- The `--animation-scale` toggle in the game block no longer overwrites the game's selected animation scale (`--animation-scale-v`).
- Location, auto time, and auto timezone are properly restored to their original conditions after a game.

## Removals

- `sfdo force-client-composition enabled` in `cmd_game_loop`: conflicted with `debug.composition.type mdp`.
- `am compact full` in `boost_pkg`: conflicted with `preload_exec` (vmtouch).
- `settings delete global game_driver_all_apps` and `updatable_driver_all_apps` in the `game_driver` default branch: overwrote profile values.
- `cmd display set-match-content-frame-rate-pref 2` in `cmd_game_loop`: overwrote profile values.
- `VK_LAYER_KHRONOS_validation`, `enable_gpu_debug_layers`, `gpu_debug_app`, `gpu_debug_layers` in the Vulkan branch of `game_render`: these were heavy debug layers.
- `cmd activity idle-maintenance` and `cmd blob_store idle-maintenance` in `cmd_game_loop`: were immediately canceled by `sm idle-maint abort`.
- Unconditional `start_adpf_cpu_hint` calls in every `core_engine` loop.
