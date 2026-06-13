# op-connect

[![GitHub stars](https://img.shields.io/github/stars/simeondahl/op-connect?style=social)](https://github.com/simeondahl/op-connect/stargazers)

A small CLI that connects to SSH and RDP targets stored in [1Password](https://1password.com),
using `op`, `ssh`, and `xfreerdp`.

- **SSH** targets are 1Password "SSH Key" items with a `url` field formatted as
  `ssh://user@host[:port]`. Authentication goes through the 1Password SSH agent,
  so private keys never touch disk.
- **RDP** targets are 1Password "Login" items tagged `rdp`, with a primary URL
  formatted as `rdp://user@host[:port]` and optionally a password field.
  Connections are launched via `xfreerdp`, with resolution/scaling automatically
  matched to your screen.

## Requirements

- [1Password CLI](https://developer.1password.com/docs/cli/get-started/) (`op`),
  signed in via the desktop app integration (Settings → Developer → "Integrate
  with 1Password CLI")
- [1Password SSH agent](https://developer.1password.com/docs/ssh/get-started/)
  enabled (Settings → Developer → "Use the SSH agent")
- `jq`
- `fzf`
- `freerdp` / `xfreerdp` (for RDP connections)
- `xrandr` and `xdpyinfo` (for screen-resolution detection, usually preinstalled on X11)

## Install

```bash
mkdir -p ~/bin
cp op-connect ~/bin/
chmod +x ~/bin/op-connect
ln -sf op-connect ~/bin/opc
```

Make sure `~/bin` is on your `PATH` (e.g. add `export PATH="$HOME/bin:$PATH"` to
`~/.bashrc`).

The `opc` symlink is a short alias — `opc`, `opc ssh plex`, and `opc rdp` all
work the same as `op-connect`.

## Usage

```bash
op-connect            # combined fzf picker across all SSH and RDP targets
op-connect ssh        # fzf picker for SSH targets only
op-connect ssh plex   # connect directly to the SSH_KEY item titled "plex"
op-connect rdp        # fzf picker for RDP targets only
op-connect rdp "ODIN - Remote Desktop"
op-connect list       # list all 1Password items with category/vault/tags
```

When using a picker (`op-connect`, `op-connect ssh`, or `op-connect rdp` with
no name), the menu reappears after a connection closes — press `Esc` to exit
back to your shell. For SSH sessions, `Ctrl-D` (or `exit`) closes the
connection and returns you to the menu.

## Setting up items in 1Password

**SSH**: any item of category "SSH Key" with a `url` field (or primary URL) set
to `ssh://user@host[:port]` is picked up automatically — no tagging needed.

**RDP**: create or edit a "Login" item, set its primary URL to
`rdp://user@host[:port]`, optionally fill in the password field, and add the
tag `rdp`.

## Notes

- The RDP password (if set) is piped to `xfreerdp` via `/from-stdin:force`,
  so it never appears in `ps`/`/proc` output or process-argument logs.
- `~/.1password/agent.sock` is the default 1Password SSH agent socket path;
  override with `OP_SSH_AGENT_SOCK` if yours differs.
- RDP connections use `/cert:tofu` (trust-on-first-use) by default: the
  server certificate is pinned on first connect, and `xfreerdp` will warn if
  it changes later (e.g. a MITM). If a target's certificate changes often
  (e.g. after firmware updates on an iDRAC), set `OP_RDP_CERT_MODE=ignore` to
  skip verification entirely for that session.
- The 1Password item list is fetched once per session and reused across
  picker loop iterations, so reopening the menu after a connection closes
  doesn't re-query 1Password.
