#  Curve Parameter Estimation

This project reconstructs a complex 2D curve by estimating three unknown parameters using nonlinear optimization. Find the fully documented and commented implementation inside the notebook: Parameter_estimation.ipynb.


### **Final Estimated Parameters:**
| Parameter | Value |
|----------|--------|
| **θ (degrees)** | **29.999973°** |
| **M** | **0.030000** |
| **X** | **54.999998** |

### **L1 Distance:**  
**L1 Score = 0.00017028**

This extremely low value shows an almost perfect match between predicted and expected curves.

### **Submission Expression (Desmos / LaTeX Format):**

\[
\left(t*\cos(0.523598)-e^{0.030000|t|}\cdot\sin(0.3t)\sin(0.523598)+54.999998,\;
42+t*\sin(0.523598)+e^{0.030000|t|}\cdot\sin(0.3t)\cos(0.523598)\right)
\]

---

# Project Overview

Given a dataset of points lying on a parametric curve for **6 < t < 60**, the task is to recover the unknown parameters in:

x(t) = tcos(θ) - exp(M|t|) * sin(0.3t) * sin(θ) + X <br>y(t) = 42 + tsin(θ) + exp(M|t|) * sin(0.3t) * cos(θ)

This curve is generated from a simpler intrinsic representation, then rotated and translated.  
The challenge is to **reverse engineer θ, M, and X from only the sampled (x,y) values**.

---

#  Mathematical Insight

The key observation:

### The curve is a **rotation + translation** of the intrinsic curve:
u = t<br>v = exp(M * |t|) * sin(0.3t)

The transformation applied during generation is:
- Rotation by **θ**
- Translation by **(X, 42)**

So we invert this transformation:

u = (x - X) * cos(theta) + (y - 42) * sin(theta)

v = -(x - X) * sin(theta) + (y - 42) * cos(theta)

The true intrinsic rule must hold:
v = e^(M * |u|) * sin(0.3 * u)

Thus, the fitting problem becomes:
Find (θ, M, X) that minimize |v − e^(M * |u|) * sin(0.3u)|.

We also add a soft constraint to ensure:
6 ≤ u ≤ 60.

---

# Optimization Strategy

### Techniques Used:
- **Nonlinear least squares** (`scipy.optimize.least_squares`)
- **Soft-L1 loss** for robustness
- **Bounded optimization**
- **Multi-start initialization** (20 runs) to avoid local minima
- Enforcing **soft constraints** on u range
- **Interpolation** for expected curve reconstruction
- **Uniform sampling** for L1 metric

### Why this works?
Because the intrinsic curve is deterministic and smooth, and the transformation is linear, reversing it becomes a well-conditioned optimization problem.

---

# Evaluation Metric — L1 Distance

The dataset is not directly ordered by t, so we:

1. Compute u-values from the transformed coordinates  
2. Sort the data by u  
3. Interpolate to get the “expected curve”  
4. Generate the predicted curve from fitted parameters  
5. Sample both curves uniformly on t ∈ [6,60]  
6. Compute:

L1 = (1 / N) * Σ ( |x_true − x_pred| + |y_true − y_pred| )


### Result: **0.00017028**  
A nearly perfect match.

---

# Visual Outputs (Generated in Notebook)

The notebook displays:

### Intrinsic space (u, v)  
Data collapses exactly onto
v = e^(M * |u|) * sin(0.3u)

### XY space  
The predicted curve overlaps the original data extremely closely.

### Expected vs Predicted curve comparison  
Used for final L1 metric.

