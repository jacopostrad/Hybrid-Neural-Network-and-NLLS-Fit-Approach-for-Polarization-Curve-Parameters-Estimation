# Hybrid-Neural-Network-and-NLLS-Fit-Approach-for-Polarization-Curve-Parameters-Estimation

This repository contains Python scripts for generating, training, and applying **Neural Network (CNN + LSTM)** models to estimate the parameters of **polarization curves** in Zinc-Air Flow Batteries (ZAFBs). The goal is to accurately estimate the following parameters from experimental or synthetic data:

- $E_0$ → equilibrium potential  
- $b$ → Tafel slope  
- $i_0$ → exchange current density  
- $R$ → ohmic resistance  
- $i_L$ → limiting current density  

---

## Repository Overview

This repository contains scripts for:

1. **Generating synthetic polarization curve datasets**  
   - `1_NN_dataset_creation.py` creates two datasets: `curves_with_iL` and `curves_without_iL`, along with the corresponding parameter files (`params_with_iL` and `params_without_iL`).  

2. **Training Neural Networks (CNN + LSTM) to estimate parameters**  
   - `2A_NN_with_iL_CNN+LSTM.py` → trains on curves **with mass transport limitation** (including $i_L$).  
   - `2B_NN_without_iL_CNN+LSTM.py` → trains on curves **without mass transport limitation** (excluding $i_L$).  

3. **Applying a hybrid Neural Network + Nonlinear Least Squares (NLLS) method to experimental data**  
   - `3_param_estimation_NN+NLLS.py` → predicts parameters from experimental polarization curves using a trained NN and refines them via NLLS fitting.  

---

## File Descriptions

### `1_NN_dataset_creation.py`
- Generates a large synthetic dataset of polarization curves using an analytical model.  
- Curves are truncated when voltage reaches 0.2 V.  
- Saves datasets in `.npy` format:  
  - `curves_n.npy` → full current–voltage curves  
  - `params_n.npy` → corresponding parameters  
- Further splits curves based on the derivative to separate mass transport effects:  
  - `curves_with_iL_n.npy` / `params_with_iL_n.npy`  
  - `curves_without_iL_n.npy` / `params_without_iL_n.npy`  

This allows training **separate NNs** depending on whether the **mass transport limitation** is present.

---

### `2A_NN_with_iL_CNN+LSTM.py`
- Trains a CNN+LSTM neural network to estimate all **5 parameters** on curves that **include limiting current effects**.  
- Minimizes **MSE loss**, monitors $R^2$ globally and per parameter, and evaluates performance on a test set.

### `2B_NN_without_iL_CNN+LSTM.py`
- Same as `2A`, but for curves **without limiting current** ($i_L$ not included).  

---

### `3_param_estimation_NN+NLLS.py`
- Applies a **trained CNN-LSTM model** to **experimental polarization curves**, then refines estimates with NLLS.  

**Workflow:**
1. Load experimental curve from a `.txt` file.  
2. Detect mass transport limitation via derivative of potential vs. current:  
   - `J = 1` → mass transport limitation detected → use NN trained **with $i_L$**  
   - `J = 0` → no mass transport limitation → use NN trained **without $i_L$**  
3. Predict parameters with the appropriate NN and denormalize to physical values.  
4. If $i_L$ is present, perform post-processing.  
5. Refine all parameters using **NN + NLLS**.  
6. Compute **95% confidence intervals** and $R^2$ between experimental and fitted curves.

---

## Workflow

1. Run `1_NN_dataset_creation.py` → generate synthetic datasets.  
2. Train the neural network using either `2A` (with $i_L$) or `2B` (without $i_L$).  
3. Use `3_param_estimation_NN+NLLS.py` to estimate parameters from experimental curves.  

---

## Included Files

- **Datasets**: `.npy` and `.npz` files with synthetic curves and parameters (5 values per parameter).  
- **Trained NN Models**:  
  - `CNN_LSTM_noregu_5val_withiL.keras`  
  - `CNN_LSTM_noregu_5val_withoutiL.keras`  
- **Example experimental curves**:  
  - `curve1.txt` → with mass transport limitation  
  - `curve2.txt` → without mass transport limitation  

---

## Requirements

- Python 3.13.7  
- `numpy`, `scipy`, `tensorflow` (for CNN + LSTM), `matplotlib` (for plotting, optional)  
