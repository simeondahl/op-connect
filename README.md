# op-connect

[![GitHub stars](https://img.shields.io/github/stars/simeondahl/op-connect?style=social)](https://github.com/simeondahl/op-connect/stargazers)

A small CLI that connects to SSH and RDP targets stored in [1Password](https://1password.com),
using `op`, `ssh`, and `xfreerdp`.

> A Rust port with `zeroize`-based secret handling is also available:
> [op-connect-rs](https://github.com/simeondahl/op-connect-rs).

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

> **Note:** `opc` is reserved as an alias for [op-connect-rs](https://github.com/simeondahl/op-connect-rs),
> the Rust port. If you want a short alias for this bash version instead, use
> something like `ln -sf op-connect ~/bin/opcb`.

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

## Security

- **No credentials are stored by op-connect.** It calls the `op` CLI for
  every lookup and relies entirely on the 1Password desktop app integration
  for authentication — there's no separate sign-in, token, or cache file on
  disk.
- **SSH private keys never touch disk.** Authentication goes through the
  1Password SSH agent (`~/.1password/agent.sock` by default, override with
  `OP_SSH_AGENT_SOCK`); `op-connect` never reads or copies key material.
- **RDP passwords are passed via stdin**, not argv or environment variables,
  using `xfreerdp /from-stdin:force` — so they never appear in `ps`,
  `/proc/*/cmdline`, or process-argument logs.
- **Revealed item data has a minimal lifetime.** The decrypted JSON from
  `op item get --reveal` (which includes the RDP password) is held in a
  `local` shell variable and explicitly `unset` as soon as the needed fields
  are extracted; the password itself is `unset` once the RDP session ends.
  Note: bash cannot guarantee secrets are zeroed from process memory —
  `unset` drops the reference but doesn't securely wipe the underlying
  bytes. This minimizes, but doesn't eliminate, the exposure window.
- **RDP certificates use `/cert:tofu`** (trust-on-first-use) by default: the
  server certificate is pinned on first connect, and `xfreerdp` warns if it
  changes later (e.g. a MITM). If a target's certificate changes often (e.g.
  after firmware updates on an iDRAC), set `OP_RDP_CERT_MODE=ignore` to skip
  verification for that session — only use this for trusted targets on a
  trusted network.
- **The 1Password item list is cached in memory only**, for the lifetime of
  a single `op-connect` invocation (including picker loop iterations). It is
  never written to disk and is discarded when the process exits. This cache
  only contains metadata from `op item list` (id, title, vault, category,
  tags) — no passwords, private keys, or other field values. Those are only
  fetched (via `op item get`) for the single item you select to connect to.
