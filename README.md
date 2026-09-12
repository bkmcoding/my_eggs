<div align="center">

# my_eggs

[![Architecture](https://img.shields.io/badge/Architecture-ARM64-orange?style=flat-square&logo=arm)](https://github.com/yourusername/my_eggs)
[![License: AGPL v3](https://img.shields.io/badge/License-AGPL_v3-blue.svg?style=flat-square)](https://www.gnu.org/licenses/agpl-3.0)

A curated collection of Pterodactyl eggs adapting x86/x64 game servers for ARM64 architecture via box64 emulation.

</div>

---

## Egg Status Tracker

| Game / Service | Architecture | Emulation | Status |
| :--- | :---: | :---: | :--- |
| **[Valheim](./valheim/)** | `ARM64` | `Box86` / `Box64` | Working / Up-to-date |
| More to come maybe |

---

## Valheim (ARM64)
**Directory:** `/valheim/`

Valheim lacks a native ARM64 server binary. This egg translates x86/x64 instructions using [ptitSeb's Box86 and Box64](https://github.com/ptitSeb):
*   **Installation:** A temporary Debian container compiles Box86 to run the 32-bit `steamcmd` updater.
*   **Runtime:** The `tsxcloud/valheim-arm` image uses Box64 to execute the 64-bit game binary.

### Emulation Context & Sources
*   **[Box86](https://github.com/ptitSeb/box86):** Emulates 32-bit x86 Linux binaries on 32-bit/64-bit ARM. Used exclusively here for SteamCMD.
*   **[Box64](https://github.com/ptitSeb/box64):** Emulates 64-bit x86_64 Linux binaries on ARM64. Used to run the Valheim server runtime.
*   These are userspace emulators. They intercept API calls and pass them to native ARM libraries, offering significantly higher performance than full-system emulators like QEMU.

### Server Configuration Gotchas
*   **Disable Crossplay:** Microsoft's PlayFab library (`libParty.so`) crashes under Box64. Set `Enable Crossplay` to `0`. Steam users can connect; Xbox users will time out.
*   **Port Allocations:** Valheim requires the primary port and the query port open. If your primary port is `2456`, allocate `2457` in Pterodactyl's Network settings.
*   **Password Characters:** Do not use single quotes (`'`) in the server password. It will prematurely close the bash startup string and crash the server on boot.

### SteamCMD Troubleshooting on ARM
If you modify the install script, keep these ARM/SteamCMD quirks in mind:
*   **Exit Code 42:** SteamCMD self-updates on launch and exits with code `42`. Standard scripts treat this as a failure. You must wrap the `app_update` command in a `while [ $EXIT_CODE -eq 42 ]` loop to allow it to restart.
*   **Missing Architecture:** SteamCMD is strictly 32-bit. Your install container must run `dpkg --add-architecture armhf` and install `libc6:armhf` before compiling Box86.
*   **Library Paths:** SteamCMD often fails to find its own dependencies under emulation. Always export `LD_LIBRARY_PATH=/mnt/server/steamcmd/linux32:$LD_LIBRARY_PATH` before executing it.

### Updating the Server
Because the runtime bypasses the standard container entrypoint, updates are handled during Pterodactyl's install phase.

1. Enable the **Update on Startup** toggle in server settings.
2. Click **Reinstall Server**.

> **Note:** Reinstalling only runs SteamCMD to fetch binaries. World data is stored safely in `.config/unity3d/IronGate/Valheim/worlds_local/` and is not overwritten.
