# Skill: background-claude-cli

This skill enables background coding tasks using the Claude CLI, optimized for low-cost hours. It coordinates asynchronous prompts, cost-aware scheduling, and monitoring to keep development momentum while minimizing compute charges.

## Goals
- Run coding tasks with Claude in a background/work queue manner
- Prefer lower-cost Claude models during off-peak hours
- Return results to the main session when complete
- Log cost and progress for auditing

## Prerequisites
- Access to Claude CLI (claude) installed and authenticated
- Knowledge of cost model for Claude variants (e.g., Claude-3 vs Claude-4)
- A local workspace with code to work on (e.g., a git repository)
- OpenClaw workspace context loaded (this repo)

## How it works
1) A coding task is enqueued into a background queue with a target window (e.g., off-peak hours).
2) The background worker selects a Claude model based on cost policy (lower-cost model during designated hours).
3) The Claude CLI runs the task, producing code, tests, and reasoning (if configured to emit CoT).
4) Outputs are captured, validated with tests if available, and results are written back to memory or logs.
5) If successful, results are merged back into the main workspace; if failed, the task may be remediated or re-queued according to policy.

## Commands (examples)
- Start a background task with low-cost Claude:
  - claude-background run --task "Implement helper to parse config" --hours off-peak
- Check status:
  - claude-background status --task-id <id>
- Retrieve results:
  - claude-background fetch --task-id <id>
- Cancel a running task:
  - claude-background cancel --task-id <id>

> Note: Commands are placeholders. Please adapt to your environment’s actual Claude CLI and wrapper tooling. The goal is to have a reproducible, auditable background workflow.

## Flow with OpenClaw
- Use a dedicated subagent (background-claude) or an acp-based worker to run tasks in the background.
- Use the existing OpenClaw cost controls (if configured) to select a lower-cost model during the pre-defined hours.
- Log outputs to memory/background-claude.log and append to memory/heartbeat-state if relevant.

## Safety & Boundaries
- Do not expose secrets or credentials in logs.
- Ensure tasks do not perform privileged actions without explicit consent.
- Respect Iven’s preferences for concise, low-noise backgrounds; avoid noisy prompts in the main session.

## Deliverables
- A small, testable workflow that can enqueue a task and return results with a cost report.
- Documentation in this SKILL.md for maintainers to implement and operate the background-Claude workflow.
