# SafetyBench-UAV

SafetyBench-UAV is a large-scale offline benchmark of UAV altitude-control
trajectories with graded safety-risk labels, logged safety signals, injected
disturbances, and varied operating conditions. It is intended for research on
learning to anticipate safety risk from trajectory data and on safety filtering
for learning-based control.

> Anonymized for double-anonymous review. Author, affiliation, funding, and
> citation details are omitted and will be added after acceptance.

---

## At a glance

| Property | Value |
|---|---|
| Episodes | 50,000 |
| Steps per episode | 150 |
| Total labeled timesteps | 7,500,000 |
| State dimension | 20 |
| Action dimension | 4 |
| Disturbance dimension | 3 |
| Risk labels | graded, {0, 1, 2, 3} |
| Environments | 5 (indoor, urban, forest, open field, dynamic) |
| Missions | 8 (hover, tracking, swarm, emergency, inspection, pursuit, navigation, avoidance) |
| Zero-disturbance fraction | 37.99% |

Risk-level distribution:

| Risk level | Samples | Percentage |
|---|---|---|
| 0 (safe) | 7,353,880 | 98.05% |
| 1 | 123,726 | 1.65% |
| 2 | 21,924 | 0.29% |
| 3 (violation) | 470 | 0.01% |

Worst-case safety margin over the corpus: `h_min = -0.213`. Level-3 labels
coincide exactly with margin violations (`h <= 0`); there are 470 of each.

---

## Contents

```
SafetyBench-UAV/
  episodes.h5            # trajectory corpus (HDF5)
  metadata/
    splits.json          # train / calibration / test episode-id splits
  README.md              # this file
```

---

## Data format

Episodes are grouped under `/episodes`, one subgroup per episode id. Each episode
subgroup carries a JSON `metadata` attribute (scenario id, seed, environment,
mission, dynamics profile, disturbance profile, mass, and length) and the
following per-timestep arrays:

| Dataset | Shape | Description |
|---|---|---|
| `states` | (150, 20) | UAV state per step (see state layout below) |
| `actions` | (150, 4) | commanded control (thrust and torque channels) |
| `disturbances` | (150, 3) | applied exogenous disturbance |
| `cbf_values` | (150, 3) | per-step barrier signals for the three constraints |
| `risk_levels` | (150,) | graded risk label in {0, 1, 2, 3} |

State layout (columns of `states`):

| Columns | Meaning |
|---|---|
| 0:3 | position (x, y, z); z is altitude |
| 3:6 | linear velocity (vx, vy, vz) |
| 6:10 | orientation quaternion |
| 10:13 | angular velocity |
| 13 | battery |
| 14 | motor efficiency |
| 15 | payload |
| 16 | GPS quality |
| 17:20 | IMU bias |

Load an episode:

```python
import h5py, json

with h5py.File("SafetyBench-UAV/episodes.h5", "r") as f:
    ep_ids = list(f["episodes"].keys())
    ep = f["episodes"][ep_ids[0]]
    meta    = json.loads(ep.attrs["metadata"])
    states  = ep["states"][:]        # (150, 20)
    actions = ep["actions"][:]       # (150, 4)
    dist    = ep["disturbances"][:]  # (150, 3)
    risk    = ep["risk_levels"][:]   # (150,)
```

---

## Safety labels

The scalar safety margin is derived from the state as the minimum of an altitude
term and a speed term:

```python
import numpy as np

def safety_margin(state):
    z = state[2]
    speed = np.linalg.norm(state[3:6])
    alt_margin = min(z - 5.0, 20.0 - z) / 5.0     # safe altitude band [5, 20] m
    vel_margin = max(0.0, (10.0 - speed) / 5.0)   # speed limit 10 m/s
    return min(alt_margin, vel_margin)
```

The graded label is a quantization of this margin:

| Condition | Risk level |
|---|---|
| margin <= 0.0 | 3 (violation) |
| 0.0 < margin <= 0.4 | 2 |
| 0.4 < margin <= 0.8 | 1 |
| margin > 0.8 | 0 |

By construction, a level-3 label is equivalent to a physical violation
(margin <= 0). This equivalence and the corpus statistics can be reproduced
directly from `states` and `risk_levels`.

---

## Splits

`metadata/splits.json` defines disjoint episode-id lists for the training,
calibration, and test partitions. The calibration split is held out and is
intended for post-hoc calibration of a risk predictor; it must not be used for
training.

```python
import json
splits = json.load(open("SafetyBench-UAV/metadata/splits.json"))
train_ids = splits["train"]["episodes"]
calib_ids = splits["calibration"]["episodes"]
```

---

## Reproducing the statistics

The following prints the total step count, the risk-level distribution, the
worst-case margin, and the violation count, all directly from the file:

```python
import h5py, numpy as np

with h5py.File("SafetyBench-UAV/episodes.h5", "r") as f:
    e = f["episodes"]
    H, R = [], []
    for k in e:
        s = e[k]["states"][:]
        z, speed = s[:, 2], np.sqrt((s[:, 3:6] ** 2).sum(1))
        am = np.minimum(z - 5.0, 20.0 - z) / 5.0
        vm = np.maximum(0.0, (10.0 - speed) / 5.0)
        H.append(np.minimum(am, vm))
        R.append(e[k]["risk_levels"][:].ravel())
    h = np.concatenate(H); r = np.concatenate(R)

print("total steps:", r.size)
u, c = np.unique(np.rint(r).astype(int), return_counts=True)
print("risk distribution:", dict(zip(u.tolist(), c.tolist())))
print("h_min:", round(float(h.min()), 4))
print("violations (h<=0):", int((h <= 0).sum()),
      "| count(risk==3):", int((np.rint(r) == 3).sum()))
```

---

## Intended use

- Learning a representation of a short trajectory window that predicts future
  safety risk (e.g., the maximum risk over a look-ahead horizon).
- Calibrating a risk predictor on the held-out split (e.g., split-conformal
  prediction) to obtain distribution-free error bounds.
- Studying safety filters that adapt their conservatism to predicted risk under
  disturbances and actuator degradation.

The graded labels allow evaluation on both discrete risk prediction and the
continuous margin, and the disturbance and dynamics-profile metadata support
stress testing under distribution shift.

---

## Scope and limitations

- The trajectories come from a simplified vertical (altitude) UAV control task.
  The action includes thrust and torque channels, but the dynamics used for this
  release primarily exercise the vertical axis; treat the corpus as an
  altitude-safety benchmark rather than a full 6-DoF flight benchmark.
- The label distribution is intentionally safety-dominated, mirroring real
  operation; violation samples are rare (0.01%). Report class-aware metrics
  accordingly.

---

## License

Released under the Creative Commons Attribution 4.0 (CC BY 4.0) license.
See `LICENSE`.

---

## Citation

Citation details are withheld during double-anonymous review and will be added
here upon acceptance.
