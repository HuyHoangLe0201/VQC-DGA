# VQC Transformer Fault Diagnosis

**Transformer Fault Diagnosis Using an Efficient Simulation-Driven Variational Quantum Classifier with Domain-Aware Feature Encoding**

---

## Overview

This repository contains the full implementation of a 2-qubit Variational Quantum Classifier (VQC) for power transformer fault diagnosis using dissolved gas analysis (DGA) data. The model uses a custom **ZX-YY feature map** combined with an **EfficientSU2 ansatz** (reps=4, 20 parameters), trained with COBYLA via One-vs-Rest (OVR) classification.

| Setting | Value |
|---|---|
| Qubits | 2 |
| Feature map | ZX-YY (RYY + RZX, 4 gates) |
| Ansatz | EfficientSU2, reps=4, SCA entanglement |
| Parameters | 20 |
| Optimizer | COBYLA (maxiter=200) |
| Classification | One-vs-Rest (OVR), 4 heads |
| Dataset | IEC TC10, 705 samples (587 train / 118 test) |
| Fault classes | D, PD, T1, T2 (T1&T2 absent from training) |

---

## Results

### Simulator (Qiskit Statevector)

| Metric | Point | 95% CI (Bootstrap, B=100,000) |
|---|---|---|
| Accuracy | 0.9915 | [0.9746, 1.0000] |
| Precision (w) | 0.9924 | [0.9797, 1.0000] |
| Recall (w) | 0.9915 | [0.9746, 1.0000] |
| F1 (w) | 0.9916 | [0.9746, 1.0000] |

### IBM Quantum Hardware — `ibm_brisbane` (Eagle r3, 127 qubits)

| Metric | Point | 95% CI (Bootstrap, B=100,000) |
|---|---|---|
| Accuracy | 0.9800 | [0.9000, 1.0000] |

Hardware settings: `optimization_level=3` · `initial_layout=[u,v]` · `resilience.level=1` · `shots=8192`

> **Note:** `ibm_brisbane` (Eagle r3) has been retired from the IBM Quantum free tier. Results on currently available Heron backends (`ibm_marrakesh`, `ibm_fez`, `ibm_torino`) yield lower accuracy (~84%) due to deeper transpiled circuits from architectural differences in native gate sets.

---

## Repository Structure

```
VQC_hardware_repo.ipynb   ← Main notebook
X.xlsx                    ← IEC TC10 DGA dataset (upload via Colab)
README.md                 ← This file
```

### Notebook Structure

| Section | Description |
|---|---|
| **1 — Simulator** | Full VQC training + evaluation on Qiskit Statevector |
| **1b — Bootstrap CI** | 95% CI for all 4 metrics, n=118, vectorized numpy |
| **2 — IBM Hardware** | Account setup + EstimatorV2 job submission |
| **2b — Bootstrap CI** | 95% CI for hardware accuracy, self-contained |
| **3.1 — Hilbert Coverage** | Feature map vs ansatz Hilbert space coverage |
| **3.2 — IQP Comparison** | IQP feature map baseline comparison |
| **3.3 — Comparison Plot** | Feature map comparison visualization |
| **3.4 — KTA** | Kernel Target Alignment |
| **3.5 — Frame Potential** | Frame potential analysis |
| **3.6 — Entanglement** | Entanglement entropy |
| **3.7 — Gradient Variance** | Barren plateau / gradient variance |
| **3.8 — Effective Dimension** | Effective dimension |
| **3.9 — Bloch Sphere** | Bloch sphere visualization by label |

---

## Requirements

```
qiskit==1.4.2
qiskit-aer
qiskit-algorithms
qiskit-ibm-runtime
qiskit-machine-learning
scikit-learn
pandas
numpy
matplotlib
openpyxl
pylatexenc
```

All dependencies are installed in Cell 1 of the notebook.

---

## How to Run

**Step 1 — Open in Google Colab**

Upload `VQC_hardware_repo.ipynb` to [colab.research.google.com](https://colab.research.google.com).

**Step 2 — Upload dataset**

Run Cell 2 to upload `X.xlsx`.

**Step 3 — Install dependencies**

Run Cell 1, then **Restart Runtime**.

**Step 4 — Simulator (no IBM account needed)**

Run Cells 4–6 (Section 1 + Bootstrap CI).

**Step 5 — Hardware (optional)**

Set your credentials in Cell 8:

```python
import os
os.environ["IBM_QUANTUM_TOKEN"] = "YOUR_TOKEN"
os.environ["IBM_INSTANCE"]      = "YOUR_CRN_INSTANCE"
```

Then run Cell 9 to submit the job. The bootstrap CI in Cell 11 is self-contained and does not require re-running the hardware job — it reproduces the original `ibm_brisbane` result from hardcoded data.

---

## Circuit Architecture

### ZX-YY Feature Map

```
|ψ(x)⟩ = RZX(x₃) · RYY(x₁) · RZX(x₂) · RYY(x₀) |00⟩
```

```python
qc.ryy(x[0], 0, 1)
qc.rzx(x[2], 1, 0)
qc.ryy(x[1], 1, 0)
qc.rzx(x[3], 0, 1)
```

### EfficientSU2 Ansatz

- `reps=4`, `entanglement="sca"`, `insert_barriers=True`
- 20 trainable parameters θ[0]…θ[19]

### OVR Classification

4 binary heads trained independently: class 0 (D), 1 (PD), 2 (T1), 4 (T2). Class 3 (T1&T2) is absent from training — a relaxed rule accepts T1 or T2 prediction as correct when ground truth is T1&T2.

---

## IBM Hardware Notes

The original 98% result was obtained on `ibm_brisbane` (Eagle r3) with:

```python
generate_preset_pass_manager(optimization_level=3,
                             backend=backend_obj,
                             initial_layout=[u, v])
opts.execution.shots  = 8192
opts.resilience.level = 1   # readout error mitigation
```

Eagle r3 uses CNOT as a native gate, which maps directly to the RZX-based circuit without additional decomposition. Heron r1/r2 backends do not have CNOT as a native gate, resulting in deeper transpiled circuits and higher noise.

---

## Data

The dataset follows the IEC TC10 standard for DGA-based transformer fault diagnosis. Four gas ratio features are used as input. The 705-sample split (587 train / 118 test) is fixed and sequential.

---

## Citation

```bibtex
@article{vqc_transformer_fault,
  title   = {Transformer Fault Diagnosis Using an Efficient Simulation-Driven
             Variational Quantum Classifier with Domain-Aware Feature Encoding},
  author  = {[Authors]},
  journal = {[Journal]},
  year    = {2025}
}
```
