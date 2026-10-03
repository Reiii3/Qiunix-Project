# Change Log | v10195-UFX-S-PRO

## Penambahan

- Helper `get_max_freq` dan `get_min_freq`: membaca frekuensi dari `cpufreq/policy*`, fallback ke cpu1.
- Variabel baru: `THERMAL_CAP`, `GAME_STATE_FILE`, `NET_MODE_FILE`, `REAPER_LOCK_FILE`.
- `save_game_state` dan `restore_game_state`: menyimpan dan mengembalikan kondisi asli lokasi, auto time, dan auto timezone.
- `stop_adpf_cpu_hint`: menghentikan loop ADPF dan membersihkan prop `debug.hwui.target_*` saat keluar game.
- Re-evaluasi thermal berkala (`thermal_tick`, tiap 4 loop) supaya override mengikuti suhu aktual selama sesi.
- Restore animation scale ke 1.0 di `daily_feature_daemon`.
- Restore `cmd greezer enable true` dan `cmd miui.downscale disable-downscale false` di `cmd_daily_loop`.
- Simpan dan kembalikan `preferred_network_mode` asli (sebelumnya dipaksa 9 terus).
- Lock PID untuk `background_reaper` agar run tidak numpuk.
- Skip `background_reaper` saat multitask game di mode High.
- Pembersihan `enable_gpu_debug_layers`, `gpu_debug_app`, `gpu_debug_layers`, dan `debug.vulkan.layers` di branch restore `game_render`.

## Fix

- `safe_thermal_override`: `cmd thermalservice reset` dulu sebelum membaca status, supaya `mStatus` bukan nilai override sendiri.
- `quality_core`: memakai `safe_thermal_override 2` (sebelumnya `override-status 2` mentah tanpa cek suhu).
- `perf_core`, `balance_core`, `quality_core`: menyimpan cap ke `THERMAL_CAP`.
- ADPF (`start_adpf_cpu_hint`):
  - Hanya dijalankan saat masuk game, bukan tiap loop.
  - Guard pembagian nol (`delta_total`).
  - Batas bawah `cpu_percent` 30.
  - `target_gpu_time_percent` memakai nilai tetap 56, tidak lagi dari beban CPU.
- `boost_pkg`: `service call activity 51 i32 0` hanya berjalan di multitask Low.
- `restore_boost_pkg`: `service call activity 51` memakai `i32 -1` (sebelumnya `5`).
- Cluster freq di tiga profil: memakai `get_min_freq` / `get_max_freq`, tidak lagi cpu1 (little).
- `network_adjustment_restore`: mengembalikan `preferred_network_mode` ke nilai asli.
- Toggle `--animation-scale` di blok game tidak lagi menimpa animation scale pilihan game (`--animation-scale-v`).
- Lokasi, auto time, dan auto timezone dikembalikan sesuai kondisi asli setelah game.

## Hapus

- `sfdo force-client-composition enabled` di `cmd_game_loop`: bentrok dengan `debug.composition.type mdp`.
- `am compact full` di `boost_pkg`: melawan `preload_exec` (vmtouch).
- `settings delete global game_driver_all_apps` dan `updatable_driver_all_apps` di default branch `game_driver`: menimpa nilai dari profil.
- `cmd display set-match-content-frame-rate-pref 2` di `cmd_game_loop`: menimpa nilai dari profil.
- `VK_LAYER_KHRONOS_validation`, `enable_gpu_debug_layers`, `gpu_debug_app`, `gpu_debug_layers` di branch Vulkan `game_render`: layer debug yang berat.
- `cmd activity idle-maintenance` dan `cmd blob_store idle-maintenance` di `cmd_game_loop`: langsung dibatalkan oleh `sm idle-maint abort`.
- Pemanggilan `start_adpf_cpu_hint` tanpa syarat di setiap loop `core_engine`.
