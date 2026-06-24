# Data-Driven-Thermal-Modeling-Wire-Arc-Additive-Manufacturing-Process (PINNs approach)
Data-driven thermal modeling of WAAM using Physics-Informed Neural Network Approach 
_______________________________________________________________________________________________________________________________

**Overview**

This file contains physics-informed neural-network surrogate modelling work developed for a master-thesis on data-driven thermal modeling of Wire Arc Additive Manufacturing process.  
The input data for PINN model training is obtained from *Simufact Welding 2022 software* where author performed 25 Finite Element Simulations by creating 25 Design of Experiments. The data coming from each simulation is stored in the Parquet file format. The developed NN model is trained 25 times (for all 25 DOE cases). 

____________________________________________________________________________________________________________________________

**Project Workflow**

1. Load the pre_processed files in the parquet file format (Simulation data).
2. Train the PINN model for each DOE case.
3. Save the 25 trained models in .h5 file format.
4. Evaluate each model using Root Mean Squared Error (RMSE) , Mean Absolute Error (MAE), and coefficient of determination (R^2).
5. Generate training, validation losses plots , PINNs losses (PDE, BC, IC losses), scatter plots, NN_predicted-verses-actual temperature-time plots for all 25 Experiments.
____________________________________________________________________________________________________________________________

## Project Summary

A Design of Experiments containing **25 simulations** was designed from combinations of:

- Robot travel speed: `200, 400, 600, 800, 1000 mm/min`
- Wire feed rate: `2, 4, 6, 8, 10 m/min`
- We have 25 combination pairs like (200,2), (200,4), (200,6),...(600,6),...,(1000,10).

Each simulation is converted into one Parquet file containing:

```text
time, x, y, z, Temp_K

PINN model is trained 25 times (25 DOE)

## Workflow

After performing Simufact Welding simulations for all 25 DOE cases, time, nodal coordinates and temperature data is extracted in .UNV files format (per time step). For each DOE, .UNV files are combined in such a way to have one big file, called Parquet file, containing time, nodal coordinates, and temperature in Kelvin, for PINN model training. 72% of data is used for training, 18% for validation, and 10 % for testing. Training is performed in two stages: First 100 epochs for  Data-only neural-network training and physics-informed learning afterwards. Each model is saved and evaluated based on metrics such as, RMSE, MAE, and R^2. All the loss plots, scatter plots (R^2) and PINN predicted verses original temperature-time plots are saved in the folders.

____________________________________________________________________________________________________________________________

## Code working

**Model inputs and Outputs**

The inputs to the model are  (time, x, y, z) and it approximates temperature in Kelvin (output). The inputs are min-max scaled. Temperature is kept in Kelvin and not scaled because it is later used in calculating derivatives with respect to time and nodal coordinates where true temperature values are required.


**PINN Architecture**

```text
Input: 4 variables
Dense 128, ReLU
Dropout 0.05
Dense 128, ReLU
Dense 64, ReLU
Dense 64, ReLU
Dense 32, ReLU
Dense 16, ReLU
Output: 1 temperature value
```
 
 



