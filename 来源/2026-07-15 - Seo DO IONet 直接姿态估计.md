---
type: source
tags: [imu, deep-learning, inertial-odometry, neural-network, orientation]
created: 2026-07-15
updated: 2026-07-15
---

# Seo et al. — DO IONet: 9-Axis IMU 6-DOF Odometry via Direct Orientation Estimation

## Bibliographic Reference

- **Title**: DO IONet: 9-Axis IMU-Based 6-DOF Odometry Framework Using Neural Network for Direct Orientation Estimation
- **Authors**: Hong-Il Seo, Ju-Won Bae, Won-Yeol Kim, Dong-Hoan Seo (Korea Maritime & Ocean University / Chosun University)
- **Journal**: IEEE Access, Vol. 11, pp. 55380–55389, 2023
- **DOI**: 10.1109/ACCESS.2023.3281970
- **License**: CC BY-NC-ND 4.0
- **Received** 27 April 2023, **Accepted** 28 May 2023, **Published** 1 June 2023
- **PDF**: `参考/论文/Seo 2023 - DO IONet 9-Axis IMU 6-DOF Odometry.pdf`

## Abstract

> With the development of mobile devices, low-cost inertial measurement unit (IMU)-based research on inertial odometry for indoor localization is being actively conducted. Inertial odometry estimates the amount of change in position and orientation relative to the initial value based on the data measured by the IMU and creates a trajectory via a multi-integration process. However, existing approaches primarily focus on estimating the position in a two-dimensional (2D) plane. Additionally, drift errors occur because the position is estimated by integrating the position change and the orientation change. Herein, we propose a novel six-degree of freedom inertial odometry framework that directly estimates the orientation to minimize drift errors. The proposed framework is a direct-orientation inertial odometry network (DO IONet) that outputs the velocity and orientation of the IMU by using linear acceleration, gyro data, gravitational acceleration, and geomagnetic data as inputs to structurally eliminate drift errors that occur during orientation calculations. DO IONet is composed of a convolutional neural network-based encoder to extract features from inertial data and decoder to extract sequential features. Structurally, it does not require initial values and has no cumulative error despite estimating orientation over tens of seconds.

## Key Contributions

1. **First IONet to directly estimate orientation (q) rather than orientation change (Δq)** — eliminates the integration step that causes drift in all prior inertial odometry systems.
2. **Structural drift elimination**: Because q is estimated independently per window (not integrated from a prior state), orientation errors cannot accumulate. This is a framework-level innovation, not merely an improvement in filtering or network architecture.
3. **No initial orientation required**: The reference-coordinate orientation is regressed directly from IMU raw data, so no initial pose calibration is needed — useful for ad-hoc sensor placement (smartphones, wearables).
4. **Demonstrates the necessity of all 9 axes**: 9-axis IONet (a+ω+g+m → Δp+Δq) and 6-axis IONet (a+ω only) are compared side-by-side. The 9-axis input is necessary; geomagnetic data provides the absolute yaw reference without which heading drifts.

## Architecture Summary

**Encoder**: 4 independent 1D CNN branches (one per input type: linear acceleration a, angular velocity ω, gravitational acceleration g, geomagnetic m). Each branch: Conv1D (128 filters) → Conv1D (64 filters) → MaxPooling. Outputs concatenated.

**Decoder**: 2-layer Bi-LSTM (128 units each) → Fully Connected → Δp (3D body-frame translation) + q (unit quaternion in reference frame).

**Window**: 200 frames (2 s @ 100 Hz), stride 10 frames.

**Loss**: Multi-task loss (homoscedastic uncertainty weighting) combining L_DPMAE (L1 on Δp) + L_QME (quaternion multiplicative error).

**Parameters**: 1,793,287 (vs. 1,161,735 for 6-axis IONet).

## Dataset

**OxIOD** (Oxford Inertial Odometry Dataset): iPhone 7 IMU at 100 Hz, Vicon motion capture ground truth. Handheld sequences only. 17 training + 6 testing sequences. Preprocessing: removed first 12 s / last 3 s per sequence, removed samples with > 20 deg inter-frame orientation change. Total training samples: 54,885.

**Training setup**: TensorFlow 1.13.1, Keras 2.3.1, Nvidia RTX 6000, Adam lr=1e-4, batch 32, 500 epochs.

## Key Results

| Metric | Short-term (0–20 s / 100–120 s) | Long-term (100–200 s) |
|--------|--|--|
| Trajectory RMSE | DO IONet 4/14 best; 9-axis IONet has better mean | _Not evaluated for long trajectory_ |
| Orientation RMSE | DO IONet 5/14 best; **lowest mean** | **DO IONet dominates: lowest mean in almost all sequences, does NOT diverge** |
| 3D Translation RMSE | 9-axis IONet 9/14 best | **DO IONet best in most sequences, does NOT diverge** |

**Critical finding**: DO IONet does not always win on short-term trajectory accuracy, but it is the **only method whose orientation error does not grow with time**. 6-axis and 9-axis IONet both diverge in orientation over 100+ seconds; DO IONet's long-term error is comparable to its short-term error.

## Tables in Original Paper

| Table | Content |
|-------|---------|
| 1 | 17 training + 6 testing OxIOD sequences (data1–data6 split) |
| 2 | Trajectory RMSE per testing sequence, 0–20 s and 100–120 s |
| 3 | Short-term orientation RMSE (0–20 s, 100–120 s) |
| 4 | Long-term orientation RMSE (0–100 s, 100–200 s) — DO IONet wins almost all |
| 5 | Short-term 3D translation RMSE — 9-axis IONet leads |
| 6 | Long-term 3D translation RMSE — DO IONet best in most |

## Figures

| Figure | Content |
|--------|---------|
| 1 | Framework overview — data flow from 9-axis IMU to trajectory |
| 2 | Network architecture — 4 independent CNN branches → Bi-LSTM → Δp + q |
| 3 | Qualitative trajectory (data1/seq2, 0–20 s) — DO IONet matches ground truth closely |
| 4 | Qualitative trajectory (data5/seq1, 100–120 s) — 6-axis drifts, 9-axis distorts, DO IONet tracks |
| 5 | Orientation error norm over time — DO IONet stays bounded; 6-axis and 9-axis diverge |

## 2024 Extension (JAMET)

**Reference**: Han, Kim, Seo, Seo, "Direct orientation estimation through inertial odometry based on a deep transformer model," JAMET 48(2):96–106, 2024. DOI: 10.5916/jamet.2024.48.2.96

Key change: Bi-LSTM decoder → 2-layer Transformer block (multi-head self-attention + FFN). CNN encoder unchanged. Achieves >10% reduction in both orientation and translation error over the 2023 version.

## Limitations Noted by the Reviewer

- Short-term trajectory RMSE is not always best — the tradeoff for long-term stability
- No uncertainty/confidence output — single point estimate only
- Only position-_change_ drift is prevented; position _accumulation_ drift (from Δp integration) still exists
- OxIOD-only evaluation — generalization to other IMUs, attachment positions, and motion patterns unvalidated
- No evaluation on resource-constrained platforms (MCU, DSP)
- Geomagnetic interference robustness not addressed

## Related Pages

- [DO IONet Transformer直接姿态](../知识/姿态解算/DO%20IONet%20Transformer直接姿态.md) — Full knowledge page with detailed analysis
- [IMU姿态解算算法演进](../知识/姿态解算/IMU姿态解算算法演进.md) — Algorithm landscape
