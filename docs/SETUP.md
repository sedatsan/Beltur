# Team Setup

Do everything here **before writing product code**. The slow steps come first.
Anything marked *(verify)* is something we could not confirm from documentation alone. The smoke test in step 8 checks it.

## 0. Start these now (they take the longest)

1. **Huawei ID and developer registration** (each person) at [developer.huawei.com](https://developer.huawei.com/consumer/en/). You need a Huawei ID to download DevEco Studio, to download emulator images and to sign apps automatically for real devices. Downloading the standalone Command Line Tools also needs a **verified** developer account. Start the identity verification now, because it may not be instant *(verify)*.
2. **Download DevEco Studio.** It's several GB. Get it from the Download Center on the [DevEco Studio page](https://developer.huawei.com/consumer/en/deveco-studio/), logged in with your Huawei ID.

## 1. Accounts

| Account | Who | Why | Status |
|---------|-----|-----|--------|
| GitHub | both | Repo, issues, PRs | Done: `sedatsan` (admin), `yeet30` (write) |
| Huawei ID / Huawei Developer | both | DevEco, emulator, signing | To do |
| Claude Code | both | Coding agent | Sedat: done. yeet30: log in with his own account |
| Prelint | both | PR review and the MCP server | Each person signs in once (step 7) |
| Antigravity (`agy`) | Sedat only | Worker agent under Sedat's Claude | Installed |

## 2. Machine requirements

Huawei's minimum for DevEco Studio:

| | Windows | macOS |
|---|---|---|
| OS | Windows 10 or 11, 64-bit | x86: macOS 11–15, Arm: macOS 12–15 |
| RAM | 16 GB or more | 8 GB or more (the emulator wants more) |
| Disk | 100 GB or more | 100 GB or more |
| Screen | 1280 × 800 or higher | 1280 × 800 or higher |

**Emulator on Windows:** turn on **Hyper-V**, **Windows Hypervisor Platform** and **Virtual Machine Platform** (Control Panel → Programs → Turn Windows features on or off), then reboot. Virtualization must be enabled in the BIOS. The emulator does not work inside a VM. It needs a lot of RAM, so close games, browsers and other heavy apps while it runs.

**Emulator on macOS:** if yeet30 is on a Mac, confirm on day one that an API 20 emulator starts on his machine *(verify)*.

### Sedat's machine (checked 2026-10-03)

- RAM 15.8 GB: right at the minimum. Close other apps while the emulator runs.
- A hypervisor is already running, so Hyper-V is on. Still confirm that **Windows Hypervisor Platform** is ticked.
- Free disk: C: 65 GB, X: 131 GB. **Install DevEco, the SDK and emulator images on X:**, because C: is below the 100 GB recommendation.
- DevEco Studio is not installed yet.
- `git config user.email` is `sedat03san@gmail.com`. Make sure that address is on your GitHub account, or commits won't link to your profile.

## 3. DevEco Studio and the SDK (API 20)

The challenge requires **API 20 or later, with API 20 as the minimum** (rule R10). API 20 is HarmonyOS 6.0.0.

1. Install the **latest DevEco Studio release**. 6.0.2 or newer bundles the HarmonyOS SDK, Node.js, hvigor, ohpm and the emulator.
2. Use paths without spaces, for example `X:\Huawei\DevEco Studio` and SDK `X:\Huawei\Sdk`.
3. On first launch, sign in with your Huawei ID (top-right), let it download the SDK, and run the environment check if it offers one.
4. In the SDK manager, make sure an SDK with **API 20 or newer** is installed.

When the project is created (step 8), set **Compatible SDK = `6.0.0(20)`**. That is the minimum the jury checks.

## 4. Emulator

1. **Tools → Device Manager → Local Emulator → New Emulator.**
2. Pick a **phone** image with **API 20 or newer**, download it to X:, and start it.
3. It works when the Hello World app from step 8 runs on it.

Things the emulator cannot do (plan the demo around them):
- **No NFC.** NFC features need a real phone (mentors have devices on site) plus NFC tags, or a second phone emulating a card.
- **Microphone input:** whether the emulator receives the PC microphone is unconfirmed *(verify)*. Test it in the first hour if the idea uses audio.
- Sensors such as the accelerometer and location are limited or simulated.

## 5. Real device (only if the idea needs NFC, sensors or audio)

1. On the phone: **Settings → About phone →** tap the software version 7 times to unlock developer options. Then **Settings → System → Developer options → USB debugging** *(verify exact menu names)*.
2. Connect it by USB and accept the prompt. `hdc list targets` should show the device.
3. In DevEco: **File → Project Structure → Signing Configs → Automatically generate signature** (needs the Huawei ID login).

## 6. Command-line build (the agents need this)

Agents build and install from the terminal, not the IDE. The tools come with DevEco Studio:

| Tool | Purpose | Check |
|------|---------|-------|
| `ohpm` | Package manager (like npm) | `ohpm -v` |
| `hvigorw` | Build tool | `hvigorw -v` |
| `hdc` | Device connector (like adb) | `hdc version` |

Add their folders inside the DevEco install to `PATH`. Typically these are `tools\ohpm\bin`, `tools\hvigor\bin`, `tools\node` and the SDK's `toolchains` folder for `hdc` *(verify on your install)*. Also set `DEVECO_SDK_HOME` to the SDK folder, which command-line builds need *(verify)*. Restart the terminal and Claude Code afterwards.

Build, install and launch from the repo root:
```bash
ohpm install
hvigorw assembleHap --mode module -p product=default --no-daemon
hdc install entry/build/default/outputs/default/entry-default-signed.hap
hdc shell aa start -a EntryAbility -b <bundleName>
```
The `.hap` path and bundle name are fixed in step 8. Update this block then.

## 7. Repo and agents

```bash
gh auth login
git clone https://github.com/sedatsan/Beltur.git
cd Beltur
git config user.name "<your name>"
git config user.email "<an email on your GitHub account>"
```

- **Line endings:** `.gitattributes` handles them. Leave `core.autocrlf` at its default.
- **Claude Code:** start it in the repo folder. It loads `CLAUDE.md`, which imports `AGENTS.md`, the shared rulebook. Approve the `prelint` MCP server when asked, then run `/mcp` → **prelint** → sign in with your own account.
- **Antigravity (Sedat only):** Sedat's Claude runs it through `/delegate` in a separate worktree `X:\Beltur-agy`, sandboxed. Never start it with `--dangerously-skip-permissions`.
- **Huawei PDFs** (`RULES.pdf`, `CRITERIA*.pdf`) stay local and are never committed (R7).

### Signing gotcha for a two-person team

Automatic signing writes a `signingConfigs` block into `build-profile.json5`. It holds paths to *your* certificate files and encrypted passwords. **Never commit that block.** It is personal, it breaks the other person's build, and signing material must stay out of the repo (R8). After signing, leave that change unstaged. Agents check for it before every commit.

## 8. Smoke test together (about 30 minutes)

Once both machines pass steps 3–7:

1. **One person** creates the project at the repo root on a `chore/scaffold` branch: **Empty Ability**, ArkTS, Stage model, **Compatible SDK `6.0.0(20)`**, bundle name `com.beltur.<appname>` (chosen with the idea).
2. Build with the command line (step 6), install it and launch it on **both** emulators.
3. Fill in the real `.hap` path and bundle name in step 6 of this file. Ship it as one PR. It's a shared-files change, so it gets its own PR (spec §5).
4. Both people pull and build again from the command line. The setup is done when both see the app running.

## 9. Checklist before product work

- [ ] Both: Huawei developer account ready, DevEco with API 20+, emulator runs
- [ ] Both: command-line `ohpm`, `hvigorw`, `hdc` work
- [ ] Both: Claude Code in the repo, Prelint signed in
- [ ] Roles decided (app / platform) and written in `AGENTS.md`
- [ ] Idea decided and written as a short spec in `docs/`
- [ ] Coordination system in place (labels, commands, `AGENTS.md` rules)
- [ ] Submission deadline written in `AGENTS.md`
- [ ] Confirmed that Prelint reviews PRs on `sedatsan/Beltur`
- [ ] Smoke test passed on both machines

## Sources

- [Downloading and installing DevEco Studio (Huawei)](https://developer.huawei.com/consumer/en/doc/harmonyos-guides/ide-software-install)
- [DevEco Studio (Huawei)](https://developer.huawei.com/consumer/en/deveco-studio/)
- [Using hvigorw, command-line tools (Huawei)](https://developer.huawei.com/consumer/en/doc/harmonyos-guides/ide-hvigor-commandline)
- [HarmonyOS version history (Wikipedia)](https://en.wikipedia.org/wiki/HarmonyOS_version_history)
