# init-files (bootstrap)

Entrypoint for bootstrapping house shell config on a new machine.

Private config lives in [`thehcma/init-files`](https://github.com/thehcma/init-files). This public repo only mirrors `bootstrap_host` so a fresh machine can download it without already having the clone.

Mirrored from private `main` at `c1a4e14` (ed25519-only SSH hosts + IdentityFile filtering).

## Exact steps (new host)

### A) House key path (default)

You need SSH access to a donor host that already has the house key (password login is fine once).

```bash
# 1) Download the script (preferred over curl|bash — clearer errors, no stale pipe)
curl -fsSL https://raw.githubusercontent.com/the-hcma/init-files/main/bootstrap_host \
  -o /tmp/bootstrap_host
chmod +x /tmp/bootstrap_host

# 2) Optional: inspect
less /tmp/bootstrap_host
grep -n 'GitHub-only ed25519\|bootstrap_host_rev' /tmp/bootstrap_host

# 3) Run — replace HOST (e.g. meerkat, saratoga, hcma@meerkat)
/tmp/bootstrap_host --key-from HOST
# minimal host:  /tmp/bootstrap_host --key-from HOST --no-dev
```

Prompts (in order):

1. SSH password (or key) to HOST — once (fetches the house key)
2. Passphrase for the house key — once (ssh-add)

Wait until you see the **bootstrap_host verify** block with `bashrc: … OK`.
**Only then** reload the shell:

```bash
# 4) After verify succeeds:
source ~/.bashrc
```

`source ~/.bashrc` before a successful verify does nothing useful — there is no symlink yet.

### B) GitHub-only SSH (no house RSA on this host)

Use when this account should **not** hold `id_rsa-sha2-256-hcma-at-hcma-dot-info` (e.g. `house_meister` on a new box). Needs `id_ed25519_github` (registered on GitHub).

```bash
mkdir -p ~/.ssh && chmod 700 ~/.ssh
scp hcma@saratoga:.ssh/id_ed25519_github{,.pub} ~/.ssh/
chmod 600 ~/.ssh/id_ed25519_github

# optional: remove house RSA if it was copied earlier
rm -f ~/.ssh/id_rsa-sha2-256-hcma-at-hcma-dot-info \
      ~/.ssh/id_rsa-sha2-256-hcma-at-hcma-dot-info.pub

curl -fsSL https://raw.githubusercontent.com/the-hcma/init-files/main/bootstrap_host \
  -o /tmp/bootstrap_host
chmod +x /tmp/bootstrap_host
# Confirm new script (must mention GitHub-only ed25519):
grep -n 'GitHub-only ed25519' /tmp/bootstrap_host

ssh-add -t 4h ~/.ssh/id_ed25519_github
ssh -T git@github.com    # expect: Hi …!

/tmp/bootstrap_host --github-ssh
# optional: --no-dev

# after verify OK:
source ~/.bashrc
```

### Confirm it worked

```bash
ls -l ~/.bashrc
# expect: ~/.bashrc -> …/init-files/bashrc

ls -ld ~/.local/share/init-files/.git
git -C ~/.local/share/init-files rev-parse --short HEAD
type refresh_init_files
```

### Examples

```bash
/tmp/bootstrap_host --key-from meerkat
/tmp/bootstrap_host --key-from hcma@meerkat --no-dev
/tmp/bootstrap_host --key-from hcma@saratoga
/tmp/bootstrap_host --github-ssh          # ed25519 already on disk; no house RSA
```

HOST may be a short name, FQDN, or user@host. Env alternative: `INIT_FILES_KEY_HOST=meerkat`.

### Key already on this host

If a previous attempt already copied the key (common after a partial run):

```bash
curl -fsSL https://raw.githubusercontent.com/the-hcma/init-files/main/bootstrap_host \
  -o /tmp/bootstrap_host
chmod +x /tmp/bootstrap_host
/tmp/bootstrap_host
# or: /tmp/bootstrap_host --github-ssh
# then, after verify OK:
source ~/.bashrc
```

To re-fetch the house key from the donor anyway: `/tmp/bootstrap_host --key-from HOST -f`

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

Fetches `~/.ssh/id_rsa-sha2-256-hcma-at-hcma-dot-info` (+ `.pub` if present) from HOST in one SSH session, starts ssh-agent if needed, caches the key, verifies GitHub SSH, clones under `~/.local/share/init-files`, runs `provision_init_files`, and prints the verify summary.

Requires OpenSSH client tools (`ssh`, `ssh-add`, `ssh-agent`, `scp`). If missing, the script prints a distro install command.

### Flags

| Flag | Meaning |
| --- | --- |
| `--key-from HOST` | fetch house key from HOST (or `INIT_FILES_KEY_HOST`) |
| `--no-dev` / `--dev` | forwarded to `provision_init_files` |
| `--github-https` / `--github-ssh` | GitHub transport (SSH is default; `--github-ssh` forces SSH + works with ed25519-only) |
| `-f` / `--force` | provision force; also re-fetch key if `--key-from` is set |
| `-q` / `--quiet` | less output |

```bash
/tmp/bootstrap_host --help
```

After bootstrap, day-to-day updates:

```bash
refresh_init_files              # pull main + always provision
refresh_init_files --no-dev     # switch this host to non-dev
~/.local/share/init-files/provision_init_files   # tools/ssh/vim only
```

### GitHub Permission denied (publickey)

Typical causes:

1. **Missing `IdentityFile` listed in config** — bootstrap/provision now omit keys that are not on disk. Re-download bootstrap and re-run, or remove missing `IdentityFile` lines from `~/.ssh/config.d/init-files-github.conf`.
2. **Old OpenSSH** (common on Rocky/CentOS) — GitHub may decline house RSA. Prefer `~/.ssh/id_ed25519_github` (`ssh-add` / `cache_ssh`), then `/tmp/bootstrap_host --github-ssh`.
3. **Pubkey not on GitHub** — the matching `.pub` must be on the GitHub account that can read init-files.

```bash
ssh -V
ssh -v -o User=git -o IdentitiesOnly=yes \
  -i ~/.ssh/id_ed25519_github -T git@github.com
# or house RSA:
ssh -v -o User=git -o IdentitiesOnly=yes \
  -i ~/.ssh/id_rsa-sha2-256-hcma-at-hcma-dot-info -T git@github.com
```

Re-download bootstrap after fixing keys:

```bash
curl -fsSL https://raw.githubusercontent.com/the-hcma/init-files/main/bootstrap_host \
  -o /tmp/bootstrap_host
chmod +x /tmp/bootstrap_host
/tmp/bootstrap_host --github-ssh
```

### CDN note

GitHub `raw.githubusercontent.com/.../main/...` can lag a minute. If you still see the old “missing house/GitHub SSH key” error, wait and re-curl, or:

```bash
gh api repos/the-hcma/init-files/contents/bootstrap_host --jq .content | base64 -d > /tmp/bootstrap_host
chmod +x /tmp/bootstrap_host
```

## Ownership

CODEOWNERS: @thehcma.
