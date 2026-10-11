---
sidebar_position: 19
title: "Use a Windows PC as a Desktop SSH Host"
description: "Make a native-Windows Hermes install reachable from Hermes Desktop on another machine over SSH: OpenSSH Server, key-only login, the Hermes path, and the known pitfalls"
---

# Use a Windows PC as a Desktop SSH Host

Hermes Desktop can open an **SSH** connection to another machine and run its
backend there, so sessions, memory, skills, and cron all live on one host and
every Desktop that connects shares them. This guide covers the case where that
host is a **native Windows** install (not WSL) and the client is another
Desktop (for example a Mac on the same LAN).

For the connection registry itself, see
[Connecting Desktop to Many Hermes Instances](../user-guide/multi-connection-desktop.md).

## 1. Enable OpenSSH Server on the Windows host

In an **elevated** PowerShell:

```powershell
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
Set-Service -Name sshd -StartupType Automatic
Start-Service sshd
# The capability normally adds this rule; create it if your network profile is "Public".
New-NetFirewallRule -Name OpenSSH-Server-In-TCP -DisplayName 'OpenSSH Server (sshd)' `
  -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22 -Profile Any
```

## 2. Key-only login

If the Windows account is in the **Administrators** group, sshd reads keys from
`C:\ProgramData\ssh\administrators_authorized_keys`, **not**
`%USERPROFILE%\.ssh\authorized_keys`. The file must be ASCII and its ACL must
allow only Administrators and SYSTEM, or sshd silently ignores it:

```powershell
$ak = "$env:ProgramData\ssh\administrators_authorized_keys"
Add-Content -Path $ak -Value '<contents of the client's id_ed25519.pub>' -Encoding ascii
icacls.exe $ak /inheritance:r /grant "*S-1-5-32-544:F" /grant "SYSTEM:F"
```

Then set `PasswordAuthentication no` in `C:\ProgramData\ssh\sshd_config`,
validate with `sshd.exe -t`, and `Restart-Service sshd`.

From the client, `ssh -o BatchMode=yes user@host hermes --version` must print a
version with no prompt. Desktop runs SSH non-interactively, so a key with a
passphrase has to be in the agent (`ssh-add --apple-use-keychain ~/.ssh/id_ed25519`
on macOS).

## 3. Add the connection in Desktop

**Settings → Gateways → Add connection → SSH**:

- **SSH host** — `user@host:22`
- **Key file** — the **private** key (`~/.ssh/id_ed25519`), never the `.pub`.
  Pointing it at the public key fails with `bad permissions` / `Permission denied (publickey)`.
- **Hermes path** — see below.

### Choosing the Hermes path

Desktop needs an executable that satisfies two checks at once:

1. A `python.exe` sits **in the same directory** (the platform probe rejects it otherwise).
2. It answers `--print-runtime-command` (the spawn step asks the launcher for the
   real interpreter command).

On a default native-Windows source install neither candidate satisfies both:

| Path | `python.exe` beside it | `--print-runtime-command` |
|---|---|---|
| `%LOCALAPPDATA%\hermes\bin\hermes.cmd` (on `PATH`) | no | yes |
| `…\installs\<id>\environments\<env>\venv\Scripts\hermes.exe` | yes | no (`unrecognized arguments`) |

The venv directory name also changes on every `hermes update`. Until Desktop
resolves this itself, point **Hermes path** at a stable directory holding two
small forwarders — `hermes.exe` that runs `bin\hermes.cmd`, and `python.exe` that
runs the current venv's interpreter (read from `installs\*\facts.json`). Leaving
the field blank only works when `hermes.exe` with a `python.exe` beside it is on
the SSH session's `PATH`.

## Known issues

### "命令行太长" / "The command line is too long"

Windows OpenSSH runs remote commands through `cmd.exe` by default, which caps a
command line at 8191 characters. The Desktop spawn command
(`powershell.exe -EncodedCommand <base64>`) is longer than that. The probe
already sends its script over stdin; the spawn and helper commands do not yet.

Setting `HKLM\SOFTWARE\OpenSSH\DefaultShell` to `powershell.exe` is **not** a
fix: the probe's `-Command [ScriptBlock]::Create(...).Invoke()` is then
re-parsed by the outer PowerShell and fails with
`[System.String] does not contain a method named 'Invoke'`. A working
workaround is a `DefaultShell` that passes short commands to `cmd.exe /c`
unchanged and starts over-long commands directly with `CreateProcess`
(32767-character limit).

### "Remote Windows backend exited before announcing its port" with `owned:false`

If the error's `state` shows `"alive":true,"owned":false,"reason":"argv"` and
the argv starts with `python.exe -I -c …`, the backend is running but the
ownership check did not recognise it: it required `-c` immediately after the
interpreter, while the launcher's runtime command inserts `-I` first. Fixed in
`hermes_cli/windows_ssh_runtime.py` (`process_state` skips interpreter flags).

### `profile inventory failed: Unsafe remote Hermes home`

The SSH profile inventory resolves the remote home with a POSIX `echo`
command and validates it as a POSIX path. On a Windows host this always fails;
it only affects the roster preview and does not block connecting.

## Operating notes

- Two backends — the host's own Desktop and the SSH-spawned one — share
  `state.db` (WAL mode handles this) and the built-in memory files. Avoid
  driving the same session from both machines at once.
- A DHCP address can change; a DHCP reservation, a DNS name, or a VPN such as
  Tailscale gives the connection a stable host.
- `hermes update` replaces the venv the forwarders resolve; re-run **Test**
  afterwards.
