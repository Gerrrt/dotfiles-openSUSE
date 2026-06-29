<div align="center">

# 🦎 dotfiles-openSUSE

### openSUSE — both flavors, one config.

The openSUSE layer (zypper) — Tumbleweed and Leap, over the shared core.

`zypper` · `zsh` · `nvim` · `tmux`

[![showcase](https://img.shields.io/badge/showcase-live-7aa2f7?style=flat-square)](https://gerrrt.github.io/dotfiles-web/)
[![openSUSE](https://img.shields.io/badge/openSUSE-ready-9ece6a?style=flat-square)]()

</div>

---

The **OS-native layer** for openSUSE (Tumbleweed + Leap). Core (zsh/tmux/nvim/git)
is vendored under `core/` from [`dotfiles-core`](../dotfiles-core); this repo adds
only what is genuinely openSUSE — zypper, Packman, AppArmor, Btrfs/snapper, the
Wayland clipboard shim.

Stamped from the `dotfiles-Fedora` template per `core/PORTING-MATRIX.md`: same
structure, swapped package manager (`dnf`→`zypper`) and a couple of distro quirks.

## Install (fresh openSUSE)

```bash
git clone <you>/dotfiles-openSUSE ~/dotfiles-openSUSE
cd ~/dotfiles-openSUSE
# one-time: vendor Core (skip if the repo already contains core/)
git subtree add --prefix=core <you>/dotfiles-core main --squash
./bootstrap.sh
exec zsh
```

Flags: `--links-only` (re-link without touching zypper), `--no-flatpak`.

## Layout

```
bootstrap.sh           zypper provision + Core/OS symlink wiring (idempotent)
install/packages.txt   zypper package list (modern CLI stack)
os/opensuse.zsh        OS-native shell layer -> symlinked to ~/.config/zsh/os.zsh
os/opensuse.gitconfig  OS git layer (credential helper) -> ~/.config/git/os.gitconfig
os/opensuse.conf       tmux netspeed/battery bits -> ~/.config/tmux/os.conf
ssh/config             hardened SSH client config -> ~/.ssh/config (keys never tracked)
wsl/wsl.conf           installed to /etc/wsl.conf on WSL
core/                  vendored from dotfiles-core (git subtree; do not hand-edit)
```

Load order in `.zshrc`: `core/tools → core/aliases → core/functions → core/fzf →
core/bindings → core/plugins → core/op → os/opensuse → local`.

## openSUSE specifics baked in

- **Tumbleweed vs Leap — the update command differs, and it bites.** Tumbleweed
  (rolling) upgrades with `zypper dup` (aliased `zdup`); Leap (stable) uses
  `zypper up` (aliased `zup`). Get it wrong and you either don't really update or
  you half-update. `bootstrap.sh` only refreshes metadata — it never force-runs
  an upgrade — so the choice stays yours.
- **zypper's solver is the best of these distros** — lean on it. On a Tumbleweed
  `dup`, vendor-change / package-split prompts are normal; read them, don't
  reflexively decline.
- **AppArmor, not SELinux**, is openSUSE's default MAC. The shell layer ships
  `aa-status` / `aa-complain` / `aa-enforce` / `aa-unconfined` (from
  `apparmor-utils`) where Fedora had `se-*` helpers.
- **Rollback is Btrfs + snapper, not package history.** zypper has no
  `history undo`; instead the root filesystem is snapshotted around each zypper
  transaction. `snaps` lists them; revert with `snapper undochange` or by booting
  a snapshot. This is one of openSUSE's best features.
- **Packman** is the third-party repo for codecs/multimedia (openSUSE's analog to
  Fedora's RPM Fusion). It's **not** auto-added by bootstrap because the repo URL
  differs Tumbleweed-vs-Leap and isn't needed for the CLI stack. Add it manually
  if you want codecs, then `zypper dup --from packman` to switch vendors.
- **fd** is packaged as `fd` here (binary `fd`), unlike Debian's `fdfind`;
  `core/zsh/tools.zsh` resolves the name automatically.
- **starship / atuin / yazi / tree-sitter-cli** aren't reliably in openSUSE repos,
  so `bootstrap.sh` installs them from upstream (`cargo` for yazi + tree-sitter)
  to match the other distro repos exactly.
- **WSL:** openSUSE Tumbleweed/Leap WSL images work fine; `bootstrap.sh` writes
  `/etc/wsl.conf` (systemd + your user + interop). Run `wsl.exe --shutdown` after.
