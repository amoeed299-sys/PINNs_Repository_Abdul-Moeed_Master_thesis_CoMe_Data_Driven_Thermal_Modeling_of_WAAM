# Data-Driven-Thermal-Modeling-Wire-Arc-Additive-Manufacturing-Process (PINNs approach)
Data-driven thermal modeling of WAAM using Physics-Informed Neural Network Approach 
_______________________________________________________________________________________________________________________________

**Overview**

This file contains physics-informed neural-network surrogate modelling work developed for a master-thesis on data-driven thermal modeling of Wire Arc Additive Manufacturing process.  
The input data for PINN model training is obtained from *Simufact Welding 2022 software* where author performed 25 Finite Element Simulations by creating 25 Design of Experiments. The data coming from each simulation is stored in the Parquet file format. The developed NN model is trained 25 times (for all 25 DOE cases). 

_______________________________________________________________________________________________________________________________
**Project Workflow**

1. Load the pre_processed files in the parquet file format (Simulation data).
2. Train the PINN model for each DOE case.
3. Save the 25 trained models in .h5 file format.
4. Evaluate each model using Root Mean Squared Error (RMSE) , Mean Absolute Error (MAE), and coefficient of determination (R^2).
5. Generate training, validation losses plots , PINNs losses (PDE, BC, IC losses), scatter plots, NN_predicted-verses-actual temperature-time plots for all 25 Experiments.
_______________________________________________________________________________________________________________________________
