# 🛡️ SafetyBench-UAV

### A Large-Scale Offline Benchmark for Anticipatory Safety Learning in UAV Control

<div align="center">

[![Dataset](https://img.shields.io/badge/dataset-SafetyBench--UAV-1f6feb.svg)](#dataset-at-a-glance)
[![Episodes](https://img.shields.io/badge/episodes-50%2C000-1f6feb.svg)](#dataset-at-a-glance)
[![Timesteps](https://img.shields.io/badge/timesteps-7.5M-1f6feb.svg)](#dataset-at-a-glance)
[![Environments](https://img.shields.io/badge/environments-5-2ea44f.svg)](#dataset-at-a-glance)
[![Missions](https://img.shields.io/badge/missions-8-2ea44f.svg)](#dataset-at-a-glance)
[![License](https://img.shields.io/badge/license-CC%20BY%204.0-blue.svg)](LICENSE)

</div>

---

## 🚁 Overview

**SafetyBench-UAV** is a large-scale offline benchmark of UAV altitude-control trajectories designed for research on:

- **Anticipatory safety-risk prediction**
- **Calibrated risk estimation**
- **Adaptive safety filtering**
- **Risk-aware learning-based control**
- **Robustness under disturbances**
- **Safety-aware reinforcement learning**
- **Distribution-shift evaluation**

The benchmark contains **50,000 UAV episodes and 7.5 million labeled timesteps**, covering multiple environments, missions, disturbances, and operating conditions.

Each timestep contains:

> **UAV state + control action + disturbance + safety signals + graded risk label**

The central research question is:

> **Can a learning system anticipate future safety risk before a physical constraint is violated, and can that information be used to improve safety filtering for learning-based control?**

---

## 🎯 Research Motivation

Learning-based control provides powerful mechanisms for autonomous decision-making, but deploying learned policies on physical systems introduces an important challenge:

> **A policy that performs well is not necessarily a policy that remains safe under disturbances, uncertainty, and changing operating conditions.**

Traditional safety evaluation often reduces the problem to a binary distinction:

```text
SAFE ─────────────────────────────── UNSAFE
  0                                      1
````

SafetyBench-UAV instead provides **graded safety-risk labels** together with a continuous safety margin:

```text
SAFE          LOW RISK       ELEVATED RISK       VIOLATION
  │               │                │                  │
  ▼               ▼                ▼                  ▼
Risk 0          Risk 1           Risk 2             Risk 3
```

This enables researchers to investigate **how risk evolves before a violation occurs**, rather than detecting a violation only after it has happened.

---

# 📊 Dataset at a Glance

<div align="center">

| **50,000** |  **7.5M** |     **150**     |     **5**    |   **8**  |
| :--------: | :-------: | :-------------: | :----------: | :------: |
|  Episodes  | Timesteps | Steps / Episode | Environments | Missions |

|      **20**      |       **4**       |          **3**         |    **4**    |         **37.99%**        |
| :--------------: | :---------------: | :--------------------: | :---------: | :-----------------------: |
| State Dimensions | Action Dimensions | Disturbance Dimensions | Risk Levels | Zero-Disturbance Fraction |

</div>

---

## 🌍 Environments

The benchmark contains trajectories from five operating environments:

| Environment   |
| :------------ |
| 🏢 Indoor     |
| 🏙️ Urban     |
| 🌲 Forest     |
| 🌾 Open Field |
| 🌐 Dynamic    |

---

## 🎯 Mission Types

The trajectories cover eight mission categories:

| Mission    |
| :--------- |
| Hover      |
| Tracking   |
| Swarm      |
| Emergency  |
| Inspection |
| Pursuit    |
| Navigation |
| Avoidance  |

These conditions provide multiple operating contexts for evaluating the generalization of safety-risk prediction and filtering methods.

---

# 🚨 Safety-Risk Distribution

SafetyBench-UAV uses four graded risk levels:

|    Risk   |       Condition       | Interpretation   |       Samples | Percentage |
| :-------: | :-------------------: | :--------------- | ------------: | ---------: |
|  🟢 **0** |      $h(s) > 0.8$     | Safe             |     7,353,880 | **98.05%** |
|  🟡 **1** | $0.4 < h(s) \leq 0.8$ | Low Risk         |       123,726 |  **1.65%** |
|  🟠 **2** |  $0 < h(s) \leq 0.4$  | Elevated Risk    |        21,924 |  **0.29%** |
|  🔴 **3** |     $h(s) \leq 0$     | Safety Violation |           470 |  **0.01%** |
| **Total** |                       |                  | **7,500,000** |   **100%** |

### Important Safety Statistic

The worst-case safety margin in the corpus is:

$$
h_{\min} = -0.213.
$$

There are exactly:

$$
470
$$

level-3 samples and exactly **470 samples with $h(s) \leq 0$**.

Thus, by construction:

$$
\boxed{
\text{Risk}=3
\iff
h(s)\leq0
}
$$

The severe-violation class is intentionally rare. Researchers should therefore use class-aware and safety-specific evaluation metrics rather than relying only on overall accuracy.

---

# 🧭 Dataset Contents

Each episode contains the following data:

| Dataset        |    Shape    | Description                    |
| :------------- | :---------: | :----------------------------- |
| `states`       | `(150, 20)` | UAV state at each timestep     |
| `actions`      |  `(150, 4)` | Commanded control inputs       |
| `disturbances` |  `(150, 3)` | Applied exogenous disturbances |
| `cbf_values`   |  `(150, 3)` | Per-step barrier signals       |
| `risk_levels`  |   `(150,)`  | Graded risk labels `{0,1,2,3}` |

---

# 🧩 State Representation

The 20-dimensional state vector contains physical and system-level information:

| Columns | Variable                       |
| :-----: | :----------------------------- |
|  `0:3`  | Position `(x, y, z)`           |
|  `3:6`  | Linear velocity `(vx, vy, vz)` |
|  `6:10` | Orientation quaternion         |
| `10:13` | Angular velocity               |
|   `13`  | Battery                        |
|   `14`  | Motor efficiency               |
|   `15`  | Payload                        |
|   `16`  | GPS quality                    |
| `17:20` | IMU bias                       |

The state therefore includes both **vehicle motion information** and **system-condition information** relevant to safety.

---

# 🛡️ Safety Formulation

SafetyBench-UAV represents safety using a continuous scalar safety margin derived from altitude and velocity.

For a state $s$, let

$$
z=s_2
$$

denote altitude, and let

$$
v=\left\|s_{3:6}\right\|
$$

denote the magnitude of the linear velocity.

---

## Altitude Safety

The altitude margin is

$$
h_{\mathrm{alt}}
=
\frac{\min(z-5,\;20-z)}{5}.
$$

This corresponds to the safe altitude interval

$$
5 \leq z \leq 20\text{ m}.
$$

---

## Velocity Safety

The velocity-related margin is

$$
h_{\mathrm{vel}}
=
\max\left(0,\frac{10-v}{5}\right).
$$

The corresponding velocity limit is

$$
v\leq10\text{ m/s}.
$$

---

## Overall Safety Margin

The overall safety margin is defined as

$$
\boxed{
h(s)=
\min
\left(
h_{\mathrm{alt}},
h_{\mathrm{vel}}
\right)
}
$$

A non-positive margin indicates a physical safety violation.

---

# 🎚️ Graded Risk Labels

The continuous safety margin is quantized into four risk levels:

| Condition         | Risk Level | Interpretation |
| :---------------- | :--------: | :------------- |
| $h(s)>0.8$        |    **0**   | Safe           |
| $0.4<h(s)\leq0.8$ |    **1**   | Low Risk       |
| $0<h(s)\leq0.4$   |    **2**   | Elevated Risk  |
| $h(s)\leq0$       |    **3**   | Violation      |

This gives researchers two complementary targets:

### Discrete risk prediction

$$
r_t\in\{0,1,2,3\}
$$

### Continuous safety estimation

$$
h(s_t)\in\mathbb{R}.
$$

This allows evaluation of both **risk classification** and **safety-margin estimation**.

---

# 🔬 What Can Be Studied with SafetyBench-UAV?

The benchmark is designed to support research beyond conventional binary safety classification.

## 1. Anticipatory Safety Prediction

Given a short trajectory window:

$$
(s_{t-k:t},a_{t-k:t})
$$

can a learned model predict future safety risk?

For example:

$$
\hat{r}_{t:t+H}
\rightarrow
\max_{j\in[t,t+H]}r_j.
$$

---

## 2. Risk Representation Learning

Can a model learn a compact representation of trajectory evolution that captures the progression from:

$$
\text{safe}
\rightarrow
\text{low risk}
\rightarrow
\text{elevated risk}
\rightarrow
\text{violation}?
$$

---

## 3. Risk Calibration

Can predicted risk probabilities be calibrated so that confidence estimates correspond to observed safety outcomes?

The dedicated calibration split enables post-hoc calibration methods such as:

* Probability calibration
* Uncertainty estimation
* Split-conformal prediction

---

## 4. Adaptive Safety Filtering

Can a safety filter change its conservatism according to predicted future risk?

Conceptually:

```text
                 Trajectory History
                        │
                        ▼
                Risk Representation
                        │
                        ▼
                Future Risk Estimate
                        │
                        ▼
              ┌─────────────────────┐
              │   Safety Filter     │
              └─────────────────────┘
                        │
                        ▼
                 Safe Action
```

---

## 5. Disturbance Robustness

The benchmark explicitly records applied disturbances, enabling investigation of:

* disturbance-aware prediction
* robustness to external perturbations
* safety degradation under disturbances
* risk prediction under changing operating conditions

---

## 6. System Degradation

The state contains system-level variables such as:

* battery
* motor efficiency
* payload
* GPS quality
* IMU bias

This enables research into safety prediction under changing system conditions.

---

## 7. Distribution Shift

The metadata includes environment, mission, dynamics, and disturbance information.

This allows researchers to investigate whether a learned safety model generalizes beyond the conditions observed during training.

---

# 🧠 Intended Research Pipeline

A representative research pipeline using SafetyBench-UAV is:

```text
┌─────────────────────────────────────────────┐
│               SAFETYBENCH-UAV               │
│                                             │
│  States · Actions · Disturbances · Risk     │
└──────────────────────┬──────────────────────┘
                       │
                       ▼
             ┌───────────────────┐
             │ Trajectory Window │
             └─────────┬─────────┘
                       │
                       ▼
             ┌───────────────────┐
             │ Risk Representation│
             └─────────┬─────────┘
                       │
                       ▼
             ┌───────────────────┐
             │ Future Risk Model │
             └─────────┬─────────┘
                       │
                       ▼
             ┌───────────────────┐
             │ Risk Calibration  │
             └─────────┬─────────┘
                       │
                       ▼
             ┌───────────────────┐
             │ Adaptive Safety   │
             │      Filter       │
             └─────────┬─────────┘
                       │
                       ▼
             ┌───────────────────┐
             │ Learning-Based    │
             │     Control       │
             └───────────────────┘
```

This pipeline is illustrative rather than a requirement of the benchmark.

Researchers may use the dataset for individual components or for an integrated safety-learning system.

---

# 📦 Repository Structure

```text
SafetyBench-UAV/
│
├── README.md
├── LICENSE
│
├── episodes.h5
│
├── metadata/
│   └── splits.json
│
├── examples/
│   └── load_episode.py
│
└── scripts/
    └── reproduce_statistics.py
```

---

# 🔀 Dataset Splits

The benchmark provides disjoint episode-level splits through:

```text
metadata/splits.json
```

The available partitions are:

```text
                    SAFETYBENCH-UAV
                           │
             ┌─────────────┼─────────────┐
             │             │             │
             ▼             ▼             ▼
          TRAIN       CALIBRATION       TEST
             │             │             │
             ▼             ▼             ▼
        Model fitting   Calibration   Final evaluation
```

The **calibration split must remain separate from training**.

It is intended for post-hoc calibration of risk predictors and should not be used to fit the prediction model.

---

# 💻 Quick Start

## Requirements

The dataset can be loaded using standard Python scientific-computing tools.

Example:

```bash
pip install h5py numpy
```

---

## Load an Episode

```python
import h5py
import json

with h5py.File("SafetyBench-UAV/episodes.h5", "r") as f:

    episode_ids = list(f["episodes"].keys())

    episode = f["episodes"][episode_ids[0]]

    metadata = json.loads(
        episode.attrs["metadata"]
    )

    states = episode["states"][:]
    actions = episode["actions"][:]
    disturbances = episode["disturbances"][:]
    cbf_values = episode["cbf_values"][:]
    risk_levels = episode["risk_levels"][:]

print("States:", states.shape)
print("Actions:", actions.shape)
print("Disturbances:", disturbances.shape)
print("CBF values:", cbf_values.shape)
print("Risk labels:", risk_levels.shape)
```

Expected shapes:

```text
States:        (150, 20)
Actions:       (150, 4)
Disturbances:  (150, 3)
CBF values:    (150, 3)
Risk labels:   (150,)
```

---

# 🔍 Load Dataset Splits

```python
import json

with open(
    "SafetyBench-UAV/metadata/splits.json",
    "r"
) as f:

    splits = json.load(f)

train_ids = splits["train"]["episodes"]
calib_ids = splits["calibration"]["episodes"]
test_ids = splits["test"]["episodes"]

print("Training episodes:", len(train_ids))
print("Calibration episodes:", len(calib_ids))
print("Test episodes:", len(test_ids))
```

---

# 📏 Reproduce Dataset Statistics

The following code reproduces the principal safety statistics directly from the released data:

```python
import h5py
import numpy as np

with h5py.File("SafetyBench-UAV/episodes.h5", "r") as f:

    episodes = f["episodes"]

    H = []
    R = []

    for episode_id in episodes:

        states = episodes[episode_id]["states"][:]

        z = states[:, 2]

        speed = np.sqrt(
            (states[:, 3:6] ** 2).sum(axis=1)
        )

        altitude_margin = (
            np.minimum(z - 5.0, 20.0 - z) / 5.0
        )

        velocity_margin = np.maximum(
            0.0,
            (10.0 - speed) / 5.0
        )

        H.append(
            np.minimum(
                altitude_margin,
                velocity_margin
            )
        )

        R.append(
            episodes[episode_id]
            ["risk_levels"][:]
            .ravel()
        )

h = np.concatenate(H)
r = np.concatenate(R)

print("Total steps:", r.size)

u, c = np.unique(
    np.rint(r).astype(int),
    return_counts=True
)

print(
    "Risk distribution:",
    dict(zip(u.tolist(), c.tolist()))
)

print(
    "Worst-case safety margin:",
    round(float(h.min()), 4)
)

print(
    "Violations:",
    int((h <= 0).sum())
)

print(
    "Risk-3 samples:",
    int((np.rint(r) == 3).sum())
)
```

Expected corpus-level results:

```text
Total steps:              7,500,000
Worst-case safety margin: -0.213
Violations:               470
Risk-3 samples:           470
```

---

# 📥 Dataset Access

The complete dataset is available for research use.

### Download

**[⬇️ Download SafetyBench-UAV](https://drive.google.com/file/d/1El6AFVLi72daQ6DSXFUvWefcqTNmDEyz/view)**

The release contains:

```text
episodes.h5
metadata/splits.json
README.md
LICENSE
```

---

# 📐 Recommended Evaluation

Because the benchmark is highly safety-dominated, researchers should report more than overall classification accuracy.

## Risk Prediction

Recommended metrics include:

* Macro-F1
* Per-class precision
* Per-class recall
* Violation recall
* False-positive rate
* Confusion matrix

## Safety-Margin Prediction

Possible metrics include:

* MAE
* RMSE
* Correlation with the true safety margin
* Near-boundary prediction performance

## Calibration

Possible metrics include:

* Calibration error
* Prediction-set coverage
* Risk-specific coverage
* Reliability diagrams

## Safety Filtering

Recommended evaluation includes:

* Violation rate
* Safety-margin statistics
* Safety-filter intervention rate
* Performance degradation
* Robustness under disturbances
* Generalization under distribution shift

---

# ⚠️ Scope and Limitations

SafetyBench-UAV is intentionally designed as a focused benchmark for safety learning rather than a complete UAV flight simulator.

## Simplified UAV Dynamics

The trajectories originate from a simplified **vertical UAV altitude-control task**.

Although the action space contains thrust and torque channels, the dynamics used for this release primarily exercise the vertical axis.

Therefore:

> **SafetyBench-UAV should be interpreted as an altitude-safety benchmark rather than a full 6-DoF UAV flight benchmark.**

Results should be interpreted within this scope.

---

## Highly Safety-Dominated Distribution

The benchmark intentionally reflects a safety-dominated operating distribution:

$$
P(r=3)=0.01\%.
$$

Severe violations are therefore rare.

This makes the benchmark useful for studying the practical problem of **rare safety events**, but it also means that overall accuracy can hide poor performance on the critical violation class.

Researchers should report class-aware and safety-specific metrics.

---

## Risk Labels Are Derived from the Safety Margin

Risk levels are generated from the defined safety-margin formulation.

They should therefore be interpreted as **graded safety states derived from the benchmark's specified safety constraints**, rather than as human annotations.

---

# 🔬 Reproducibility

SafetyBench-UAV is designed so that its core statistics and safety labels can be independently checked from the released trajectory data.

In particular:

* The safety margin can be computed directly from `states`.
* Risk levels are explicitly stored in `risk_levels`.
* Level-3 risk corresponds to $h(s)\leq0$.
* Episode-level train/calibration/test splits are provided.
* Corpus-level statistics can be reproduced using standard Python tools.
* No proprietary software is required to read the released HDF5 dataset.

The goal is to support **transparent, reproducible, and independently auditable safety-learning experiments**.

---

# 🔭 Future Research Directions

SafetyBench-UAV is intended to provide a foundation for research in:

```text
Trajectory Representation Learning
                ↓
Future Safety-Risk Prediction
                ↓
Uncertainty & Calibration
                ↓
Safety-Margin Estimation
                ↓
Adaptive Safety Filtering
                ↓
Risk-Aware Reinforcement Learning
                ↓
Learning-Based Control
                ↓
Robust Autonomous Operation
```

Potential future directions include:

* Temporal safety-risk representation learning
* Future violation prediction
* Uncertainty-aware risk estimation
* Conformal safety prediction
* Disturbance-aware risk prediction
* Adaptive safety filtering
* Risk-sensitive reinforcement learning
* Model-predictive safety filtering
* Distribution-shift evaluation
* Integration with formal safety mechanisms

---

# 📚 Associated Research

SafetyBench-UAV was developed as part of ongoing research on:

> **Anticipatory safety filtering for UAVs via calibrated latent risk representations.**

The benchmark is intended to support investigation of the relationship between:

$$
\boxed{
\text{Trajectory Learning}
\rightarrow
\text{Risk Prediction}
\rightarrow
\text{Calibration}
\rightarrow
\text{Safety Filtering}
\rightarrow
\text{Safe Control}
}
$$

The associated manuscript is currently under journal review.

Citation details are intentionally withheld during the review process and will be added following publication.

---

# 📖 Citation

Citation information is currently withheld during the review process.

A citation entry will be added after publication.

```bibtex
@dataset{SafetyBenchUAV,
  title        = {SafetyBench-UAV},
  author       = {To be added after publication},
  year         = {2026},
  publisher    = {To be added},
  version      = {1.0},
  note         = {Large-scale offline benchmark for UAV safety-risk
                  prediction and safety filtering}
}
```

---

# 📜 License

SafetyBench-UAV is released under the:

**Creative Commons Attribution 4.0 International (CC BY 4.0)**

See [`LICENSE`](LICENSE) for the complete license terms.

---

# 🤝 Contributing

Research use, independent evaluation, and methodological extensions are welcome.

If you use SafetyBench-UAV in your research, please cite the associated dataset/manuscript once the citation information becomes available.

For questions, issues, or suggestions, please open a GitHub issue in this repository.

---

<div align="center">

## 🛡️ SafetyBench-UAV

### Learning to anticipate safety before a violation occurs.

**50,000 Episodes · 7.5 Million Timesteps · 5 Environments · 8 Missions**

</div>
```

### Two important notes before you paste it

**1. Your equations should now render correctly.**
The key difference from your screenshot is that every mathematical expression is enclosed as a complete block:

```markdown
$$
h_{\mathrm{alt}}
=
\frac{\min(z-5,\;20-z)}{5}
$$
```
