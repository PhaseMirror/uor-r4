# uor‑r4 – Hybrid Verification Project

## Overview
`uor‑r4` implements **ADR 0007 – Production‑Grade Hybrid Verification Pipeline** that merges **Zero‑Mathlib Lean 4 core** with a **Rust/Kani execution engine**. The goal is to provide formally verified architectural decision records (ADRs) while guaranteeing runtime correctness of the physics engine (Complex Gravitational Coupling).

## Key Features
- **Lean 4 core** (`lean/`) with *zero‑sorry* policy (except for the quarantined `Axioms.lean`).
- **Rust engine** (`src/engine/complex_gravitational.rs`) using `num_rational::Rational64` for exact arithmetic.
- **Kani model checking** (`cargo kani`) validates invariants such as:
  - *Immutable status after acceptance*.
  - *No circular supersession*.
  - *Exact rational bounds* on forces and coupling terms.
- **ADR system** (`src/adr/`) with Rust structs, state‑transition methods, and Kani proofs.
- **CI pipeline** (`.github/workflows/apex-integrity.yml`) runs the Lean build, Kani verification, and Rust tests on every push/PR.
- **Validation script** (`scripts/validate_zero_drift.sh`) ensures no stray `sorry` tokens appear outside `lean/Core/Axioms.lean`.

## Repository Layout
```
.
├── .github/workflows/apex-integrity.yml   # CI workflow (Lean + Kani)
├── contracts/fpes_invariants.yaml          # Formal contract spec
├── docs/adr/0007-hybrid-verification-pipeline.md  # ADR document
├── lean/
│   ├── lakefile.lean                     # Minimal Lake config (no Mathlib)
│   └── Core/
│       ├── Axioms.lean                  # Quarantined file (allowed `sorry`)
│       └── GravitationalCoupling.lean   # Zero‑sorry proof stub
├── scripts/
│   ├── validate_zero_drift.sh            # Pre‑commit `sorry` guard
│   └── generate_certificates.py          # Stub for Kani→Lean certificates
├── src/
│   ├── adr/
│   │   ├── link.rs                       # ArtifactLink struct
│   │   ├── mod.rs                        # ADR core + Kani invariants
│   │   └── status.rs                     # ADRStatus enum
│   ├── engine/
│   │   └── complex_gravitational.rs       # Exact‑rational physics engine
│   └── main.rs                           # Registers ADR 0006 and runs demo
├── tests/adr_invariants.rs               # Test harness for Kani proofs
├── Cargo.toml                             # Rust manifest (num_rational dep)
└── README.md                              # **You are here**
```

## Getting Started
```bash
# Clone the repo and cd into the project
git clone <repo-url>
cd uor-r4

# Install Rust (if not already installed) and Kani
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
source $HOME/.cargo/env
cargo install kani

# Validate Lean source (zero‑sorry)
bash scripts/validate_zero_drift.sh

# Build Lean core
cd lean && lake build && cd ..

# Run Kani model checking (verifies ADR invariants & engine bounds)
cargo kani

# Run Rust unit tests
cargo test
```

## ADR Workflow
1. **Create** a new ADR file under `docs/adr/` (e.g., `0008‑new-feature.md`).
2. **Implement** the corresponding Rust module or Lean definition.
3. **Register** the ADR in `src/main.rs` (or another appropriate entry point) via `ADR::new(...).accept()`. The acceptance makes the record immutable unless superseded.
4. **Add** any necessary Kani proofs in `src/adr/mod.rs` or a dedicated `src/adr/proofs/` module.
5. **Run** the full verification pipeline (`cargo kani && lake build && cargo test`).
6. **Commit** – the CI will enforce all checks automatically.

## Extending the Pipeline
- **More Lean proofs**: Add additional theorems in `lean/Core/` and keep them `sorry`‑free.
- **Advanced physics**: Replace the simple coupling term with a full relativistic model, still using `Rational64` for scalar calculations.
- **Certificate generation**: Flesh out `scripts/generate_certificates.py` to produce deterministic Lean axioms from Kani logs, then `git add` the resulting `KaniCertificates.lean`.

## License
This project is licensed under the **MIT License**. See `LICENSE` for details.

---
*Hybrid verification enables us to reason about both the mathematical specification (Lean) and the concrete implementation (Rust/Kani) under a single, auditable pipeline.*
