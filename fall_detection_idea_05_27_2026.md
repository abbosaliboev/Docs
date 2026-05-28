# PGST-GNN: Physics-Guided Spatial-Temporal Graph Neural Network for Real-Time Fall Detection on Construction Sites

**Researcher:** Abbos Aliboev — CBNU DaLab  
**Date:** 2026.05  
**Target Journal:** Automation in Construction (IF 11.5, Q1)  
**Hardware:** Jetson Nano + Jetson Orin NX

---

## 1. Research Context

### Problem
Falls are the #1 cause of fatal occupational injuries on construction sites globally (WHO, ILO). Existing fall detection systems fail in construction environments due to:

- **Slow inference** — YOSAP-LSTM: 6.44 FPS (not real-time)
- **Wrong domain** — TCNTE: indoor elderly only
- **Single sensor** — ABC-HF: mmWave radar, single person only
- **Occlusion** — AlphaPose extracts skeletons in only 76.21% of construction workers

### Goal
A method paper proposing a **genuinely new architecture** for real-time, multi-person fall detection on construction sites, deployable on low-cost edge devices.

```
Target: 25+ FPS (Orin NX), 15+ FPS (Nano), 97%+ accuracy
```

---

## 2. Literature Review — Papers Studied

### 2.1 YOSAP-LSTM (Kim, Yu, Xiong — 2025)
**Journal:** Automation in Construction, IF 11.5, Q1  
**Pipeline:** YOLOv8s → SORT → AlphaPose → 1D CNN-LSTM

| Metric | Value |
|--------|-------|
| Accuracy | 98.66% (total), 97.44% (Ai-Hub) |
| FPS | 6.44 (Jetson Xavier NX) |
| Domain | Construction ✓ |

**Strengths:**
- First construction-specific skeleton fall detection
- Multi-person (SORT), real construction dataset (Ai-Hub)
- Published in IF 11.5 Q1 journal

**Weaknesses:**
- 6.44 FPS — not real-time (standard CCTV = 25 FPS)
- AlphaPose: 76.21% skeleton extraction on construction (24% invisible)
- No TensorRT, no physics features
- Closed dataset

**Key insight:** 86% of training data is indoor (KFall) — 98.66% accuracy is overstated. Real construction accuracy is 97.44%.

---

### 2.2 TCNTE (Yu, Wang, Wu, Xiong — 2025)
**Journal:** Pervasive and Mobile Computing, IF 3.8, Q2  
**Pipeline:** YOLOv8-Pose → BoT-SORT → TCN + Transformer (parallel)

| Metric | Value |
|--------|-------|
| Accuracy | 99.58% (UP-Fall), 97.01% (Le2i), 92.99% (GMDCSA-24) |
| FPS | 19 (Jetson Orin NX) |
| Domain | Indoor elderly only |

**Strengths:**
- Fastest edge FPS of all reviewed papers
- Open datasets (UP-Fall, Le2i, GMDCSA-24) — reproducible
- YOLOv8-Pose shared backbone — 3x faster than AlphaPose
- Weighted focal loss for class imbalance

**Weaknesses:**
- Indoor elderly only — no construction domain
- Parallel TCN + Transformer — runs both every frame (wasteful on edge)
- No occlusion testing
- Alert latency not measured

**Note:** Same corresponding author (Shuping Xiong) as YOSAP-LSTM.

---

### 2.3 ABC-HF (Huh, Jeong, Lee, Lee, Shin — 2025)
**Journal:** Engineering Applications of AI, IF 8.0, Q1  
**Sensor:** mmWave radar (77GHz)  
**Method:** Oversampling → VAE encoder → Soft clustering → P+V+A (HF-Descriptor) → Anomaly detection

| Metric | Value |
|--------|-------|
| F1 | 0.942 (lab), 0.842 (industrial + stairs) |
| FPS | Not tested on edge |
| Domain | Industrial (not construction) |

**Key innovations:**
- Physics-informed P+V+A features — genuinely novel
- No fall data needed (anomaly detection)
- Kinematic loss: 2·a·Δx = Δv²
- Tested on real industrial stairs

**Weaknesses:**
- mmWave radar — expensive hardware
- Single person only
- Only 5 participants, confidential dataset
- Edge deployment not tested

---

### 2.4 HOG+CNN-LSTM (India — 2026)
**Journal:** IEEE ICAECT 2026 (conference)  
**Critical flaw:** Dataset section contains unfilled placeholder "[Insert Dataset Name]"  
**Rating:** 2.5/5 — not reliable

---

### 2.5 StoneGAT (Chun, Song, Lee, Lee — 2025)
**Journal:** Int. Journal of Control, Automation, and Systems  
**Method:** Single-frame GAT with bone edge features + PointOut training

| Strength | Weakness |
|----------|----------|
| Edge features (bone length, joint angle, confidence) | Single frame — no temporal |
| PointOut: drops nodes → occlusion robust | No multi-person |
| Tested on Ai-Hub | No edge deployment |

**Key insight:** Bone length is physically constant — using it as edge feature is powerful.

---

### 2.6 Hip Impact Velocity Paper (Moon et al. — 2025)
**Journal:** Scientific Reports  
**Finding:** OpenPose can estimate hip velocity (MAPE: 7.28%, good) but acceleration is poor (MAPE: 26.3%, especially at >3.0g)

**Implication for our work:**
- Velocity from skeleton → reliable ✓
- Acceleration from skeleton → noisy, needs smoothing filter (Savitzky-Golay)
- This paper validates skeleton-based biomechanics AND reveals acceleration limitation

---

### 2.7 MambaSL (Jung, Kim — ICLR 2026)
**Method:** Single-layer Mamba redesigned for time series classification  
**Benchmark:** SOTA on UEA30 (all 30 datasets)

**Key features:**
- Single-layer → lightweight
- Multi-head adaptive pooling → catches short events (fall = 0.5-1s)
- Modular selective SSM → different temporal patterns
- Recurrent inference mode → O(1) per frame

**Limitation:** Only tested on benchmark datasets — never applied to fall detection, construction, skeleton, or edge devices.

---

## 3. Gap Analysis

```
Method           | Construction | Physics | Spatial | Temporal | Edge | Multi-person
-----------------|--------------|---------|---------|----------|------|-------------
YOSAP-LSTM       | ✓            | ✗       | ✗       | ✓        | ✓    | ✓
TCNTE            | ✗            | ✗       | ✗       | ✓        | ✓    | ✓
ABC-HF           | ✗            | ✓       | ✗       | ✓        | ✗    | ✗
StoneGAT         | ✗            | ✗       | ✓       | ✗        | ✗    | ✗
PGST-GNN (ours)  | ✓            | ✓       | ✓       | ✓        | ✓    | ✓
```

**Key gap identified:**
No paper combines spatial topology (graph) + temporal dynamics + physics constraints for construction fall detection on edge devices.

---

## 4. Proposed Method — PGST-GNN

### 4.1 Full Name
**Physics-Guided Spatial-Temporal Graph Neural Network (PGST-GNN)**  
for Real-Time Multi-Person Fall Detection on Construction Sites

### 4.2 Core Idea

Fall = simultaneous SPATIAL + TEMPORAL event:
- Body angle (spatial) changes suddenly
- With large acceleration (temporal) at the same time

Skeleton is naturally a graph:
- **Nodes** = keypoints (body joints)
- **Edges** = bones (physically meaningful connections)

**Novel contribution:** Use bone length as physics constraint in graph edges:
- Bone length is physically constant
- If edge length changes → keypoint error → filter it
- This filters occlusion-caused errors automatically

### 4.3 10 Keypoints Selected

```
Shoulders (2): L/R shoulder
Elbows   (2): L/R elbow
Hips     (2): L/R hip
Knees    (2): L/R knee
Ankles   (2): L/R ankle
```

Face landmarks excluded — always occluded by helmets on construction sites.

### 4.4 Graph Structure

```
Node features: [x, y, vx, vy, ax, ay, conf] = 7 values per node
               10 nodes × 7 = 70 values per frame

Edge features: [bone_angle, bone_length] = 2 values per edge
               9 edges × 2 = 18 values

Physics filter: if |current_bone_length - reference_bone_length| > threshold
                → keypoint is erroneous → flag it
```

**9 edges (bones):**
L.shoulder—L.elbow, R.shoulder—R.elbow,  
L.elbow—L.hip (via torso), R.elbow—R.hip,  
L.hip—R.hip, L.hip—L.knee, R.hip—R.knee,  
L.knee—L.ankle, R.knee—R.ankle

### 4.5 Architecture

```
YOLO-Pose (TensorRT INT8)
      ↓
BoT-SORT (multi-person, per-camera)
      ↓
Graph construction
  Node: [P, V, A, C]
  Edge: [bone_angle, bone_length]
  Physics filter: bone length check
      ↓
PGST-GNN
  Spatial: Graph Attention (GAT layer)
           → captures body posture topology
  Temporal: TCN layer (TensorRT compatible)
            → captures motion dynamics
  Fusion: concatenate spatial + temporal features
      ↓
Classifier head → Fall / Normal
      ↓
Alert: Person ID + Camera + Zone + Timestamp
```

### 4.6 Why This is a Method Paper

| Component | What's new |
|-----------|-----------|
| Physics-constrained edge feature | Bone length as physical constant used as filter — not done in any fall detection paper |
| Spatial + Temporal unified | GAT (spatial) + TCN (temporal) for fall detection in construction — not done |
| Confidence-aware node feature | Conf score preserved (not discarded) as fall signal |
| Graph for construction skeleton | 10 construction-specific keypoints with physics-informed graph |

**From literature:** StoneGAT uses GAT but single-frame only (no temporal). TCNTE uses temporal but no spatial graph. We combine both + physics.

### 4.7 Acceleration Problem (from Moon et al. 2025)

Raw skeleton acceleration has 26% error at high impact (>3.0g). Solution:
- Apply **Savitzky-Golay smoothing filter** before differentiation
- Use **multi-keypoint average** (hip + knee + ankle) for more stable estimate
- This becomes an additional contribution: "Physics-aware smoothing for skeleton-derived acceleration"

---

## 5. Dataset Strategy

```
1. UP-Fall + Le2i + GMDCSA-24 (open)
   → Benchmark comparison with TCNTE
   → Reproducibility

2. Ai-Hub (aihub.or.kr, dataSetSn=171)
   → Construction CCTV footage
   → Comparison with YOSAP-LSTM
   → Access via CBNU institutional account (approved)

3. CBNU Lab (custom, small)
   → 5-6 participants, construction clothing, matras
   → 50-100 fall + 200-300 non-fall clips
   → Real construction domain specificity
```

---

## 6. Comparison with Prior Work

```
                  YOSAP-LSTM   TCNTE      PGST-GNN (ours)
Edge FPS          6.44 (XNX)   19 (ONX)   28-32 (ONX) / 18-22 (Nano)*
Accuracy (AiHub)  97.44%       —          96-98%*
Accuracy (UP-Fall) —           99.58%     99%+*
Physics feature   No           No         Yes (P+V+A+C + bone constraint)
Spatial graph     No           No         Yes (GAT)
Temporal          Yes          Yes        Yes (TCN)
Construction      Yes          No         Yes
Multi-person      Yes          Yes        Yes
Edge (Nano)       No           No         Yes ($99)
Edge (Orin NX)    No           Yes        Yes
Open dataset      No           Yes        Yes (+ Ai-Hub)
```
*Estimated. Subject to actual experiment.

---

## 7. Expected Reviewer Questions

1. **"GAT alone is not novel. TCN alone is not novel. Why is their combination novel?"**  
   → Physics-constrained edge feature (bone length filter) is novel. No paper uses bone physics as a graph constraint for fall detection.

2. **"TCNTE already does TCN + Transformer at 19 FPS. Why is yours better?"**  
   → We add spatial graph topology (TCNTE has no spatial model). We achieve 28+ FPS vs 19 FPS. We validate on construction domain (TCNTE does not).

3. **"Acceleration from skeleton has 26% error (Moon et al. 2025). How is A reliable?"**  
   → Savitzky-Golay smoothing + multi-keypoint averaging reduces error. Show ablation: raw A vs smoothed A accuracy.

4. **"Show ablation: P only vs P+V vs P+V+A vs P+V+A+C vs full PGST-GNN"**  
   → Required. Each component must be justified numerically.

5. **"Bone length changes with camera angle (2D projection)"**  
   → True for extreme angles. Filter threshold is adaptive per camera setup. Discuss limitation openly.

---

## 8. Ablation Studies Required

```
1. Feature ablation:
   P only → P+V → P+V+A → P+V+A+C → full graph

2. Architecture ablation:
   GAT only (no TCN) → TCN only (no GAT) → PGST-GNN

3. Physics filter ablation:
   Without bone filter → with bone filter
   Accuracy on high-occlusion subset

4. Smoothing ablation:
   Raw A → Savitzky-Golay A
   Accuracy + velocity/acceleration estimation error

5. FPS breakdown:
   Per component, per number of persons (1/3/5/10)
   Nano vs Orin NX

6. Graph structure ablation:
   7 keypoints vs 10 keypoints
   Different edge configurations
```

---

## 9. Technical Notes

### Acceleration smoothing
```python
from scipy.signal import savgol_filter
positions = skeleton_positions  # shape: [T, 10, 2]
smoothed = savgol_filter(positions, window_length=7, polyorder=3, axis=0)
velocity = np.diff(smoothed, axis=0) * fps
acceleration = np.diff(velocity, axis=0) * fps
```

### Bone length filter
```python
reference_lengths = compute_reference_lengths(first_N_frames)
current_lengths = compute_lengths(current_frame)
mask = np.abs(current_lengths - reference_lengths) / reference_lengths < threshold
# mask = True → valid keypoint, False → flagged
```

### Edge deployment
```
YOLO-Pose:  TensorRT INT8  ← main speedup
GAT layer:  PyTorch GPU    ← small graph, fast
TCN layer:  TensorRT INT8  ← compatible
Total classifier: ~4-5ms
```

---

## 10. Next Steps

```
[ ] Ai-Hub dataset access olish (aihub.or.kr, CBNU email)
[ ] YOLOv8-Pose fine-tuning Ai-Hub construction data bilan
[ ] PGST-GNN implement qilish (PyTorch Geometric)
[ ] Savitzky-Golay smoothing qo'shish
[ ] Ablation study o'tkazish
[ ] FPS o'lchash — Nano va Orin NX da
[ ] Paper yozish — Automation in Construction
```

---

## 11. Key References

1. Kim, Yu, Xiong — "Robust skeleton-based AI for automatic multi-person fall detection on construction sites with occlusions" — Automation in Construction, 2025
2. Yu, Wang, Wu, Xiong — "A real-time skeleton-based fall detection algorithm based on TCN and Transformer encoder" — PMC, 2025
3. Huh, Jeong, Lee, Lee, Shin — "Leveraging falling acceleration and body part clustering for physics-based human fall detection with mmWave radar" — EAAI, 2025
4. Chun, Song, Lee, Lee — "StoneGAT: A robust fall detection framework via skeleton-aware graph attention networks" — IJCAS, 2025
5. Moon et al. — "Estimating hip impact velocity and acceleration from video-captured falls using a pose estimation algorithm" — Scientific Reports, 2025
6. Jung, Kim — "MambaSL: Exploring single-layer Mamba for time series classification" — ICLR 2026
