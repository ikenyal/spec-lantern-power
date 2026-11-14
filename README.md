# spec-lantern Kiro Power

Revive abandoned "vibe-coded" TypeScript/React apps. spec-lantern reverse-engineers
a Kiro spec, turns every claim a comment makes into an **executable
Correctness_Property**, locks a read-only prediction, then **executes** the
properties in an isolated runner and compares predicted vs actual. The guiding
rule: an agent that only *reads* code trusts misleading comments — so this Power
never stops at reading.

- Demo video (2 min): https://youtu.be/H2C7ndx9C8c
- Live report and the "Trust or Test?" game: https://d2xayupxggkiqt.cloudfront.net/
- Source, spec and design: https://github.com/ikenyal/spec-lantern
- Kiroween 2026 submission: https://devpost.com/software/spec-lantern

[![spec-lantern demo video](https://img.youtube.com/vi/H2C7ndx9C8c/maxresdefault.jpg)](https://youtu.be/H2C7ndx9C8c)

## Requirements

- **Kiro** (the AI IDE that loads Powers).
- **Node.js 20 or newer** — the bundled MCP server is launched with `npx`.
- **rootless Podman** — the server runs all target code inside Podman, never on
  your host. On macOS, start the Podman machine first:

  ```sh
  brew install podman        # if not installed
  podman machine init        # first time only
  podman machine start
  ```

## Install

In Kiro:

1. Open **Powers**.
2. **Add Custom Power** -> **Import power from GitHub**.
3. Paste `https://github.com/ikenyal/spec-lantern-power`.

Kiro reads `POWER.md` and registers the `spec-lantern` MCP server from
`mcp.json` (launched with `npx -y github:ikenyal/spec-lantern-power#v0.1.0`).
No build step runs — the server ships as one bundled file. Confirm Kiro lists all
six tools (`lock_prediction`, `supersede_prediction_lock`,
`execute_properties`, `replay_counterexample`, `apply_fixes_and_reexecute`,
`compare_predictions`) before you start.

## Use it

Open a TypeScript/React repo in Kiro and just ask, for example:

> Check the comments in this repo — are any of them lying about what the code does?

Kiro activates the Power, reverse-engineers a spec, and runs the
execute-don't-trust pipeline to produce a Lantern_Report of confirmed bugs with
counterexamples.

## Safety model (5 lines)

1. Target code runs **only inside rootless Podman**, never on your host.
2. Dependencies install behind an internal-only network + registry proxy with
   lifecycle scripts disabled; the test phase runs with **no network at all**.
3. The Power itself **never executes** target code — it only writes artifacts
   and calls the six MCP tools.
4. **The original is sacred**: every tool that runs your code
   (`execute_properties`, `replay_counterexample`, `apply_fixes_and_reexecute`)
   first copies the repo into a throwaway working copy and runs against the copy
   — your original repo is never mounted or written to.
5. `apply_fixes_and_reexecute` is **not auto-approved** — you confirm every
   fix-and-re-execute run.

## Working copies (where your code is copied to)

Because the original is never mounted, each run copies your repo into a fresh,
throwaway working copy (excluding `node_modules`, `.git`, and build output) and
removes it when the run finishes. Those copies live under:

- `$SPEC_LANTERN_WORKDIR` when that environment variable is set, otherwise
- `~/.spec-lantern/worktrees` (created automatically on first use).

The default is anchored to your home directory on purpose: on macOS the Podman
machine shares `/Users` but **not** `/tmp`, so the copy must live under your
home to be visible inside the container. Set `SPEC_LANTERN_WORKDIR` to relocate
the copies (for example to a larger volume), but keep it somewhere Podman can
bind-mount — under your home on macOS.
