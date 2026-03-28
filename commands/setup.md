---
description: "Configure autoresearch hardware targets — Apple Silicon (MLX), NVIDIA server (CUDA), or RunPod cloud GPU. One-time setup stored persistently."
argument-hint: ""
allowed-tools: ["Bash", "Read", "Write", "AskUserQuestion"]
---

# Autoresearch Setup

Guide the user through configuring their autoresearch hardware targets. This is a one-time process — the config persists across sessions and projects.

## Step 1: Check existing config

```bash
cat "${CLAUDE_PLUGIN_DATA:-${HOME}/.autoresearch}/config.json" 2>/dev/null || echo "NO_CONFIG"
```

If config exists, show the current targets and ask if the user wants to update or start fresh.

## Step 2: Auto-detect local hardware

```bash
python3 -c "import mlx; print('MLX_AVAILABLE')" 2>/dev/null || echo "NO_MLX"
uname -ms 2>/dev/null
sysctl -n machdep.cpu.brand_string 2>/dev/null || echo ""
system_profiler SPHardwareDataType 2>/dev/null | grep -E "Chip|Memory" || echo ""
```

## Step 3: Configure each target

Ask the user about each target conversationally. Build the config JSON as you go.

### Local Mac (MLX)

If Apple Silicon detected (even without MLX installed — it may be in a project venv), ask for the path to the autoresearch repo. Validate:
```bash
[ -f "<path>/program.md" ] && [ -f "<path>/train.py" ] && echo "VALID" || echo "INVALID"
```

Auto-detect description from system info. Repo recommendation: any MLX port of autoresearch.

### Remote NVIDIA Server

Ask if they have an SSH-accessible NVIDIA GPU server. If yes, collect:
- SSH hostname (e.g., `ml-server`, `user@192.168.1.50`)
- Remote path to autoresearch repo (e.g., `~/ml-projects/autoresearch`)

Verify connectivity AND identify which fork is installed:
```bash
ssh -o ConnectTimeout=3 "<ssh_host>" "nvidia-smi --query-gpu=name,memory.total --format=csv,noheader" 2>/dev/null
ssh -o ConnectTimeout=3 "<ssh_host>" "cd <path> && git remote get-url origin 2>/dev/null && head -5 train.py" 2>/dev/null
```

The git remote URL tells you which fork: `autoresearch-blackwell` (consumer optimized), `autoresearch-win-rtx`, or upstream `karpathy/autoresearch`. Include this in the config description.

Repo recommendations:
- Consumer NVIDIA (RTX 20/30/40/50): [flight505/autoresearch-blackwell](https://github.com/flight505/autoresearch-blackwell) — torch.compile, OOM cascade, --smoke-test
- Datacenter (H100, A100): upstream [karpathy/autoresearch](https://github.com/karpathy/autoresearch) — Flash Attention 3

### RunPod (Cloud GPU)

Ask if the user wants RunPod cloud GPU access. If yes:
- Collect API key. If no account: "Sign up at https://runpod.io?ref=wjm4q5bw"
- Note: pod provisioning coming in a future update; API key stored for readiness.
- Default GPU type: "NVIDIA RTX 4090"

## Step 4: Write config

Assemble the JSON and write it:

```bash
cat << 'CONFIGEOF' | bash "${CLAUDE_PLUGIN_ROOT}/scripts/write-config.sh"
{
  "version": 1,
  "targets": {
    "local": {
      "enabled": true/false,
      "path": "<path>",
      "backend": "mlx",
      "description": "<auto-detected>"
    },
    "server": {
      "enabled": true/false,
      "ssh_host": "<hostname>",
      "path": "<remote path>",
      "backend": "cuda",
      "gpu_type": "<detected>",
      "repo": "<git remote URL>",
      "description": "<GPU name + fork name>"
    },
    "runpod": {
      "enabled": true/false,
      "api_key": "<key>",
      "gpu_type": "<preferred>",
      "description": "RunPod cloud GPU"
    }
  }
}
CONFIGEOF
```

## Step 5: Verify

Read back and display a clean summary:
```bash
cat "${CLAUDE_PLUGIN_DATA:-${HOME}/.autoresearch}/config.json"
```

Then show available commands:
- `/autoresearch:advisor` — analyze any project for autoresearch opportunities
- `/autoresearch:run` — start an experiment loop
- `/autoresearch:status` — check results

$ARGUMENTS
