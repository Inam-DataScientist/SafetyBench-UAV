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
SafetyBench-UAV instead provides graded safety-risk labels together with a continuous safety margin:

</div>

## 🌍 Environments

The benchmark contains trajectories from five operating environments:

| Environment   |
| :------------ |
| 🏢 Indoor     |
| 🏙️ Urban     |
| 🌲 Forest     |
| 🌾 Open Field |
| 🌐 Dynamic    |
## 🚨 Safety-Risk Distribution

SafetyBench-UAV uses four graded risk levels:
