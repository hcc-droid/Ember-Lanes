# AGENTS.md — Ember Lanes

## 1. Project

Ember Lanes is a **mobile-first 2D greybox** combining **Draft** and **Lane Defense**. The player protects a base by choosing cards, building a defensive lineup, and deploying units across **three combat lanes**. Enemies approach from the opposite side. A run progresses through multiple waves, with Draft choices shaping the player's strategy.

Core loop: start a wave → enemies spawn and advance → the player uses cards/deploys units → units fight under the simulation rules → the wave ends → the player receives the next Draft choice → continue until victory or defeat.

Initial high-level outcomes:
- **Victory:** complete all required waves and defeat the final boss, if the approved specification includes one.
- **Defeat:** base health reaches zero.

Build a playable greybox first. Use simple geometry, clear colors, and readable UI. Do not create complex art or polish outside the approved phase.

## 2. Required Documents — Read Before Every Phase

Before starting each phase or making a substantial change, read:
- `docs/SPEC.md` — approved gameplay behavior and requirements.
- `docs/OPEN.md` — unresolved questions, missing values, and conflicts.
- `docs/DECISIONS.md` — decisions explicitly approved by the user.

Also read this file and inspect the current project state/Git status before editing.

Document roles:
- `AGENTS.md`: engineering constraints and working process.
- `SPEC.md`: current approved game specification.
- `DECISIONS.md`: user-approved decisions.
- `OPEN.md`: unresolved or contradictory items.

If documents conflict, **do not choose silently**. Record the conflict and its implementation impact in `docs/OPEN.md`, then ask the user. Chat discussion is not a permanent decision until recorded in the appropriate document.

Never turn a `TBD`, placeholder, proposal, target, or assumption into a permanent game rule without user approval and a documentation update.

## 3. Stack and Change Restrictions

- Phaser 3
- TypeScript (`strict` mode)
- Vite
- Vitest

Do not add a dependency without explaining its purpose, why the existing stack is insufficient, and its maintenance/testing impact. Wait for explicit user approval. Do not replace the engine, language, build tool, or framework without explicit approval.

## 4. Mandatory Architecture

The simulation must be deterministic and runnable without opening the game screen.

### `src/sim/` — simulation
- Plain TypeScript only; no Phaser, DOM, or browser API imports.
- No `Math.random()` or `Date.now()`.
- Use a seeded RNG. The same seed and input sequence must produce the same result.
- Use a fixed timestep of **1/30 second**.
- Own gameplay rules and state transitions (movement, targeting, collision, damage, status effects, waves, Draft, victory/defeat), but implement only rules approved in `SPEC.md`.

### `src/render/` — Phaser and input
- Phaser reads and displays state from `src/sim/`.
- Player actions are sent to the simulation as input commands.
- Rendering must not decide gameplay rules or independently modify authoritative game state.
- Keep controls mobile-first and practical for one-finger use where possible. Do not assume unapproved screen dimensions, orientation, or interactions.

### `src/config/` — balance
- Store all tunable values here: health, damage, speed, attack rate, spawn timing, wave composition, card pools/weights, status effects, and other balance values.
- Do not hardcode balance values in simulation logic.
- Values appearing only in a draft/example are provisional until approved.

### `src/telemetry/` — events
- Record stable, documented event names and payloads.
- Support JSON export. Do not invent event semantics that conflict with the approved documents.

### `scripts/sim.ts` — headless runner
- Run without Phaser or a browser.
- Support reproducible seeded runs and machine-readable JSON output.
- Enable large batches of simulations for balance analysis.

### `tests/` — verification
- Test approved rules, edge cases, architecture boundaries, and deterministic behavior.
- Do not make an unresolved proposal appear approved by encoding it as a required test.

## 5. Legacy Design Proposals — Not Automatically Approved

Older Ember Lanes notes contain details beyond the current high-level specification. Treat these as **candidates only** until reconciled with and approved in `SPEC.md` and `DECISIONS.md`:

- Portrait layout around 360 × 640; enemies enter from the top and the base is at the bottom.
- Five waves per run, boss in the final wave, 25–35 seconds per wave, and an approximately three-minute run.
- Draft of three cards with one selection before every wave, including the first.
- Six unit cards from three roles (Guard, Archer, Mage) × two tags (Fire, Frost).
- Enemy behaviors such as Walker, Swerver, Breaker, and Boss.
- A lane bonus for at least two units with the same tag in one lane; proposed effects include Fire/Burn and Frost/slow.
- A Last Stand reward when base health is exactly one HP.
- A per-lane unit limit.

The current high-level specification also lists different enemy/card taxonomies (for example, Normal/Fire/Ice enemies and Fire/Ice/Defense/Upgrade/Spell cards). **Do not merge, rename, or discard these categories without an explicit decision.** Record alternatives in `docs/OPEN.md` and ask before implementing the affected systems.

The same rule applies to missing values/mechanics: HP, damage, speed, attack range, targeting, collision, spawn schedule, card weights/costs/limits, status duration/stacking, boss behavior, and exact Draft/Last Stand rules. An `Energy` UI label does not authorize an energy mechanic; a card or enemy name does not define its behavior. Ask instead of guessing.

## 6. Work Process — One Phase at a Time

- One user prompt authorizes **one phase only**. Never start or prepare the next phase unless asked.
- Use **Planning mode** for every phase.
- Read the required documents and inspect the code/Git status first.
- Before editing, present a plan covering the goal, steps, exact files/directories to change, unresolved questions, and intended tests/acceptance checks.
- **Stop and wait for explicit user approval before coding.** A request for a plan is not permission to implement.
- After approval, implement only the approved phase and approved decisions.
- If a missing rule blocks progress, add it to `docs/OPEN.md` and ask. Do not invent a default.
- If checks fail, report the real failure and fix only issues within the current phase. Do not expand scope.
- Never claim completion without actually running the relevant checks.
- At phase end, report changed files, commands, pass/fail results, and relevant measurements. Identify checks not run and why.
- Show the diff/change summary. **Commit only after the user reviews the diff and explicitly approves the commit.**

## 7. Phase Roadmap and Gates

The roadmap is a sequence of separately approved phases, not authorization to execute them together. Confirm each phase's scope against the current documents.

### P1 — Core simulation
Build TypeScript simulation, seeded RNG, fixed timestep, config, tests, and a minimal headless runner. No Phaser rendering. Test approved lane/combat/wave/base/end rules, repeatability with identical seed+inputs, and crash-free headless runs. Do not fill unresolved gaps with invented mechanics.

### P2 — Rendering and input
Add the greybox battlefield, three lanes, base, units, enemies, approved HUD, and mobile-first input. Phaser must display sim state and send commands to sim. If input replay is added, verify headless replay produces the same final state.

### P3 — Draft and cards
Implement only approved card pools, Draft rules, placement, tags, lane bonuses, and Last Stand. Store pools/weights in config. Test duplicate handling and approved trigger conditions.

### P4 — Telemetry and bot balance
Implement approved telemetry and headless bot policies. Legacy proposals include `random`, `greedy-by-dps`, and `tag-stacker`; 1,000 runs per policy; and reports for win rate by policy, loss rate by wave, pick share by card, average base/hearth HP at run end, and crashes/abnormal runs.

Legacy target ranges (random bot 20–40% wins, stronger bot 60–80%, no card above 45% pick share for the stronger bot) are **hypotheses, not industry benchmarks or automatic pass/fail rules**. Confirm them with the user before adopting them as acceptance criteria. If a stronger bot wins above 90% or a card exceeds 45% pick share after an approved balance experiment, stop before the next phase, report the data, and propose changes for approval. Do not change gameplay logic just to force target metrics.

### P5 — Tutorial, feedback, results, retry
After the balance review, implement only approved tutorial, simple combat feedback, result screen, and retry behavior. Legacy proposals include a text-free hand/finger tutorial, three enemies in tutorial wave 1, and one-tap retry; confirm them in `SPEC.md` first. Test tutorial with a recorded input sequence, retry taps, and telemetry. Bot tests do not replace human playtests: observe a first-time player and check whether they understand the basic action within about five seconds without explanation. This is a usability test, not a game rule.

### P6 — Shareable web build
Create a static build suitable for static hosting; no backend unless approved. Legacy proposals include `?seed=N` for reproducible runs, `?debug=1` to display debug information without changing gameplay, and `docs/AD_SEEDS.md` for naturally occurring demo/test situations found through simulation. Verify listed seeds reproduce in the built version. Do not force game state to manufacture a demo scene.

## 8. Verification and Reporting

Use the repository's actual scripts. Intended checks:

```bash
npm run typecheck
npm test
npm run build
npm run sim -- --runs 200 --seed 1
```

Run checks appropriate to the active phase in a clear sequence. If a script does not exist, report that; do not claim it passed. On failure, stop the gate, show the error, and explain its relevance. For simulation work, report run count, seed, policy, output path, and measured results.

Writing code is not proof of completion. A phase requires actual verification against approved criteria and user review of the diff.

## 9. Agent Coordination

- Start with **one agent**.
- Use multiple agents only when there is a clear benefit and the user approves or requests it.
- Give each agent exclusive ownership of a separate directory or isolated file set. Never let two agents edit the same file or own overlapping directories.
- Define ownership/integration points first. The integrating agent reviews the combined diff and runs relevant checks.

## 10. Out of Scope

Do not implement unless the user explicitly changes scope:

- PvP/multiplayer, gacha, liveops, multiple biomes.
- IAP, social features, backend services.
- Real advertisements or ad-network integration.
- Complex or production-quality art.

Greybox visuals use simple geometric shapes, basic colors, and clear indicators only.

## 11. Decision Discipline

- Implement only features approved for the current phase.
- Do not add helpful extras, hidden mechanics, progression systems, or speculative balance rules.
- Do not treat examples, bot targets, or legacy notes as approved requirements.
- Keep gameplay logic separate from rendering/UI and all balance values in config.
- Store durable project knowledge in project documents, not only in chat.
- When a decision is made, update `docs/DECISIONS.md`; if gameplay requirements change, update `docs/SPEC.md` too.
- Keep `docs/OPEN.md` current and remove an item only after its decision is recorded.
