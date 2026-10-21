# force-ca-cert-upgrade

Forces an upgrade of the `ca-certificates` package and verifies that hosts
no longer trust the compromised eMudhra (emSign) intermediate CAs disclosed
in October 2026.

Background: https://emudhra.we-have-a-problem.com/

## Requirements

- Debian/Ubuntu (apt-based) hosts – other OS families are skipped
- Outbound HTTPS from the targets for the verification step

## Caveats

To my knowledge RHEL has not released an updated package

## What it does

1. Upgrades `ca-certificates` if already installed (`state: latest`, `only_upgrade: true`)
2. Runs `curl` against the eMudhra test site – the task **fails** if the
   connection still succeeds, i.e. the host still trusts the compromised chain

## Usage

Test on a single host first:

```bash
ansible-playbook -i inventory.ini ca-update.yml --limit your_test_host
```

Then roll out to everything (raise forks for larger inventories):

```bash
ansible-playbook -i inventory.ini ca-update.yml -f 20
```

Add `-K` if `become` needs a sudo password.

Hosts listed as `failed` in the recap still accept the compromised chain.

## Not covered

This only fixes the system trust store. Check separately:

- Container images (rebuild with updated `ca-certificates`)
- Python venvs using `certifi` (≥ 2026.10.19)
- Go binaries (rebuild with a patched Go release)
- PHP apps bundling `composer/ca-bundle` (≥ 1.6.0 / 1.4.16)
