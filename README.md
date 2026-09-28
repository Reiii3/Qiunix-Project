# Qiunix: Apollo — Plugin Documentation

## Overview

**Qiunix: Apollo** adalah plugin Android yang menyediakan seperangkat tools untuk membantu pengelolaan dan optimasi pengalaman bermain game.

Plugin ini menggunakan beberapa komponen native yang bekerja sebagai satu sistem:

- `qiunix` — **CLI / command interface**
- `qiunixD` — **Daemon**
- `scr` — **Feature Extension / extension**
- `game.txt` — daftar game untuk perangkat/versi Android tertentu
- `service.sh` — proses inisialisasi setelah boot
- `system.prop` — konfigurasi properti sistem
- `webroot/` — antarmuka WebUI dan bridge JavaScript

> Catatan: nama file binary pada paket instalasi menggunakan format `QiunixC_*`, `QiunixD_*`, dan `QiunixF_*`. Saat instalasi, file tersebut dipilih berdasarkan arsitektur perangkat dan diubah menjadi nama runtime yang digunakan plugin.

---

## Component Architecture

```text
Qiunix: Apollo
│
├── qiunix
│   └── CLI / Command Interface
│       └── menerima dan menjalankan perintah Qiunix
│
├── qiunixD
│   └── Daemon
│       └── menjalankan proses/background service Qiunix
│
├── scr
│   └── Feature Extension
│       └── komponen tambahan untuk fitur/extension Qiunix
│
├── service.sh
│   └── boot initialization
│       ├── menunggu Android selesai boot
│       ├── menyiapkan daftar game
│       ├── membersihkan cache tertentu
│       ├── menangani reaktivasi daemon
│       └── membuka WebUI
│
├── system.prop
│   └── system/rendering properties
│
└── webroot/
    └── WebUI + JavaScript bridge
```

---

# Binary Components

## 1. QiunixC — CLI

### Runtime name

```text
qiunix
```

### Source binary

```text
QiunixC_arm32
QiunixC_arm64
```

`QiunixC` merupakan komponen **CLI (Command-Line Interface)** utama dari Qiunix.

Setelah instalasi, binary yang sesuai arsitektur perangkat akan dipindahkan dan diberi nama:

```text
/system/bin/qiunix
```

CLI digunakan sebagai titik akses utama untuk mengontrol berbagai operasi Qiunix melalui command line.

Contoh pola penggunaan yang terlihat pada plugin:

```sh
qiunix --auto-reactivated-daemon --get
```

dan:

```sh
qiunix --daemon --restart
```

Komponen lain seperti `service.sh` menggunakan CLI ini untuk berkomunikasi dengan daemon dan menjalankan operasi Qiunix.

### Tanggung jawab utama

- Menjadi command interface Qiunix.
- Menerima command dan argument dari shell.
- Mengontrol operasi daemon.
- Menyediakan akses command yang digunakan oleh service dan sistem Qiunix.
- Menjadi entry point untuk operasi yang membutuhkan kontrol langsung terhadap engine.

---

# 2. QiunixD — Daemon

### Runtime name

```text
qiunixD
```

### Source binary

```text
QiunixD_arm32
QiunixD_arm64
```

`QiunixD` merupakan **daemon** Qiunix.

Daemon adalah proses yang berjalan di background dan menangani operasi Qiunix secara terus-menerus atau berdasarkan kebutuhan sistem.

Setelah instalasi, binary daemon yang sesuai arsitektur perangkat menjadi:

```text
/system/bin/qiunixD
```

Daemon dapat dikontrol melalui CLI:

```sh
qiunix --daemon --restart
```

dan dihentikan ketika plugin di-uninstall:

```sh
qiunix --daemon --stop
```

### Tanggung jawab utama

- Menjalankan proses background Qiunix.
- Mempertahankan fungsi Qiunix yang membutuhkan proses daemon.
- Menerima kontrol dari CLI `qiunix`.
- Mendukung restart/re-activation daemon.
- Menjalankan operasi yang tidak harus dilakukan langsung oleh CLI.

### Hubungan CLI dan daemon

Secara sederhana:

```text
User / WebUI / Service
          │
          ▼
       qiunix
          │
          │ command
          ▼
       qiunixD
          │
          ▼
   Background Engine
```

`qiunix` berperan sebagai pengendali/command interface, sedangkan `qiunixD` menjalankan proses daemon di background.

---

# 3. QiunixF — Feature Extension

### Runtime name

```text
scr
```

### Source binary

```text
QiunixF_arm32
QiunixF_arm64
```

`QiunixF` merupakan **extension component** untuk fitur Qiunix.

Pada saat instalasi, binary ini dipilih berdasarkan ABI perangkat dan dipindahkan menjadi:

```text
/system/bin/scr
```

Komponen ini dipisahkan dari CLI dan daemon sehingga fungsi extension dapat digunakan sebagai komponen tambahan dari sistem Qiunix.

### Tanggung jawab utama

- Menyediakan extension untuk fitur Qiunix.
- Menjadi komponen tambahan di luar CLI utama dan daemon.
- Menyediakan fungsi native yang dibutuhkan oleh fitur tertentu.
- Memungkinkan arsitektur Qiunix memisahkan fitur tambahan dari core CLI dan daemon.

Struktur konseptualnya:

```text
qiunix
  │
  ├── Core CLI
  │
  ├── qiunixD
  │     └── Daemon / Background
  │
  └── scr
        └── Feature Extension
```

---

# Architecture Support

Qiunix menyediakan dua binary untuk setiap komponen native:

```text
ARM64
├── QiunixC_arm64
├── QiunixD_arm64
└── QiunixF_arm64

ARM32
├── QiunixC_arm32
├── QiunixD_arm32
└── QiunixF_arm32
```

Installer membaca ABI perangkat:

```sh
getprop ro.product.cpu.abi
```

Jika perangkat menggunakan:

```text
arm64-v8a
```

maka versi ARM64 digunakan.

Jika perangkat menggunakan:

```text
armeabi-v7a
```

maka versi ARM32 digunakan.

Binary yang tidak sesuai arsitektur dihapus untuk menjaga instalasi tetap bersih.

---

# Boot Service

File:

```text
service.sh
```

bertanggung jawab terhadap proses setelah Android selesai boot.

### Alur utama

```text
Android Boot
     │
     ▼
Wait boot completed
     │
     ▼
Wait storage available
     │
     ▼
Prepare game list
     │
     ▼
Clean Qiunix cache
     │
     ▼
Check daemon state
     │
     ├── Auto reactivation enabled
     │        │
     │        ▼
     │     Restart daemon
     │
     ▼
Open Qiunix WebUI
```

Service juga membuat/menyiapkan:

```text
/data/local/tmp/game.txt
```

untuk daftar game yang digunakan oleh sistem Qiunix.

Pada Android 12 atau lebih baru, daftar game diperoleh melalui:

```sh
dumpsys game
```

Sedangkan pada Android versi lebih lama, daftar tersebut difilter berdasarkan isi:

```text
game.txt
```

---

# Game Database

File:

```text
game.txt
```

berisi daftar package game.

Contohnya:

```text
com.activision.callofduty.shooter
com.epicgames.fortnite
com.miHoYo.GenshinImpact
com.mobile.legends
```

Daftar ini digunakan sebagai sumber package game terutama pada jalur kompatibilitas Android versi lama.

Pada Android 12+, `service.sh` mencoba mengambil daftar game secara langsung dari Android melalui `dumpsys game`.

---

# System Properties

File:

```text
system.prop
```

berisi property sistem yang berkaitan dengan rendering, SurfaceFlinger, HWUI, dan pengelolaan UI/RAM.

Contoh kategori konfigurasi yang terdapat di dalamnya:

- SurfaceFlinger
- HWUI
- Render Engine
- UI scheduling
- purgeable assets

Contoh property:

```text
debug.sf.multithreaded_present=1
debug.sf.no_vsyncs_on_screen_off=1
debug.hwui.use_partial_updates=true
debug.renderengine.graphite=true
sys.use_fifo_ui=1
persist.sys.purgeable_assets=1
```

Property ini menjadi bagian dari konfigurasi optimasi sistem yang dibawa oleh plugin.

---

# WebUI

Qiunix menyediakan `webroot/` untuk integrasi WebUI.

Strukturnya:

```text
webroot/
├── index.html
└── js/
    └── kernel.js
```

`index.html` mengaktifkan fullscreen dan mengarahkan UI ke:

```text
https://qiunixnewgen.pages.dev/
```

Sementara `kernel.js` menyediakan JavaScript bridge untuk berkomunikasi dengan KernelSU.

Bridge tersebut menyediakan fungsi seperti:

```text
exec()
spawn()
fullScreen()
enableEdgeToEdge()
toast()
moduleInfo()
listPackages()
getPackagesInfo()
exit()
```

Dengan demikian, WebUI dapat menjadi antarmuka untuk berinteraksi dengan fungsi native/shell melalui bridge yang tersedia.

---

# Installation Flow

Secara umum proses instalasi berjalan seperti berikut:

```text
Qiunix ZIP
   │
   ▼
Magisk / KernelSU Module Installer
   │
   ▼
customize.sh
   │
   ├── Detect ABI
   │
   ├── Select ARM32 / ARM64
   │
   ├── Remove unused binaries
   │
   ├── Rename QiunixC → qiunix
   │
   ├── Rename QiunixD → qiunixD
   │
   └── Rename QiunixF → scr
   │
   ▼
Set executable permissions
   │
   ▼
Module ready
```

---

# Uninstallation

File:

```text
uninstall.sh
```

digunakan untuk menghentikan daemon sebelum module dihapus.

Perintah yang digunakan:

```sh
qiunix --daemon --stop
```

Tujuannya adalah memastikan proses daemon Qiunix dihentikan saat plugin di-uninstall.

---

# Component Summary

| Component | Source Binary | Runtime Name | Role |
|---|---|---|---|
| QiunixC | `QiunixC_arm32/arm64` | `qiunix` | CLI / command interface |
| QiunixD | `QiunixD_arm32/arm64` | `qiunixD` | Background daemon |
| QiunixF | `QiunixF_arm32/arm64` | `scr` | Feature extension |
| `service.sh` | Shell script | — | Boot/service initialization |
| `system.prop` | System properties | — | Rendering/system configuration |
| `game.txt` | Package list | — | Game package database |
| `webroot/` | Web files | — | WebUI and KernelSU bridge |

---

# Important Naming Note

Nama `QiunixC`, `QiunixD`, dan `QiunixF` adalah nama binary **di dalam package/module**.

Setelah proses instalasi:

```text
QiunixC → qiunix
QiunixD → qiunixD
QiunixF → scr
```

Jadi ketika melakukan debugging pada perangkat yang sudah terpasang, nama yang biasanya ditemukan adalah:

```text
/system/bin/qiunix
/system/bin/qiunixD
/system/bin/scr
```

Sedangkan nama `QiunixC_*`, `QiunixD_*`, dan `QiunixF_*` digunakan sebagai binary source untuk pemilihan arsitektur ketika proses instalasi berlangsung.

---

## Version Information

Current package:

```text
Name    : Qiunix: Apollo — BASIC
Version : 10100-060926-B
Build   : 2026-09-06
Engine  : Universal Quick Xtorm
Author  : @ReiiEja
```

## Disclaimer

Dokumentasi ini menjelaskan fungsi dan hubungan komponen berdasarkan struktur package, script installer/service, konfigurasi, serta penggunaan command yang terlihat pada module.

Fungsi internal spesifik dari binary native (`QiunixC`, `QiunixD`, dan `QiunixF`) tidak dijabarkan lebih jauh apabila implementasi internalnya tidak tersedia sebagai source code di dalam package.
