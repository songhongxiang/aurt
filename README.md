![Install and run api tests](https://github.com/INTO-CPS-Association/aurt/actions/workflows/run_tests.yml/badge.svg)
# Aarhus University Robotics Toolbox (AURT) Overview

# Installation

To install the tool, type:
```
pip install aurt
```

# Command Line Interface

The following shows the different use cases that aurt supports.
In order to improve performance, the model is compiled in different stages, 
in a way that allows the user to try alternative joint dynamics models without having to re-create the full model, 
which is a computationally demanding procedure.

## Compile Rigid Body Dynamics Model

```
aurt compile-rbd --mdh mdh.csv --gravity 0.0 0.0 -9.81 --out rigid_body_dynamics
```
Reads the Modified Denavit-Hartenberg (MDH) parameters in file `mdh.csv` and outputs rigid-body dynamics model to file `rigid_body_dynamics`.
The gravity vector determines the orientation of the robot base for which the parameters will be calibrated.
The generated model does not include the joint dynamics.
To visualize the mdh model of the robot, make sure the `roboticstoolbox-python` is installed, and add the `--plot` argument to the `compile-rbd` command. 
<img src="resources/robot_Plot.png" alt="MDH plot" width="400"/>

## Compile Robot Dynamics Model

```
aurt compile-rd --model-rbd rigid_body_dynamics --friction-load-model square --friction-viscous-powers 2 1 4 --out robot_dynamics
```

Reads the rigid-body dynamics model created with the `compile-rbd` command, and generates the robot dynamics model, 
taking into account the joint dynamics configuration.

The friction configuration options are:
- `--friction-load-model TYPE` where `TYPE in {none, square, absolute}` are depicted in the figure below for, respectively, part (a), (b), and (c).
- `--friction-viscous-powers POWERS` where `POWERS` has the format `P1 P2 ... PN`, and `PN` is a positive real number representing the `N`-th power of the polynomial.
<img src="resources/friction_load_models.png" alt="The different possibilities for load-dependent friction models" width="400"/>
<img src="https://render.githubusercontent.com/render/math?math=\mathrm{f}_v(\dot{q}) = \sum_{n=1}^{M}F_{v,n}\,b_n,">

<img src="https://render.githubusercontent.com/render/math?math=b_n = |\dot{q}|\,\dot{q}^{n-1}, \text{if } n \text{ is even}">

## Calibrate

```
aurt calibrate --model robot_dynamics --data measured_data.csv --out-params calibrated_parameters.csv --out-calibration-model robot_calibration --plot
```

Reads the model produced by the `compile-rd` command, the measured data in `measured_data.csv`, writes the calibrated base parameter values to `calibrated_parameters.csv`, and writes the calibrated model parameters to `robot_calibration`.
For showing the calibration plot, use the argument `--plot`.

The measured data should contain the following fields:
- `timestamp` of type float, representing the number of seconds passed from a given reference point.
- `target_qd_N` of type float, representing the `N`th joint target angular velocity, as computed by the robot controller, where `N` is an integer in `{0, 1, ...}`.
- `actual_q_N` of type float, representing the `N`th joint angle, as measured by the robot controller, where `N` is an integer in `{0, 1, ...}`.
- `actual_current_N` of type float, representing the `N`th joint current, as measured by the robot controller, where `N` is an integer in `{0, 1, ...}`.

## Predict

```
aurt predict --model robot_calibration --data measured_data.csv --prediction predicted_output.csv
```

Reads the model produced by the `calibrate` command, 
the measured data in `measured_data.csv`, 
and writes the predicted output to `predicted_output.csv`.

The prediction fields are:
- `timestamp` of type float, referring to the time of the measured data, as in [Calibrate](#predict).
- `predicted_current_N` of type float, representing the `N`th joint current, as predicted by the robot model, where `N` is an integer in `{0, 1, ...}`.

## Calibrate and Validate
```
aurt calibrate-validate --model robot_dynamics --data measured_data.csv --calibration_data_relative FRACTION --out-params calibrated_parameters.csv --out-calibration-model robot_calibration --prediction predicted_output.csv --plot
```
Simultaneously calibrates and cross-validates the robot dynamics model using the dataset `measured_data.csv`. The command implements the functionalities of the commands `calibrate` and `predict`. The data of `measured_data.csv` is separated into two consecutive parts 1) calibration data and 2) validation data. The calibration data has a duration of 0.1 < `FRACTION` < 0.9 times the duration of `measured_data.csv` while the remaining part of the data is used for cross-validation.

# Contributing

## Development environment

To setup the development environment:
1. Open terminal in the current folder.
2. Install all packages for development: `pip install -e .`. You can also install the visualization packages, using `pip install .[vis]`
3. Unpack the datasets (see [Dataset management](#dataset-management))
4. To run `api` tests, open a command prompt or powershell in the repository root, and run `python build.py --run-tests api`. If you are using Linux, use `python3` instead of `python`.
<!--5. To run the full tests, use the command `python --run-tests full-offline`.-->
<!--2. Optional: create a virtual environment: `python -m venv venv`
3. Optional: activate the virtual environment: 
   1. Windows (Powershell):`.\venv\Scripts\Activate.ps1`
   2. Linux: `source venv/bin/activate`
4. Install all packages for development: `pip install -e .[dev]`
5. Unpack the datasets (see [Dataset management](#dataset-management))
6. To run all tests, open a powershell in the repository root, and run the `.\build_script.ps1 --run-tests offline` script. 
7. Optional: open Pycharm in the current folder.-->


## Dataset management

### Small dataset (< 100MB compressed)

If the data is small, then:
- Each round of experiments should be placed in a folder with an informative name, inside the Dataset folder.
- There should be a readme file in there explaining the steps to reproduce the experiment, parameters, etc...
- The csv files should be 7ziped and committed. Do not commit the csv file.
- There should be tests that use the data there.

### Large Datasets (>= 100MB compressed)

If the data is large, then:

- A "lite" version of the dataset should be in the dataset folder (following the same guidelines as before)
  - This is important to run the tests.
- the larger version should be placed in the shared drive (see below).

There is a shared drive for large datasets.
The shared drive **Nat_robot-datasets** has been created with **Emil Madsen** as owner.

| **Shared Drive**   | **Owner**              | **E-mail**                            | **Department**                            |
| ------------------ | ---------------------- | ------------------------------------- | ----------------------------------------- |
| Nat_robot-datasets | au504769 (Emil Madsen) | [ema@ece.au.dk](mailto:ema@ece.au.dk) | Electrical and Computer Engineering (ECE) |


 **Read/write access is assigned to:** 

| **Username** | **Name**                       | **E-mail**                                                | **Department**                            |
| ------------ | ------------------------------ | --------------------------------------------------------- | ----------------------------------------- |
| au602135     | Cláudio Ângelo Gonçalves Gomes | [claudio.gomes@ece.au.dk](mailto:claudio.gomes@ece.au.dk) | Electrical and Computer Engineering (ECE) |
| au522101     | Christian Møldrup Legaard      | [cml@ece.au.dk](mailto:cml@ece.au.dk)                     | Electrical and Computer Engineering (ECE) |
| au513437     | Daniella Tola                  | [dt@ece.au.dk](mailto:dt@ece.au.dk)                       | Electrical and Computer Engineering (ECE) |

For more information on access, self-service and management of files: https://medarbejdere.au.dk/en/administration/it/guides/datastorage/data-storage/
