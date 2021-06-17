# AU Robotics Toolbox (AURT) Overview

# Installation

To install the tool, type:
```
pip install aurt
```

# Command Line Interface

The following shows the different use cases that aurt supports.
In order to improve performance, the model is compiled in different stages, 
in a way that allows the user to try alternative friction models without having to re-create the full model, 
which takes a long time.

## Compile Rigid Body Dynamics Model

```
aurt compile-rbd --mdh mdh.csv --gravity 0.0 0.0 -9.81 --out rigid-body_dynamics.pickle
```
Reads the Modified Denavit-Hartenberg (MDH) parameters in file `mdh.csv` and outputs the linear and minimal robot dynamics model to file `rigid-body_dynamics.pickle`.
The gravity vector determines the orientation of the robot for which the parameters will be calibrated.
The generated model does not include the joint dynamics.

## Compile Robot Dynamics Model

```
aurt compile-rd --model-rbd rigid-body_dynamics.pickle --friction-load-model square --friction-viscous-powers 2 1 4 --friction-hysteresis-model sign --out robot_dynamics.pickle
```

Reads the rigid-body dynamics model created with the `compile-rbd` command, and generates the robot dynamics model, 
taking into account the joint dynamics configuration.

The friction configuration options are:
- `--friction-load-model TYPE` where `TYPE in {none, square, absolute}` and:
  - `TYPE=none` means TODO
  - `TYPE=square` means TODO
  - `TYPE=absolute` means TODO 
- `--friction-viscous-powers POWERS` where `POWERS` has the format `P1 P2 ... PN`, and `PN` is a positive real number representing the `N`-th power of the polynomial.
- `--friction-hysteresis-model TYPE` where `TYPE in {sign, maxwell-slip}`, and:
  - `TYPE=sign` means TODO
  - `TYPE=maxwell-slip` means TODO
  
## Calibration

```
aurt calibrate --model robot_dynamics.pickle --data measured_data.csv --out-base-params calibrated_parameters.csv
```

Reads the model produced in [Compile Robot Dynamics Model](#compile-joint-dynamics-model), the measured data in `measured_data.csv`, 
and writes the base parameter values to `calibrated_parameters.csv`,

In order to generate the original parameters described in `mdh.csv`, 
provided in [Compile Rigid Body Dynamics Model](#compile-rigid-body-dynamics-model), 
use the `--out-full-params calibrated_parameters.csv` instead of `--base-params`

The measured data should contain the following fields:
- `time` of type float, representing the number of seconds passed from a given reference point.
- `target_qd_N` of type float, representing the `N`th joint target angular velocity, as computed by the robot controller, where `N` is an integer in `{0, 1, ...}`.
- `actual_q_N` of type float, representing the `N`th joint angle, as measured by the robot controller, where `N` is an integer in `{0, 1, ...}`.
- `actual_current_N` of type float, representing the `N`th joint current, as measured by the robot controller, where `N` is an integer in `{0, 1, ...}`.

## Predict

```
aurt predict --model robot_dynamics.pickle --data measured_data.csv --base-params calibrated_parameters.csv --prediction prediction.csv
```

Reads the model produced in [Compile Joint Dynamics Model](#compile-joint-dynamics-model), 
the measured data in `measured_data.csv`, 
and the base parameter values produced in [Calibration](#calibration), and writes the prediction to `prediction.csv`.

The prediction fields are:
- `time` of type float, referring to the time of the measured data, as in [Calibration](#calibration).
- `predicted_current_N` of type float, representing the `N`th joint current, as predicted by the robot model, where `N` is an integer in `{0, 1, ...}`.

# Contributing

## Development environment

To setup the development environment:
1. Open terminal in the current folder.
2. Optional: create a virtual environment: `python -m venv venv`
3. Optional: activate the virtual environment: 
   1. Windows (Powershell):`.\venv\Scripts\Activate.ps1`
   2. Linux: `source venv/bin/activate`
4. Install all packages for development: `pip install -e .[dev]`
5. Unpack the datasets (see [Dataset management](#dataset-management))
6. To run all tests, open a powershell in the dynamic folder, and run the `.\build_script.ps1 --run-tests offline` script.
7. Optional: open Pycharm in the current folder.


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
The shared drive **Nat_UR-robot-datasets** has been created with **Emil Madsen** as owner.

| **Navn/Name**         | **Drev-ansvarlig/Shared  drive owner** |
| --------------------- | -------------------------------------- |
| Nat_UR-robot-datasets | au504769  (Emil Madsen)                |


 **Read/write access is assigned to:** 

| **Brugernavn/Username** | **Navn/Name**                   | **Afdeling/department** | **E-mail**                                                | **Tilføjet via  gruppe/assigned by group** |
| ----------------------- | ------------------------------- | ----------------------- | --------------------------------------------------------- | ------------------------------------------ |
| au602135                | Cláudio Ângelo Gonçalves Gomes  | Cyber-Physical Systems  | [claudio.gomes@ece.au.dk](mailto:claudio.gomes@ece.au.dk) |                                            |
| au522101                | Christian Møldrup Legaard       | Cyber-Physical Systems  | [cml@ece.au.dk](mailto:cml@ece.au.dk)                     |                                            |
| au513437                | Daniella Tola                   | Cyber-Physical Systems  | [dt@ece.au.dk](mailto:dt@ece.au.dk)                       |                                            |

For more information on access, self-service and management of files: https://medarbejdere.au.dk/en/administration/it/guides/datastorage/data-storage/

