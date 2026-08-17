# Mission03-Airuma-Rocketpy
UFABC Rocket Design — Project Airumã (Mission ID 03). RocketPy simulation, Kn diagnostics, and Monte Carlo/sensitivity analysis — LASC 2026.

# Project Airumã — RocketPy Flight Simulation & Analysis

**Team:** UFABC Rocket Design
**Competition:** LASC 2026 (Latin American Space Challenge), September 1–5, 2026 — Iacanga/Bauru-SP
**Vehicle:** Airumã (10K SRAD Solid, 3 km apogee category)
**Motor:** Cururu (SRAD KNSB, finocyl grain)

This repository contains the RocketPy-based simulation code supporting Project Airumã's Mission Report, submitted per LASC RCSM Ed. 7 Rev. 1 (March 2026), Computational Simulation section (p. 58) and CRS 10.1.10-14.

## What this notebook does

The notebook builds a complete deterministic flight simulation for Airumã (Environment, Rocket, Motor, Flight), then extends it with:

- A custom `FinocylGrain` diagnostic layer that models the motor's internal finocyl grain geometry (core + 8 fins) — a geometry RocketPy's native `SolidMotor` does not represent — enabling burn-area and Kn (Klemmung ratio) evolution over the burn, validated against real static fire data (TE01).
- An independent inertia verification comparing RocketPy's native inertia calculation against two alternative estimates, testing whether the finocyl geometry meaningfully affects inertia beyond RocketPy's built-in hollow-cylinder model.
- A launch-angle sensitivity sweep (rail inclination and heading) supporting operational decision-making on competition day.
- A 200-run Monte Carlo simulation with a convergence check, followed by a per-target sensitivity analysis (`SensitivityModel`) mapping every relevant RCSM requirement to its dominant source of uncertainty.
- The four required CRS 10.1.10-14 flight cases: Nominal, Ballistic, Drogue Only, and Main At Apogee, all run under the same motor calibration for direct comparability.

## How simulation results shaped the design

RocketPy was used iteratively throughout the design cycle, not only as a final validation step. Key examples:

- **Fin geometry:** the sensitivity analysis identified fin span as the dominant parameter for both static stability margins (~52% sensitivity each), which was recorded as a manufacturing quality-control priority for the fins.
- **Recovery system:** impact velocity was found to be dominated by the main parachute's effective drag coefficient (~93% sensitivity), confirming the current CdS sizing already provides a comfortable margin against the REC 8.1.4/8.1.6 limit rather than requiring further resizing.
- **Motor model:** an early large apogee discrepancy between the finocyl and generic hollow-cylinder ("BATES") models led to a multi-step debugging investigation (grain-count bug, geometry re-derivation against OpenMotor, and a grain-density recalibration after the geometry correction silently broke a previous mass calibration) — the full history is documented inline in the notebook.
- **Compliance strategy:** motor burn-out time was found to dominate rail-departure velocity (~70% sensitivity), and the primary FLT 4.3.4 threshold is not met even at the best percentile under uncertainty — a property of the motor itself. This was resolved as a compliance decision (RCSM's alternate pathway), not a redesign.
- **Operations:** launch heading was found to dominate Y-direction landing dispersion (~60% sensitivity) more than any manufacturing tolerance, motivating a recommendation to the Operations and Launch Personnel team for heightened attention to rail-pointing precision on launch day.

The full reasoning behind each of these points, including negative/inconclusive results (e.g., the attempted Saint-Robert burn-rate calibration and the unreliable X-impact dispersion ranking) documented as methodological limitations rather than acted upon, is in the notebook's markdown cells alongside the code that produced them.

## Software used

- **[RocketPy](https://github.com/RocketPy-Team/RocketPy)** — primary 6-DOF flight simulation, Monte Carlo, and sensitivity analysis framework.
- **[OpenMotor](https://github.com/reilleya/openMotor)** — independent cross-validation of the finocyl grain geometry and 2D burnback simulation (used as a reference source, not run from this repository).
- **NumPy / SciPy** — numerical computation, including root-finding (`brentq`) for the grain regression inverse problem.
- **Pandas** — tabular result summaries.
- **Matplotlib** — all plots and the grain-regression animation.

See `requirements.txt` for exact package versions.

## Repository structure

```
├── notebooks/
│   └── RocketpyAiruma.ipynb      # Main notebook (all sections below)
├── data/
│   ├── Airuma.eng                # Motor thrust curve (static fire TE01, rebased to t=0)
│   ├── mach_vs_power_on_Airuma.csv
│   └── mach_vs_power_off_Airuma.csv
├── requirements.txt
└── README.md
```

## Notebook contents

1. Setup and parameter dictionary
2. Main deterministic flight simulation (Nominal case)
3. Finocyl grain Kn diagnostics and cross-validation against static fire data (TE01)
4. Inertia verification (native BATES vs. independent estimates)
5. Launch angle sensitivity sweep (inclination/heading)
6. Monte Carlo simulation and convergence check
7. Sensitivity analysis (RCSM compliance mapping)
8. Recovery failure case simulations (Ballistic, Drogue Only, Main At Apogee)
9. Design decisions informed by this analysis

## Running the notebook

The notebook is self-contained and runs top to bottom in Google Colab or a local Jupyter environment. Required files (`Airuma.eng` and the two drag-coefficient CSVs) are downloaded automatically from Google Drive in the setup section — no manual upload needed.

```bash
pip install -r requirements.txt
```


## A note on data privacy

The static fire cross-validation cell (Section 3) uses **anonymized** pressure and thrust data from the team's TE01 static fire test, transformed via an affine transformation (`x → a·x + b`) that exactly preserves the Pearson correlation coefficients used for validation while concealing the real physical magnitudes of the team's proprietary motor test data. Time values are unaltered. The empirical thrust coefficient (Cf), which is not preserved under this transformation, is reported separately in the Mission Report, computed locally from the team's private, non-anonymized dataset.
