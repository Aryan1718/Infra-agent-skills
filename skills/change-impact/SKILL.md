---
name: change-impact
description: 'Use this skill when the user asks what recent code changes will affect, what might break, what depends on a modified file, or wants an impact review before committing, opening a PR, or merging. Triggers on: "change-impact", "what will this affect", "what might break", "what depends on this", "analyze the impact of my changes", "did I miss anything after changing this", or similar post-change impact analysis requests. Focus on uncommitted changes, staged changes, the last commit, or a branch diff, then identify confirmed impact, likely impact, risk signals, and practical follow-up checks using read-only repository analysis.'
---

# Change Impact

A post-change analysis skill that explains, in plain language, what the user's
recent code changes are likely to affect.

This skill is useful after editing code, before committing, before opening a
PR, or when the user wants to understand the blast radius of a change.

The goal is not just to list changed files. The goal is to answer:

- What did I change?
- What else depends on it?
- What could break because of it?
- What follow-up files or tests may also need changes?

## Use This Skill When

Use this skill when the user:

- asks what their recent changes impact
- asks what might break because of a change
- wants to understand dependency impact before a PR
- changed a core module, API, schema, config, auth flow, or shared utility
- wants a quick risk review after coding
- asks whether a change is isolated or system-wide

Example trigger phrases:

- `change-impact`
- `what will this affect`
- `what might break`
- `what depends on this`
- `analyze the impact of my changes`
- `did I miss anything after changing this`

## Core Principles

- Focus on recent changes first.
- Prefer evidence from the codebase over guesses.
- Explain impact in plain language.
- Distinguish confirmed impact from likely impact.
- Call out uncertainty clearly.
- Do not overwhelm the user with a huge graph unless requested.
- Prioritize actionable findings over exhaustive listing.

## What This Skill Does

This skill analyzes:

1. What files changed
2. What symbols or logic changed
3. Where those symbols are referenced
4. Which modules, routes, jobs, tests, configs, or database layers depend on them
5. What follow-up changes may be required

Then it produces a simple impact report such as:

- changed component
- affected areas
- likely risks
- recommended follow-up checks

## Read-Only Checks Allowed Without Confirmation

These checks are safe because they do not modify the repository:

```bash
git status --short
git diff --name-only
git diff --stat
git diff
git log --oneline --decorate -n 10
rg <pattern> .
grep -R <pattern> .
```

The assistant may also inspect relevant files to understand imports, function
calls, route usage, schema references, test coverage, and shared utilities.

## Step 1 - Determine Scope of Change

First determine what to analyze.

Preferred order:

### A. Current uncommitted and staged changes
Use when the user just made changes locally.

Useful commands:

```bash
git status --short
git diff --name-only
git diff --stat
```

### B. Recent commit or commit range
Use when the user asks about a specific commit or recent work.

Useful commands:

```bash
git log --oneline --decorate -n 10
git show --stat <commit>
git diff --name-only <base>..<head>
```

If the scope is unclear, ask one short question to determine whether the user
wants analysis for:
- current uncommitted changes
- last commit
- current branch diff
- specific files

## Step 2 - Identify What Actually Changed

For each changed file, determine the kind of change.

Classify changes into categories such as:

- API or route changes
- schema or model changes
- auth or permission changes
- shared utility changes
- UI component changes
- environment or config changes
- background job or worker changes
- external API integration changes
- test-only changes

Summarize in plain language, for example:

- Added a new parameter to the auth helper
- Changed request validation in the login route
- Renamed a shared utility function
- Updated database model fields
- Modified response shape for an endpoint

## Step 3 - Trace Dependencies

For each meaningful change, inspect where it is used.

Look for:

- imports
- function calls
- route registrations
- model or schema references
- background job usage
- config lookups
- test references
- frontend or backend consumers of an API contract

Use code search and local file inspection to trace direct dependencies first,
then nearby indirect dependencies when useful.

Examples:

- If an auth helper changed, inspect middleware, login handlers, protected routes
- If an API response changed, inspect clients, serializers, frontend consumers
- If a schema changed, inspect queries, writes, migrations, validators, tests
- If a shared utility changed, inspect all call sites
- If a config or env key changed, inspect all lookup locations and deployment docs

## Step 4 - Determine Impact Level

Classify findings into:

### Confirmed Impact
Code paths directly referencing the changed symbol, file, or contract.

### Likely Impact
Places that are probably affected based on architecture or naming, but not yet
confirmed directly.

### Risk Signals
Places that often break after this kind of change, such as:

- tests not updated
- validators not updated
- clients still using old response shape
- schema changed without migration
- shared utility changed without reviewing all call sites
- auth logic changed without reviewing protected endpoints

Use simple risk levels:

- Low
- Medium
- High

Do not inflate severity without evidence.

## Step 5 - Produce the Report

The report should be concise, practical, and easy to scan.

Recommended format:

### Change Summary
What changed in simple terms.

### Affected Areas
List the main impacted modules, routes, services, or files.

### Why They Are Affected
One line per affected area.

### Risk Level
Low, Medium, or High.

### Recommended Follow-Up
Practical next checks, such as:

- review these call sites
- update tests for these modules
- validate API response compatibility
- inspect auth coverage on these routes
- verify migration exists
- run targeted test suite

Example style:

```text
Change Summary
- Modified auth middleware to require a new token field

Affected Areas
- login route - token creation path changed
- protected API routes - middleware behavior changed
- auth tests - existing tests may not cover the new requirement

Risk Level
- High

Recommended Follow-Up
- check all middleware call sites
- review login and token refresh flows
- run auth and integration tests
```

## When to Ask a Clarifying Question

Ask only if needed. Good cases:

- the user did not specify whether to analyze working tree, commit, or branch
- the repo has no visible recent changes
- there are too many changes and the scope should be narrowed

Prefer one short question instead of multiple.

## Edge Cases

### No Changes Found
If there are no visible local changes, explain that clearly and ask whether the
user wants analysis for:
- last commit
- current branch compared to main
- a specific file or commit

### Large Change Set
If many files changed, prioritize:
- shared utilities
- auth
- schema
- API contracts
- config
- any file imported in many places

Then summarize rather than dumping everything.

### Rename or Move
If a file was renamed or a symbol was moved, check whether old references remain.

### Test-Only Changes
If only tests changed, say impact is probably limited unless fixtures, helpers,
or shared mocks changed.

## Safety Rules

- Never modify files as part of this skill
- Never claim something is definitely broken without evidence
- Separate direct references from likely downstream impact
- Prefer concise, useful reporting over exhaustive noise
- If uncertain, say so explicitly

## Minimal Success Criteria

This skill is successful if it can:

- identify the scope of recent changes
- summarize what changed in plain language
- trace key dependencies of important changed code
- explain what parts of the system are affected
- assign a reasonable impact level
- recommend concrete follow-up checks
