# 🛡️ SafetyBench-UAV

### A Large-Scale Offline Benchmark for Risk-Aware UAV Safety Learning

**SafetyBench-UAV** is a large-scale offline benchmark of UAV altitude-control trajectories designed to study **anticipatory safety prediction, calibrated risk estimation, and adaptive safety filtering for learning-based control**.

The benchmark contains **50,000 trajectories and 7.5 million labeled timesteps**, covering diverse operating environments, missions, disturbances, and system conditions. Each timestep contains UAV state, control action, external disturbance, barrier-related safety signals, and a graded safety-risk label.

> **Core idea:** Instead of asking only whether a UAV is currently safe, SafetyBench-UAV enables research on whether a learning system can **anticipate future safety risk and use that prediction to adapt its control behavior.**

---

## ✨ Why SafetyBench-UAV?

Learning-based control has demonstrated significant potential for autonomous systems, but safety remains a fundamental challenge when learned policies operate under disturbances, uncertainty, and changing system conditions.

SafetyBench-UAV provides a controlled benchmark for investigating:

* 🧠 **Learning to anticipate future safety risk**
* 📈 **Calibrated risk prediction**
* 🛡️ **Adaptive safety filtering**
* 🌪️ **Robustness under disturbances**
* ⚙️ **Safety under actuator/system degradation**
* 🔄 **Generalization under distribution shift**
* 📐 **Learning with continuous safety margins**
* 🎯 **Risk-aware learning-based control**

The benchmark intentionally contains a highly safety-dominated distribution, reflecting the fact that severe safety violations are rare during normal operation.

---

# 📊 Dataset at a Glance

|                                   |                      |
| --------------------------------- | -------------------: |
| **Episodes**                      |           **50,000** |
| **Total timesteps**               |        **7,500,000** |
| **Steps / episode**               |              **150** |
| **State dimension**               |               **20** |
| **Action dimension**              |                **4** |
| **Disturbance dimension**         |                **3** |
| **Risk levels**                   | **4 — {0, 1, 2, 3}** |
| **Environments**                  |                **5** |
| **Mission types**                 |                **8** |
| **Zero-disturbance trajectories** |           **37.99%** |

### Environments

**Indoor · Urban · Forest · Open Field · Dynamic**

### Missions

**Hover · Tracking · Swarm · Emergency · Inspection · Pursuit · Navigation · Avoidance**

---

# 🚨 Safety-Risk Distribution

SafetyBench-UAV provides **graded risk labels** rather than a simple safe/unsafe classification.

| Risk Level | Interpretation   |       Samples | Percentage |
| :--------: | ---------------- | ------------: | ---------: |
|  🟢 **0**  | Safe             |     7,353,880 | **98.05%** |
|  🟡 **1**  | Low risk         |       123,726 |  **1.65%** |
|  🟠 **2**  | Elevated risk    |        21,924 |  **0.29%** |
|  🔴 **3**  | Safety violation |           470 |  **0.01%** |
|  **Total** |                  | **7,500,000** |   **100%** |

The benchmark contains **470 level-3 violations**, corresponding exactly to the **470 timesteps with safety margin ≤ 0**.

> **Important:** Because severe violations are intentionally rare, conventional accuracy can be misleading. Class-aware evaluation and safety-focused metrics are recommended.

---

# 🧭 What Does Each Timestep Contain?

Each trajectory records the physical state, control command, external disturbance, barrier-related safety signals, and risk level.

```text
┌─────────────────────────────────────────────────────────────┐
│                    UAV TRAJECTORY                            │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   STATE ────────────────┐                                   │
│   20 dimensions          │                                   │
│                          ▼                                   │
│   ACTION ───────────► UAV DYNAMICS ───────► NEXT STATE      │
│   4 dimensions           ▲                                   │
│                          │                                   │
│   DISTURBANCE ───────────┘                                   │
│   3 dimensions                                              │
│                                                             │
│   SAFETY SIGNALS ─────────────► RISK LEVEL {0,1,2,3}        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

Each episode contains:

```text
states          → (150, 20)
actions         → (150, 4)
disturbances    → (150, 3)
cbf_values      → (150, 3)
risk_levels     → (150,)
```

---

# 🧩 State Representation

The 20-dimensional state contains both physical and system-level information.

| Dimensions | Variable                       |
| ---------- | ------------------------------ |
| `0:3`      | Position `(x, y, z)`           |
| `3:6`      | Linear velocity `(vx, vy, vz)` |
| `6:10`     | Orientation quaternion         |
| `10:13`    | Angular velocity               |
| `13`       | Battery                        |
| `14`       | Motor efficiency               |
| `15`       | Payload                        |
| `16`       | GPS quality                    |
| `17:20`    | IMU bias                       |

The benchmark therefore includes not only motion state, but also system conditions that can influence safety.

---

# 🛡️ Safety Formulation

SafetyBench-UAV defines a continuous safety margin from altitude and velocity.

For a state \(s\), let

$$
z = s_2
$$

denote altitude and

$$
v = \left\|s_{3:6}\right\|
$$

denote the magnitude of linear velocity.

The altitude margin is

$$
h_{\text{alt}}
=
\frac{\min(z-5,\;20-z)}{5},
$$

corresponding to the safe altitude interval

$$
5 \leq z \leq 20\text{ m}.
$$

The velocity-related margin is

$$
h_{\text{vel}}
=
\max\left(0,\frac{10-v}{5}\right),
$$

with a velocity limit of

$$
v \leq 10\text{ m/s}.
$$

The overall safety margin is

$$
\boxed{
h(s)=
\min\left(h_{\text{alt}},h_{\text{vel}}\right)
}
$$

---

# 🎚️ Graded Risk Labels

The continuous safety margin is converted into four discrete risk levels:

| Safety condition        |          Risk         |
| ----------------------- | :-------------------: |
| \(h(s) > 0.8\)          |      **0 — Safe**     |
| \(0.4 < h(s) \leq 0.8\) |    **1 — Low Risk**   |
| \(0 < h(s) \leq 0.4\)   | **2 — Elevated Risk** |
| \(h(s) \leq 0\)         |   **3 — Violation**   |

This provides two complementary evaluation targets:

**Discrete risk prediction**

$$
\{0,1,2,3\}
$$

and

**Continuous safety-margin prediction**

$$
h(s)\in\mathbb{R}.
$$

This distinction enables research beyond conventional binary safety classification.

---

# 🔬 Research Questions Enabled by the Benchmark

SafetyBench-UAV is designed to support research questions such as:

### 01 — Can safety risk be anticipated?

Given a short history of UAV states and actions, can a learned representation predict the **maximum future risk over a look-ahead horizon**?

### 02 — Can predictions be calibrated?

Can a risk predictor provide reliable uncertainty estimates rather than only point predictions?

### 03 — Can risk prediction improve control?

Can predicted future risk be used to make a safety filter **adaptive rather than permanently conservative**?

### 04 — How does safety prediction behave under disturbance?

How does prediction quality change with external disturbances and changing operating conditions?

### 05 — Can learned safety mechanisms generalize?

Can a risk-aware model remain reliable under different environments, missions, dynamics profiles, and system conditions?

### 06 — Can calibration provide statistical guarantees?

The held-out calibration split enables research into methods such as **post-hoc calibration and split-conformal prediction**.

---

# 🧠 Intended Research Pipeline

A natural research workflow using SafetyBench-UAV is:

```text
                    SAFETYBENCH-UAV
                           │
                           ▼
                  Trajectory Window
                           │
                           ▼
                 ┌──────────────────┐
                 │  Risk Predictor   │
                 └──────────────────┘
                           │
                           ▼
                  Future Risk / h(s)
                           │
                           ▼
                 ┌──────────────────┐
                 │ Risk Calibration  │
                 └──────────────────┘
                           │
                           ▼
                 ┌──────────────────┐
                 │ Safety Filter     │
                 └──────────────────┘
                           │
                           ▼
                  Learning-Based Control
                           │
                           ▼
                  Safe UAV Behavior
```

The benchmark therefore supports a complete research loop from **trajectory data → risk representation → calibration → safety filtering → learning-based control**.

---

# 📦 Dataset Structure

```text
SafetyBench-UAV/
│
├── episodes.h5
│
│   └── episodes/
│       ├── episode_000000/
│       │   ├── states
│       │   ├── actions
│       │   ├── disturbances
│       │   ├── cbf_values
│       │   ├── risk_levels
│       │   └── metadata
│       │
│       ├── episode_000001/
│       ├── episode_000002/
│       └── ...
│
├── metadata/
│   └── splits.json
│
├── README.md
│
└── LICENSE
```

The episode metadata records information including:

* scenario ID
* random seed
* environment
* mission
* dynamics profile
* disturbance profile
* payload/mass
* episode length

---

# 🔀 Reproducible Dataset Splits

The benchmark provides disjoint episode-level splits:

```text
                    SAFETYBENCH-UAV
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
          TRAIN         CALIBRATION      TEST
             │             │             │
        Model fitting   Calibration    Final evaluation
```

The **calibration split must remain separate from training** and is intended for post-hoc calibration of the risk predictor.

This separation enables reproducible evaluation of calibrated safety prediction.

---

# 💻 Quick Start

## Load an episode

```python
import h5py
import json

with h5py.File("SafetyBench-UAV/episodes.h5", "r") as f:

    ep_ids = list(f["episodes"].keys())

    ep = f["episodes"][ep_ids[0]]

    metadata = json.loads(ep.attrs["metadata"])

    states = ep["states"][:]
    actions = ep["actions"][:]
    disturbances = ep["disturbances"][:]
    cbf_values = ep["cbf_values"][:]
    risk_levels = ep["risk_levels"][:]
```

The resulting arrays have the following structure:

```text
states          (150, 20)
actions         (150, 4)
disturbances    (150, 3)
cbf_values      (150, 3)
risk_levels     (150,)
```

---

# 📏 Reproduce the Safety Statistics

The benchmark statistics can be independently reproduced directly from the trajectory data.

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
            episodes[episode_id]["risk_levels"][:].ravel()
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

The expected corpus-level values are:

```text
Total steps:              7,500,000
Worst-case margin:        -0.213
Violations:               470
Risk-3 samples:           470
```

---

# 📥 Dataset Access

The complete dataset is provided as an HDF5 trajectory corpus together with metadata and reproducible episode-level splits.

### Dataset

**SafetyBench-UAV — 50,000 Episodes · 7.5 Million Timesteps**

**Download:**
`<INSERT YOUR GOOGLE DRIVE DATASET LINK HERE>`

> The dataset is released for research and reproducibility under the **Creative Commons Attribution 4.0 International (CC BY 4.0)** license.

---

# 📚 Scope and Limitations

SafetyBench-UAV is intentionally designed as a focused benchmark rather than a complete UAV flight simulator.

### Current scope

The released trajectories come from a **simplified vertical UAV altitude-control task**.

Although the action space contains thrust and torque channels, the current dynamics primarily exercise the vertical axis.

Therefore:

> **SafetyBench-UAV should be interpreted as an altitude-safety benchmark rather than a full 6-DoF UAV flight benchmark.**

This limitation is important when interpreting results and comparing methods.

### Safety imbalance

Safety violations are intentionally rare:

$$
P(\text{risk}=3)=0.01\%.
$$

This reflects the safety-dominated nature of autonomous operation but creates a challenging class imbalance.

Researchers should therefore report appropriate class-aware and safety-specific metrics rather than relying solely on overall accuracy.

---

# 🎯 Recommended Evaluation

For research using this benchmark, we recommend reporting performance at multiple levels:

### Risk prediction

* Macro-F1
* Per-risk-level precision/recall
* Confusion matrix
* Violation recall
* False-alarm rate

### Continuous safety estimation

* MAE / RMSE of predicted safety margin
* Correlation with true margin
* Near-boundary prediction performance

### Calibration

* Calibration error
* Coverage of prediction sets
* Risk-specific coverage

### Safety filtering

* Violation rate
* Safety-margin statistics
* Intervention rate
* Performance degradation
* Robustness under disturbances
* Generalization under distribution shift

---

# 🔭 Future Research Directions

SafetyBench-UAV is intended to serve as a foundation for research on:

```text
Trajectory Representation
        ↓
Future Risk Prediction
        ↓
Uncertainty / Calibration
        ↓
Safety-Margin Estimation
        ↓
Adaptive Safety Filtering
        ↓
Learning-Based Control
        ↓
Robust Autonomous Operation
```

Potential extensions include:

* temporal risk representation learning
* uncertainty-aware risk prediction
* conformal safety prediction
* disturbance-aware risk estimation
* adaptive safety filters
* safety-aware reinforcement learning
* distribution-shift evaluation
* risk-sensitive control
* model-predictive safety filtering
* integration with formal safety mechanisms

---

# 🧪 Reproducibility Philosophy

SafetyBench-UAV is designed so that the core dataset statistics and safety labels can be independently verified from the released trajectory data.

In particular:

* The safety margin is directly computable from the recorded state.
* Risk levels are explicitly stored.
* Level-3 risk corresponds exactly to a non-positive safety margin.
* Episode-level train/calibration/test splits are provided.
* Corpus statistics can be reproduced without proprietary software.
* The dataset format is directly readable using standard Python scientific-computing tools.

Our goal is to make safety-learning experiments **transparent, reproducible, and independently auditable**.

---

# 📖 Citation

Citation information is intentionally withheld during double-blind review and will be added following publication.

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

# 📄 Associated Research

SafetyBench-UAV was developed as part of research on **anticipatory safety filtering for learning-based UAV control**.

The benchmark is intended to support reproducible investigation of the relationship between:

**trajectory learning → safety-risk prediction → calibrated uncertainty → adaptive filtering → safe control.**

---

# 📝 License

SafetyBench-UAV is released under the:

**Creative Commons Attribution 4.0 International (CC BY 4.0)**

See [`LICENSE`](LICENSE) for the complete license terms.

---

<div align="center">

### 🛡️ SafetyBench-UAV

**Learning to anticipate safety before a violation occurs.**

*50,000 Episodes · 7.5 Million Timesteps · 5 Environments · 8 Missions*

</div>
