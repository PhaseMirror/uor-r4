# uor‑r4 – Hybrid Verification Project

## Overview
`uor‑r4` implements **ADR 0007 – Production‑Grade Hybrid Verification Pipeline**. The repository fuses a **zero‑Mathlib Lean 4 core** with a **Rust/Kani bounded‑execution engine** to provide formally verified Architecture Decision Records (ADRs) and provably correct physics code for **Complex Gravitational Coupling**.

## Current Project State (2026‑08‑29)
- **Lean core** (`lean/`):
  - `lakefile.lean` – minimal Lake configuration, no external dependencies.
  - `Core/Axioms.lean` – the *only* file allowed to contain `sorry` tokens (placeholder axiom).
  - `Core/GravitationalCoupling.lean` – zero‑sorry stub (currently a trivial arithmetic lemma).
- **Rust ADR core** (`src/adr/`):
  - `status.rs` – `ADRStatus` enum (`Proposed`, `Accepted`, `Deprecated`, `Superseded`).
  - `link.rs` – `ArtifactLink` struct for external references.
  - `mod.rs` – `ADR` struct, lifecycle methods (`new`, `accept`, `deprecate`, `supersede`), and Kani‑enabled invariants (`immutable_after_accept`, `no_circular_supersession`).
- **Physics engine** (`src/engine/complex_gravitational.rs`):
  - Exact rational arithmetic via `num‑rational::Rational64`.
  - Newtonian pairwise forces + higher‑order coupling term.
  - `apply_coupling` updates velocities deterministically.
  - Unit test `basic_two_body` validates opposite‑velocity behavior.
- **Workspace configuration** (`Cargo.toml`):
  - Workspace resolver set to `2` for edition‑2021 compatibility.
  - Dependency `num‑rational = "0.4"` (note the hyphenated crate name).
- **CI / Automation** (`.github/workflows/apex-integrity.yml`):
  1. Checkout repository.
  2. Install Rust toolchain and Kani.
  3. Run `scripts/validate_zero_drift.sh` (ensures no stray `sorry`).
  4. Build Lean core (`lake build`).
  5. Run Kani model checking (`cargo kani`).
  6. Execute Rust unit tests (`cargo test`).
- **Support scripts**:
  - `scripts/validate_zero_drift.sh` – aborts CI if any `sorry` appears outside `Axioms.lean`.
  - `scripts/generate_certificates.py` – stub for future Kani→Lean certificate generation.

## Repository Layout
```
.
├── .github/workflows/apex-integrity.yml      # CI pipeline (Lean + Kani)
├── contracts/fpes_invariants.yaml           # Formal contract spec (future work)
├── docs/adr/0007-hybrid-verification-pipeline.md  # ADR documenting this pipeline
├── lean/
│   ├── lakefile.lean                        # Minimal Lake config (zero‑Mathlib)
│   └── Core/
│       ├── Axioms.lean                     # Quarantine for temporary `sorry`s
│       └── GravitationalCoupling.lean      # Zero‑sorry proof stub
├── scripts/
│   ├── validate_zero_drift.sh               # Pre‑commit `sorry` guard
│   └── generate_certificates.py             # Stub for Kani→Lean certificates
├── src/
│   ├── adr/
│   │   ├── link.rs                          # ArtifactLink struct
│   │   ├── mod.rs                           # ADR core + Kani invariants
│   │   └── status.rs                        # ADRStatus enum
│   ├── engine/
│   │   └── complex_gravitational.rs          # Exact‑rational physics engine
│   └── main.rs                              # Demo: registers ADR 0006 and runs simulation
├── tests/adr_invariants.rs                  # Kani‑compatible test harness
├── Cargo.toml                                # Rust manifest (workspace resolver, num‑rational dep)
└── README.md                                 # This file – comprehensive project overview
```

## Build & Verification Workflow
```bash
# Clone and enter the repository
git clone <repo-url>
cd uor-r4

# Install Rust (if needed) and Kani
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
source $HOME/.cargo/env
cargo install kani   # installs the `kani` subcommand

# Validate Lean sources (zero‑sorry policy)
bash scripts/validate_zero_drift.sh

# Build the Lean core (requires `lake` – bundled with Lean)
cd lean && lake build && cd ..

# Run Kani model checking (verifies ADR invariants & engine bounds)
cargo kani

# Run Rust unit tests (including the physics engine demo)
cargo test
```
All steps must succeed for the repository to be considered **consistent**.

## ADR Workflow Recap
1. **Create** a new ADR markdown under `docs/adr/` (e.g., `0008‑new-feature.md`).
2. **Implement** the associated Rust module or Lean definition.
3. **Register** the ADR in code via `ADR::new(...).accept()`. The acceptance locks the record unless superseded.
4. **Add** any required Kani proofs in `src/adr/mod.rs` (or a dedicated proofs module).
5. **Run** the full verification suite (`cargo kani && lake build && cargo test`).
6. **Commit** – CI enforces all checks automatically.

## Extending the Project
- **More Lean proofs**: Add theorems to `lean/Core/` while keeping the repository `sorry`‑free (except for `Axioms.lean`).
- **Advanced physics**: Replace the simple coupling term with a full relativistic or tidal‑interaction model, still using `Rational64` for deterministic scalar calculations.
- **Certificate generation**: Implement `scripts/generate_certificates.py` to parse Kani logs and emit a deterministic `KaniCertificates.lean` file; CI can then `git diff --check` to enforce consistency.

## License
This project is licensed under the **MIT License**. See the `LICENSE` file for details.

---
*Hybrid verification enables us to reason about both the mathematical specification (Lean) and the concrete implementation (Rust/Kani) under a single, auditable pipeline.*
