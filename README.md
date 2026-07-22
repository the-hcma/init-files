# init-files (bootstrap)

Entrypoint for bootstrapping house shell config on a new machine.

Private config lives in [`thehcma/init-files`](https://github.com/thehcma/init-files). This public repo only mirrors `bootstrap_host` so a fresh machine can download it without already having the clone.

## Exact steps (new host)

You need SSH access to a donor host that already has the house key (password login is fine once).

```bash
# 1) Download the script (preferred over curl|bash — clearer errors, no stale pipe)
curl -fsSL https://raw.githubusercontent.com/the-hcma/init-files/2f67b348749158f1db2a57f885602939475c3f0c/bootstrap_host \
  -o /tmp/bootstrap_host
chmod +x /tmp/bootstrap_host

# 2) Optional: inspect
less /tmp/bootstrap_host

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
```

HOST may be a short name, FQDN, or user@host. Env alternative: `INIT_FILES_KEY_HOST=meerkat`.

### Key already on this host

If a previous attempt already copied the key (common after a partial run):

```bash
curl -fsSL https://raw.githubusercontent.com/the-hcma/init-files/2f67b348749158f1db2a57f885602939475c3f0c/bootstrap_host \
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
curl -fsSL https://raw.githubusercontent.com/the-hcma/init-files/2f67b348749158f1db2a57f885602939475c3f0c/bootstrap_host \
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
| `--github-https` / `--github-ssh` | GitHub transport (optional; default is SSH via the house key) |
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

The house key can be on disk and in ssh-agent while GitHub still rejects it. On the new host run:

```bash
ssh -V
ssh -v -o User=git -o IdentitiesOnly=yes \
  -i ~/.ssh/id_rsa-sha2-256-hcma-at-hcma-dot-info -T git@github.com
```

Typical causes:

1. **Old OpenSSH** (common on Rocky/CentOS) — GitHub requires RSA SHA-2 signatures. Prefer `~/.ssh/id_ed25519_github` when present (`cache_ssh ~/.ssh/id_ed25519_github`), or upgrade `openssh-clients` / `openssh-client`, then retry `/tmp/bootstrap_host`.
2. **Pubkey not on GitHub** — the matching `.pub` must be on the GitHub account that can read init-files (Settings → SSH keys). Compare fingerprints with: `ssh-keygen -lf ~/.ssh/id_rsa-sha2-256-hcma-at-hcma-dot-info`

Re-download bootstrap after upgrading OpenSSH or fixing the pubkey:

```bash
curl -fsSL https://raw.githubusercontent.com/the-hcma/init-files/2f67b348749158f1db2a57f885602939475c3f0c/bootstrap_host \
  -o /tmp/bootstrap_host
chmod +x /tmp/bootstrap_host
/tmp/bootstrap_host
```

### CDN note

GitHub `raw.githubusercontent.com/.../main/...` can lag. Prefer the commit-pinned URL above, or:

```bash
gh api repos/the-hcma/init-files/contents/bootstrap_host --jq .content | base64 -d > /tmp/bootstrap_host
chmod +x /tmp/bootstrap_host
```

## Ownership

CODEOWNERS: @thehcma.
