# Team Setup

We follow the organizers' recommended route: **DevEco Studio on Windows, plus the hackathon agent tooling and project template**. Full official guide: [onirodeveloper/hackyeah2026-challenge](https://github.com/onirodeveloper/hackyeah2026-challenge) (README and FAQ).

Do everything here **before writing product code**.

## 1. Accounts

| Account | Who | Why |
|---------|-----|-----|
| GitHub | both | Done: `sedatsan` (admin), `yeet30` (write) |
| Huawei ID | both | Downloading DevEco Studio and signing a shareable `.hap` |
| Claude Code | both | Coding agent |
| Prelint | both | PR review and the MCP server. Each person signs in once |
| Antigravity (`agy`) | Sedat only | Worker agent under Sedat's Claude |

**Huawei verification still pending (it takes 1–2 working days)?** You can still work:
- If the official DevEco Studio download is blocked, the organizers share a fallback installer (link in their README, step 1).
- **Debug builds need no account.** Run/Debug in DevEco signs them with a development certificate automatically, and they install on your emulator.
- An account-backed signature is only needed for a `.hap` you hand in or share. **Ask the Huawei mentors** how to sign the submission if verification isn't done by then.

## 2. Windows prerequisites

| Item | Requirement | Sedat (checked 2026-10-03) |
|------|-------------|----------------------------|
| Windows | 10 or 11, 64-bit, 16 GB RAM, ~100 GB disk | Windows 11 Pro, 15.8 GB RAM (close other apps while the emulator runs) |
| Virtualization | **Hyper-V** and **Windows Hypervisor Platform** enabled, then reboot | Both enabled (rebooted 13:47) OK |
| Git | on `PATH` | 2.52 OK |
| Node.js 22+ with npm | on `PATH` (the DevEco CLI needs it) | Node 24.14, npm 11.11 OK |
| Python 3 | the `python` command works | 3.12.10 OK |
| npm global folder on `PATH` | `npm.cmd config get prefix` | `X:\npm-global`, on PATH OK |
| JDK | only for Oniro signing | JDK 21 OK |

Enable virtualization from an **admin** PowerShell, then reboot:
```powershell
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V-All, HypervisorPlatform -All
```
Or use Control Panel → Programs → Turn Windows features on or off.

**Disk:** keep DevEco Studio in its **default install location**, so the agent tooling finds it without extra configuration. Put the **SDK and emulator images on X:**, because they are the large parts (C: has 65 GB free, X: has 131 GB).

## 3. DevEco Studio

1. Install the latest DevEco Studio. Setup guide: [Oniro docs, DevEco Studio installation](https://docs.oniroproject.org/application-development/environment-setup-guide/deveco-studio/installation/).
2. **Launch it once and finish the first-launch setup.** This creates the config files the next step edits. Then close it.

## 4. Agent tooling (the organizers' installer prompt)

Paste [`INSTALLATION_PROMPT.md`](https://github.com/onirodeveloper/hackyeah2026-challenge/blob/main/INSTALLATION_PROMPT.md) into Claude Code. It:
- **switches DevEco's region to China** (`country.region.xml`). Outside China, DevEco only offers watch emulators. In the China region it also offers phone, tablet, 2-in-1 and TV.
- installs 9 agent skills at user level: `ohos-app-scaffold`, `ohos-app-dev`, `ohos-system-app-dev`, `ohos-system-dev`, `conductor-dev`, `hmos-arkts-knowledge-retriever`, `hmos-arkui-scenario-development`, `hmos-arkui-develop-skill`, `hmos-arkui-mvvm-pattern`
- installs `@deveco/deveco-cli` **1.3.4** (pinned), applies the organizers' two patches, and runs `devecocli init --skill`
- prepares the **Hackathon Template** and **Conductor Hackathon Template** folders. **You** copy them into `<DevEco Studio>\plugins\openharmony\lib\templates\project` with File Explorer. Windows will ask for approval.

Restart DevEco Studio and Claude Code afterwards.

**Sedat's status (DevEco 6.1.1.280, default location):** skills, DevEco CLI 1.3.4 and its patches, region switched to CN, `devecocli init --skill` all done. Emulator images and instances are moved to `X:\Huawei\Sdk` and `X:\Huawei\Emulator\deployed` (Device Manager settings). The bundled HarmonyOS SDK (6.3 GB) stays inside the DevEco install folder, since it's part of the installation. Still to do: copy the two templates from `%TEMP%\oni-tpl`.

**macOS (if yeet30 is on a Mac):** the installer is Windows-only. Change the region by hand (FAQ: "How do I switch the DevEco Studio region to China manually?") and install the skills with `npx skills add -g <url> -a claude-code -y`. The DevEco emulator runs only on supported **Apple Silicon** Macs.

## 5. Emulator

1. DevEco → **Device Manager** → create a **Phone** device with the **newest system image** (API 24). Download it to X:. It's several GB. Start it.
2. This is GUI-only. Agents cannot do it.

What the DevEco emulator can and cannot do:

| Works | Does not work |
|-------|---------------|
| Push notifications, widgets, internet | **NFC** |
| **System apps and extra app privileges** | **Distributed / multi-device features** |
| Basic sensors (accelerometer, gyroscope, GPS) | Real camera, Bluetooth pairing, SIM, biometrics |
| Phone, tablet, 2-in-1, wearable and TV profiles | Accurate battery and thermal behaviour |

Microphone input is not listed. Test it early if the idea needs audio.

## 6. Create the project (one person, on a `chore/scaffold` branch)

1. DevEco → **Create Project** → **Hackathon Template** → **Phone**.
2. **Compatible SDK `6.0.0 (API 20)`** (the challenge minimum).
3. In the root `build-profile.json5`: `compileSdkVersion` **API 23**, `targetSdkVersion` **API 24**.
4. Create it at the repo root. The template brings `AGENTS.md`, `CLAUDE.md`, `GEMINI.md`, `AI_WORKFLOW.md`, `HACKATHON_BRIEF.md` and `hackathon-resources/`. **Merge** our existing `AGENTS.md` rules into the template's files instead of overwriting either set.
5. Build and run on the emulator, then ship it as one PR (shared files, spec §5).

## 7. Signing rules for two people

- Never commit signing material (`.p12`, `.cer`, `.p7b`) or its passwords (rule R8). `.gitignore` already covers the files.
- DevEco writes your personal signing config into `build-profile.json5`. **Never commit that `signingConfigs` block.**
- `9568332 install sign info inconsistent` means the app on the device was signed by someone else's machine. Uninstall it first, but ask before doing this, because it deletes the app's data.

## 8. Repo and agents

```bash
gh auth login
git clone https://github.com/sedatsan/Beltur.git
```
- Start Claude Code in the repo. It loads `CLAUDE.md`, which imports `AGENTS.md`. Approve the `prelint` MCP server, then run `/mcp` → **prelint** → sign in.
- Huawei PDFs (`RULES.pdf`, `CRITERIA*.pdf`) stay local (R7).
- Antigravity runs only through `/delegate`, sandboxed, in `X:\Beltur-agy`.

## 9. Alternative: Oniro App Builder and the Oniro Emulator

This is an open-source route that needs no DevEco emulator. The emulator is QEMU-based and runs OpenHarmony 6.1 (API 23). It's less mature: no sensors, no multi-device profiles. On Windows, the command-line tools ZIP must come from the Huawei download page. See the [Oniro setup guide](https://docs.oniroproject.org/application-development/environment-setup-guide/oniro/setup/). Use it only if the DevEco route is blocked.

## 10. Checklist before product work

- [ ] Both: Hyper-V and Windows Hypervisor Platform enabled
- [ ] Both: DevEco installed, launched once, region switched to CN, templates copied
- [ ] Both: phone emulator (API 24) downloaded and starting
- [ ] Both: Claude Code in the repo, Prelint signed in
- [ ] Asked the Huawei mentors about verification and signing the submitted `.hap`
- [ ] Roles and idea decided
- [ ] Project created from the Hackathon Template, builds and runs on both emulators
