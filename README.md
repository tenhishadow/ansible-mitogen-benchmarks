# ansible-mitogen-benchmarks

Reproducible benchmark harness to measure Ansible runtime overhead and the speedup from Mitogen.

This repo is intentionally small and boring: pinned dependencies, a single benchmark playbook, and a single command to run A/B tests.

## What it does

- Runs the same playbook twice using `hyperfine`
  - baseline: plain `ansible-playbook`
  - Mitogen: `ANSIBLE_STRATEGY=serverscom.mitogen.mitogen_linear`
- Targets `localhost` by default, so anyone can reproduce the results
- Can target remote hosts if you replace the inventory

## How Mitogen works (short version)

Classic Ansible pays per-task overhead: staging modules, starting Python, imports, and process churn.

Mitogen keeps persistent Python interpreters and drives modules via RPC over SSH, which removes much of that overhead.

## Requirements

- `uv`
- `hyperfine`
- `go-task` (Task)
- `ansible-core` is installed via `uv` from `pyproject.toml`

## Quickstart

```bash
task deps
task galaxy-deps
task bench
```

The benchmark command is:

```bash
hyperfine --warmup 1 --runs 3 \
  'uv run ansible-playbook playbooks/bench.yml' \
  'ANSIBLE_STRATEGY=serverscom.mitogen.mitogen_linear uv run ansible-playbook playbooks/bench.yml'
```

## Inventory

Default inventory runs locally (no SSH).

To benchmark remote hosts, change the inventory used by the playbook (or update the inventory file in this repo) and rerun `task bench`.

## Results and blog post

Full write-up with diagrams, methodology, and real numbers:

[Post](https://tenhishadow.github.io/posts/2026-01-27-ansible-warp-drive-mitogen/)

## Files to read first

- `Taskfile.yml`
- `ansible.cfg`
- `playbooks/bench.yml`
- `requirements.yml`
- `.pre-commit-config.yaml`

## Notes

This repo is a benchmark harness, not a Mitogen support project.
If you hit a Mitogen bug, upstream issue tracker is the right place.

