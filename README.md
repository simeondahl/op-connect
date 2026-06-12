# op-connect

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
```

Make sure `~/bin` is on your `PATH` (e.g. add `export PATH="$HOME/bin:$PATH"` to
`~/.bashrc`).

## Usage

```bash
op-connect            # combined fzf picker across all SSH and RDP targets
op-connect ssh        # fzf picker for SSH targets only
op-connect ssh plex   # connect directly to the SSH_KEY item titled "plex"
op-connect rdp        # fzf picker for RDP targets only
op-connect rdp "ODIN - Remote Desktop"
op-connect list       # list all 1Password items with category/vault/tags
```

## Setting up items in 1Password

**SSH**: any item of category "SSH Key" with a `url` field (or primary URL) set
to `ssh://user@host[:port]` is picked up automatically — no tagging needed.

**RDP**: create or edit a "Login" item, set its primary URL to
`rdp://user@host[:port]`, optionally fill in the password field, and add the
tag `rdp`.

## Notes

- The RDP password (if set) is passed to `xfreerdp` as a command-line argument,
  so it's briefly visible to other local users via `ps` on shared machines.
- `~/.1password/agent.sock` is the default 1Password SSH agent socket path;
  override with `OP_SSH_AGENT_SOCK` if yours differs.
