# Quantum Troptics — Data Analysis

**Authors:** Lorenzo Gabellini, Aurora Folcarelli, Luca Martini, Emanuele Ceccarani

Measurements acquired in the **Nanophotonics Lab of Prof. Rinaldo Trotta** at Sapienza University of Rome.

---

Quantum state tomography pipeline for two-photon polarization experiments. Reconstructs the two-qubit density matrix from coincidence-count histograms acquired with time-correlated single-photon counting (TCSPC), and computes standard entanglement metrics.

## Project structure

```
data_analysis/
├── inference.ipynb      # Main analysis notebook
└── data/
    └── LASER/           # Calibration dataset: laser in |HH⟩ polarization
        ├── HH.txt
        ├── HV.txt
        ├── ...          # 16 measurement settings total
        └── RL.txt
```

## Measurement settings

Tomography uses 16 two-photon polarization projectors spanning the operator space of a two-qubit system:

| Alice \ Bob | H | V | D | L/R |
|-------------|---|---|---|-----|
| **H** | HH | HV | HD | HL |
| **V** | VH | VV | VD | VL |
| **D** | DH | DV | DD | DR |
| **R** | RH | RV | RD | RL |

Basis states: H (horizontal), V (vertical), D (diagonal), A (anti-diagonal), R (right circular), L (left circular).

## Data format

Each `.txt` file is a two-column tab-separated histogram:

```
"Time differences (ps)"    "Counts per bin"
-2000                       4
-1980                       3
...
```

- 200 bins, bin width 20 ps, range −2000 to +2000 ps.
- The coincidence peak sits at the optical delay of the setup and may shift between files.

## Analysis pipeline (`inference.ipynb`)

1. **Peak extraction** — `integrate_highest_peak()` locates the dominant coincidence peak in each histogram, estimates the accidental background from side bins, and integrates the background-subtracted count with its Poisson uncertainty.

2. **Density matrix reconstruction** — `naive_density_matrix()` uses linear inversion: the 16 measured probabilities are fit to the Gram-matrix equation `G c = p`, where `G_ij = Tr(Mᵢ† Mⱼ)` and the POVM elements `Mᵢ` are rank-1 polarization projectors. The result is Hermitian but not constrained to be positive-semidefinite.

3. **Entanglement metrics** — `density_matrix_metrics()` reports:
   - `Tr(ρ)` — normalization check
   - `Tr(ρ²)` — purity
   - `F` — fidelity to a target state (default: Φ⁺ Bell state; LASER run uses |HH⟩)
   - `C` — Wootters concurrence
   - `N` — negativity (Peres–Horodecki criterion)
   - `E_F` — entanglement of formation
   - eigenvalues of ρ

4. **LaTeX output** — `latex_counts_table()` and `latex_metrics_table()` produce ready-to-paste tables for publications.

## LASER calibration results

The LASER dataset injects a coherent |HH⟩ state and serves as a system calibration. Key results from the last run:

| Metric | Value |
|--------|-------|
| Fidelity F(ρ, HH) | 0.9988 |
| Purity Tr(ρ²) | 1.012 |
| Concurrence C | 0 |
| Negativity N | 0.027 |

The small negativity and purity > 1 are artifacts of unconstrained linear inversion; a maximum-likelihood with positivity constraint would correct this. This is part of future work.

## Dependencies

```
numpy
matplotlib
```

Install with:

```bash
pip install -r requirements.txt
```

## Running the notebook

```bash
jupyter lab inference.ipynb
```

or open it directly in VS Code.
