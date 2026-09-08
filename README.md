# this readme was made by claude opus 5 to prevent fucking graves time to organize , if the strings n shit is broken dm grave on discord

# 🛡️ Hunt Scanner - Complete Reverse Engineering & Forensic Documentation

This repository contains the full static analysis, extracted embedded artifacts, decrypted network endpoints, memory inspection rules, and categorized client detection signatures for **`Hunt Scanner.exe`** (also known as *Hunt Service* / *Hunt SS Tool*).

---

## 📁 Repository Directory Structure

```
c:\Users\User\source\repos\hunt scanner\
│
├── 📄 Hunt Scanner.exe               # Target binary analyzed (NOT executed)
├── 📄 README.md                      # Master Forensic Documentation (This file)
│
├── 📂 scanner files/                 # Extracted Embedded Templates & Payloads
│   ├── 📄 detection-results.html     # Standalone HTML report template (38.2 KB) with Anti-DevTools
│   ├── 📄 payload_started.json       # Startup telemetry JSON payload sent to Webhook #1
│   ├── 📄 payload_results.json       # Scan completion JSON payload sent to Webhook #2
│   ├── 📄 webhooks.json              # Structured JSON summary of decrypted Webhook URLs
│   └── 📄 delete_me.bat              # Standalone report self-deletion batch script
│
└── 📂 scanner modules/               # Categorized Memory Detection Modules & Signatures
    ├── 📄 combat.txt                 # 54 Combat, Aim, Crystal PvP & Auto-Totem signatures
    ├── 📄 movement_and_utility.txt    # 12 Movement, Render, X-Ray & Automation signatures
    ├── 📄 cheat_clients.txt          # 35 Detected hack client brands & mod suites
    ├── 📄 anti_forensics_and_cleaners.txt # 9 USN journal wipers, RAM cleaners & Anti-SS tools
    ├── 📄 obfuscation_and_bypasses.txt    # 12 JNIC obfuscator, Mixin & Replace bypass signatures
    ├── 📄 all_extracted_strings.txt       # Raw dump of all 4,868 extracted binary strings
    │
    └── 📂 clients/                   # Itemized Per-Client Forensic Analysis Files (21 Files)
        ├── 📄 all_client_signatures.txt
        ├── 📄 meteor_client.txt
        ├── 📄 thunderhack_client.txt
        ├── 📄 argon_client.txt
        ├── 📄 krypton_client.txt
        ├── 📄 grim_client.txt
        ├── 📄 prestige_client.txt
        ├── 📄 skliggahack_client.txt
        ├── 📄 nova_client.txt
        ├── 📄 scrim_client.txt
        ├── 📄 wurst_client.txt
        ├── 📄 catlean_client.txt
        ├── 📄 coffee_client.txt
        ├── 📄 doomsday_client.txt
        ├── 📄 francium_client.txt
        ├── 📄 ghost_bleach_client.txt
        ├── 📄 lumina_client.txt
        ├── 📄 novoware_client.txt
        ├── 📄 shoreline_client.txt
        ├── 📄 surge_wing_client.txt
        └── 📄 donutsmp_bypass.txt
```

---

## 🌐 Network Destinations & Decrypted Webhook Endpoints

During execution, `Hunt Scanner.exe` communicates over HTTPS via Windows `WINHTTP.dll`. The Webhook URLs were decrypted from the `.rdata` section using XOR key `0x004B`:

| Destination / Purpose | Decrypted Endpoint URL | Trigger Event & Action |
| :--- | :--- | :--- |
| **Authentication & Telemetry** | `https://discord.com/api/webhooks/1445499289781407874/kOHJ0nnkcbyqM3hLv4J4P6h0KkmZvYW4fbo8TACi8M3FPp78wz4gM26hehmcaPXpVSBS` | Triggered on launch/login. Sends PC name, Username, RAM, license expiration, and Discord code. |
| **Scan Results & HTML Report Exfiltrator** | `https://discord.com/api/webhooks/1397242448052490381/_Let2o5puqLbdTGIdfAy_8dylM_7nJxwjKL8Z9NR6Kgt3G09GjpICB7XbMXP-JNfn9rE` | Triggered on scan completion. Sends RED/YELLOW/ORANGE counts and uploads `detection-results.html`. |
| **Support & License Portal** | `https://dsc.gg/huntservice` | Discord invite link for support tickets and user licensing. |
| **CDN Assets** | `https://i.imgur.com/l4taeNs.png` & `https://fonts.googleapis.com` | Discord webhook avatar logo and HTML report web fonts. |

---

## 🔬 Low-Level Memory Inspection Mechanics (`ScanThread`)

When scanning a target process (e.g. `java.exe` / `javaw.exe`), `ScanThread` executes the following Win32 assembly routine:

```
[Target Process Selection: java.exe]
         │
         ▼
[OpenProcess(PROCESS_VM_READ | PROCESS_QUERY_INFORMATION)]
         │
         ▼
[VirtualQueryEx Loop over Address Space]
         │ ──► Filter: MEM_COMMIT (Allocated Memory Pages)
         │ ──► Filter: PAGE_READWRITE (Java Heap) & PAGE_EXECUTE_READWRITE (Native Code)
         ▼
[ReadProcessMemory (Sliding Window 8192-byte Buffers)]
         │
         ▼
[String & Byte Signature Matcher]
         │ ──► Match Found: Flag Process, Increment RED/YELLOW Counter
         │ ──► Log to detection-results.html & Send Discord Payload
```

### Targeted JVM Memory Structures:
1. **Java Class Constant Pool (`CONSTANT_Utf8_info`):** Searches RAM for loaded class strings, package names (e.g. `net/meteorclient`, `net/wurstclient`, `com/thunder/client`), and method annotations.
2. **SpongePowered Mixin Transformers:** Detects unauthorized class bytecode transformers (`org/spongepowered/asm/mixin/transformer/MixinTransformer`) modifying vanilla Minecraft classes.
3. **Memory String Replacement Bypasses:** Ghost clients attempt to overwrite cheat strings in RAM with benign mod names (e.g. replacing `Krypton` with `OptiFine`). The scanner audits RAM for trailing pointers (`Replace bypass Found (Krypton client)`).
4. **JNI Native Method Bridges:** Inspects process module tables (PEB LDR data) for unlinked C++ `.dll` handles and `PAGE_EXECUTE_READWRITE` code stubs.
5. **Anti-Forensics Checks:**
   * **USN Journal Wiping:** Queries NTFS USN Journal (`FSCTL_QUERY_USN_JOURNAL`). If cleared, flags `Delete USN Journal module Found`.
   * **RAM String Cleaners:** Scans `java.exe` heap for active zeroing loops (`String Cleaner Found`).
   * **Anti-SS Overlay Tools:** Checks window flags (`WS_EX_LAYERED`) and GDI/DirectX hooks (`Anti SS Tool Found`).

---

## 📋 Comprehensive Category Overview

### 1. Cheat Clients (`scanner modules/cheat_clients.txt` & `scanner modules/clients/`)
* **Anarchy & Utility:** Meteor Client, Thunderhack, Wurst Client, Shoreline, Syracruse Client, Asteria, Doomsday, Francium.
* **Ghost & PvP:** Argon, Catlean, Coffee Client, Gardenia, Ghost Bleach, Grandline, Grim (and Grim Cracked), Krypton, Lattia, Lumina, Minced, Nova Client (V1, Class, Replace), NovoWare, Prestige Client, Scrim Client, Skliggahack, St-Api, Surge/Wing, TipTap, Wazo Client, Xyla.

### 2. Combat & PvP Modules (`scanner modules/combat.txt`)
* **Auto-Crystal & Anchor:** Auto Hit Crystal, Auto Crystal, CrystalAura, Cw Crystal, Anchor Placer, Anchor Macro, AutoAnchor, ClickCrystal.
* **Auto-Totem & Armor:** Auto Inventory Totem, Auto Pot Refill, Auto Armor, Auto Totem, AutoDoubleHand, Legit Totem.
* **Aim & Weapon Delays:** Aimassist, Click Aimassist, Sticky Aim, TriggerBot, Axe Delay, Sword Delay, Switch Delay, Equip Delay, No Miss Delay, Only Crit Axe/Sword, Disable Shields.

### 3. Anti-Forensics & Bypasses (`scanner modules/anti_forensics_and_cleaners.txt` & `obfuscation_and_bypasses.txt`)
* **Anti-Forensics:** Delete USN Journal, String Cleaner, Self Destruct, Anti SS Tool, HWID Auth Bypass.
* **Obfuscation & Injections:** JNIC Obfuscator, Base64 Obfuscation, SpongePowered Mixin Transformer, Prestige Injector, Donut SMP Bypass.
