---
description: "Start an autoresearch experiment loop — pre-flight checks, reads program.md, enters autonomous keep-or-revert loop. Specify target: local, server, or auto-detect."
argument-hint: "[local|server] [options like 'overnight, aim for 50+ experiments']"
---

# Autoresearch Run

You are starting an autonomous research session using Karpathy's autoresearch framework.

## 1. Load config

Read the autoresearch config to find hardware targets:

```bash
cat "${CLAUDE_PLUGIN_DATA:-${HOME}/.autoresearch}/config.json" 2>/dev/null || echo "NO_CONFIG"
```

If `NO_CONFIG`: stop and tell the user to run `/autoresearch:setup` first.

## 2. Determine target

If `$ARGUMENTS` specifies a target (`local` or `server`), use that.
Otherwise, auto-detect: if the current directory is inside a configured local path, use `local`. If unsure, ask the user.

## 3. Pre-flight checks

### For `local` target:

```bash
# Confirm we're in the right repo
[ -f program.md ] && [ -f train.py ] && [ -f prepare.py ] && echo "repo OK" || echo "FAIL: not in autoresearch repo"

# Detect backend
python3 -c "import mlx; print('backend: mlx')" 2>/dev/null || \
  python3 -c "import torch; print('backend: cuda (' + str(torch.cuda.get_device_name(0)) + ')')" 2>/dev/null || \
  echo "FAIL: no backend"

# Check data cache
ls ~/.cache/autoresearch/ 2>/dev/null && echo "data OK" || echo "FAIL: run uv run prepare.py first"
```

### For `server` target:

Read `ssh_host` and `path` from config, then:

```bash
# Test SSH connectivity (2s timeout)
ssh -o ConnectTimeout=2 <ssh_host> "echo 'connected'" 2>/dev/null || echo "FAIL: cannot reach server"

# Check repo on server
ssh <ssh_host> "[ -f <path>/program.md ] && [ -f <path>/train.py ] && echo 'repo OK' || echo 'FAIL: repo not found at <path>'"

# Check GPU
ssh <ssh_host> "nvidia-smi --query-gpu=name,memory.total --format=csv,noheader 2>/dev/null || echo 'FAIL: no GPU'"
```

If any check fails, report the issue and stop.

## 4. Read the protocol

Read `program.md` in the target directory. It is your authoritative research protocol.
Also read `train.py` and `prepare.py` to understand the training setup.

For server targets: read the files via SSH.

## 5. Set up the run

Propose a run tag based on today's date (e.g. `mar25`). Create the branch:

```bash
git checkout -b autoresearch/$(date +%b%d | tr '[:upper:]' '[:lower:]')
```

Initialize `results.tsv` with the header:
```
commit	val_bpb	memory_gb	status	description
```

## 6. Enter the LOOP FOREVER

Follow the experiment loop from `program.md` exactly:
- Establish baseline first (run train.py unmodified)
- Each experiment: modify train.py -> git commit -> run -> read results -> keep or revert
- Log every run to results.tsv
- **Never pause to ask if you should continue** — run until manually stopped

## Notes
- This session uses your Claude Max subscription via Claude Code OAuth — no API key billing
- Experiments are 5 min each; you can run ~12/hour overnight
- For server targets, run commands via SSH to the configured host

$ARGUMENTS
