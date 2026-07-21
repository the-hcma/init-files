# init-files (bootstrap)

Entrypoint for bootstrapping house shell config on a new machine.

## Exact steps (new host)

You need SSH access to a donor host that already has the house key (password login is fine once).

```bash
# 1) Download the script (preferred over curl|bash — clearer errors, no stale pipe)
curl -fsSL https://raw.githubusercontent.com/the-hcma/init-files/main/bootstrap_host \
  -o /tmp/bootstrap_host
chmod +x /tmp/bootstrap_host

# 2) Optional: inspect
less /tmp/bootstrap_host

# 3) Run — replace HOST (e.g. meerkat, saratoga, hcma@meerkat)
/tmp/bootstrap_host --key-from HOST
# minimal host:  /tmp/bootstrap_host --key-from HOST --no-dev
```

Prompts (in order):

1. SSH password (or key) to `HOST` — **once** (fetches the house key)
2. Passphrase for the house key — **once** (`ssh-add`)

Wait until you see the **`=== bootstrap_host verify ===`** block with `bashrc: … OK`.  
**Only then** reload the shell:

```bash
# 4) After verify succeeds:
source ~/.bashrc
```

`source ~/.bashrc` before a successful verify does nothing useful — there is no symlink yet.

### Confirm it worked

```bash
ls -l ~/.bashrc
# expect: ~/.bashrc -> …/init-files/bashrc

ls -ld ~/.local/share/init-files/.git
git -C ~/.local/share/init-files rev-parse --short HEAD
type refresh_bashrc
```

### Examples

```bash
/tmp/bootstrap_host --key-from meerkat
/tmp/bootstrap_host --key-from hcma@meerkat --no-dev
/tmp/bootstrap_host --key-from hcma@saratoga
```

`HOST` may be a short name, FQDN, or `user@host`. Env alternative: `INIT_FILES_KEY_HOST=meerkat`.

### Key already on this host

If a previous attempt already copied the key (common after a partial run):

```bash
curl -fsSL https://raw.githubusercontent.com/the-hcma/init-files/main/bootstrap_host \
  -o /tmp/bootstrap_host
chmod +x /tmp/bootstrap_host
/tmp/bootstrap_host
# then, after verify OK:
source ~/.bashrc
```

To re-fetch the key from the donor anyway: `/tmp/bootstrap_host --key-from HOST -f`

### curl|bash alternative

Works if `/dev/tty` is available (normal interactive SSH/terminal):

```bash
curl -fsSL https://raw.githubusercontent.com/the-hcma/init-files/main/bootstrap_host \
  | bash -s -- --key-from HOST
# after verify OK:
source ~/.bashrc
```

Downloading to `/tmp/bootstrap_host` is still preferred when debugging.

### What `--key-from` does

Fetches `~/.ssh/id_rsa-sha2-256-hcma-at-hcma-dot-info` (+ `.pub` if present) from `HOST` in one SSH session, starts `ssh-agent` if needed, caches the key, verifies GitHub SSH, clones/installs init-files under `~/.local/share/init-files`, and prints the verify summary.

Requires OpenSSH client tools (`ssh`, `ssh-add`, `ssh-agent`, `scp`). If missing, the script prints a distro install command.

### Flags

| Flag | Meaning |
| --- | --- |
| `--key-from HOST` | fetch house key from HOST (or `INIT_FILES_KEY_HOST`) |
| `--no-dev` / `--dev` | forwarded to install |
| `--github-https` / `--github-ssh` | GitHub transport (optional; default is SSH via the house key) |
| `-f` / `--force` | install force; also re-fetch key if `--key-from` is set |
| `-q` / `--quiet` | less output |

```bash
/tmp/bootstrap_host --help
```

### GitHub "Permission denied (publickey)"

The house key is on disk and in SSH_AUTH_SOCK=/Users/hcma/.ssh/agent/s.Jyiqvffivu.agent.xIhUB9mtDH; export SSH_AUTH_SOCK;
SSH_AGENT_PID=90974; export SSH_AGENT_PID;
echo Agent pid 90974;, but GitHub rejected it. On the new host run:



Typical causes:

1. **Old OpenSSH** (common on Rocky/CentOS) — GitHub requires RSA SHA-2. Upgrade:  (or ), then retry .
2. **Pubkey not on GitHub** — add  to the GitHub account that can read init-files (fingerprint must match).

Re-download bootstrap after fixes:

Mode:    HTTPS GitHub on saratoga
OK       no https→ssh insteadOf (HTTPS mode)
OK       gh authenticated (HTTPS git)
OK       clone already present: /Users/hcma/.local/share/init-files
Running  /Users/hcma/.local/share/init-files/install --github-https
GitHub: HTTPS (remembered for host saratoga in /Users/hcma/.config/init-files/github-https.saratoga)
Checking tools for init-files bashrc (host saratoga)...
Platform: modern macOS (Homebrew paths enabled)

  OK       bash -> /opt/homebrew/bin/bash
  OK       git -> /opt/homebrew/bin/git
  OK       gpg -> /opt/homebrew/opt/gnupg/bin/gpg
  OK       vim -> /opt/homebrew/bin/vim
  OK       curl -> /usr/bin/curl
  OK       python3 -> /opt/homebrew/bin/python3
  OK       ssh -> /usr/bin/ssh
  OK       ssh-add -> /usr/bin/ssh-add
  OK       ssh-agent -> /usr/bin/ssh-agent
  OK       ssh-keygen -> /usr/bin/ssh-keygen
  OK       cmp -> /usr/bin/cmp
  OK       make -> /usr/bin/make
  OK       rsync -> /usr/bin/rsync
  OK       colordiff -> /opt/homebrew/bin/colordiff
  missing  astyle (optional) — brew install astyle
  OK       patch -> /usr/bin/patch
  missing  gdb (optional) — brew install gdb
  OK       ls -> /opt/homebrew/opt/coreutils/libexec/gnubin/ls
  OK       grep -> /opt/homebrew/opt/grep/libexec/gnubin/grep
  OK       rm -> /opt/homebrew/opt/coreutils/libexec/gnubin/rm
  OK       ln -> /opt/homebrew/opt/coreutils/libexec/gnubin/ln
  OK       bc -> /usr/bin/bc
  OK       fzf -> /opt/homebrew/bin/fzf
  OK       clear -> /usr/bin/clear
  OK       less -> /usr/bin/less
  OK       more -> /usr/bin/more
  missing  emacs (optional) — brew install emacs
  OK       mvim -> /opt/homebrew/bin/mvim
  OK       scutil -> /usr/sbin/scutil
  OK       osascript -> /usr/bin/osascript
  OK       open -> /usr/bin/open
  OK       system_profiler -> /usr/sbin/system_profiler
  OK       killall -> /usr/bin/killall
  OK       launchctl -> /bin/launchctl
  OK       vlc -> /Applications/VLC.app/Contents/MacOS/VLC
  OK       corepack -> /opt/homebrew/opt/corepack/bin/corepack


Optional tools missing — install if you want the related aliases/features:
  astyle          install: brew install astyle
  gdb             install: brew install gdb
  emacs           install: brew install emacs
Wrote tool paths -> /Users/hcma/.config/init-files/tools.saratoga
OK       ~/.ssh/authorized_keys (house keys already present)
OK       /Users/hcma/.ssh/config.d/init-files-house.conf
OK       /Users/hcma/.ssh/config.d/init-files-github.conf
OK       no https→ssh insteadOf (HTTPS preferred on saratoga)
OK       GitHub HTTPS via gh auth git-credential
OK       GitHub HTTPS mode (SSH key optional for git on saratoga)
OK       ~/.bashrc -> /Users/hcma/.local/share/init-files/bashrc
Run: source ~/.bashrc
OK       ~/.vimrc -> /Users/hcma/.local/share/init-files/vim/vimrc
OK       no ~/.gvimrc (single vimrc model)
OK       vim-plug (/Users/hcma/.vim/autoload/plug.vim)
OK       vim plugins (PlugInstall)
Note: Nerd Font for Vim glyphs: brew install --cask font-meslo-lg-nerd-font
OK       /Users/hcma/.bash_profile already resolves to init-files bashrc
Note: this shell still needs: source ~/.bashrc   (new login shells pick it up automatically)

=== bootstrap_host verify (saratoga) ===
bashrc:   /Users/hcma/.bashrc -> /Users/hcma/.local/share/init-files/bashrc
          OK (symlink into clone)
HEAD:     53bf530
tools:    /Users/hcma/.config/init-files/tools.saratoga
GitHub:   HTTPS mode
          gh auth: OK

Next: source ~/.bashrc   (or open a new shell)
Optional: check_tool_versions

## Ownership

CODEOWNERS: `@thehcma`.
