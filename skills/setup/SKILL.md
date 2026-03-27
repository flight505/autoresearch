---
name: setup
description: "One-time configuration of autoresearch hardware targets. Guides user through setting up Apple Silicon (MLX), NVIDIA server (CUDA), and/or RunPod cloud GPU targets. Writes persistent config. Use when user runs /autoresearch:setup or when SessionStart hook reports no config."
user-invocable: false
---

# Autoresearch Setup

Guide the user through configuring their autoresearch hardware targets. This is typically a one-time process — the config persists across sessions and projects.

## Step 1: Check existing config

```bash
cat "${CLAUDE_PLUGIN_DATA:-${HOME}/.autoresearch}/config.json" 2>/dev/null || echo "NO_CONFIG"
```

If config exists, show the current targets and ask: "You already have autoresearch configured. Want to update it or start fresh?"

## Step 2: Auto-detect local hardware

```bash
# Check for MLX (Apple Silicon)
python3 -c "import mlx; print('MLX_AVAILABLE')" 2>/dev/null || echo "NO_MLX"

# System info
uname -ms 2>/dev/null
sysctl -n machdep.cpu.brand_string 2>/dev/null || echo ""
system_profiler SPHardwareDataType 2>/dev/null | grep -E "Chip|Memory" || echo ""
```

## Step 3: Configure each target

Ask the user about each target. Build a JSON config as you go.

### Local Mac (MLX)

If MLX was detected, ask:

> "I detected Apple Silicon with MLX. Where is your autoresearch repo?
> (Leave blank to skip this target)"

If the user provides a path, validate it:
```bash
[ -f "<path>/program.md" ] && [ -f "<path>/train.py" ] && echo "VALID" || echo "INVALID"
```

Ask for a description (e.g., "MacBook M4 Max, 64GB") or auto-detect from the system info above.

**Repo recommendation if they haven't cloned yet:**
- For Apple Silicon, any MLX port of autoresearch will work. Popular options include community forks adapted for MLX.

### Remote NVIDIA Server

Ask:

> "Do you have a remote NVIDIA GPU server accessible via SSH? (e.g., a home lab, cloud instance, or office workstation)"

If yes, ask for:
- SSH hostname (e.g., `ml-server`, `user@192.168.1.50`, `user@my-gpu-box.local`)
- Path to autoresearch repo on the server (e.g., `~/ml-projects/autoresearch`)
- GPU type (optional, for description — or detect via SSH)

Optionally verify connectivity:
```bash
ssh -o ConnectTimeout=3 "<ssh_host>" "nvidia-smi --query-gpu=name,memory.total --format=csv,noheader" 2>/dev/null
```

**Repo recommendations:**
- For consumer NVIDIA GPUs (RTX 20/30/40/50 series): recommend [flight505/autoresearch-blackwell](https://github.com/flight505/autoresearch-blackwell) — works on Turing through Blackwell, has torch.compile, OOM cascade, and `--smoke-test` flag
- For datacenter GPUs (H100, A100): recommend the upstream [karpathy/autoresearch](https://github.com/karpathy/autoresearch) which uses Flash Attention 3

### RunPod (Cloud GPU)

Ask:

> "Do you want to configure RunPod for cloud GPU access? This lets you run experiments on rented GPUs without owning hardware."

If yes:
- Ask for their RunPod API key
- If they don't have an account: "Sign up at https://runpod.io?ref=wjm4q5bw"
- Note: "RunPod pod provisioning will be available in a future update. For now, the API key is stored so you're ready when it launches."

Ask for preferred GPU type (default: "NVIDIA RTX 4090").

## Step 4: Write config

Assemble the JSON config object based on the user's answers:

```json
{
  "version": 1,
  "targets": {
    "local": {
      "enabled": true,
      "path": "<user's path>",
      "backend": "mlx",
      "description": "<user's description>"
    },
    "server": {
      "enabled": true,
      "ssh_host": "<user's ssh host>",
      "path": "<remote path>",
      "backend": "cuda",
      "gpu_type": "<detected or user-provided>",
      "description": "<description>"
    },
    "runpod": {
      "enabled": false,
      "api_key": "<user's key>",
      "gpu_type": "<preferred GPU>",
      "description": "RunPod cloud GPU"
    }
  }
}
```

Write it:
```bash
echo '<assembled JSON>' | bash "${CLAUDE_PLUGIN_ROOT}/scripts/write-config.sh"
```

## Step 5: Verify

Read back the config and display a clean summary:

```bash
cat "${CLAUDE_PLUGIN_DATA:-${HOME}/.autoresearch}/config.json"
```

Show something like:

```
Autoresearch configured:
  local:  MacBook M4 Max (MLX) @ /Users/you/autoresearch-mlx
  server: RTX 4090 (CUDA) @ gpu-server:~/autoresearch
  runpod: not configured

You can now use:
  /autoresearch:advisor  — analyze any project for autoresearch opportunities
  /autoresearch:run      — start an experiment loop
  /autoresearch:status   — check results
```
