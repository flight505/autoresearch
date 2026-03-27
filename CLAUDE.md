# autoresearch plugin — development notes

## Structure
- `skills/advisor/` — project analysis + hardware recommendation (user-invocable)
- `skills/setup/` — interactive config wizard (invoked by setup command)
- `commands/run.md` — pre-flight + experiment loop
- `commands/status.md` — results viewer
- `commands/setup.md` — thin wrapper for setup skill
- `hooks/session-start` — injects hardware config into session context
- `scripts/write-config.sh` — persists config.json

## Config
User config stored at `${CLAUDE_PLUGIN_DATA}/config.json` (fallback: `~/.autoresearch/config.json`).
Schema version 1. Contains `targets.local`, `targets.server`, `targets.runpod`.

## Testing
- `chmod +x hooks/session-start scripts/write-config.sh`
- SessionStart hook should output JSON with `hookSpecificOutput.additionalContext`
- Config file permissions should be 600 (owner-read-only)
