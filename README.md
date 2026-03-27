# autoresearch — Claude Code Plugin

[Karpathy's autoresearch](https://github.com/karpathy/autoresearch) as a Claude Code plugin. Run autonomous fixed-budget experiments overnight using your Claude Max subscription — zero per-token billing.

## What it does

The autoresearch pattern: an AI agent iteratively edits one file, runs a fixed-budget experiment (typically 5 minutes), measures a single scalar metric, and keeps the change if it improved. Repeat forever. The agent runs 50-100 experiments overnight while you sleep.

This plugin adds:

| Command | What it does |
|---|---|
| `/autoresearch:setup` | One-time hardware configuration (local Mac, remote server, RunPod) |
| `/autoresearch:run` | Pre-flight checks + start the experiment loop |
| `/autoresearch:status` | View results.tsv and summarize progress |
| `/autoresearch:advisor` | Analyze any project for autoresearch opportunities |

The **advisor** works in any project — not just ML training. It identifies code performance, pipeline throughput, prompt engineering, build speed, and other optimization targets where the autoresearch pattern applies.

## Install

```bash
# Add the marketplace (if you haven't already)
claude plugin marketplace add flight505/flight505-marketplace

# Install the plugin
claude plugin install autoresearch@flight505-plugins
```

## Quick start

```bash
# 1. Configure your hardware targets (one-time)
/autoresearch:setup

# 2. cd into your autoresearch repo
cd ~/autoresearch

# 3. Start an experiment loop
/autoresearch:run

# 4. Check results anytime
/autoresearch:status
```

### Unattended overnight run

```bash
cd ~/autoresearch
claude --dangerously-skip-permissions \
  -p "/autoresearch:run overnight, aim for 50+ experiments"
```

The `--dangerously-skip-permissions` flag enables fully autonomous operation. Use **only** in the autoresearch repo.

## Hardware targets

The plugin supports three hardware targets. Configure them once with `/autoresearch:setup`.

### Apple Silicon (MLX)

For MacBook M-series. Uses MLX — no PyTorch or CUDA needed. Best for quick daytime iteration.

### NVIDIA CUDA (remote server)

For any NVIDIA GPU accessible via SSH. The recommended repo depends on your GPU:

| GPU | Recommended Repo |
|---|---|
| Consumer (RTX 20/30/40/50 series) | [flight505/autoresearch-blackwell](https://github.com/flight505/autoresearch-blackwell) |
| Datacenter (H100, A100) | [karpathy/autoresearch](https://github.com/karpathy/autoresearch) |

The **Blackwell fork** works on all consumer NVIDIA GPUs (Turing through Blackwell). Key features:
- PyTorch SDPA attention (no Flash Attention 3 dependency)
- `torch.compile` enabled by default (Linux + Triton)
- OOM cascade with automatic activation checkpointing
- `--smoke-test` flag for quick 10-second validation
- 28+ GPU FLOPS entries for accurate MFU reporting

### RunPod (cloud GPU)

No hardware? Rent GPUs on demand with [RunPod](https://runpod.io?ref=wjm4q5bw). Cloud provisioning is coming in a future update — for now, `/autoresearch:setup` stores your API key so you're ready when it launches.

## The autoresearch pattern — beyond ML

The advisor skill (`/autoresearch:advisor`) identifies optimization targets in any project. The pattern works wherever you have:

1. **A single file to edit** — the mutable artifact
2. **A scalar metric** — computable without human judgment
3. **A fixed time budget** — equal cost per experiment
4. **Automated execution** — no human in the loop

Examples: API latency, build duration, bundle size, query execution time, prompt accuracy, pipeline throughput, inference speed.

## Why Claude Max

Claude Code authenticates via OAuth with your Claude.ai account. With a Max plan, usage is billed against your subscription's included quota — not per-token API billing. An overnight run of 50-100 five-minute experiments is completely practical on the flat monthly fee.

## License

MIT
