# init-files (public bootstrap)

Public, curl-friendly entrypoint for bootstrapping the **private** [`thehcma/init-files`](https://github.com/thehcma/init-files) shell config.

This repository intentionally contains only the bootstrap surface (`bootstrap_host` + docs). The private repo remains the source of truth for `bashrc`, `install`, SSH house materials, and host tooling.

## Quick start (new host)

**1. Put the house/GitHub SSH key on the host** (passphrase-protected; never committed here):

```bash
# From a working machine:
scp ~/.ssh/id_rsa-sha2-256-hcma-at-hcma-dot-info{,.pub} NEWHOST:~/.ssh/
```

On the new host:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_rsa-sha2-256-hcma-at-hcma-dot-info
chmod 644 ~/.ssh/id_rsa-sha2-256-hcma-at-hcma-dot-info.pub
```

**2. Inspect, then run** (prefer inspect before pipe-to-shell):

```bash
curl -fsSL https://raw.githubusercontent.com/the-hcma/init-files/main/bootstrap_host | less
curl -fsSL https://raw.githubusercontent.com/the-hcma/init-files/main/bootstrap_host | bash
```

Flags (same as private `bootstrap_host` / `install`):

```bash
curl -fsSL https://raw.githubusercontent.com/the-hcma/init-files/main/bootstrap_host | bash -s -- --no-dev
curl -fsSL https://raw.githubusercontent.com/the-hcma/init-files/main/bootstrap_host | bash -s -- --github-https
```

**3. Reload:**

```bash
source ~/.bashrc
```

`bootstrap_host` sets up GitHub SSH (or HTTPS), clones `thehcma/init-files` into `~/.local/share/init-files`, runs `./install`, and prints a short verify summary.

## HTTPS / `gh` path

If this host uses GitHub HTTPS via `gh auth` instead of the house SSH key:

```bash
gh auth login
curl -fsSL https://raw.githubusercontent.com/the-hcma/init-files/main/bootstrap_host | bash -s -- --github-https
```

## What this repo is not

- Not a mirror of the private bashrc/install tree
- Not a place for private keys, `authorized_keys`, or host `tools` files

## Syncing `bootstrap_host`

After changing `bootstrap_host` in the private repo, copy it here (PR to `main`) so the public raw URL stays current.

## License / ownership

Maintained for house use. CODEOWNERS: `@thehcma`.
