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
