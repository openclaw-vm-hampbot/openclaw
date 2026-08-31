# PR openclaw/openclaw#133163 — served Control UI proof artifacts

Captured from the served Control UI bundle built at branch head
`1cbd25ca3232b65417fa0861a7eda068d9730646`
(`feat/agent-safe-start-dreaming`), served by a real Gateway booted from that
exact head (isolated dev profile, loopback `127.0.0.1:18801`, memory-core
active, per-agent dreaming config present; UI footer shows
`2026.8.1 · feat/agent-saf…@1cbd25c*`).

Driven with headless Chromium (Playwright 1.62.1) as a real browser client of
the served UI, exercising the UI's own CAS-bound config path (no CLI-side
mutations):

- `01-landing.png` — login gate before token entry (no credentials stored).
- `02-agents.png` — Agents page before opening the Memory tab.
- `03-memory-included.png` — Agents → Memory tab: header renders
  `Using default: Enabled · AGENT INCLUDED`; Dreaming status panel renders
  with phases.
- `04-confirmation-copy.png` — participation toggle clicked: confirmation
  screen renders the per-agent copy shipped by this branch: title
  "Exclude This Agent from Dreaming" (destructive-direction flow, not the old
  global "All Agents" wording).
- `05-agent-excluded.png` — after confirming exclusion: header re-renders live
  to `Default: Enabled · Dreaming is excluded for this agent. · AGENT
  EXCLUDED`; the previously loaded Dreaming status is no longer presented as
  current runtime state for the excluded agent.
- `06-restored.png` — participation restored via the same UI flow: header back
  to `AGENT INCLUDED` (config net-zero mutated).

Local loopback URLs and ephemeral dev-profile state only; no secrets in
frame.


## Exact-head cache-gate proof (head `8ad886a1a54`, 2026-08-31)

Captured from a real Gateway booted from the exact branch head (build
`2026.8.1-8ad886a1a541`, dev profile, loopback `127.0.0.1:19001`, served
production UI bundle built from `ui/` at the same head). The browser
session holds a cached enabled `dreamingStatus` payload while an external
config writer changes gates; no page reload between frames:

- `07-cache-enabled-active.png` — proof-dreamer-3 participating: header
  renders `Default: Enabled · AGENT INCLUDED`; the Dreaming status panel
  presents the live payload with phases (`Light 3:00 AM`, `Deep 3:00 AM`,
  `Rem 3:00 AM`).
- `08-external-exclusion-cache-gated.png` — an external config writer
  excludes the agent while the panel is open. Without a reload the header
  re-renders to `Default: Enabled · Dreaming is excluded for this agent. ·
  AGENT EXCLUDED` and the cached enabled payload is gated: the phase rows
  render `—` (no cached next-run times), counts drop to zero.
- `09-participation-restored.png` — the external writer restores
  participation: the live phases render again (config net-zero mutated).
- `10-global-disable-cache-gated.png` — the external writer disables the
  global Dreaming master switch (`plugins.entries["memory-core"].config.
  dreaming.enabled = false`) while the enabled payload stays cached: the
  panel drops the active phases/counts without reload (rows render `—`).

Console transcript of the same run (capture-script assertions):

    [A1-enabled] toggle="Agent Included" on=true phases=3
    [A1-enabled] phaseStatuses=["Light 3:00 AM","Deep 3:00 AM","Rem 3:00 AM"]
    [A2-excluded] toggle="Agent Excluded" on=false phases=3
    [A2-excluded] muted="Default: Enabled · Dreaming is excluded for this agent."
    [A2-excluded] phaseStatuses=["Light —","Deep —","Rem —"]
    [A3-restored] toggle="Agent Included" on=true phases=3
    [A3-restored] phaseStatuses=["Light 3:00 AM","Deep 3:00 AM","Rem 3:00 AM"]
    [B-global-off] toggle="Agent Included" on=true phases=3
    [B-global-off] phaseStatuses=["Light —","Deep —","Rem —"]
    [dreaming-cache-gate-proof] PASS

Local loopback URLs and ephemeral dev-profile state only; no secrets in
frame.


## Exact-final-head shared-workspace ambiguity proof (head `816d6d4d28d8`, 2026-08-31)

Captured from a real Gateway booted from the exact final branch head (build
`2026.8.1-816d6d4d28d8`, `dist/build-info.json` confirms commit
`816d6d4d28d87135cd82b85abce4f7b172e22069`; dev profile, loopback
`127.0.0.1:19001`; served production UI bundle built from `ui/` at the same
head). This exercises the exact changed UI path of the final head:
`ui/src/pages/agents/memory/memory-panel.ts` shared-workspace diagnosis
recovery for an excluded co-owner (lines ~445–466). Scenario mirrors the
shipped regression `preserves the shared-workspace diagnosis for an excluded
co-owner with a cached payload` (`memory-panel.test.ts`, added at this head):

Setup: three agents share one workspace (`shared-proof`); proof-dreamer-3
participates (enabled), proof-dreamer-2 is excluded, plus an additional
included co-owner — the mixed included/excluded owner set that Doctor
reports as `shared-workspace-ambiguity` to both co-owners.

- `11-ambiguity-included-coowner.png` — proof-dreamer-3 (included co-owner)
  Memory tab: the panel presents the ambiguity status payload with the
  specific notice and `Agent Included`.
- `12-excluded-coowner-ambiguity-preserved.png` — an external config writer
  excludes proof-dreamer-3 while the panel is open, no reload: the header
  toggles to `Agent Excluded` AND the specific shared-workspace diagnosis
  survives the participation gate ("Dreaming is paused because this
  workspace is shared with an excluded agent."), the generic exclusion
  notice is NOT shown, status renders `Dreaming Idle`, `0 promoted` (stale
  count suppressed), and all phase next-runs are gated to `—` (no cached
  schedule).

Console transcript of the same run (capture-script assertions):

    [A-included] toggle="Agent Included" on=true
    [A-included] notice present, cached ambiguity payload acquired
    [B-excluded] toggle="Agent Excluded"
    [B-excluded] ambiguityNotice=true genericNotice=false
    [B-excluded] statusLabel="Dreaming Idle" promotedZero=true
    [B-excluded] phaseNexts=["—","—","—"]
    [shared-workspace-ambiguity-proof] PASS

Local loopback URLs and ephemeral dev-profile state only; no secrets in
frame.
