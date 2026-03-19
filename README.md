# Bayesian Optimization of Nanoparticle Reinforced Tire Compounds for Enhanced Durability and Efficiency

**Author:** Andy Kapoor

## Repository Structure
```
.
├── Tire_Compound_Optimized.ipynb   # Main analysis notebook
└── README.md
```

---

## Overview

This project applies Bayesian Optimization (BO) to identify nanoparticle reinforced tire compound formulation parameters that minimize wear (abrasion) and heat generation (rolling resistance). Two key drivers of tire degradation in both commuter and endurance racing applications. Rather than exhaustive grid search or manual tuning, BO uses a Gaussian Process surrogate model to intelligently navigate a 6-dimensional parameter space, proposing the most informative experiment at each iteration.

---

## Data Source

Compound performance data was generated using a nanocomposite tire simulator hosted at `http://abaqus.oit.duke.edu:8000/` (accessible on the Duke wired network, DukeBlue WiFi, or Duke VPN). Jobs were submitted via Duke NetID. All simulator outputs were copied directly into the notebook. No external data files are required. The notebook is fully self-contained.

---

## Problem

Nanoparticle reinforced tire compound development involves balancing competing objectives: durability, grip, rolling resistance, and cost. The reinforcing nanoparticle fillers (e.g. carbon black) and processing conditions interact in complex, nonlinear ways that make manual tuning inefficient. This project explores whether a data-efficient sequential optimization strategy can converge on high-performing formulations in a limited number of simulator evaluations, reducing the need for costly physical prototyping.

---

## Parameters

| Parameter | Description | Range |
|---|---|---|
| `vulc_time` | Vulcanization time (s) | 300 - 1000 |
| `vulc_temp` | Vulcanization temperature (K) | 403 - 453 |
| `sulph_MP` | Sulfur crosslinker content | 1 - 10 |
| `carbB_MP` | Carbon black nanoparticle filler content | 1 - 35 |
| `mixer_set` | Mixing intensity | 1 - 11 |
| `carbB_grade` | Rubber grade | 1 - 100 |

## Outputs Tracked

`material_cost`, `heating_cost`, `mixing_cost`, `sound_damping`, `rolling_resistance`, `abrasion`, `dry_grip`, `wet_grip`

---

## Methods

### Gaussian Process Surrogate Model
- Kernel: Rational Quadratic
- Input features scaled with MinMaxScaler
- GP fit to observed data, then used to predict mean and uncertainty across 50,000 random candidate points per iteration
- Initialized with 5 points before optimization begins

### Acquisition Functions Implemented
- **EI** — Expected Improvement (used for optimization)
- **UCB** — Upper Confidence Bound
- **PI** — Probability of Improvement

### Optimization Runs
- **Optimization 1:** Minimize abrasion. 20 sequential iterations starting from 5 initialization points.
- **Optimization 2:** Minimize rolling resistance. 20 sequential iterations starting from the same 5 initialization points.

Each iteration: fit GP → score candidates via EI → select next point → submit to simulator → copy result into notebook → append → repeat.

### Dimensionality Reduction and Visualization
The 6-dimensional parameter space was projected into 3D using three complementary techniques to visualize how BO explored the space and where the best formulations were found:
- **PCA** (linear, with loadings heatmap)
- **UMAP** (nonlinear, preserves local structure)
- **Kernel PCA** (nonlinear, RBF kernel)

A final bar chart compares all 8 output metrics across the baseline compound, the abrasion-optimized compound, and the rolling resistance-optimized compound.

---

## Results Summary

| Objective | Initial Best | Final Best | Improvement |
|---|---|---|---|
| Abrasion | 241.64 | ~170 | ~30% reduction |
| Rolling Resistance | 0.1404 | 0.1009 | ~28% reduction |

BO successfully converged toward lower-wear and lower-resistance nanoparticle reinforced formulations within 20 iterations, demonstrating data-efficient optimization over a 6-dimensional discrete/continuous parameter space.

---

## Dependencies
```
numpy
matplotlib
scipy
scikit-learn
umap-learn
```

Install with:
```bash
pip install numpy matplotlib scipy scikit-learn umap-learn
```

---

## Limitations & Future Work

- Objective functions are simulated. Experimental validation would be needed for real-world deployment.
- Single-objective optimization runs were conducted separately. A true multi-objective Pareto front (e.g. balancing abrasion vs. rolling resistance vs. grip) would be a natural extension.
- Convergence warnings from GP kernel bounds suggest hyperparameter tuning may improve surrogate model quality.
- Future work: multi-objective BO (e.g. using NSGA-II or qNEHVI), transfer learning across compound families, integration with physical tire models.
