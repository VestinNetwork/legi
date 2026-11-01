# REA — Referensi Lengkap Command

Dokumen ini berisi semua command REA (`rea`, paket npm `rea-agents` v6.8.0) beserta penjelasannya.

- **Sumber**: `rea --help`, `rea --llms`, `rea <command> --help`, dan repo https://github.com/morluto/rea
- **Biner**: `~/.local/bin/rea`
- **Runtime**: Node.js v26.13.0
- **Total command**: 177 (175 command utama + 2 grup integrasi: `mcp` dan `completions`)

---

## Daftar Isi

1. [Ringkasan & Status Setup](#1-ringkasan--status-setup)
2. [Setup & Manajemen](#2-setup--manajemen)
3. [Global Options](#3-global-options)
4. [Analyze & Inspect Umum](#4-analyze--inspect-umum)
5. [Inspect — Format Biner Native](#5-inspect--format-biner-native)
6. [Inspect — Android](#6-inspect--android)
7. [Inspect — Web / JavaScript / Electron](#7-inspect--web--javascript--electron)
8. [Inspect — Jaringan & Packet Capture](#8-inspect--jaringan--packet-capture)
9. [Inspect — Artifact, Firmware & Lain-lain](#9-inspect--artifact-firmware--lain-lain)
10. [Decompile, Decode & Disassembly](#10-decompile-decode--disassembly)
11. [Trace](#11-trace)
12. [Compare](#12-compare)
13. [Capture & Observe](#13-capture--observe)
14. [ADB (Android Debug Bridge)](#14-adb-android-debug-bridge)
15. [Evidence & Environment](#15-evidence--environment)
16. [Debug, Frida & Instrumentasi](#16-debug-frida--instrumentasi)
17. [Project & Application Graph](#17-project--application-graph)
18. [Contoh Alur Kerja](#18-contoh-alur-kerja)

---

## 1. Ringkasan & Status Setup

Perintah `rea setup` dijalankan dengan konfigurasi berikut:

| Item | Nilai |
| --- | --- |
| Client yang dikonfigurasi | **Gemini CLI** (`gemini_cli`) |
| File konfigurasi | `~/.gemini/settings.json` |
| Registrasi MCP | `{"mcpServers":{"rea":{"command":"/home/openhands/.local/bin/rea","args":["mcp"]}}}` |
| Skill terpasang | `~/.agents/skills/reverse-engineer-anything` (versi 49, 229 tool) |
| Hasil | `status: ready`, aksi: `configured_gemini_cli`, `installed_skill` |

Status `rea doctor`:

| Check | Status | Keterangan |
| --- | --- | --- |
| node | ✅ healthy | 26.13.0 |
| host | ✅ healthy | debian 13 |
| hopper | ❌ missing_analysis_engine | Jalankan `rea setup` atau set `HOPPER_LAUNCHER_PATH` |
| ghidra | ❌ missing_analysis_engine | Set `GHIDRA_INSTALL_DIR` |
| ida-registration | ❌ missing_analysis_engine | Set `REA_IDA_MCP_CONFIG` |
| jadx | ❌ missing_analysis_engine | Set `REA_JADX_MCP_JAR` |
| kuna-installation | ❌ missing_analysis_engine | Set `REA_KUNA_COMMAND` |

> **Catatan**: `healthy: false` pada `doctor` bukan berarti instalasi gagal. Yang belum siap adalah **analysis engine** (Hopper/Ghidra/IDA/JADX/Kuna) yang bersifat opsional dan harus dikonfigurasi terpisah. Fitur seperti `analyze`, `decompile`, `function`, dan `xrefs` butuh salah satu engine ini.

---

## 2. Setup & Manajemen

### `rea setup`
Mengonfigurasi integrasi agent dan provider analisis opsional. Menampilkan rencana (plan) perubahan dan meminta persetujuan sebelum menulis file.

**Opsi:**

| Opsi | Penjelasan |
| --- | --- |
| `--yes`, `-y` | Setujui aksi setup tanpa prompt |
| `--install-hopper` | Ikut menyetujui instalasi Hopper saat dipakai bersama `--yes` |
| `--client <array>` | Integrasi agent yang dikonfigurasi; bisa diulang untuk beberapa agent |
| `--skill-client <array>` | Destination skill client yang sudah ada untuk di-update; bisa diulang |
| `--all-detected` | Konfigurasi semua agent terdeteksi |
| `--skill` | Override skill bawaan yang disertakan pada integrasi agent (mis. `--skill=false`) |
| `--dry-run` | Cetak rencana tanpa menerapkannya |
| `--accessible` | Gunakan prompt sekuensial yang aksesibel |

**Contoh:**
```bash
rea setup                                   # interaktif
rea setup --client gemini_cli --yes         # non-interaktif untuk Gemini CLI
rea setup --client codex --client cursor --skill=false --dry-run
rea setup --yes --all-detected --install-hopper --json
```

### `rea doctor`
Memeriksa apakah REA siap dipakai.

**Opsi:** `--target <path>` (path app opsional), `--client <array>`, `--provider <array>`, `--skill`

```bash
rea doctor
rea doctor --json
rea doctor --client gemini_cli --skill
```

### `rea update`
Update instalasi REA dan verifikasi rilis yang terpasang.

```bash
rea update
```

### `rea uninstall`
Menghapus konfigurasi agent dan file skill milik REA.

**Opsi:** `--purge-data` (juga menghapus cache dan state REA)

```bash
rea uninstall
rea uninstall --purge-data
```

### `rea capabilities`
Menampilkan daftar kemampuan provider dan efek sampingnya.

```bash
rea capabilities
```

### `rea providers`
Menampilkan daftar analysis provider yang terkonfigurasi.

```bash
rea providers
```

### `rea mcp`
Registrasi sebagai MCP server.

| Subcommand | Penjelasan |
| --- | --- |
| `rea mcp add` | Registrasi sebagai MCP server |
| `rea mcp doctor` | Validasi startup MCP server dan listing tool |

```bash
rea mcp add
rea mcp doctor
rea --mcp                    # jalankan sebagai MCP stdio server
```

### `rea completions <bash|fish|nushell|zsh>`
Membuat script shell completion.

```bash
eval "$(rea completions bash)"     # tambahkan ke ~/.bashrc
rea completions fish | source      # tambahkan ke ~/.config/fish/config.fish
eval "$(rea completions zsh)"      # tambahkan ke ~/.zshrc
```

---

## 3. Global Options

Opsi ini berlaku untuk semua command:

| Opsi | Penjelasan |
| --- | --- |
| `--filter-output <keys>` | Filter output berdasarkan key path (mis. `foo,bar.baz,a[0,3]`) |
| `--format <toon\|json\|yaml\|md\|jsonl>` | Format output |
| `--full-output` | Tampilkan envelope output lengkap |
| `--help` | Tampilkan bantuan |
| `--llms`, `--llms-full` | Cetak manifest yang mudah dibaca LLM |
| `--mcp` | Mulai sebagai MCP stdio server |
| `--schema` | Tampilkan JSON Schema untuk command |
| `--token-count` | Cetak jumlah token dari output (bukan outputnya) |
| `--token-limit <n>` | Batasi output ke n token |
| `--token-offset <n>` | Lewati n token pertama dari output |
| `--update` | Update ke versi terbaru |
| `--version` | Tampilkan versi |

> Prompt UI dan progres ditulis ke **stderr**, sehingga **stdout** tetap bersih untuk hasil terstruktur/pipeline. Gunakan `NO_COLOR=1` untuk menonaktifkan warna.

---

## 4. Analyze & Inspect Umum

| Command | Penjelasan |
| --- | --- |
| `rea analyze <path>` | Gambaran umum sebuah app |
| `rea inspect <path>` | Inspect overview app beserta evidence |
| `rea inspect-artifact <path>` | Inspect satu artifact dan kembalikan semua observasi serta probe berikutnya |
| `rea inspect-binary-layout <path>` | Inspect layout file/linked binary tanpa mengeksekusi objek |
| `rea inspect-byte-entropy <path>` | Inspect entropi byte per-file dan per-blok tanpa eksekusi |
| `rea inspect-desktop-application <path>` | Identifikasi kandidat runtime desktop dari observasi direktori dan header |
| `rea inspect-native-load-image <path>` | Verifikasi byte native yang dimuat, source mapping, relokasi, dan entry |
| `rea inspect-evidence-bundle <path>` | Inspect metadata discovery lengkap atau terpilih dari Evidence bundle lokal |
| `rea inspect-analysis-view <json>` | Proyeksi view terpilih dari Evidence JSON hasil operasi analisis |
| `rea function <path> [address]` | Analisis satu fungsi lengkap beserta evidence |
| `rea xrefs <path> [address]` | Daftar referensi (bounded) ke sebuah alamat |
| `rea search <path> [pattern]` | Cari di semua string atau nama prosedur yang dianalisis |
| `rea trace <path> [query]` | Trace fitur literal melalui referensi yang dianalisis |
| `rea decompile <path> [address]` | Baca satu bagian app sebagai kode |
| `rea decompile-all <path>` | Tulis pseudocode semua fungsi ke direktori dan kembalikan manifest |
| `rea instructions <path> [address]` | Baca setiap instruksi mentah tanpa decompile |
| `rea read-bytes <path> [address]` | Baca byte memori provider pada satu alamat analisis |
| `rea search-bytes <path>` | Cari pola byte bermasker pada memori provider yang terinisialisasi |
| `rea address-to-file-offset <path> <address>` | Resolve alamat analisis ke offset byte file aslinya |
| `rea identify-flutter-build <path>` | Identifikasi hash Dart snapshot dan fakta engine Flutter murni dari parsing APK |
| `rea extract-artifact <path>` | Ekstrak isi reguler artifact yang didukung (ekstraksi DMG tidak tersedia) |
| `rea extract-firmware <path>` | Ekstrak byte firmware terpilih, kembalikan file dan provenance terverifikasi |
| `rea demangle-swift <path> <symbols...>` | Demangle simbol Swift tanpa Hopper |

---

## 5. Inspect — Format Biner Native

| Command | Penjelasan |
| --- | --- |
| `rea inspect-macho <path>` | Jalankan `inspect_macho` tanpa meluncurkan Hopper |
| `rea inspect-signature <path>` | Jalankan `inspect_signature` tanpa meluncurkan Hopper |
| `rea inspect-plist <path>` | Parse metadata plist app tanpa meluncurkan Hopper |
| `rea list-architectures` | Jalankan `list_architectures` tanpa meluncurkan Hopper |
| `rea inspect-elf-notes <path>` | Inspect identitas ELF `PT_NOTE` dan `SHT_NOTE` tanpa eksekusi |
| `rea inspect-go-binary <path>` | Inspect compiler Go, modul, dan build setting yang tertanam |
| `rea inspect-rust-binary <path>` | Inspect daftar dependensi Rust `cargo-auditable` yang tertanam |
| `rea inspect-pe-imports <path>` | Inspect identitas tabel import dan delay-load PE tanpa eksekusi |
| `rea inspect-pe-resources <path>` | Inspect identitas resource PE, rentang payload, hash, relasi ikon |
| `rea inspect-pe-certificates <path>` | Inspect identitas tabel sertifikat Authenticode PE (tanpa verifikasi tanda tangan) |
| `rea inspect-pe-load-config <path>` | Inspect TLS callback dan load configuration PE tanpa eksekusi |
| `rea inspect-native-exports <path>` | Inspect tabel export image PE atau ELF tanpa eksekusi |
| `rea inspect-managed-artifact <path>` | Inspect identitas PE/CLI tanpa memuat kode target |
| `rea inspect-managed-members <path>` | Inspect member metadata PE/CLI, signature, CIL anchor, literal string `ldstr` |
| `rea inspect-managed-native-boundaries <path>` | Inspect deklarasi boundary PInvoke PE/CLI tanpa memuat kode target |
| `rea inspect-native-instruction <path>` | Inspect satu instruksi native beserta fakta operand terdekode |
| `rea inspect-native-api <path> [address]` | Rekonstruksi satu boundary API fungsi native dengan evidence |
| `rea inspect-native-data-type <path>` | Inspect satu tipe database atau objek data bertipe yang dipulihkan |
| `rea inspect-native-dispatch-metadata <path>` | Inspect metadata dispatch bertipe Objective-C dan Swift |
| `rea inspect-kernel-module <path>` | Inspect metadata kernel module Linux tanpa mengeksekusinya |
| `rea inspect-devicetree <path>` | Inspect flattened devicetree blob tanpa eksekusi |
| `rea inspect-wasm-artifact <path>` | Inspect artifact WASM lokal eksplisit dengan WABT |
| `rea inspect-evm-interface <path>` | Infer kandidat interface dari bytecode EVM lokal eksplisit |
| `rea inspect-gpu-code-objects <path>` | Inspect target CUDA/HIP tertanam dan metadata kernel statis terpilih |
| `rea inspect-model-artifact <path>` | Inspect header GGUF atau safetensors tanpa memuat bobot model |
| `rea inspect-python-pyc <path>` | Inspect header pyc CPython 3.7–3.13 dan code object marshal tanpa eksekusi |
| `rea inspect-erlang-beam <path>` | Inspect atom, export, import, dan header Code Erlang BEAM tanpa eksekusi |
| `rea inspect-il2cpp-metadata <path>` | Inventaris nama, owner, dan literal string IL2CPP `global-metadata.dat` |
| `rea inspect-apple-ips <path>` | Inspect Apple `.ips` crash report tanpa eksekusi target |
| `rea inspect-asset-catalog <path>` | Inspect metadata asset catalog Apple yang dikompilasi |
| `rea inspect-keyed-archive <path>` | Inspect Foundation keyed archive sebagai graph referensi objek |
| `rea inspect-recorded-crash <path>` | Inspect register/signal core yang disediakan dan bukti pemetaan debugger opsional |
| `rea inspect-with-objdump <path>` | Inspect artifact lokal dengan GNU objdump dan simpan output mentah sebagai Evidence |
| `rea list-objdump-architectures` | Daftar target BFD yang diumumkan GNU objdump terpasang |
| `rea match-native-functions` | Cocokkan dua inventaris fungsi native yang teramati tanpa eksekusi target |
| `rea resolve-native-call-targets` | Resolve satu call site native statis |
| `rea trace-native-values <path> <procedure>` | Trace def-use native dan dependensi panggilan (bounded) |
| `rea annotate-native-function <path> <procedure>` | Edit anotasi fungsi dan kembalikan analisis yang di-refresh |
| `rea verify-managed-native-boundaries <json>` | Verifikasi deklarasi P/Invoke managed terhadap Evidence native terautentikasi |

---

## 6. Inspect — Android

| Command | Penjelasan |
| --- | --- |
| `rea inspect-android-package <path>` | Inspect deklarasi APK dan manifest terdekode tanpa eksekusi |
| `rea inspect-android-class <path>` | Inspect inventaris member satu kelas APK spesifik |
| `rea inspect-android-method <path>` | Decompile satu method APK dengan pemilihan overload eksplisit |
| `rea inspect-apk-signature <path>` | Inspect skema signing APK, signer, dan digest konten |
| `rea search-android-classes <path>` | Cari nama kelas APK; query kosong = inventaris semua kelas |
| `rea trace-android-references <path> <className>` | Trace node incoming APK untuk sebuah kelas atau method terpilih |
| `rea decode-android-resources <path>` | Dekode manifest, metadata, dan string resource satu APK dengan apktool |
| `rea inspect-apktool-client` | Inspect launcher apktool yang dipilih caller tanpa mendekode apa pun |
| `rea decompile-jeb-item <item-address> <item-kind>` | Decompile satu tipe atau method JEB ke pseudo-code engine |
| `rea inspect-jeb-client` | Inspect client JEB MCP yang berjalan tanpa membuka target |
| `rea list-jeb-units` | Daftar unit project JEB berdasarkan path dan tipe dengan filter opsional |
| `rea open-jeb-project` | Buka atau buat project JEB dari artifact atau database `.jdb2` |
| `rea identify-flutter-build <path>` | Identifikasi hash Dart snapshot dan fakta engine Flutter dari parsing APK |

---

## 7. Inspect — Web / JavaScript / Electron

| Command | Penjelasan |
| --- | --- |
| `rea inspect-web-page <endpoint>` | Inspect satu halaman secara pasif via CDP |
| `rea inspect-web-network-capture <path>` | Inspect HAR, mitmproxy native, Fiddler SAZ, atau qlog aioquic tanpa fetch URL tercatat |
| `rea inspect-web-event-listeners` | Inspect source location listener native pada match CSS dokumen utama pertama |
| `rea inspect-electron-fuses <path>` | Inspect byte fuse Electron yang dipaketkan tanpa meluncurkan aplikasi |
| `rea inspect-electron-page <path>` | Inspect satu file page Electron secara pasif |
| `rea list-electron-targets` | Daftar file page lokal yang diekspos endpoint CDP Electron terpilih |
| `rea list-browser-targets` | Daftar target page dari browser CDP loopback terpilih (filter origin opsional) |
| `rea list-javascript-runtime-targets` | Daftar target V8 Inspector Node/Electron |
| `rea observe-javascript-runtime` | Observe satu target Node/Electron Inspector spesifik secara pasif |
| `rea observe-web-execution` | Observe eksekusi page secara presisi; reset counter, nonaktifkan optimasi sementara |
| `rea observe-web-session` | Observe navigasi page yang dijalankan pengguna selama jendela waktu terpilih |
| `rea analyze-web-bundle <endpoint> <targetId>` | Capture dan analisis bundle page secara statis |
| `rea analyze-javascript-application <path>` | Rekonstruksi statis aplikasi JavaScript/Electron lokal |
| `rea export-web-scripts <path>` | Ekspor script website yang tersimpan ke direktori kosong untuk analisis statis |
| `rea recover-javascript-sources <path>` | Pulihkan modul JavaScript yang terbaca beserta provenance dan file terverifikasi |
| `rea discover-webmcp-tools <endpoint> <targetId>` | Discover tool WebMCP yang dideklarasikan page secara pasif |
| `rea reconcile-javascript-runtime <json>` | Rekonsiliasi Evidence aplikasi statis dan runtime pasif |
| `rea compare-javascript-export-shapes <json>` | Bandingkan bentuk return export JavaScript statis eksak tanpa eksekusi |
| `rea trace-javascript-semantics <json>` | Trace relasi semantik statis bounded melalui Evidence aplikasi terautentikasi |
| `rea trace-web-module-imports <manifestPath> <scriptIndex>` | Trace import native satu script yang ditangkap ke URL dan kandidat source |
| `rea trace-web-source-location <manifestPath> <scriptIndex> <sourceMapPath> <sourceMapUrl> <line> <column>` | Trace satu posisi script melalui source map lokal yang dipilih eksplisit |

---

## 8. Inspect — Jaringan & Packet Capture

| Command | Penjelasan |
| --- | --- |
| `rea inspect-packet-capture <path>` | Inspect packet capture yang ada dengan TShark yang disediakan caller |
| `rea inspect-packet-fields <path>` | Cari definisi protokol dan field TShark yang terpasang |
| `rea inspect-pcap-capture <path>` | Inspect satu capture pcap/pcapng: endpoint, DNS, TLS SNI, distribusi protokol |
| `rea inspect-pcap-client` | Inspect biner tshark dan capinfos yang dipilih caller tanpa membaca capture |
| `rea inspect-signal-decoders <path>` | Cari decoder sigrok terpasang atau bantuan decoder terpilih |
| `rea decode-signal-recording <path>` | Dekode signal recording yang ada dengan sigrok yang disediakan caller |

---

## 9. Inspect — Artifact, Firmware & Lain-lain

| Command | Penjelasan |
| --- | --- |
| `rea inspect-sqlite-database <path>` | Inspect schema SQLite dan baris tabel biasa (opsional) dari snapshot lokal |
| `rea inspect-firmware-regions <path>` | Inspect kandidat signature firmware tanpa mengekstraknya |
| `rea decode-interface-builder <path>` | Dekode storyboard dan nib archive yang dikompilasi dari app bundle |
| `rea trace-dylib-resolution <path>` | Trace resolusi load-path dyld untuk Mach-O atau tiap executable dalam app bundle |
| `rea compare-managed-members <leftPath> <rightPath>` | Bandingkan inventaris member PE/CLI managed berdasarkan tipe, nama, dan signature deklarasi eksak |

---

## 10. Decompile, Decode & Disassembly

| Command | Penjelasan |
| --- | --- |
| `rea decompile <path> [address]` | Baca satu bagian app sebagai kode. Butuh minimal satu dari address posisional atau `--procedure`; bisa juga `--file-offset` saja |
| `rea decompile-all <path>` | Tulis pseudocode setiap fungsi ke direktori dan kembalikan manifest |
| `rea instructions <path> [address]` | Baca setiap instruksi mentah tanpa decompile |
| `rea function <path> [address]` | Analisis satu fungsi lengkap dengan evidence |
| `rea decompile-jeb-item <item-address> <item-kind>` | Decompile satu tipe atau method JEB ke pseudo-code engine |
| `rea decode-android-resources <path>` | Dekode manifest, metadata, dan string resource APK dengan apktool |
| `rea decode-interface-builder <path>` | Dekode storyboard dan nib archive yang dikompilasi |
| `rea decode-signal-recording <path>` | Dekode signal recording dengan sigrok caller |
| `rea demangle-swift <path> <symbols...>` | Demangle simbol Swift tanpa Hopper |
| `rea execute-rizin-command <path>` | Eksekusi satu command Rizin terhadap artifact lokal |
| `rea cutter-command <sessionId> <expectedGeneration> <command>` | Eksekusi satu command Rizin pada instance Cutter live terpilih |
| `rea list-cutter-sessions` | Daftar instance Cutter live yang terhubung via bridge plugin Python REA |

---

## 11. Trace

| Command | Penjelasan |
| --- | --- |
| `rea trace <path> [query]` | Trace fitur literal melalui referensi yang dianalisis |
| `rea trace-android-references <path> <className>` | Trace node incoming APK untuk kelas atau method terpilih |
| `rea trace-application-feature <json>` | Trace seed bertipe melalui Evidence aplikasi terautentikasi |
| `rea trace-dylib-resolution <path>` | Trace resolusi load-path dyld untuk Mach-O atau tiap executable dalam bundle |
| `rea trace-javascript-semantics <json>` | Trace relasi semantik statis bounded melalui Evidence aplikasi |
| `rea trace-native-ui-action <path> <action>` | Trace selector aksi UI terkompilasi ke handler native dan callee langsung |
| `rea trace-native-values <path> <procedure>` | Trace def-use native dan dependensi panggilan (bounded) |
| `rea trace-web-module-imports <manifestPath> <scriptIndex>` | Trace import native satu script ke URL dan kandidat source |
| `rea trace-web-source-location <manifestPath> <scriptIndex> <sourceMapPath> <sourceMapUrl> <line> <column>` | Trace satu posisi script melalui source map lokal |

---

## 12. Compare

| Command | Penjelasan |
| --- | --- |
| `rea compare <left> <right>` | Bandingkan dua canonical Evidence bundle |
| `rea compare-application-versions <json>` | Bandingkan dua versi JavaScript Application Graph terautentikasi |
| `rea compare-javascript-export-shapes <json>` | Bandingkan bentuk return export JavaScript statis eksak tanpa eksekusi |
| `rea compare-managed-members <leftPath> <rightPath>` | Bandingkan inventaris member PE/CLI managed |
| `rea compare-process-captures <left> <right> [traceSpec]` | Bandingkan dua file Evidence process capture JSON |
| `rea compare-source-to-bundle <json>` | Bandingkan source historis dengan Evidence aplikasi terautentikasi |
| `rea compare-wasm-interfaces <input>` | Bandingkan interface WASM core dari dua record Evidence inspection eksak |
| `rea compare-web-captures <beforeJson> <afterJson>` | Bandingkan dua capture pasif ternormalisasi atau scenario browser terekam |
| `rea compare-web-screenshots <beforeJson> <afterJson>` | Bandingkan dua nilai JSON artifact PNG self-verifying |

---

## 13. Capture & Observe

| Command | Penjelasan |
| --- | --- |
| `rea capture-adb-screen <serial>` | Capture satu frame layar sebagai PNG yang di-digest |
| `rea capture-browser-scenario <json>` | Jalankan satu scenario Playwright bounded dan kembalikan Evidence |
| `rea capture-electron-scenario <json>` | Jalankan satu scenario Electron milik sendiri dengan aksi yang bisa dibatalkan |
| `rea capture-native-ui-scenario <path>` | Jalankan aksi AX deklaratif atau wait pada satu jendela aplikasi native spesifik |
| `rea capture-process <scenario>` | Capture satu scenario proses yang dipilih caller |
| `rea capture-web-screenshot <endpoint> <targetId>` | Capture satu viewport page terlihat dalam scope origin yang diberikan |
| `rea collect-adb-bugreport <serial>` | Kumpulkan dan digest satu arsip bugreport buatan device |
| `rea observe-javascript-runtime` | Observe satu target Node/Electron Inspector secara pasif |
| `rea observe-native-calls` | Luncurkan Mach-O di bawah LLDB dan rekam entry ke fungsi atau method Objective-C terpilih |
| `rea observe-native-ui <path>` | Observe satu jendela aplikasi native yang sudah berjalan via PID dan window ID |
| `rea observe-web-execution` | Observe eksekusi page presisi |
| `rea observe-web-session` | Observe navigasi page yang dijalankan pengguna |

---

## 14. ADB (Android Debug Bridge)

### Perangkat & Inspeksi

| Command | Penjelasan |
| --- | --- |
| `rea list-adb-devices` | Daftar device terhubung dengan serial, state, transport, dan jenis yang diinfer |
| `rea inspect-adb-client` | Inspect client adb yang dipilih caller tanpa menghubungi device |
| `rea inspect-adb-device <serial>` | Inspect identitas build satu device via whitelist `getprop` tetap |
| `rea inspect-adb-display <serial>` | Laporkan ukuran dan densitas display yang dideklarasikan |
| `rea inspect-adb-package <serial> <package>` | Inspect versi, installer, dan flag satu package terpasang via `dumpsys` |
| `rea inspect-adb-window <serial>` | Laporkan deklarasi fokus window manager saat ini |
| `rea list-adb-features <serial>` | Daftar fitur hardware dan software yang dideklarasikan device |
| `rea list-adb-packages <serial>` | Daftar package terpasang pada satu device beserta base APK path |
| `rea list-adb-processes <serial>` | Daftar proses berjalan persis seperti `ps -A` melaporkannya |
| `rea list-adb-services <serial>` | Daftar binder service dan interface device |
| `rea list-adb-directory <serial> <path>` | Daftar satu direktori device dengan jenis, permission, dan ukuran |
| `rea read-adb-logcat <serial>` | Baca dump log device yang bounded untuk satu buffer |
| `rea read-adb-setting <serial>` | Baca satu nilai setting dari `system`, `secure`, atau `global` |
| `rea resolve-adb-packages <serial>` | Temukan package terpasang berdasarkan substring nama (case-insensitive) |

### Transfer & Operasi File

| Command | Penjelasan |
| --- | --- |
| `rea pull-adb-file <serial>` | Pull satu file device ke salinan lokal dengan digest SHA-256 |
| `rea pull-adb-package <serial>` | Pull set APK lengkap satu package terpasang dengan digest SHA-256 |
| `rea push-adb-file <serial>` | Push satu file lokal ke device; menolak target yang sudah ada tanpa `--overwrite` |

### Manajemen Aplikasi

| Command | Penjelasan |
| --- | --- |
| `rea install-adb-package <serial>` | Install satu APK lokal; `--replace` minta mode reinstall untuk mempertahankan data |
| `rea uninstall-adb-package <serial> <package>` | Uninstall satu package beserta datanya dari device |
| `rea start-adb-app <serial> <package>` | Mulai satu app terpasang dengan resolve dan launch launcher activity-nya |
| `rea start-adb-activity <serial> <action>` | Mulai intent dengan `am start` (action, URI, component, extras) |
| `rea stop-adb-app <serial> <package>` | Force-stop setiap proses dari satu package |

---

## 15. Evidence & Environment

| Command | Penjelasan |
| --- | --- |
| `rea evidence-export <path>` | Validasi dan ekspor canonical Evidence JSON secara atomik |
| `rea evidence-import <path>` | Validasi dan import Evidence bundle lokal yang bounded |
| `rea environment-build <path>` | Bangun bundle environment agent dari capture terekam dalam satu langkah offline |
| `rea environment-compare <path>` | Bandingkan environment dengan probe capture dari aplikasi asalnya |
| `rea environment-export <path>` | Ekspor manifest environment dan Evidence capture sebagai folder bundle agent environment |
| `rea environment-serve <path>` | Sajikan manifest environment sebagai agent environment via streamable HTTP sampai SIGINT/SIGTERM |
| `rea evaluate-reconstruction-coverage <json>` | Evaluasi coverage rekonstruksi fail-closed inline |
| `rea build-reconstruction-obligation-ledger <inputJson>` | Hasilkan halaman ledger obligation rekonstruksi yang deterministik dan didukung Evidence |
| `rea import-managed-reconstruction <json>` | Import rekonstruksi managed hasil decompiler terhadap Evidence member statis eksak |
| `rea import-reference-source <path>` | Import source tree sebagai referensi historis saja |
| `rea reconcile-javascript-runtime <json>` | Rekonsiliasi Evidence aplikasi statis dan runtime pasif |

---

## 16. Debug, Frida & Instrumentasi

| Command | Penjelasan |
| --- | --- |
| `rea debug-with-gdb <command>` | Jalankan satu command console GDB unrestricted di sesi MI ephemeral |
| `rea debug-with-rizin <path> <command>` | Jalankan satu command debugger Rizin unrestricted di sesi ephemeral |
| `rea instrument-with-frida <path>` | Jalankan satu script Frida terhadap proses terpilih lalu detach |
| `rea list-frida-devices` | Daftar device Frida lokal dan endpoint remote opsional |
| `rea list-frida-processes <path>` | Daftar proses yang terlihat melalui device Frida |
| `rea execute-rizin-command <path>` | Eksekusi satu command Rizin terhadap artifact lokal |
| `rea cutter-command <sessionId> <expectedGeneration> <command>` | Eksekusi satu command Rizin pada instance Cutter live |
| `rea list-cutter-sessions` | Daftar instance Cutter live via bridge plugin Python |
| `rea observe-native-calls` | Luncurkan Mach-O di bawah LLDB dan rekam entry fungsi |

---

## 17. Project & Application Graph

| Command | Penjelasan |
| --- | --- |
| `rea project-android-application-graph <json>` | Proyeksikan Evidence inventaris APK terautentikasi ke graph aplikasi Android |
| `rea project-apple-application-graph <json>` | Proyeksikan Evidence inventaris IPA atau macOS app ke graph aplikasi Apple |
| `rea project-harmony-application-graph <json>` | Proyeksikan Evidence inventaris HarmonyOS HAP/HSP/App Pack ke graph aplikasi |
| `rea project-managed-application-graph <json>` | Proyeksikan Evidence statis managed terautentikasi ke graph aplikasi |

---

## 18. Contoh Alur Kerja

### Setup awal
```bash
# 1. Cek kesiapan
rea doctor

# 2. Konfigurasi agent (mis. Gemini CLI)
rea setup --client gemini_cli --yes

# 3. Verifikasi
rea doctor --client gemini_cli --skill
```

### Analisis binary native
```bash
rea analyze /path/to/binary            # gambaran umum
rea inspect-binary-layout /path/to/binary
rea function /path/to/binary --procedure main
rea xrefs /path/to/binary --address 0x401000
rea decompile /path/to/binary --procedure main
```

### Analisis APK
```bash
rea inspect-android-package app.apk
rea decode-android-resources app.apk
rea search-android-classes app.apk "com.example"
rea inspect-android-class app.apk
```

### Analisis web / Electron
```bash
rea analyze-javascript-application /path/to/app.asar
rea inspect-electron-fuses /path/to/app
rea list-browser-targets
rea observe-web-execution <endpoint>
```

### Operasi ADB
```bash
rea list-adb-devices
rea list-adb-packages <serial>
rea capture-adb-screen <serial>
rea pull-adb-package <serial> <package>
rea start-adb-app <serial> <package>
```

---

## Lampiran — Daftar Cepat Semua Command (Alfabetis)

```
address-to-file-offset                  analyze
analyze-javascript-application          analyze-web-bundle
annotate-native-function                build-reconstruction-obligation-ledger
capabilities                            capture-adb-screen
capture-browser-scenario                capture-electron-scenario
capture-native-ui-scenario              capture-process
capture-web-screenshot                  collect-adb-bugreport
compare                                 compare-application-versions
compare-javascript-export-shapes        compare-managed-members
compare-process-captures                compare-source-to-bundle
compare-wasm-interfaces                 compare-web-captures
compare-web-screenshots                 completions
cutter-command                          debug-with-gdb
debug-with-rizin                        decode-android-resources
decode-interface-builder                decode-signal-recording
decompile                               decompile-all
decompile-jeb-item                      demangle-swift
discover-webmcp-tools                   doctor
environment-build                       environment-compare
environment-export                      environment-serve
evaluate-reconstruction-coverage        evidence-export
evidence-import                         execute-rizin-command
export-web-scripts                      extract-artifact
extract-firmware                        function
identify-flutter-build                  import-managed-reconstruction
import-reference-source                 inspect
inspect-adb-client                      inspect-adb-device
inspect-adb-display                     inspect-adb-package
inspect-adb-window                      inspect-analysis-view
inspect-android-class                   inspect-android-method
inspect-android-package                 inspect-apk-signature
inspect-apktool-client                  inspect-apple-ips
inspect-artifact                        inspect-asset-catalog
inspect-binary-layout                   inspect-byte-entropy
inspect-dart-aot                        inspect-desktop-application
inspect-devicetree                      inspect-electron-fuses
inspect-electron-page                   inspect-elf-notes
inspect-erlang-beam                     inspect-evidence-bundle
inspect-evm-interface                   inspect-firmware-regions
inspect-go-binary                       inspect-gpu-code-objects
inspect-il2cpp-metadata                 inspect-jeb-client
inspect-kernel-module                   inspect-keyed-archive
inspect-macho                           inspect-managed-artifact
inspect-managed-members                 inspect-managed-native-boundaries
inspect-model-artifact                  inspect-native-api
inspect-native-data-type                inspect-native-dispatch-metadata
inspect-native-exports                  inspect-native-instruction
inspect-native-load-image               inspect-packet-capture
inspect-packet-fields                   inspect-pcap-capture
inspect-pcap-client                     inspect-pe-certificates
inspect-pe-imports                      inspect-pe-load-config
inspect-pe-resources                    inspect-plist
inspect-python-pyc                      inspect-recorded-crash
inspect-rust-binary                     inspect-signal-decoders
inspect-signature                       inspect-sqlite-database
inspect-wasm-artifact                   inspect-web-event-listeners
inspect-web-network-capture             inspect-web-page
inspect-with-objdump                    install-adb-package
instructions                            instrument-with-frida
list-adb-devices                        list-adb-directory
list-adb-features                       list-adb-packages
list-adb-processes                      list-adb-services
list-architectures                      list-browser-targets
list-cutter-sessions                    list-electron-targets
list-frida-devices                      list-frida-processes
list-javascript-runtime-targets         list-jeb-units
list-objdump-architectures              match-native-functions
mcp                                     observe-javascript-runtime
observe-native-calls                    observe-native-ui
observe-web-execution                   observe-web-session
open-jeb-project                        project-android-application-graph
project-apple-application-graph         project-harmony-application-graph
project-managed-application-graph       providers
pull-adb-file                           pull-adb-package
push-adb-file                           read-adb-logcat
read-adb-setting                        read-bytes
reconcile-javascript-runtime            recover-javascript-sources
resolve-adb-packages                    resolve-native-call-targets
search                                  search-android-classes
search-bytes                            setup
start-adb-activity                      start-adb-app
stop-adb-app                            trace
trace-android-references                trace-application-feature
trace-dylib-resolution                  trace-javascript-semantics
trace-native-ui-action                  trace-native-values
trace-web-module-imports                trace-web-source-location
uninstall                               uninstall-adb-package
update                                  verify-managed-native-boundaries
xrefs
```
