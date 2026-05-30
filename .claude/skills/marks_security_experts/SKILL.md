---
name: marks_security_experts
description: |
  Spawns 5 PhD-level security subagents in parallel to DESIGN (or review)
  a security + IP-protection architecture for a product — especially edge/
  embedded devices that ship to sites you don't control. Five fixed lenses:
  Security Architect, Applied Cryptography & Key Management, ML Model-
  Protection, Platform/Hardware Security, and Field Ops & Revocation. Each
  designs its slice (components, decisions, credential/data flows,
  dependencies); the skill synthesizes ONE unified target architecture with
  a phased build plan and cross-lens contradictions surfaced.
  Triggers on: "security experts", "security architecture review",
  "model-IP protection design", "threat model X", "design the key
  management", "how do we protect the model/keys on the device",
  "secure this deployment". Use proactively before a first commercial ship,
  when an attacker may gain physical possession of a device, or when keys /
  model weights / a control surface need protecting on an untrusted network.
---

# Mark's Security Experts

A specialization of [[marks_team_of_experts]] for security + IP-protection
architecture. Born from the 2026-05-29 AI Precise session: a headless Jetson
robot that fires 250 °F steam, ships to a non-technical customer (no SSH) at
a site you don't control, carrying a valuable model + secrets. The general
5-PhD panel had no lens for threat modeling, key management, model protection,
or revocation — so this panel bakes those in.

## When to invoke

- User says "security experts", "security review", "model-IP protection",
  "threat model this", "design the key management / KMS", "secure boot",
  "how do we protect the model/keys", or invokes `/marks_security_experts`
- Before a first commercial ship of a device to an untrusted environment
- When an adversary may gain **physical possession** of a device that holds
  IP (model weights), secrets (keys/credentials), or a control surface
- Designing encryption, key distribution, revocation, or attestation for a fleet

## When NOT to invoke

- A single, focused security question (use one specialist or answer directly)
- A pure software-quality / non-security architecture question (use
  [[marks_team_of_experts]] instead)
- A compliance checklist with a known framework (do the framework, not a panel)

## The threat-model framing (state this up front, every run)

On a device an attacker can physically possess and root, you **cannot** make
extraction impossible — the GPU/CPU needs cleartext to run. The architecture's
job is therefore to: **(1) raise extraction cost above attacker motivation,
(2) make theft provable, (3) make a stolen unit revocable** — not to achieve
secrecy. Always classify threats as *defend* vs *accept*, and name the accepts
honestly. Full-disk encryption with auto-unlock is theater vs whole-box theft;
passphrase-unlock breaks a headless boot model — design accordingly.

## The 5 fixed lenses

| # | Lens | What they design |
|---|------|------------------|
| 1 | **Security Architect / Threat-Modeling** | Threat model (actors × assets × capability), trust boundaries, attack surface, **control-surface authN/authZ**, how the other 4 slices compose into one coherent defense-in-depth design. Sets the frame. |
| 2 | **Applied Cryptography & Key Management** | Key hierarchy (root of trust → per-unit identity → data/model keys), envelope + **network-bound** encryption, the key-fetch protocol (mutual auth, replay/MITM, offline behavior), rotation, **revocation**. |
| 3 | **ML Model-Protection / MLOps Security** | Model encryption at-rest + in-memory exposure minimization, **watermarking/fingerprinting** (+ the verification procedure), provenance/version binding (checkpoint-sha ↔ config), device-locked compiled engines, honest extraction limits. |
| 4 | **Platform / Hardware Security** | Root of trust (TPM / secure-element / fuses / TrustZone-OP-TEE), **secure boot** feasibility + bricking risk, disk-crypto realities, targeted encrypted volume vs FDE, OS hardening, and the honest **hardware limits** (e.g. weights are cleartext in GPU memory; no practical GPU TEE). |
| 5 | **Field Ops / Deployment & Revocation** | Per-unit provisioning + inventory (no baked long-lived secrets), **offline-tolerance** so a network-bound key doesn't brick a dead-zone unit, the **revocation runbook** (timed, attacker-window analysis), recovery-vs-security tension (degrade to a tablet tap, never a truck roll), license/contract hooks. |

**Customize sparingly.** These five are tuned for an edge device with model IP +
a control surface. Swap lens 3 (Model-Protection) for "Data-Protection /
Privacy" if there's no model IP; swap lens 4 (Platform) for "Cloud/Infra
Security" if it's a server product. Keep it at 5 distinct, non-overlapping lenses.

**MANDATORY 6th seat — the premise red-team.** Per the parent skill
[[marks_team_of_experts]], always spawn a first-principles premise red-team
alongside the 5 security lenses. In a security design its job is to attack the
*threat-model premise*: "Is the asset we're protecting the right one? Is the
threat we're defending the one that actually matters, or the one that's easiest to
model? What are we accepting without saying so?" This is exactly the seat that, on
the 2026-05-29 run, produced the headline reframe ("the model isn't the #1 asset —
the unauthenticated steam control surface is"). Never drop it; lead the synthesis
with its verdict.

## Workflow

### Step 1 — Confirm scope (one round max)
Confirm: **what to secure** (the system + its valuable assets), **DESIGN vs
REVIEW** (propose a target architecture, or audit an existing one), and the
**deployment threat environment** (who can touch the device, what network).
Skip if the request is unambiguous.

### Step 2 — Gather shared context
Identify the files every expert needs + per-lens pointers:
- The existing security posture / decisions (HANDOFF, decision log, state docs)
- The control surface (API/server routes — is there ANY auth?)
- The deploy/provisioning chain (where secrets/keys/model flow today)
- The model-load path (the decrypt-at-load hook), the secret-handling code
- Ground-truth hardware/platform facts (SoC, OS, TPM/secure-element availability)

### Step 3 — Spawn 5 agents in parallel (background)
`Agent` tool, `subagent_type: general-purpose`, `model: opus`,
`run_in_background: true`, **all 5 calls in ONE message**. Each prompt carries:
identity, shared context + ground-truth facts, ~5-10 lens-specific file paths,
the DESIGN-output format below, and an anti-flattery directive ("push back; call
out where the decided posture is wrong or internally inconsistent").

**DESIGN-output format (mandatory) — each agent returns:**
1. Their slice of the **target architecture** — components, trust boundaries,
   and at least one credential/data **flow sequence** (named concrete tech).
2. **3-5 key DECISIONS** — each: chosen option / rejected alternatives / why /
   effort / what it depends on (which other lens).
3. **Phasing** — pre-first-ship vs later.
4. End with EXACTLY one sentence: "The single most important architectural
   decision in my domain is ___."

### Step 4 — Surface progress, wait for all completions
Tell the user "5 security experts running; will synthesize once all return."
Do NOT poll output files — the harness notifies on each completion.

### Step 5 — Synthesize ONE unified architecture (your job, not a 6th agent)
Produce:
1. **Threat model** (the agreed actors × assets × defend/accept table).
2. **Unified target architecture** — components + a single end-to-end
   credential/data-flow diagram (provision → boot → key-fetch → model-decrypt →
   run → revoke). Reconcile the 5 slices into one coherent system.
3. **Key decisions** with provenance tags `[Lens]`, deduped.
4. **Cross-lens contradictions surfaced honestly** — especially the recurring
   one: crypto/security want a network-bound key; Field Ops warns it can brick a
   field unit. Resolve it in the synthesis (grace window? fail-open vs closed?).
5. **Phased build plan** — what ships before first customer, what's later.
6. **What we explicitly will NOT build** (and why) — e.g. FDE, GPU TEE, secure
   boot now, weight-secrecy guarantees.
7. End with the single highest-leverage architectural decision.

## Lessons learned (from the 2026-05-29 canonical run)

- **Lead with the threat model and the honest limits.** "You cannot stop a
  motivated owner from extracting on-device weights" prevents the panel from
  over-designing crypto theater. State accepts explicitly.
- **Field Ops is the reality check on every other lens.** Network-bound keys,
  secure boot, encrypted volumes all sound great until a dead-zone or a
  secure-boot mismatch bricks a no-recovery field unit. Make Field Ops
  pressure-test the others' proposals in its prompt.
- **All 5 spawn calls in ONE message** to parallelize. **Opus** for design depth.
- **Give each agent the decrypt/auth hook-point file paths**, not "look around."
- **Synthesis is a unified architecture, not a fix-list.** Unlike the parent
  skill's Pareto top-N, the deliverable here is one coherent design + a phased
  build plan. Draw the single end-to-end flow.
- **Don't read agent output files via the shell** — wait for the notification.

## Anti-patterns to avoid

- **Crypto theater** — designing FDE/secure-boot/encryption that doesn't change
  the realistic threat (whole-box theft of an auto-booting unit). Name what each
  control actually defends.
- **Stranding the field unit** — any security mechanism that turns a connectivity
  blip or a key-fetch failure into an unrecoverable brick for a no-SSH operator.
- **Overlapping lenses** — keep Architect (frame) distinct from Crypto (keys)
  distinct from Platform (hardware root of trust).
- **Recursive spawning** / asking agents to converge with each other — they run
  independently; convergence is the synthesis step.
- **Sycophantic prompts** — explicitly tell each agent to refute the decided
  posture where it's wrong.

## Output to user

1. The unified target architecture (components + the single end-to-end flow)
2. Key decisions with `[Lens]` provenance
3. Cross-lens contradictions, resolved honestly (esp. network-bound-key vs
   field-brick)
4. The phased build plan (pre-first-ship vs later)
5. The explicit "will NOT build" list
6. The single highest-leverage decision

Keep the synthesis tight (~1000-1800 words). The agents produce thousands of
words; the user gets the coherent design, not a digest.
