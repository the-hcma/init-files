# init-files (bootstrap)

Curl-friendly entrypoint for bootstrapping house shell config on a new machine.

## Quick start

You need SSH access to a host that already has the house key (password login is fine for this one step). Point `--key-from` at that host:

```bash
# Inspect first (recommended):
curl -fsSL https://raw.githubusercontent.com/the-hcma/init-files/main/bootstrap_host | less

# Fetch the house key from HOST, then finish bootstrap:
curl -fsSL https://raw.githubusercontent.com/the-hcma/init-files/main/bootstrap_host \
  | bash -s -- --key-from HOST
```

Examples:

```bash
curl -fsSL https://raw.githubusercontent.com/the-hcma/init-files/main/bootstrap_host \
  | bash -s -- --key-from meerkat

curl -fsSL https://raw.githubusercontent.com/the-hcma/init-files/main/bootstrap_host \
  | bash -s -- --key-from hcma@saratoga --no-dev
```

`HOST` may be a short name, FQDN, or `user@host`. Equivalent env: `INIT_FILES_KEY_HOST=meerkat`.

Then reload:

```bash
source ~/.bashrc
```

### What `--key-from` does

On this machine it runs the equivalent of:

```bash
mkdir -p ~/.ssh && chmod 700 ~/.ssh
scp -p HOST:.ssh/id_rsa-sha2-256-hcma-at-hcma-dot-info ~/.ssh/
scp -p HOST:.ssh/id_rsa-sha2-256-hcma-at-hcma-dot-info.pub ~/.ssh/   # if present
chmod 600 ~/.ssh/id_rsa-sha2-256-hcma-at-hcma-dot-info
```

Then it caches the key (`ssh-add`), verifies GitHub SSH, clones/installs init-files, and prints a short verify summary. If the key is already on this host, fetch is skipped unless you pass `-f` / `--force`.

### Key already on this host

```bash
curl -fsSL https://raw.githubusercontent.com/the-hcma/init-files/main/bootstrap_host | bash
```

### HTTPS / `gh` path

```bash
gh auth login
curl -fsSL https://raw.githubusercontent.com/the-hcma/init-files/main/bootstrap_host \
  | bash -s -- --github-https
```

### Flags

| Flag | Meaning |
| --- | --- |
| `--key-from HOST` | scp house key from HOST (or `INIT_FILES_KEY_HOST`) |
| `--no-dev` / `--dev` | forwarded to install |
| `--github-https` / `--github-ssh` | GitHub transport |
| `-f` / `--force` | install force; also re-fetch key if `--key-from` is set |
| `-q` / `--quiet` | less output |

```bash
curl -fsSL https://raw.githubusercontent.com/the-hcma/init-files/main/bootstrap_host | bash -s -- --help
```

## What this repo is

Only the public bootstrap surface (`bootstrap_host` + this README). No private keys are stored here.

## Ownership

CODEOWNERS: `@thehcma`.
