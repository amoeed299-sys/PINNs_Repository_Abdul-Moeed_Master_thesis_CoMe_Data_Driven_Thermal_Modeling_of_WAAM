# Data-Driven-Thermal-Modeling-Wire-Arc-Additive-Manufacturing-Process (PINNs approach)
Data-driven thermal modeling of WAAM using Physics-Informed Neural Network Approach 
____________________________________________________________________________________________________________________________

**Overview**

This file contains physics-informed neural-network surrogate modelling work developed for a master-thesis on data-driven thermal modeling of Wire Arc Additive Manufacturing process.  
The input data for PINN model training is obtained from *Simufact Welding 2022 software* where author performed 25 Finite Element Simulations by creating 25 Design of Experiments. The data coming from each simulation is stored in the Parquet file format. The developed PINN model is trained 25 times (25 DOE). 

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

After performing Simufact Welding simulations for all 25 DOE cases; time, nodal coordinates and temperature data is extracted in .UNV files format (per time step). For each DOE, .UNV files are combined in such a way to have one big file, called Parquet file, containing time, nodal coordinates, and temperature in Kelvin, for PINN model training. 72% of data is used for training, 18% for validation, and 10 % for testing. Training is performed in two stages: First 100 epochs for  Data-only neural-network training and physics-informed learning afterwards. Each model is saved and evaluated based on metrics such as, RMSE, MAE, and R^2. All the loss plots, scatter plots (R^2) and PINN predicted verses original temperature-time plots are saved in the folders.

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

To all dense layers, L2 regularization is applied.

**Training Strategy**

Training is done in two stages

Stage 1: Data-only training 

- Maximum epochs: `100`
- Loss: mean squared error
- Batch size: `8192`
- Early stopping and learning-rate reduction are enabled.

Stage 2: PINN learning

- Maximum epochs: `250`
- Initial physics warm-up: `20 epochs`
- Maximum batches per epoch: `60`
- Physics batch size: `2048`
- BC batch size: `2048`
- IC batch size: `2048`
- Gradient clipping: `1.0`
- Validation-based early stopping: `40 epochs`

The total loss combines:

```text
Total loss = w_data*data loss + w_pde*PDE loss + w_bc*boundary-condition loss + w_ic*initial-condition loss
```

The pde, bc, ic loss weights are adjusted using gradient-norm-based adaptive weighting.

**Incorporated Physics**

The PINN uses the transient thermal equation with:

- Goldak double-ellipsoidal heat source.
- PDE loss.
- Convection loss.
- Radiation loss.
- Initial-temperature condition.
- Temperature dependent specific heat capacity and thermal conductivity.

The material temperature dependent properties Cp(T) and k(T) are read from CSV files:

```text
K_vs_temp_simufact_316L.csv
C_p_vs_temp_simufact_316L.csv
```

**Data Split** 

Data split is: 72% of data is used for training, 18% for validation, and 10 % for testing.

 
**Repository Structure**

```text
PINNS-WORK-WAAM/
├── master-thesis/
│   ├── data/
│   │   ├── pre_processed/
│   │   │   ├── 200_2.parquet
│   │   │   ├── experiment_200_2.yaml
│   │   │   └── ...
│   │   ├── K_vs_temp_simufact_316L.csv
│   │   └── C_p_vs_temp_simufact_316L.csv
│   └── scripts/
│       ├── surrogate_model_training_v1.2(PINNs)-latest.py
│       ├── surrogate_model_training_v1.2(PINNs)-latest.sh
│       ├── Requirements.txt
│       ├── trained_models_PINNS/
│       ├── training_plots_PINNS/
│       ├── evaluation_plots_PINNS/
│       ├── evaluation_results_PINNS/
│       └── training_log_*.log
└── README.md
```

Each Yaml file stores all the parameters of Goldak double ellipsoidal heat source, current, voltage, efficiency, emissivity and convective heat transfer coefficient values for each experiment.


**Requirements file**

It containes all the required libraries necessary for the code execution.

**Code execution**

Move to the scripts directory:

```bash
cd master-thesis/scripts
```

Run the shell script:

```bash
bash "surrogate_model_training_v1.2(PINNs)-latest.sh"
```

Or run the Python file directly:

```bash
python "surrogate_model_training_v1.2(PINNs)-latest.py"
```

The filenames should be quoted because they contain parentheses.

## Outputs 

**Trained Models**

All the trained models  (.h5 file format) and input scaler information per DOE (.npz file format) are saved in folder:
```text
trained_models_PINNS/
```

Example:

model_200_2.h5

input_scaler_200_2.npz

____________________________________________________________________________________________________________________________

**Training plots**

Training loss plots including, data-only training loss (e.g. training_history_data_only_200_2.png), PINN physics losses (PDE, BC, IC), total loss, data loss and Validation loss  (e.g. training_history_pinn_200_2_components.png) are saved in folder:
```text
training_plots_PINNS/
```
For example, in training_history_pinn_200_2_components.png, there are two subplots. The upper subplot shows the phsyics loss plots incluidng Partial Different Equation loss, boundary condition loss, initial condition loss curves over the epochs and lower subplot includes total loss , data loss and validation loss plots over epochs.


____________________________________________________________________________________________________________________________

**Evaluation plots**

Predicted verses actual temperature plots (scatter plots) and Temperature-versus-time comparison plots (PINN predcited against original temperature for test set) are saved in folder:

```text
evaluation_plots_PINNS/
```
____________________________________________________________________________________________________________________________

**Evalution metrics**

The main test-results file for all DOE is saved as model_evaluation_metrics_test.csv in the folder:

```text
evaluation_results_PINNS/
```
The metrics include: RMSE, MAE, R^2, and Pearson correlation. The folder evaluation_results_PINNS/ also contains evalution metrics files for training and validation data.

___________________________________________________________________________________________________________________________


 **Important Notes**

- We have 25 trained models (25 DOE).
- The input scaler file (.npz foramt) must always be used with its corresponding `.h5` model.
- YAML and Parquet files must be there in the data/pre_processed folder for the execution of code (model training).
- GPU memory growth is enabled automatically when a GPU is available.
- Training logs are written both to the terminal and to timestamped `.log` files.



**Project** 

Master’s thesis — Data-Driven Thermal Modeling of Wire Arc Additive Manufacturing Process

**Author** 

Abdul-Moeed_M.Sc. Computational Mechanics (@TUM)


















 
 



