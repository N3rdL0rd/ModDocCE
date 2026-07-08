---
sidebar_position: 5
---

# Pairip

Pairip (typically appearing as `com.pairip` / `libpairipcore.so`) is **Automatic Integrity Protection (AIP)** — a native app shielding service that Google injects automatically at distribution time when a developer enables it in the Play Console. Newer Google Play releases of Dead Cells ship with Pairip enabled, which is the main obstacle to modding the official package directly.

## What it does

| Check             | Purpose                                                              |
|-------------------|----------------------------------------------------------------------|
| Installer Check   | Blocks APKs not installed from Google Play — "Get this app from Play" dialog, then force quit |
| Anti-Tamper       | Detects modified, re-packaged, or re-signed APKs                     |
| Anti-Hook/Debug   | Detects Frida, GDB, and other instrumentation tools at runtime       |

## How it works

Unlike simple Java-layer checks, Pairip combines several low-level techniques:

- **Custom VM**: Sensitive methods (e.g. `onCreate`) are stripped from the dex, compiled into encrypted bytecode, and stored in assets. At runtime, `libpairipcore.so` interprets these via a native `executeVM`.
- **Direct Syscalls**: Bypasses libc wrappers — uses raw `SVC #0` for `open`, `read` etc. — rendering conventional hooking ineffective.
- **Background Scanning**: A daemon thread monitors `/proc/self/status` (`TracerPid`) and `/proc/self/maps` for injected libraries (e.g. `frida-agent`). Detected intrusions trigger `tgkill` or `exit_group`.

## Impact on modding

Newer Google Play versions of Dead Cells are fully protected by Pairip. Attempting to modify the official package directly will trigger integrity checks and crash the app. The typical workflow is to first bypass or remove Pairip from the build before making other modifications.
