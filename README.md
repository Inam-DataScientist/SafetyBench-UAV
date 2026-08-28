# Verified Latent Safety Representation (LSR) for Safe UAV Reinforcement Learning

This repository contains the reference implementation and the **SafetyBench-UAV**
benchmark for the paper *"Verified Latent Safety Representation for Adaptive
Safety Enforcement in Reinforcement Learning under Unknown Dynamics."*

The framework learns a compact, dynamics-aware latent representation of a short
trajectory window, predicts future safety risk from it, calibrates that
prediction with split-conformal prediction, and uses the calibrated risk to set
the conservatism of a feasibility-aware high-order control barrier function
(HOCBF) quadratic-program (QP) safety filter.

> Anonymized for double-anonymous review. Author, affiliation, funding, and
> citation details are intentionally omitted and will be added after acceptance.

---

## Overview

The pipeline has an offline stage and an online stage:

- **Offline (learn + calibrate).** A length-`L` window of 47-dimensional
  dynamics-aware features is encoded to a latent code `z_t`; a linear head
  predicts the future-max risk `r_hat_t`; split-conformal calibration on a
  disjoint held-out split yields the residual quantile `eps_hat`.
- **Online (verified enforcement).** A nominal controller proposes an action;
  the verified barrier gain `gamma_ver` (from the calibrated risk) sets the
  conservatism of the HOCBF-QP filter, which returns the executed action
  `u_safe = u_RL + delta_u`, with a slack term preserving feasibility.

---

## Repository structure

```
LSR-Verified-RL/
  requirements.txt
  controllers/
    pd_tracker.py            # nominal controller (proposes the nominal action)
  lsr/
    data/
      dataset.py             # feature construction (phi_t), windowing (X_t), labels
    losses/
      representation.py      # representation-learning loss terms
      risk.py                # weighted MSE risk-regression loss
    models/
      encoder.py             # LSR temporal encoder E_theta
      transformer_encoder.py # attention-based encoder variant
      risk_predictor.py      # linear risk head f_psi
      risk_predictor_v2.py   # alternative risk head
    training/
      trainer.py             # joint training + conformal calibration (Algorithm 1)
  verifier/
    cbf.py                   # control barrier function utilities
    specifications.py        # safe-set / constraint specifications
    specifications_hocbf.py  # high-order CBF specifications
    conformal_verifier.py    # split-conformal calibration and quantile
    qp_solver.py             # feasibility-aware QP (OSQP via CVXPY)
    verified_operator.py     # verified gain gamma_ver + online filtering (Algorithm 2)
  scripts/
    generate_sota_dataset.py # regenerate the SafetyBench-UAV corpus
    validate_safetybench.py  # dataset integrity checks and statistics
    train_robust.py          # end-to-end training entry point
    stress_tests.py          # closed-loop evaluation under stress
    debug_dangerous_episode.py # inspection utility for adverse episodes
  SafetyBench-UAV/
    episodes.h5              # the benchmark corpus (see Dataset section)
  Results/
    fig2_latent_space_tsne.png
    fig_adaptation_trace.png
    fig_lsr_cbf_correlation.png
    fig_lsr_separation_comparison.png
    fig_lsr_trajectory_evolution.png
    fig_risk_prediction_analysis.png
```

---

## Installation

Requires Python 3.9 or later.

```
git clone <REPOSITORY_URL>
cd LSR-Verified-RL
python -m venv .venv
source .venv/bin/activate      # on Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

Core dependencies are PyTorch (encoder and risk predictor), CVXPY with the OSQP
backend (safety-filter QP), and h5py, NumPy, SciPy, and Matplotlib (data and
figures). The pinned versions in `requirements.txt` are authoritative.

---

## SafetyBench-UAV dataset

`SafetyBench-UAV/episodes.h5` is a large-scale offline corpus of UAV trajectories
for learning to anticipate safety risk. It provides what existing safe-learning
suites do not expose together: graded risk labels, logged ground-truth safety
margins, injected disturbances, and actuator degradation, across multiple
environments and missions.

Each timestep records the state, the applied action, the disturbance, the
resulting next state (from which the one-step transition is formed), the scalar
safety margin `h(x)`, and a graded risk label in `{0, 1, 2, 3}`, where a
level-3 label coincides exactly with a physical violation (`h < 0`). The exact
HDF5 keys and shapes are defined in `lsr/data/dataset.py` and can be printed with:

```
python scripts/validate_safetybench.py
```

To regenerate the corpus from scratch:

```
python scripts/generate_sota_dataset.py
```

The concrete instantiation used in the paper (episode count, horizon, feature
dimensions, disturbance mixture, and risk-level prevalences) is reported in the
Experimental Setup section of the paper.

---

## Quickstart

Train the encoder and risk predictor and run split-conformal calibration
(Algorithm 1):

```
python scripts/train_robust.py
```

Run the online verified safety filter and reproduce the closed-loop evaluation,
including the primary stress scenario and the graceful-degradation sweep
(Algorithm 2):

```
python scripts/stress_tests.py
```

Configuration flags, output paths, and random seeds are defined at the top of
each script; adjust them to match your environment before running.

---

## Mapping from paper to code

| Paper component | Location |
|---|---|
| Dynamics-aware feature `phi_t`, window `X_t`, labels | `lsr/data/dataset.py` |
| LSR encoder `E_theta` (temporal core + attention pooling) | `lsr/models/encoder.py`, `lsr/models/transformer_encoder.py` |
| Linear risk predictor `f_psi` | `lsr/models/risk_predictor.py` |
| Weighted MSE objective | `lsr/losses/risk.py`, `lsr/losses/representation.py` |
| Joint training + calibration (Algorithm 1) | `lsr/training/trainer.py` |
| Split-conformal calibration, quantile `eps_hat` | `verifier/conformal_verifier.py` |
| Safe set and HOCBF specifications | `verifier/specifications.py`, `verifier/specifications_hocbf.py`, `verifier/cbf.py` |
| Feasibility-aware QP (OSQP via CVXPY) | `verifier/qp_solver.py` |
| Verified gain `gamma_ver` and online filter (Algorithm 2) | `verifier/verified_operator.py` |
| Nominal controller | `controllers/pd_tracker.py` |
| Benchmark generation | `scripts/generate_sota_dataset.py` |
| Closed-loop stress evaluation | `scripts/stress_tests.py` |
| Analysis figures | `Results/` |

---

## Reproducibility

- All reported metrics are computed over the evaluation episodes described in the
  paper. Set the seed at the top of each script to reproduce a specific run.
- The safety filter is deterministic given the trained encoder, the calibration
  quantile, and the barrier specifications.
- Experiments in the paper were run on a workstation with two NVIDIA A800 80 GB
  GPUs; a single GPU is sufficient to reproduce the results at reduced batch size.

---

## License

Released under the MIT License. See `LICENSE`.

---

## Citation

Citation details are withheld during double-anonymous review and will be added
here upon acceptance.
