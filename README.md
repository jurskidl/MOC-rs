**Phase 0 — Core scaffolding and numerics hygiene**
|Task|Status|
|Deterministic math: fixed thread scheduling, stable reductions (pairwise/Kahan)|❌|
|Config + validation: JSON/YAML schema (positive Σt, χ sum=1, ranges)|❌|
|Timers/logging: per-iteration Δk, L2(Δφ/φ), global balance, wall-time|❌|
|Unit: Determinism — two identical runs yield|Δk|
|Unit: Invalid config (negative Σt, malformed geometry) throws clearly|

**Phase 1 — 2D geometry and FSR tessellation**
|Task|Status|
|Primitives: half-planes (lines), circles; CSG for rectangles/annuli|❌|
|FSR tessellation: non-overlapping polygons; store area, centroid, material_id|❌|
|Adjacency: detect shared edges between neighboring FSRs|❌|
|Robust intersections and epsilon-tolerant point-in-region|❌|
|Unit: Areas/centroids — checkerboard and ring sector within 1e-12|❌|
|Unit: Classification — 1000 random points 100% correct|❌|
|Unit: Adjacency graph matches expected small geometry|❌|

**Phase 2 — Materials and multigroup cross sections**
|Task|Status|
|XS storage (SoA): Σt[g], Σs[g→g’], νΣf[g], χ[g], optional κg|❌|
|Interpolation hooks for T, ρ (linear/piecewise), enforce χ sum=1, non-negativity|❌|
|Mixtures by volume fraction|❌|
|Unit: Interpolation — linear table midpoints exact (≤1e-12)|❌|
|Unit: Mixture — volumetric mix matches hand calc (≤1e-12)|❌|
|Unit: Scattering rows satisfy ∑g’ Σs[g→g’] ≤ Σt[g] (≤1e-14)|❌|

**Phase 3 — Track sets (azimuthal) and segmentation**
|Task|Status|
|Angle set: M azimuthal angles θm ∈ [0, π), weights wm (uniform wm=π/M)|❌|
|Track generation: spacing h; parallel tracks seeded on inflow edges|❌|
|Segmenter: ordered segments {fsr_id, length ℓ, entry_edge_id, exit_edge_id}|❌|
|Track weights: Wm = wm × Δ⊥ for area consistency|❌|
|Unit: Area/path-length — ∑ Wm ℓ reconstructs domain area (≤1e-3)|❌|
|Unit: Segment endpoints on boundaries (≤1e-10), continuity (≤1e-12)|❌|
|Unit: Coverage — every FSR hit for each angle family|❌|

**Phase 4 — Fixed-source sweep (1 group, no scatter)**
|Task|Status|
|Attenuation: ψout = ψin e^(−τ) + (Q/Σt)(1−e^(−τ)), τ=Σtℓ|❌|
|Segment-averaged ψ̄ with small-τ series|❌|
|Scalar flux tally: φi += (1/Ai) ∑ Wm ℓ ψ̄|❌|
|Boundary conditions: vacuum, reflective mapping|❌|
|Unit: Uniform box absorber with uniform Q — ⟨φ⟩ ≈ Q/Σt (≤1%)|❌|
|Unit: Reflective vs mirrored — boundary match ≤1e-10; φ RMS ≤0.2%|❌|

**Phase 5 — Isotropic scattering and multigroup source iteration**
|Task|Status|
|Source: Qi,g = (1/2π)[∑g’ Σs,g’→g φi,g’ + χg ∑g’ νΣf,g’ φi,g’]|❌|
|Group sweeps with relaxation α; stop on residual thresholds|❌|
|Tallies: absorption, in/out-scatter, leakage via boundary currents|❌|
|Unit: Global balance — Prod − (Abs + Net leak)|
|Unit: Fixed Q (no scatter/fission) converges in one sweep|❌|
|Unit: Sensitivity — halve h, double M, changes ≤0.5% RMS|❌|

**Phase 6 — k-eigenvalue (power iteration)**
|Task|Status|
|Fission source: Fg,i = χg ∑g’ νΣf,g’ φi,g’; normalize ∑Fg,i=1|❌|
|Power iteration with optional Wielandt shift; update keff|❌|
|Convergence:|Δk|
|Unit: Homogeneous reflected box —|keff−kinf|
|Unit: Spectral radius monotone; Wielandt stabilizes if needed|❌|

**Phase 7 — Acceleration (pCMFD/CMFD)**
|Task|Status|
|Coarse mesh (pins/macro-cells); compute Φc and face currents Ĵ|❌|
|Coarse diffusion with conductances D̂; solve for Φ̂|❌|
|Scale fine flux: φi ← φi × (Φ̂c / Φc) in source iteration|❌|
|Unit: Equivalence — keff within 5 pcm; FSR power RMS ≤0.2%|❌|
|Unit: Speedup — ≥2× fewer source iterations on medium lattice|❌|

**Phase 8 — Power tallying and normalization**
|Task|Status|
|Power density: Pi = ∑g κg νΣf,g φi (or κ × fission rate)|❌|
|Aggregation: FSR→pin mapping; pin RMS/max deviations|❌|
|Normalization to target power P0|❌|
|Unit: Energy consistency — ∑ Pi = P0 within 1e-8|❌|
|Unit: Mesh refinement — pin-power RMS ≤0.5%, max ≤1%|❌|

**Phase 9 — Boundary conditions and symmetries (2D)**
|Task|Status|
|Implement periodic (translational/rotational), reflective, vacuum, albedo|❌|
|Periodic mapping: outgoing→incoming on opposite edge; conserve weights|❌|
|Unit: Infinite lattice via periodic BC ≈ 3×3 tiling (≤0.3% RMS)|❌|
|Unit: Albedo slab — reflected current within 1% analytic|❌|

**Phase 10 — Residuals, diagnostics, QA**
|Task|Status|
|Residual monitors: L2/L∞(Δφ/φ), Δk, fission-source change; stagnation alarm|❌|
|Optical stats: per-FSR τ; safe exp underflow; small-τ series|❌|
|Unit: Stagnation alarm triggers under poor relaxation by iteration N|❌|
|Unit: Optical extremes τ≈1e-6, τ≈50 stable ψout/ψ̄ (no NaNs)|❌|

**Phase 11 — Performance and parallelism (2D)**
|Task|Status|
|Thread-parallel sweeps with per-thread accumulators and stable reductions|❌|
|Precompute per-segment Σt; optional fast exp approximation with guard|❌|
|Unit: Reproducibility 1/2/8 threads —|Δk|
|Unit: Strong scaling — 1→8 threads ≥4× speedup (medium case)|❌|
|Unit: Sanitizers — address/UB sanitizers clean on test suite|❌|
|Regression and acceptance gates (add to CI once Phases 4–8 exist)|

|Task|Status|
|Cases: uniform box (vacuum), two-region checkerboard, 1 pin reflective, quarter vs full symmetry|❌|
Gates:
Δk
