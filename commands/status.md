---
description: "View autoresearch experiment results — shows results.tsv, keep/discard summary, and best metric. Supports local and remote server targets."
argument-hint: "[local|server]"
---

# Autoresearch Status

Display the current experiment log and summarize progress.

## 1. Determine target

Read config:
```bash
cat "${CLAUDE_PLUGIN_DATA:-${HOME}/.autoresearch}/config.json" 2>/dev/null || echo "NO_CONFIG"
```

If `$ARGUMENTS` specifies `server`, use the configured SSH host and remote path.
Otherwise, read from the current local directory.

## 2. Read results

### Local:

```bash
echo "=== Branch ===" && git branch --show-current 2>/dev/null || echo "(not in a git repo)"
echo ""
echo "=== Results ===" && cat results.tsv 2>/dev/null || echo "(no results.tsv yet)"
echo ""
echo "=== Summary ==="
grep -v "^commit" results.tsv 2>/dev/null | awk -F'\t' '{
  total++
  if ($4=="keep") keep++
  if ($4=="discard") discard++
  if ($4=="crash") crash++
  if ($4=="keep" && (best=="" || $2+0<best+0)) best=$2
} END {
  print "Total runs:   " total+0
  print "Kept:         " keep+0
  print "Discarded:    " discard+0
  print "Crashed:      " crash+0
  print "Best val_bpb: " (best ? best : "n/a")
}'
```

### Server (via SSH):

Use the `ssh_host` and `path` from config:
```bash
ssh <ssh_host> "cd <path> && git branch --show-current && echo '' && cat results.tsv 2>/dev/null" || echo "Cannot reach server"
```

## 3. Summarize

After showing the raw data, provide a brief natural language summary:
- What ideas have been tried (from the description column)
- What worked and what didn't
- What directions look promising to try next

$ARGUMENTS
