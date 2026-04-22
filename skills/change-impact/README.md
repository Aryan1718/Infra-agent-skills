# Change Impact Skill

This skill helps an agent analyze recent code changes and explain what other
parts of the system may be affected. It is designed for impact reviews before
committing, opening a PR, or merging.

## What It Does

- checks uncommitted changes, staged changes, commits, or branch diffs
- summarizes what changed in plain language
- traces references to changed symbols and contracts
- separates confirmed impact from likely impact and risk signals
- recommends practical follow-up checks without overclaiming certainty

## How To Use

Use this skill when the task is to understand the blast radius of recent code
changes.

Examples:

- "change-impact"
- "What will this affect?"
- "What might break after this change?"
- "What depends on this file?"
- "Analyze the impact of my changes"

Open [SKILL.md](./SKILL.md) and follow the impact analysis workflow.
