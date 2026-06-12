# MA: Mission Alone — Godot Project (Prototype)

Project ini adalah implementasi mekanik dasar dari GDD "MA: Mission Alone" menggunakan Godot 4.2+.
Fokus prototype: gerak karakter, sistem stamina/sprint, stealth (crouch), generator objective,
1 Hunter (The Stalker) vs 1 Runner (Maya) melawan AI bot sederhana.

---

## 1. Yang Sudah Tersedia di Project Ini

- `project.godot` — konfigurasi project (portrait, mobile renderer)
- `scripts/GameState.gd` — singleton untuk state global (role, progress generator)
- `scripts/PlayerController.gd` — base controller (gerak, stamina, crouch, sprint)
- `scripts/HunterController.gd` — skill Shadow Step (Stalker)
- `scripts/RunnerController.gd` — skill Quick Fix (Maya)
- `scripts/Generator.gd` — mekanik mengisi generator
- `scripts/BotAI.gd` — AI lawan sederhana (chase / seek generator)
- `scripts/VirtualJoystick.gd` — kontrol joystick virtual untuk mobile
- `scripts/GameUI.gd` — HUD dan tombol aksi
- `scripts/Main.gd` — koordinator scene utama
- `scripts/RoleSelect.gd` — layar pilih role

## 2. Yang PERLU Anda Buat di Editor Godot (Scene/.tscn)

Script di project ini SUDAH SIAP, tapi file `.tscn` (scene) perlu dirakit manual di Godot Editor
karena scene berisi node tree, posisi, dan resource visual yang lebih mudah dibuat lewat editor
drag-and-drop daripada ditulis manual sebagai teks. Berikut langkah-langkahnya:

### A. Install Godot
1. Download Godot 4.2+ (Standard, bukan .NET) dari https://godotengine.org/download
2. Extract dan jalankan executable-nya (tidak perlu instalasi)

### B. Buka Project
1. Jalankan Godot, klik "Import"
2. Arahkan ke folder `godot_project` ini, pilih `project.godot`

### C. Buat Scene `RoleSelect.tscn` (scene utama/start)
1. New Scene → Root node: `Control`
2. Tambahkan child: `VBoxContainer` (nama: VBoxContainer)
   - Child: `Button` (nama: HunterButton, text: "HUNTER - The Stalker")
   - Child: `Button` (nama: RunnerButton, text: "RUNNER - Maya")
3. Attach script `scripts/RoleSelect.gd` ke root Control
4. Simpan sebagai `scenes/RoleSelect.tscn`

### D. Buat Scene `Game.tscn`
1. New Scene → Root node: `Node2D` (nama: Main)
2. Tambahkan child-child berikut:
   - `CharacterBody2D` (nama: Player)
     - Child: `CollisionShape2D` (Circle, radius ~16)
     - Child: `Sprite2D` (gunakan placeholder warna/ColorRect via texture, atau import sprite Anda)
     - Attach script: `scripts/PlayerController.gd` (akan di-override otomatis oleh Main.gd)
     - Tambahkan ke group "player" (Node tab → Groups)
   - `CharacterBody2D` (nama: Bot)
     - Child: `CollisionShape2D` (Circle, radius ~16)
     - Child: `Sprite2D`
     - Attach script: `scripts/BotAI.gd`
   - `Node2D` (nama: ExitGate) — bisa pakai ColorRect hijau sebagai marker visual
   - 5x `Area2D` (nama bebas, masing-masing dengan child `CollisionShape2D` Circle radius ~28 dan `Sprite2D`)
     - Attach script `scripts/Generator.gd` ke masing-masing
     - Sebar posisi mereka di peta (misal: pojok-pojok dan tengah)
   - `CanvasLayer` (nama: GameUI) — lihat bagian E
   - Attach script `scripts/Main.gd` ke root Node2D (Main)

### E. Buat UI di dalam `GameUI` (CanvasLayer)
Struktur node yang dibutuhkan oleh `GameUI.gd`:
```
GameUI (CanvasLayer)
├── HUD (Control)
│   └── TopBar (HBoxContainer)
│       ├── RoleLabel (Label)
│       ├── ObjectiveLabel (Label)
│       ├── StaminaBar (ProgressBar)
│       └── HealthBar (ProgressBar)
├── Controls (Control)
│   ├── VirtualJoystick (Control, attach scripts/VirtualJoystick.gd)
│   │   └── Knob (Control / ColorRect kecil bulat)
│   ├── SkillButton (Button, text: "SKILL")
│   ├── ActionButton (Button, text: "ACTION")
│   └── SprintButton (Button, text: "SPRINT")
└── MessageLabel (Label, posisi tengah, visible=false default)
```
Attach script `scripts/GameUI.gd` ke root `GameUI`.

### F. Setup Input Actions
Project Settings → Input Map, tambahkan action:
- `move_up`, `move_down`, `move_left`, `move_right` (untuk keyboard testing, opsional)
- `crouch` (misal: Shift)
- `sprint` (misal: Space) — juga di-trigger via SprintButton di UI
- `interact` (juga di-trigger via ActionButton)

### G. Set Main Scene
Project Settings → Application → Run → Main Scene → pilih `scenes/RoleSelect.tscn`
(Sudah diset di `project.godot`, tapi pastikan file scene-nya sudah ada)

---

## 3. Export ke APK Android

### A. Install Prasyarat
1. **Java JDK 17** — download dari https://adoptium.net/
2. **Android SDK Command Line Tools**:
   - Download dari https://developer.android.com/studio#command-tools
   - Extract, lalu jalankan `sdkmanager` untuk install:
     ```
     sdkmanager "platform-tools" "platforms;android-34" "build-tools;34.0.0"
     ```
3. **Gradle** (biasanya sudah termasuk dalam Android SDK build tools)
4. Buat **Debug Keystore** (untuk testing, bukan rilis Play Store):
   ```
   keytool -genkey -v -keystore debug.keystore -storepass android -alias androiddebugkey -keypass android -keyalg RSA -keysize 2048 -validity 10000
   ```

### B. Konfigurasi Godot
1. Buka Godot → Editor → Manage Export Templates → Download and Install (sesuai versi Godot Anda)
2. Project → Export Templates → pastikan template Android sudah terpasang
3. Editor → Editor Settings → Export → Android:
   - Isi path ke Android SDK
   - Isi path ke debug keystore (debug.keystore), user: `androiddebugkey`, password: `android`

### C. Export
1. Project → Export...
2. Add → Android
3. Preset settings:
   - Package name: `com.yourname.missionalone`
   - Min SDK: 21 (Android 5.0) — agar kompatibel HP lama
   - Target SDK: 34
   - Orientation: Portrait
4. Klik "Export Project" → pilih lokasi simpan → akan menghasilkan file `.apk`

### D. Install ke HP
1. Aktifkan "Unknown sources" / "Install unknown apps" di Android Settings
2. Transfer file `.apk` ke HP (via kabel USB, atau `adb install nama_file.apk` jika HP terhubung lewat ADB)
3. Buka file APK di HP untuk install

---

## 4. Optimasi untuk HP Kentang (sesuai GDD)

- Gunakan sprite 2D sederhana (bukan 3D) untuk versi awal — jauh lebih ringan
- Tekstur maksimal 512x512px, format kompresi ETC2 (sudah diset di project.godot)
- Hindari banyak `_process()` yang berat; gunakan `_physics_process()` untuk gerak
- Matikan shadow/lighting dinamis untuk versi mobile
- Target build size: cek ukuran APK setelah export, idealnya di bawah 50-80MB untuk prototype 2D

---

## 5. Langkah Selanjutnya

Setelah prototype 2D ini berjalan dan terasa nyaman dimainkan:
- Tambahkan karakter Hunter/Runner lainnya (Warden, Butcher, Whisper / Rio, Dewi, Aldo) sebagai scene terpisah yang extend `PlayerController.gd`
- Tambahkan sound design (detak jantung, langkah kaki)
- Untuk multiplayer online sungguhan, integrasikan plugin networking seperti Godot High-Level Multiplayer API atau Nakama

Jika butuh bantuan lanjutan (misal menulis script karakter tambahan, setup multiplayer dasar,
atau debug error saat export), beri tahu saya.
