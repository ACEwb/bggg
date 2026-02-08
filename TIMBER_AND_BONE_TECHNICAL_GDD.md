# TIMBER & BONE — Technical GDD (v0.1)

## 1) Product Definition
**Hook:** A 2D lumberjack game where upgrades stack into absurdity until entire forests delete themselves instantly.

**Genre:** Incremental action / idle hybrid (2D side-view).

**Platforms:** Windows, macOS, Linux, ChromeOS, Android, iOS.

**Session Model:**
- Short-session friendly (30s–10m loops).
- Infinite long-term progression.
- Offline/idle simulation enabled.

---

## 2) Design Pillars
1. **Only Tree Cutting Matters** — every system directly improves chopping.
2. **Instant Reward Loop** — every action yields visible progress quickly.
3. **Absurd Escalation** — progression moves from manual effort to full automation.
4. **Stable on Low-End** — deterministic frame budget and scalable effects.

---

## 3) Core Loop and Progression Curve
1. Enter forest zone.
2. Cut trees (manual → assisted → automatic).
3. Earn Wood + Meta Currency.
4. Buy upgrades in infinite skill tree.
5. Increase clear speed and unlock deeper zones.
6. Repeat forever with soft scaling.

### Progression Bands
- **Early Game:** 2–6 swings/tree, visible stamina pressure.
- **Mid Game:** weak-point chaining, sub-second cuts.
- **Late Game:** instant fells, screen-wide chains.
- **Beyond Endgame:** spawn-kill forests and passive biome clears.

---

## 4) Systems Specification

### 4.1 Tree Entity
**State machine:** `Alive -> Damaged -> Breaking -> Fallen -> Despawned`.

**Data fields:**
- `maxIntegrity`, `currentIntegrity`
- `hardnessTier`
- `weakPointSize`, `weakPointMultiplier`
- `yieldWoodBase`, `yieldMetaBase`
- `spawnTimestamp`, `zoneId`

### 4.2 Player Chopping Model
**Base chop event:**
- Input trigger produces `SwingAction`.
- Swing resolves against tree collider and optional weak point.
- Damage applied after timing window.

**Damage formula (authoritative simulation):**
`damage = baseAxePower * speedScalar * precisionScalar * automationScalar * zoneScalarReduction`

**Time-to-fell control knobs:**
- Swing interval
- Recovery frames
- Weak-point frequency
- Hit confirmation leniency

### 4.3 Stamina / Effort Model
- Early game uses stamina drain to gate click spam.
- Upgrades reduce stamina cost and recovery delay.
- Late-game milestones disable stamina entirely.

### 4.4 Yield and Multiplication
**Wood payout:**
`woodOut = baseYield * yieldMultiplier * chainMultiplier * conversionBonus`

**Chain behaviors:**
- Adjacent tree fracture chance.
- Zone pulse events that convert one tree into N virtual fells.
- Auto-harvest ticks that process spawned trees in batches.

### 4.5 Infinite Skill Tree
Single directed graph with procedural extension.

**Generation model:**
- Seeded deterministic node generation by `depth` and `branchId`.
- Node categories weighted by progression stage.
- Every node maps to one of: speed, precision, effort, yield, structure.

**Scaling model:**
- Costs and values use softcap curves, never hardcaps.
- Benefits asymptotically slow but remain positive.

**Example cost function:**
`cost(n) = a * n^p * log(n + b)`

**Example value function:**
`value(n) = base + (gain * n) / (1 + k * n)` for soft diminishing return.

### 4.6 Zone Unlocking
- Zones increase tree durability and payout.
- Unlock criteria based on cumulative throughput (wood/sec milestones).
- Each zone introduces visual palette changes only (no mechanic bloat).

### 4.7 Idle / Offline Simulation
- Save `lastActiveTimestamp`.
- On return, simulate bounded ticks using aggregated rates.
- Cap max simulated duration per resume for performance predictability.

---

## 5) Number and Economy Layer

### 5.1 Big Number Strategy
Use a custom `BigFloatLite` representation:
- Mantissa (double)
- Exponent (int)
- Normalized scientific representation

Supports suffix formatting:
`K, M, B, T, Qa, Qi, Sx, Sp, Oc, No, Dc, ...`

Fallback:
- Scientific notation for very high exponents.
- Optional symbolic infinity display for post-threshold cosmetic milestone.

### 5.2 Safety Requirements
- No integer overflow in economy operations.
- Clamp and validate all conversion boundaries.
- Deterministic rounding policies for save/load parity.

---

## 6) Tech Architecture (C++ Core + Java Platform Layer)

### 6.1 Module Split
**C++ Core (shared):**
- ECS/gameplay simulation
- Economy math / big numbers
- Upgrade graph generation
- Save schema logic
- Deterministic simulation tick

**Java Layer (platform integration):**
- Android/ChromeOS app lifecycle
- Input abstraction glue (touch, back button, focus)
- Native bridge (JNI) for C++ core calls
- System UI, permissions, notifications
- Adapters for telemetry/crash reporting

For desktop/mobile parity:
- Keep gameplay authoritative in C++.
- Java/Kotlin/Objective-C/desktop wrappers only marshal events and render surfaces.

### 6.2 Cross-Platform Runtime Strategy
- Fixed simulation tick (e.g., 30 or 60 Hz).
- Render interpolation decoupled from simulation.
- Data-driven config for balance values per build.

### 6.3 Save System
- Versioned JSON or compact binary blob.
- Append-only migration table by schema version.
- Autosave on interval + app background events.

---

## 7) Performance Budget and Low-End Constraints

### 7.1 Frame Budget Targets
- 60 FPS target on mid/high devices.
- 30 FPS stable fallback on low-end.
- Simulation remains deterministic under either mode.

### 7.2 Optimization Rules
- Batch tree updates by chunk.
- Cap concurrent VFX and audio voices.
- Use object pools for tree fragments and popup text.
- Avoid per-frame allocations in hot loops.

### 7.3 Idle-Heavy Scaling
At high automation:
- Skip individual collision checks where possible.
- Use aggregate kill-rate calculations.
- Process forests in vectorized or chunked passes.

---

## 8) Input and UX Requirements
- Keyboard/mouse, touch, and controller parity.
- One-thumb viable core actions on mobile.
- Hold-to-cut and auto-cut unlocks reduce tap fatigue.
- Accessibility toggles: reduced flashes, simplified numbers, larger UI scale.

---

## 9) Visual and Audio Direction

### Visual
- Clean 2D side-view sprites.
- Tree damage stages become faster with power.
- Humor and glitch overlays at absurd tiers.

### Audio
- Layered chop impacts with pitch variation.
- Compression-safe mix for rapid trigger rates.
- Optional “chaos mix limit” to prevent clipping in late game.

---

## 10) Tone, Messaging, and Content Style
- Starts practical and grounded.
- Escalates into deliberate absurdity.
- Achievement and tooltip copy acknowledges broken power levels.
- Avoid cynicism; celebrate mastery and momentum.

---

## 11) Milestone Plan (Production)

### M0 — Core Prototype (2–4 weeks)
- One zone, one tree type, manual chopping.
- Basic upgrades (speed, yield, weak point).
- Save/load, big number formatter.

### M1 — Vertical Slice (4–8 weeks)
- Infinite skill tree generation (first stable pass).
- 3–5 zones, offline progress, automation unlock path.
- Basic controller + mobile touch support.

### M2 — Content and Scale (6–10 weeks)
- Hundreds+ generated upgrades and balancing pass.
- Optimization for Chromebook/mobile floor devices.
- UI polish, achievement set, absurdity feedback layer.

### M3 — Launch Candidate
- Platform cert hardening.
- Migration-tested saves.
- Crash/telemetry instrumentation.
- Performance and battery validation.

---

## 12) Risks and Mitigations
1. **Runaway economy inflation**  
   Mitigation: staged softcaps, slope dashboards, automated balance tests.

2. **Late-game CPU spikes**  
   Mitigation: aggregate simulation and event batching thresholds.

3. **Cross-platform logic drift**  
   Mitigation: gameplay only in C++, platform layers are thin wrappers.

4. **Input fatigue on touch**  
   Mitigation: early hold mechanics and progressive automation unlocks.

---

## 13) Next Practical Deliverables
1. Formal data schema for tree, upgrade node, and save file.
2. C++ interface definitions (`SimulationCore`, `EconomyService`, `UpgradeGraphService`).
3. Java JNI bridge contract doc.
4. Balance spreadsheet with progression targets (TTF, wood/sec, cost curves).
5. Test matrix for low-end Chromebook + Android devices.
