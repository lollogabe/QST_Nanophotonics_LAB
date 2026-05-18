# Quantum State Tomography of Quantum-Dot Photon Pairs

**Authors:** Lorenzo Gabellini, Aurora Folcarelli, Luca Martini, Emanuele Ceccarani

Measurements acquired in the **Nanophotonics Lab of Prof. Rinaldo Trotta** at
Sapienza University of Rome.

---

A two-photon polarization quantum-state-tomography (QST) pipeline. It reconstructs
the two-qubit density matrix ρ of the photon pairs emitted by a semiconductor
quantum dot from coincidence-count histograms acquired by time-correlated
single-photon counting (TCSPC) via SNSPDs, and quantifies the entanglement of the
reconstructed state.

The entire analysis lives in a single notebook, [`inference.ipynb`](inference.ipynb).

## Project structure

```
QST_Nanophotonics_LAB/
├── inference.ipynb        # the complete analysis (methods + inference)
├── requirements.txt
├── README.md
├── data/
│   ├── spectra/           # quantum-dot photoluminescence spectra (CSV)
│   ├── DOT/               # entangled photon pairs — 16 TCSPC histograms
│   └── LASER/             # |HH⟩ calibration dataset — 16 TCSPC histograms
└── results/               # created on first run
    ├── figures/           # PNG (slides) + PDF (articles)
    ├── tables/            # LaTeX tables
    └── *.json             # machine-readable result summaries
```

## Measurement settings

Tomography uses the 16 two-photon polarization projectors of a tomographically
complete POVM:

| Alice \ Bob | H | V | D | L/R |
|-------------|---|---|---|-----|
| **H** | HH | HV | HD | HL |
| **V** | VH | VV | VD | VL |
| **D** | DH | DV | DD | DR |
| **R** | RH | RV | RD | RL |

Single-photon basis states: H (horizontal), V (vertical), D (diagonal),
A (anti-diagonal), R (right-circular), L (left-circular). Each setting `XY`
defines the **POVM element** `Π_XY = |XY⟩⟨XY|`, a rank-1 projector; the 16
elements together form one **POVM**.

## Data format

**TCSPC histograms** (`DOT/`, `LASER/`) — two-column tab-separated files, one per
analyzer setting:

```
"Time differences (ps)"    "Counts per bin"
-20000                      5
-19980                      4
...
```

- `LASER`: 200 bins, 20 ps wide, range ±2000 ps — a single coincidence peak.
- `DOT`: 2000 bins, 20 ps wide, range ±20000 ps — a central coincidence peak
  plus laser-repetition side peaks one period (~12.5 ns) away.

**Spectra** (`spectra/`) — comma-separated `wavelength (nm), counts, flag`.

## Analysis pipeline

The notebook is split into a **methods** part (sections 1–9, function definitions)
and an **inference** part (sections 10–14, the actual run).

1. **Emission spectra** — `plot_spectra()` shows the photoluminescence of the dot
   under above-band and two-photon (resonant) excitation, and the spectrally
   filtered exciton (X) and biexciton (XX) cascade lines.

2. **Count extraction** — `integrate_peak()` locates the dominant peak inside a
   search window, estimates the Poissonian background, and integrates the
   connected run of bins above the threshold `background + nσ·√background`.
   - *LASER*: one peak over the whole delay window.
   - *DOT*: the central coincidence peak is integrated inside ±5 ns; a side peak
     (the same side for every setting) is integrated identically and provides a
     per-element scaling factor. The **normalized counts** are central / side.

3. **Density-matrix reconstruction** — two estimators, both fed the normalized
   counts:
   - `reconstruct_linear_inversion()` — solves the Gram-matrix system `G c = p`.
     Hermitian by construction, but not constrained to ρ ⪰ 0.
   - `reconstruct_mle()` — maximum likelihood with the physical parametrization
     ρ = T†T / Tr(T†T), where T is a lower-triangular (Cholesky) matrix with
     complex entries (James, Kwiat, Munro & White, *Phys. Rev. A* **64**, 052312,
     2001). The 16 real parameters are fitted by minimizing the Poisson (default)
     or Gaussian negative log-likelihood with **L-BFGS-B**, seeded from the
     linear-inversion estimate. The normalization is N = HH + HV + VH + VV.

4. **Metrics** — `state_metrics()` reports Tr(ρ), purity Tr(ρ²), fidelity F to a
   target state, Wootters concurrence C, negativity 𝒩, entanglement of
   formation E_F, and the eigenvalues of ρ.

5. **Uncertainties** — `monte_carlo_metrics()` resamples every measured count
   from a Poisson distribution (mean = the measured count), reconstructs each
   sample with the same estimator, and reports the mean and sample standard
   deviation (`ddof=1`) of every metric.

All figures and tables are written to `results/` by `save_figure()` and
`save_results()`.

## Results (DOT dataset, target Φ⁺)

| Metric | Linear inversion | Maximum likelihood |
|--------|------------------|--------------------|
| Fidelity F(ρ, Φ⁺) | 0.965 ± 0.006 | 0.945 ± 0.001 |
| Purity Tr(ρ²)     | 0.980 ± 0.011 | 0.939 ± 0.002 |
| Concurrence C     | 0.865 ± 0.010 | 0.939 ± 0.002 |
| Negativity 𝒩      | 0.486 ± 0.005 | 0.468 ± 0.001 |

Linear inversion returns a slightly unphysical ρ (purity > physical, small
negative eigenvalues) because positivity is not enforced; maximum likelihood
returns a physical state and is the result to quote.

## Dependencies

```
numpy
scipy
matplotlib
jupyterlab
ipykernel
```

Install with:

```bash
pip install -r requirements.txt
```
