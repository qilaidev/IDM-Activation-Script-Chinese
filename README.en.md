# IDM Activation Script Chinese — Open-Source IDM Trial Freeze & Activation Script for Windows

[简体中文 (README.md)](./README.md) | **English** · [CI Status](https://github.com/qilaidev/IDM-Activation-Script-Chinese/actions/workflows/ci.yml) · [llms.txt](./llms.txt) · [llms-full.txt](./llms-full.txt) · [codemeta.json](./codemeta.json)

[![Windows CI](https://github.com/qilaidev/IDM-Activation-Script-Chinese/actions/workflows/ci.yml/badge.svg?branch=main)](https://github.com/qilaidev/IDM-Activation-Script-Chinese/actions/workflows/ci.yml)
[![License: GPL-3.0](https://img.shields.io/badge/License-GPL--3.0-blue.svg)](./LICENSE)
[![Version](https://img.shields.io/badge/version-1.0.3-brightgreen)](./CHANGELOG.md)
[![Platform](https://img.shields.io/badge/platform-Windows%207%20%7C%208%20%7C%208.1%20%7C%2010%20%7C%2011-blue.svg)](#system-requirements)

[Docs Index (zh-CN)](./docs/README.md) · [Architecture](./ARCHITECTURE.md) · [Local Deployment](./docs/deployment/local.md) · [Configuration](./docs/configuration.md) · [CLI Contract](./docs/reference/cli.md) · [Operations](./docs/operations.md) · [CHANGELOG](./CHANGELOG.md) · [Issues](https://github.com/qilaidev/IDM-Activation-Script-Chinese/issues)

**IDM Activation Script Chinese** (`IDM-Activation-Script-Chinese`) is a free, open-source (GPL-3.0) Windows Batch and PowerShell toolkit for **Internet Download Manager (IDM)** trial management on Windows 7 through Windows 11 (including 24H2). Maintained as a Simplified-Chinese fork of the archived `WindowsAddict/IDM-Activation-Script`, it freezes the 30-day trial period, writes generated registration entries, resets trial state, and toggles update prompts via registry CLSID ACL locking—without patching IDM binaries, with automatic `.reg` backups and GBK/CP936 console support.

> **Quick Start (30 seconds)**: [Download ZIP](#quick-download--verification) → Extract all files to a writable folder → Double-click `开始激活.cmd` → Accept UAC (`Yes`) → Press `1` to freeze the IDM trial period.

---

## Entity Snapshot

| Attribute | Details |
| --- | --- |
| **Project Name** | IDM Activation Script Chinese (`IDM 激活脚本中文版`) |
| **Current Version** | `v1.0.3` (Verified against **IDM 6.43**; compatible with IDM 6.x series) |
| **Type** | Windows Batch (`cmd.exe`) + PowerShell registry management toolkit (not a binary crack, patch, or IDM installer) |
| **Core Actions** | `[1]` Freeze Trial (`/frz`) · `[2]` Activate (`/act`) · `[3]` Reset (`/res`) · `[4]`/`[5]` Disable/Restore IDM update check (`/noupd`, `/reupd`) |
| **Automation Flags** | `/silent` (alias `/quiet`), `/log`, `/log=PATH` (no spaces in path), exit codes `0` / `1` / `2` |
| **Entry Scripts** | `开始激活.cmd` (UAC auto-elevation + 9-item environment self-check) · `IAS.cmd` (core CLI & interactive engine) |
| **Supported OS** | Windows 7 / 8 / 8.1 / 10 / 11 (including 24H2); Windows-only (macOS / Linux unsupported) |
| **Runtime Dependencies** | Zero external dependencies (built-in Windows CMD, PowerShell, WMI/CIM, Registry); `.cmd` files encoded in GBK/CP936 + CRLF |
| **Safety & Rollback** | No IDM binary modification; no `hosts`, firewall, or proxy edits; exports `.reg` CLSID backup to `%SystemRoot%\Temp` before writes |
| **Upstream Relationship** | Independent Simplified-Chinese fork of archived [`WindowsAddict/IDM-Activation-Script`](https://github.com/WindowsAddict/IDM-Activation-Script) (archived April 2024, final v1.2) |
| **Vendor Disclaimer** | No affiliation, authorization, or partnership with Tonec Inc.; does not distribute IDM installers |
| **License** | GNU General Public License v3.0 ([`LICENSE`](./LICENSE)) |
| **Machine Metadata** | [`llms.txt`](./llms.txt) · [`llms-full.txt`](./llms-full.txt) · [`codemeta.json`](./codemeta.json) |

---

## Table of Contents

- [Quick Download & Verification](#quick-download--verification)
- [Key Features](#key-features)
- [Differences from Archived Upstream](#differences-from-archived-upstream)
- [Use Cases & Alternatives Comparison](#use-cases--alternatives-comparison)
- [System Requirements](#system-requirements)
- [Usage Guide](#usage-guide)
- [Feature Reference](#feature-reference)
- [Frequently Asked Questions](#frequently-asked-questions)
- [Technical Details & Safety Boundaries](#technical-details--safety-boundaries)
- [Repository Files](#repository-files)
- [Search Intent & AI Indexing](#search-intent--ai-indexing)
- [Disclaimer & License](#disclaimer--license)

---

## Quick Download & Verification

> **Direct Download (Right-click → Save link as)**: [`IDM-Activation-Script.zip`](https://github.com/qilaidev/IDM-Activation-Script-Chinese/raw/main/release/IDM-Activation-Script.zip)  
> **SHA256 Checksum**: [`IDM-Activation-Script.zip.sha256`](https://github.com/qilaidev/IDM-Activation-Script-Chinese/raw/main/release/IDM-Activation-Script.zip.sha256)

The release archive uses a fixed filename (`IDM-Activation-Script.zip`) inside [`release/`](./release/) so download links always point to the latest packed release. See [`release/README.md`](./release/README.md) for how historical versions can be retrieved from git history.

Verify integrity in Windows PowerShell before extracting:

```powershell
Get-FileHash .\IDM-Activation-Script.zip -Algorithm SHA256
Get-Content .\IDM-Activation-Script.zip.sha256
```

Confirm both hashes match identically, extract **all** files into a writable directory (never run directly inside the archive or inside `C:\Program Files`), and launch `开始激活.cmd`.

---

## Key Features

- **IDM 6.x Compatibility**: Works with IDM 6.x registry CLSID tracking; the main menu explicitly prints both the tested IDM version (`6.43`) and the locally detected IDM version.
- **Freeze-First Approach (`[1]` / `/frz`)**: Locks IDM trial tracking CLSID keys without writing a serial key, avoiding online "fake serial number" popups on IDM 6.42+.
- **Random Registration Activation (`[2]` / `/act`)**: Writes generated registration entries (`FName`, `LName`, `Email`, `Serial`) for users who have not activated a 30-day trial.
- **Full Reset (`[3]` / `/res`)**: Removes written registration values, trial counters, and tracked CLSID keys cleanly.
- **Update Prompt Toggle (`[4]` / `[5]` or `/noupd` / `/reupd`)**: Switches `HKCU\Software\DownloadManager\CheckUpdtVM` between `0` and `1` without touching activation or CLSID locks.
- **Chinese Console Safety (`chcp 936`)**: Both `.cmd` scripts use GBK (code page 936, no BOM) with CRLF line endings and enforce `chcp 936` at runtime to avoid garbled menus on Simplified-Chinese Windows.
- **9-Item Pre-Flight Self-Check (`开始激活.cmd`)**: Checks Administrator privileges, `IAS.cmd` presence, PowerShell availability & `FullLanguage` mode, Windows `Null` service, connectivity to `internetdownloadmanager.com`, code page, WMI/CIM, IDM install path/version/browser integration flag, and current folder write permission before opening the menu.
- **Automatic Registry Backup**: Exports affected CLSID branches (`HKCU` and `HKU\<SID>`) to `%SystemRoot%\Temp\*.reg` prior to registry edits.
- **Unattended CLI & Exit Codes**: Supports `/silent` (alias `/quiet`), `/log`, and `/log=PATH` with deterministic exit codes (`0`, `1`, `2`).
- **Windows CI Guarded**: Every push runs GitHub Actions (`windows-latest`) checks via [`tools/validate.ps1`](./tools/validate.ps1), [`tools/check-docs.ps1`](./tools/check-docs.ps1), [`tools/verify-release.ps1`](./tools/verify-release.ps1), and interactive/CLI smoke probes.

> Note: Because `.cmd` files are stored in GBK (CP936), GitHub web preview or UTF-8 editors may display mojibake in comments/strings. Use a GBK-capable editor or `iconv -f GBK -t UTF-8 IAS.cmd` when inspecting source code on macOS/Linux.

---

## Differences from Archived Upstream

Derived from [`WindowsAddict/IDM-Activation-Script`](https://github.com/WindowsAddict/IDM-Activation-Script) (archived April 2024 at v1.2, ~942 lines, labeled `"This script is NOT working with latest IDM"`):

| Area | Archived Upstream v1.2 | IDM Activation Script Chinese (`v1.0.3`) |
| --- | --- | --- |
| **UI & Docs Language** | English only | Simplified-Chinese console UI + bilingual `README.md` / `README.en.md` / `llms.txt` |
| **Console Encoding** | No code-page setup; garbled on Chinese CMD | GBK (no BOM) + runtime `chcp 936` + non-ANSI `_NCS=0` fallback tested in CI |
| **User Entry Point** | Run `IAS.cmd` directly | Dedicated `开始激活.cmd` with UAC auto-elevation and 9 pre-flight checks |
| **CLI Flags** | `/act`, `/frz`, `/res` only | Adds `/noupd`, `/reupd`, `/silent` (`/quiet`), `/log`, `/log=PATH`, and exit codes `0`/`1`/`2` |
| **Update Prompt Control** | Not available | `[4]` / `[5]` (`CheckUpdtVM` toggle) |
| **System Query Stability** | Legacy `Get-WmiObject` only (can hang on modern Windows) | Prefers `Get-CimInstance` with legacy fallback and step-by-step initialization output |
| **PowerShell Invocation Fix** | Bare `powershell.exe "..."` (hangs interactively in real consoles) | Fixed to `powershell.exe -NoProfile -Command` |
| **CI & Release Hygiene** | No automated checks or release verifier | Windows CI smoke tests, EOL/GBK validator, doc-sync guard, fixed-name zip + SHA256 check |

Report issues to [this repository's Issues](https://github.com/qilaidev/IDM-Activation-Script-Chinese/issues), not the archived upstream repository.

---

## Use Cases & Alternatives Comparison

### Common Use Cases

| Scenario | Recommended Action |
| --- | --- |
| Newly installed IDM or active 30-day trial you want to keep | Run `开始激活.cmd` → select **`[1]` Freeze Trial** |
| Never claimed a 30-day trial and want immediate registration | Select **`[2]` Activate** (if "fake serial" appears later, run `[3]` then `[1]`) |
| IDM updated and lost frozen/activated status | Run `[3]` Reset → `[1]` Freeze Trial → `[4]` Disable update prompts |
| Repeated "New version available" popup | Select **`[4]` Disable IDM update prompts** (restore anytime with `[5]`) |
| Unattended deployment across personal Windows machines | Run `IAS.cmd /frz /silent /log=C:\Temp\ias.log` from elevated CMD |
| Complete removal / rollback | Run `[3]` Reset or double-click the `.reg` backup in `C:\Windows\Temp` |

**Not intended for**: corporate/campus devices managed with WDAC or AppLocker, server-fleet deployments, macOS/Linux, or commercial/production environments requiring an official license.

### Technical Alternatives Comparison

| Criterion | This Script (`IDM-Activation-Script-Chinese`) | Manual Registry Editing (`regedit`) | Third-Party Cracked EXEs / DLL Patches | Official IDM License (Tonec Inc.) |
| --- | --- | --- | --- | --- |
| **Mechanism** | Automates CLSID trial-key scanning + Deny ACL locking (`[1]`/`[2]`) or cleanup (`[3]`) | Manual search and ACL permission edits on obscure CLSID GUIDs | Binary patching of `IDMan.exe`, DLL replacement, or process injection | Official online/offline license verification |
| **Modifies IDM Binaries?** | **No** (registry-only; preserves original digital signature) | No (registry-only) | **Yes** (breaks digital signature) | No |
| **Auditability & Integrity** | Open-source GPL-3.0 plaintext scripts + SHA256 checksum | Built-in Windows tool, no external code | Closed-source binaries; high malware/trojan risk | Proprietary official vendor software |
| **Rollback Safety** | Automatic `.reg` backup exported to `%SystemRoot%\Temp` before changes | Manual export required; mistakes can break unrelated COM entries | Requires reinstalling IDM and scanning system | Official uninstall/license transfer |
| **Maintainability** | Reset (`[3]`), re-freeze (`[1]`), toggle updates (`[4]`/`[5]`), `/silent` CLI | Error-prone and tedious to repeat after IDM updates | Breaks on updates; relies on waiting for new repacks | Official automatic updates & support |
| **Best Fit** | Personal Windows testing, trial freezing, Batch/ACL study | Advanced Windows internals debugging | **Not recommended** | Commercial, enterprise, and long-term use |

---

## System Requirements

| Requirement | Specification |
| --- | --- |
| **Operating System** | Windows 7 / 8 / 8.1 / 10 / 11 (including 24H2) |
| **Privileges** | Administrator privileges (`开始激活.cmd` prompts via UAC automatically) |
| **Built-in Tools** | Windows PowerShell (`FullLanguage` mode required) |
| **Network** | `[1]` Freeze and `[2]` Activate require reachability to `internetdownloadmanager.com` (ping + TCP port 80; exits with code `1` without touching registry if unreachable). `[3]`, `[4]`, `[5]` work offline. |
| **Console Code Page** | Automatically executes `chcp 936` inside the script |
| **Storage Path** | Extract to a normal writable folder (not inside the zip or `C:\Program Files`) |

---

## Usage Guide

### Before First Run

1. **Administrator Rights**: `开始激活.cmd` triggers UAC automatically; click **Yes**.
2. **SmartScreen Prompt**: Unsigned `.cmd` scripts may trigger Windows SmartScreen ("Windows protected your PC"). Click **More info** → **Run anyway**, or right-click the file → **Properties** → check **Unblock** → **OK**.
3. **Antivirus False Positives**: Heuristic engines (Windows Defender / third-party AV) may flag registry ACL + elevation scripts. Verify the SHA256 against [`release/IDM-Activation-Script.zip.sha256`](./release/IDM-Activation-Script.zip.sha256), inspect the plaintext scripts directly, and optionally exclude the extracted folder temporarily.

### Method 1: Interactive Menu (Recommended)

1. Double-click **`开始激活.cmd`**.
2. Wait for the 9-step environment self-check to finish.
3. Choose an option in the menu (option `[1]` is recommended for IDM 6.42+):

```text
┌─────────────────────────────────────┐
│  脚本 1.0.3 | 已适配 IDM 6.43        │
│  本机检测到的 IDM：6.43.x            │
├─────────────────────────────────────┤
│  [1] 冻结试用期      (Freeze Trial - Recommended)
│  [2] 激活            (Activate with generated registration)
│  [3] 重置激活/试用期  (Reset activation / trial)
│  [4] 禁用 IDM 更新提示 (Disable IDM update prompts)
│  [5] 恢复 IDM 更新提示 (Restore IDM update prompts)
│  [6] 下载 IDM         (Open IDM download page)
│  [7] 帮助             (Open repository help)
│  [0] 退出             (Exit)
└─────────────────────────────────────┘
```

### Method 2: Command-Line & Unattended Mode

Run from an **Administrator** Command Prompt (`cmd.exe`):

```cmd
:: Freeze trial period (recommended)
IAS.cmd /frz

:: Activate by writing generated registration info
IAS.cmd /act

:: Reset activation and trial state
IAS.cmd /res

:: Disable IDM automatic update check (CheckUpdtVM = 0)
IAS.cmd /noupd

:: Restore IDM automatic update check (CheckUpdtVM = 1)
IAS.cmd /reupd

:: Unattended silent mode with custom log path (no spaces allowed in path)
IAS.cmd /frz /silent /log=C:\Temp\ias.log
```

**CLI Behavior Notes** (full specification in [`docs/reference/cli.md`](./docs/reference/cli.md)):

- **Action flags**: `/frz`, `/act`, `/res`, `/noupd`, `/reupd` (pass one action flag per call; if both `/noupd` and `/reupd` are passed, `/noupd` takes precedence).
- **Silent flag**: `/silent` (equivalent alias: `/quiet`) suppresses menus and pauses and automatically enables logging. Calling `/silent` without any action flag exits immediately with code **`2`**.
- **Logging flags**: `/log` writes to `%SystemRoot%\Temp\IAS-<timestamp>.log`; `/log=PATH` writes to a custom path (creates parent directories if missing, falls back to default path on write error). **Do not include spaces in `PATH`** because arguments are split on whitespace after quote stripping.
- **Exit code propagation**: `开始激活.cmd` forwards all arguments to `IAS.cmd`, but if launched non-elevated it spawns a new elevated process via UAC and the original window cannot capture the child exit code. For scripts checking `%errorlevel%`, invoke `IAS.cmd` directly from an elevated shell.
- **Conhost bypass**: If `conhost.exe` re-entry is blocked by security software or policy, run `IAS.cmd -qedit` from an Administrator CMD window.

---

## Feature Reference

| Menu | CLI Flag | Network Required | Behavior & Trade-offs |
| --- | --- | --- | --- |
| `[1]` | `/frz` | Yes | Scans and locks IDM trial tracking CLSID keys with Deny ACL without writing a serial key. Most reliable against "fake serial" alerts on IDM 6.42+. |
| `[2]` | `/act` | Yes | Performs CLSID locking plus writes generated `FName`, `LName`, `Email`, and `Serial` values. Works even if a 30-day trial was never claimed, though online serial checks may occasionally trigger a fake-serial warning. |
| `[3]` | `/res` | No | Removes registration entries, trial counters, and tracked CLSID keys to restore pre-script state. |
| `[4]` | `/noupd` | No | Sets `HKCU\Software\DownloadManager\CheckUpdtVM` to `0` so IDM stops prompting for updates (does not touch activation or CLSID locks). |
| `[5]` | `/reupd` | No | Sets `CheckUpdtVM` back to `1` to re-enable IDM update checks. |

---

## Frequently Asked Questions

<a id="en-q1"></a>
<details>
<summary><b>Q1: How do I fix "Fake serial number has been blocked" or repeated registration popups after activation?</b></summary>

Option `[2]` (`/act`) writes a randomly generated serial number, which IDM's online validation can flag as invalid. Run `[3]` **Reset** (`IAS.cmd /res`) to remove the old serial, then switch to `[1]` **Freeze Trial** (`IAS.cmd /frz`), which writes no serial key and avoids online serial validation.

</details>

<a id="en-q2"></a>
<details>
<summary><b>Q2: Should I choose [1] Freeze Trial or [2] Activate for IDM 6.42 / 6.43 and newer?</b></summary>

Choose **`[1]` Freeze Trial** in nearly all cases. It locks IDM's trial-tracking CLSID registry keys without writing a serial number, making it far more resilient on IDM 6.42 and 6.43+. Use `[2]` Activate only if you have never claimed a 30-day trial and want immediate registration, then fall back to `[3]` + `[1]` if prompted.

</details>

<a id="en-q3"></a>
<details>
<summary><b>Q3: Why do Chinese characters turn into garbled text, and does this work on non-Chinese Windows?</b></summary>

`IAS.cmd` and `开始激活.cmd` are saved in GBK (code page 936, no BOM) and execute `chcp 936` automatically at startup. On Simplified-Chinese Windows, menus render cleanly; on non-Chinese Windows locales without CP936 console fonts installed, Chinese UI text may still appear as boxes or mojibake even though the registry logic and CLI flags (`/frz`, `/act`, `/res`, `/noupd`, `/reupd`, `/silent`, `/log=`) function identically.

</details>

<a id="en-q4"></a>
<details>
<summary><b>Q4: Does the script work on Windows 11 24H2, and how do I handle SmartScreen or Defender alerts?</b></summary>

Yes, Windows 11 (including 24H2) is supported. Because batch scripts are unsigned and perform registry ACL changes plus UAC elevation, SmartScreen or heuristic AV engines may flag them. Always verify `IDM-Activation-Script.zip` against `IDM-Activation-Script.zip.sha256`, right-click → **Properties** → check **Unblock**, and inspect the plaintext source before running.

</details>

<a id="en-q5"></a>
<details>
<summary><b>Q5: Why does the script freeze at "Initializing..." or flash "Access is denied" after self-checks?</b></summary>

- **Hang during initialization**: Legacy `Get-WmiObject` can stall on damaged WMI repositories or AV hooks in newer Windows builds. `IAS.cmd` prioritizes `Get-CimInstance` and prints progress steps; if it still stalls, run `winmgmt /verifyrepository` in an elevated CMD.
- **Flash "Access is denied" entering menu**: Interactive mode attempts `start conhost.exe` to disable QuickEdit. If blocked by security software or policy, open an Administrator CMD in the folder and run `IAS.cmd -qedit` directly.

</details>

<a id="en-q6"></a>
<details>
<summary><b>Q6: Why did the Edge / Chrome IDM browser extension icon turn gray or stop working after activation?</b></summary>

The script only ensures `AdvIntDriverEnabled2` is set to `1`; it does not install, modify, or manage browser extensions. Note that freeze/activate/reset finishes by terminating `IDMan.exe`, so start IDM manually afterward, verify the browser extension version matches your installed IDM version, check **IDM → Options → General** browser integration boxes, and ensure your Windows user account matches the account used to install IDM.

</details>

<a id="en-q7"></a>
<details>
<summary><b>Q7: How do I disable IDM "New version available" update popups without breaking activation?</b></summary>

Select **`[4]` Disable IDM update prompts** (or run `IAS.cmd /noupd`). This stops running `IDMan.exe` and writes only `HKCU\Software\DownloadManager\CheckUpdtVM = 0` without modifying serials or CLSID locks. You can revert anytime using `[5]` (`IAS.cmd /reupd`).

</details>

<a id="en-q8"></a>
<details>
<summary><b>Q8: What changes does the script make to Windows, and how do I completely roll back?</b></summary>

Every modified registry location is documented in [`docs/reference/registry.md`](./docs/reference/registry.md). The script never alters `hosts`, firewall rules, system proxies, scheduled tasks, or IDM executables. To revert: run `[3]` **Reset** (`IAS.cmd /res`) or double-click the automatic backup `.reg` files saved in `C:\Windows\Temp\_Backup_HK*_CLSID_*.reg`. Backup files and `IAS-*.log` logs are not auto-deleted so you can inspect or clean them manually.

</details>

---

## Technical Details & Safety Boundaries

### Execution Flow (`[1]` Freeze / `[2]` Activate)

1. **Pre-check & Backup**: Confirms `IDMan.exe` exists and `internetdownloadmanager.com` is reachable (otherwise exits `1` without touching the registry), closes running `IDMan.exe`, and exports `HKCU` / `HKU\<SID>` CLSID branches to `%SystemRoot%\Temp`.
2. **Cleanup**: Deletes existing registration and trial counter values under `HKCU\Software\DownloadManager` (and `HKU\<SID>` when unsynced) plus `%HKLM%` IDM key.
3. **Integration Restore**: Re-creates `%HKLM%\AdvIntDriverEnabled2 = 1`.
4. **CLSID Scan & Lock**: Identifies IDM trial-tracking CLSID GUID keys, takes ownership (`S-1-0-0` NULL SID), and applies a `Deny FullControl` ACL for `Everyone` (if more than 20 candidate keys are found, deletes them instead).
5. **Registration Write (`[2]` only)**: Writes generated `FName`, `LName`, `Email`, and `Serial` values; `[1]` skips this step.
6. **Verification & Re-lock**: Downloads small test images from `internetdownloadmanager.com` through IDM to verify download functionality, then re-applies CLSID locking.

### Exit Codes

| Exit Code | Meaning | Typical Triggers |
| --- | --- | --- |
| `0` | Success | Action (`/frz`, `/act`, `/res`, `/noupd`, `/reupd`) completed or interactive menu exited cleanly |
| `1` | Business Failure | IDM not installed, `internetdownloadmanager.com` unreachable, registry delete/write error, or IDM download test failed |
| `2` | Environment / Argument Error | `/silent` without action flag, Windows build `< 7600`, missing PowerShell or non-`FullLanguage`, non-admin, WMI/CIM/SID error, unwritable CLSID, archive temp directory, or LF line endings / missing trailing blank line in `IAS.cmd` |

### Registry & Filesystem Scope

- **Read/Write Targets**: `HKCU\Software\Classes\CLSID` (or `Wow6432Node\CLSID`), `HKU\<SID>\Software\Classes\CLSID`, `HKCU\Software\DownloadManager` (`FName`, `LName`, `Email`, `Serial`, `scansk`, `tvfrdt`, `radxcnt`, `LstCheck`, `ptrk_scdt`, `LastCheckQU`, `CheckUpdtVM`), `HKLM\Software\Internet Download Manager` (`Wow6432Node` on x64/ARM64; deleted then `AdvIntDriverEnabled2 = 1` restored).
- **Explicitly Untouched**: `hosts`, Windows Firewall, system proxy, scheduled tasks, services, Group Policy, and all IDM executable/DLL binaries.
- **Enterprise Policy Boundary**: Does **not** bypass WDAC or AppLocker. Contact your IT administrator on managed enterprise devices.

---

## Repository Files

| File / Directory | Purpose |
| --- | --- |
| `开始激活.cmd` | Recommended double-click entry script (UAC elevation + 9 pre-flight checks → calls `IAS.cmd`; GBK + CRLF) |
| `IAS.cmd` | Core batch/PowerShell engine (menu, CLI flags, CLSID ACL locking, backup, logging; GBK + CRLF) |
| `使用说明.txt` | Minimal quick-start text guide bundled in the zip (UTF-8 with BOM + CRLF) |
| [`README.md`](./README.md) | Primary Simplified-Chinese documentation |
| [`README.en.md`](./README.en.md) | Complete English documentation (this file; repository-only, not bundled in zip) |
| [`CHANGELOG.md`](./CHANGELOG.md) | Release changelog (verified against `IAS.cmd` `iasver` in CI) |
| [`llms.txt`](./llms.txt) / [`llms-full.txt`](./llms-full.txt) | Structured AI/LLM summaries following `llmstxt.org` (CRLF, no BOM; repository-only) |
| [`codemeta.json`](./codemeta.json) | CodeMeta 2.0 machine-readable software metadata (LF, no BOM; repository-only) |
| [`docs/`](./docs/README.md) | Architecture, module internals, CLI/registry references, deployment guides, and operations docs |
| `tools/` | PowerShell CI guards (`validate.ps1`, `check-docs.ps1`, `verify-release.ps1`, `pack-release.ps1`) |
| [`release/`](./release/README.md) | Published `IDM-Activation-Script.zip` and `.sha256` checksum |

---

## Search Intent & AI Indexing

Machine-readable project metadata and structured summaries:

- [`llms.txt`](./llms.txt) — Concise bilingual summary (`llmstxt.org` convention)
- [`llms-full.txt`](./llms-full.txt) — Extended technical reference
- [`codemeta.json`](./codemeta.json) — CodeMeta 2.0 structured software schema

<details>
<summary><b>Common Search Queries & Direct Answers (SEO / AEO / GEO Reference)</b></summary>

| Search Query | Direct Solution in This Repository |
| --- | --- |
| IDM activation script / IDM activation script Chinese | Double-click `开始激活.cmd` or run `IAS.cmd /frz` → [Usage Guide](#usage-guide) |
| How to freeze IDM trial period / IDM 30-day trial expired | Choose `[1]` Freeze Trial (`IAS.cmd /frz`), or `[3]` Reset then `[1]` → [Feature Reference](#feature-reference) |
| IDM fake serial number has been blocked fix | Run `[3]` Reset (`IAS.cmd /res`) then `[1]` Freeze Trial (`IAS.cmd /frz`) → [FAQ Q1](#en-q1) |
| IDM trial reset script / reset IDM registration | Choose `[3]` Reset (`IAS.cmd /res`) → [Feature Reference](#feature-reference) |
| Disable IDM new version update popup | Choose `[4]` (`IAS.cmd /noupd`) to set `CheckUpdtVM=0` → [FAQ Q7](#en-q7) |
| IDM unattended silent CLI activation | Run `IAS.cmd /frz /silent /log=C:\Temp\ias.log` as Admin → [Method 2: CLI](#method-2-command-line--unattended-mode) |
| IDM script works on Windows 11 24H2? | Yes, Windows 7 through Windows 11 (24H2) supported → [FAQ Q4](#en-q4) |
| IDM crack vs registry activation script | No IDM `.exe`/`.dll` modification; registry-only with `.reg` backups → [Alternatives Comparison](#technical-alternatives-comparison) |

</details>

**Keywords**: `idm`, `internet-download-manager`, `idm-activation-script`, `idm-trial-freeze`, `idm-trial-reset`, `idm-fake-serial-fix`, `windows-11-24h2`, `batch-script`, `powershell`, `unattended`, `silent-install`, `clsid`, `cp936`, `gbk`, `open-source`, `gpl-3.0`.

---

## Disclaimer & License

- **Educational & Testing Purpose**: Provided strictly for learning Windows registry ACL management and batch scripting.
- **No Vendor Affiliation**: Internet Download Manager is proprietary software by Tonec Inc. This repository has **no affiliation, sponsorship, or authorization from Tonec Inc.** and does not distribute IDM installers. For long-term or commercial use, purchase an official license at https://www.internetdownloadmanager.com/buy_now.html.
- **No Fabricated Claims**: This repository makes no unverified claims regarding star counts, download counts, benchmarks, or commercial endorsements.
- **License**: Released under the **GNU General Public License v3.0 (GPL-3.0)**. See [`LICENSE`](./LICENSE) for full terms and the "NO WARRANTY" clause.
