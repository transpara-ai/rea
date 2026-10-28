---
doc_id: REA-SETUP-WORKFLOW
title: REA setup and readiness workflow
doc_type: operations_runbook
status: draft
version: 0.1.0
created: 2026-10-28
updated: 2026-10-28
owner: Michael Saucier
steward: Codex/OpenAI
project: REA
supersedes: []
canonical: false
---

# REA setup and readiness workflow

Use this workflow to connect REA to the agents you use and prove that the
analysis needed for a task works. The authoritative setup details and supported
hosts are in the [installation manual](installation.md); this page gives the
order of work and the evidence to keep.

## 1. Choose the required capabilities

Write down the agent clients and analysis workflows needed for the task. Static
JavaScript analysis does not require a native engine. Native decompilation needs
an available provider such as Ghidra, Hopper or IDA. Some other formats need
their own caller-supplied tools. An audit-wide `rea doctor` checks optional
engines too, so its `healthy: false` is not by itself a failure of the selected
workflow. Use explicit doctor scope for the decision.

Check the installed REA version and read-only host audit:

```bash
rea --version
rea doctor --json
```

Record the CLI version, detected clients, installed skill identity, provider
findings and any settings that REA says it cannot parse. Fix a malformed
selected configuration before setup; setup preserves unrelated entries and
refuses unsafe or ambiguous ownership. On Linux, confirm a procps-compatible
`ps` is on `PATH` before running supervised analysis.

## 2. Prepare one native engine when needed

For Ghidra, install a supported release and its declared full JDK outside REA.
Compare the archive's SHA-256 with the [official release](https://github.com/NationalSecurityAgency/ghidra/releases) asset digest before
extracting it. REA setup does not install Ghidra or Java. Supply absolute
installation paths in the shell that will run setup, and prove the provider is
admitted before editing any agent registration:

```bash
export GHIDRA_INSTALL_DIR=/absolute/path/to/ghidra_12.1.4_PUBLIC
export JAVA_HOME=/absolute/path/to/jdk-21
rea doctor --provider ghidra --json
```

The paths are examples; the selected Ghidra release determines the supported
JDK range. See [Ghidra setup](installation.md#ghidra) for host and version
requirements. A Ghidra launcher on `PATH` can support direct CLI discovery,
but set `GHIDRA_INSTALL_DIR` explicitly while planning setup so agent
registrations retain the verified engine path.

If Hopper is the chosen engine, review its [license, demo limits and installer
plan](installation.md#hopper) separately. Approved setup can install Hopper on
supported hosts; it requires its own `--install-hopper` choice with `--yes` in
unattended use. IDA, JADX and Kuna require the caller-supplied components and
configuration described in their provider guides. Select providers for the
formats the task needs; other missing engines remain informational.

## 3. Preview and apply agent registration

Select exact clients with repeatable `--client` flags, or use
`--all-detected` only when every detected client is intended. Keep the provider
environment from step 2 in the same shell for planning, application and direct
CLI verification. For example:

```bash
rea setup --client codex --client claude_code --client pi --dry-run --json
```

Read `plannedActions`: confirm each destination, retained environment override,
backup path, skill destination and external command. Then apply that same scope
with `--yes --json`, or run it interactively and approve its displayed plan.
For this example, the unattended application command is:

```bash
rea setup --client codex --client claude_code --client pi --yes --json
```

Setup makes backups and checks for intervening edits; follow a reported conflict
instead of overwriting a newer client configuration. See [review setup
changes](installation.md#review-setup-changes) for recovery rules.

## 4. Verify the path the user will run

Restart every configured agent. A registration check does not prove that a
client has reloaded its configuration. Check the exact selected clients,
provider and installed skill:

```bash
rea doctor --client codex --client claude_code --client pi \
  --provider ghidra --skill --json
rea mcp doctor --json
```

For the direct CLI, retain the exported paths from step 2 or use a recognized
Ghidra launcher on `PATH`. Run a small authorized native sample through the
real provider, then inspect the returned Evidence for that provider. For
example:

```bash
rea analyze /absolute/path/to/sample --provider ghidra --json
```

In a restarted agent, call `binary_session` with no arguments and check its
server identity and provider availability. `rea mcp doctor` proves a fresh
server can start and advertise the expected tool catalog; it does not prove
each configured desktop agent restarted or that every provider operation ran.

The workflow is complete when the selected scoped doctor report is healthy,
the chosen provider has passed a real task-sized analysis, and the agent that
will use REA can see the matching live server. Record unavailable optional
engines as limits, with the workflows they affect.

Native analysis starts an engine and may create a temporary project. Check
available disk and memory before adding an engine. If resource cost affects
the choice, measure a representative completed task's elapsed time, CPU time
and peak process-family memory on the selected host.

## Revision History

| Version | Date       | Change                                                         |
| ------- | ---------- | -------------------------------------------------------------- |
| 0.1.0   | 2026-10-28 | Initial task-scoped setup, provider and verification workflow. |
