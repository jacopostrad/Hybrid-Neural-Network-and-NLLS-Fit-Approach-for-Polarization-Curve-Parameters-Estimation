These scripts are written and performed in Python 3.13.7

This repository contains scripts for:
•	Generating synthetic polarization curve datasets → 1_NN_dataset_creation.py: creates two datasets curves_with_iL and curves_without_iL, and respective parameter files (params_with_iL and params_without_iL)
•	Training Neural Networks (CNN + LSTM) to estimate parameters → 2A_NN_with_iL_CNN+LSTM.py and 2B_NN_without_iL_CNN+LSTM.py, for the respective datasets
•	Applying a hybrid Neural Network + Nonlinear Least Squares (NLLS) method to experimental data → 3_param_estimation_NN+NLLS.py
The goal is to estimate the parameters of a polarization curve:
E0 → equilibrium potential
b → Tafel slope
i0 → exchange current density
R → ohmic resistance
iL → limiting current density


FILES DESCRIPTION:
1_NN_dataset_creation.py
This script generates a large synthetic dataset of polarization curves using the analytical model, generating all combinations of parameters and truncating curves when voltage reaches 0.2 V. The primary dataset is saved in:
•	curves_n.npy → current–voltage curves
•	params_n.npy → corresponding parameters
The script also splits the dataset using the first derivative into:
•	curves_with_iL_n.npy
•	curves_without_iL_n.npy
•	corresponding parameter files
This allows training separate neural networks depending on whether the mass transport limitation is visible or not.



2A_NN_with_iL_CNN+LSTM.py
Trains a Neural Network to estimate the 5 parameters for curves that include limiting current effects, with a hybrid CNN+LSTM architecture. The NN minimizes MSE, monitors R² (global and per parameter), and evaluates performance on the test set.

2B_NN_without_iL_CNN+LSTM.py
Same as 2A, but trained on curves without limiting current effect (iL not included).



3_param_estimation_NN+NLLS.py
This script applies the previously trained CNN-LSTM model to experimental polarization curves and refines the estimates via NLLS fitting.
Workflow:
1.	Load experimental curve – Current-voltage data is read from a text file.
2.	Detect mass transport limitation – Based on the derivative of potential vs. current:
	o	J = 1: mass transport limitation detected → model includes iL;
	o	J = 0: no mass transport limitation → model excludes iL.
3.	Load the appropriate NN model – Either trained with or without iL.
4.	NN parameter prediction – Predict normalized parameters and denormalize them to obtain physical values.
5.	Post-processing for iL (only if mass transport detected).
6.	NN+NLLS refinement – Use NN (and post-processed) estimates as initial guesses in NLLS to refine all parameters.
7.	Confidence intervals and R² – Report 95% confidence intervals and compute the coefficient of determination between experimental and fitted curves.



WORKFLOW:
Run 1_NN_dataset_creation.py → Train NN using 2A or 2B → Use 3_param_estimation_NN+NLLS.py for experimental parameter estimation.



FILE ATTACHED:
Together with the scripts there are:
1.	All the .npy and .npz datasets created with 5 values per parameter, already splitted in the two datasets
2.	Two NN example models for both datasets, created with 5 values per parameter ---> CNN_LSTM_noregu_5val_withiL.keras and CNN_LSTM_noregu_5val_withoutiL.keras
3	Two examples of experimental curves: one with mass transport effect (curve1.txt) and one without (curve2.txt)
