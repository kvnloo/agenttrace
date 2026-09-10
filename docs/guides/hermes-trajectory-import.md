# Hermes Trajectory Import

agenttrace can parse Hermes Agent **trajectory** JSONL exports — ShareGPT-compatible training/debug trajectories — separately from the existing Hermes session importers (`hermes_json`, `hermes_jsonl`, `hermes_db`).

## Expected export path

This parser expects the trajectory JSONL written by Hermes when trajectory saving is enabled, **not** the default session backup from `hermes sessions export`.

| Source | Typical output | SourceTool |
| --- | --- | --- |
| Interactive / `AIAgent(..., save_trajectories=True)` or `run_agent.py --save_trajectories` | `trajectory_samples.jsonl` (completed) and `failed_trajectories.jsonl` in the process CWD | `hermes_trajectory` |
| Batch runner (`batch_runner.py`) | `data/<run_name>/trajectories.jsonl` and per-batch `batch_*.jsonl` | `hermes_trajectory` |
| `hermes sessions export backup.jsonl` (default JSONL) | session objects with `messages` + `session_id` | `hermes_json` (existing) |
| `~/.hermes/state.db` | SQLite sessions | `hermes_db` (existing) |

Official format reference: [Hermes Trajectory Format](https://hermes-agent.nousresearch.com/docs/developer-guide/trajectory-format).

## Shape

Each JSONL line is one trajectory object with a ShareGPT `conversations` array (`from` / `value` roles: `system`, `human`, `gpt`, `tool`). Optional enrichment fields mapped into agenttrace metrics:

- `usage` (or `metadata.usage`) → token counts (`input_tokens`, `output_tokens`, cache fields)
- `duration_seconds` / `latency_ms` (and aliases) → session `duration_sec` via synthesized timestamps around `timestamp`
- model pricing over reported tokens → `cost_estimated`

Tool calls embedded as `<tool_call>...</tool_call>` / `<tool_response>...</tool_response>` (and `<think>` reasoning) are normalized into agenttrace events.

## Example

```bash
agenttrace trajectory_samples.jsonl
agenttrace --overview -d /path/to/hermes/run
```

A synthetic sample lives at `testdata/generated/hermes-trajectory.jsonl`.
