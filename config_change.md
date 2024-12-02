# README: Simulation Configuration Guide

This README provides details on the configurable parameters for setting up malaria simulations. Each parameter can be adjusted to modify the behavior and scope of the simulation. Below are explanations of the key areas that can be changed, along with the rationale for why you might want to modify them.

## 1. **Simulation Settings**

- **Number of Seeds**
  ```python
  num_seeds = 15
  ser_num_seeds = 15
  ```
  The number of seeds (`num_seeds` and `ser_num_seeds`) can be adjusted to change the number of simulation replications. Increasing this value improves statistical robustness but will also increase computational cost.

- **Simulation Duration**
  ```python
  years = 7
  ```
  Modify the number of years for the simulation based on the desired study length. Longer simulations require more computational resources.

- **Serialization Settings**
  ```python
  ser_date = 18 * 365
  serialize = False
  pull_from_serialization = True
  ```
  - `ser_date` defines the serialization date in days.
  - `serialize` can be set to `True` to save intermediate states, useful for resuming simulations, though it will require more storage.
  - `pull_from_serialization` determines whether to use serialized data from previous runs. Set to `False` if starting a new simulation.

- **Event Reporting**
  ```python
  event_reporter = False
  add_event_report = True
  ```
  - `event_reporter`: Set to `True` to enable event reporting for detailed event tracking.
  - `add_event_report`: Set to `False` to reduce computational load if detailed tracking is unnecessary.

## 2. **Demographic and Climate Settings**

- **Demographic Suffix**
  ```python
  demographic_suffix = '_wSMC_risk_wIP'
  ```
  This suffix is used to specify demographic data settings. Ensure compatibility with available data files if changing this suffix.

## 3. **Intervention Suite Settings**

- **Health Service Duration**
  ```python
  int_suite.hs_duration = 365
  ```
  Modify `hs_duration` to change the duration of health services intervention. Set to `None` to follow the exact duration from the dataset, providing more flexibility.

- **ITN Retention**
  ```python
  int_suite.itn_retention_in_yr = 1.69
  ```
  Adjust `itn_retention_in_yr` to reflect changes in insecticide-treated net (ITN) usage behavior or quality.

## 4. **Config Parameters**

- **Larval Habitat, Base Population, and Birth Rate**
  ```python
  config.parameters.x_Temporary_Larval_Habitat = 3
  config.parameters.x_Base_Population = 3
  config.parameters.x_Birth = 3
  ```
  - `x_Temporary_Larval_Habitat`: Adjust to simulate different levels of temporary larval habitat intensity.
  - `x_Base_Population`: Modify to adjust overall population density.
  - `x_Birth`: Adjust to change birth rate and its effect on population growth.

## 5. **Species-Specific Parameters**

- **Anthropophily and Indoor Feeding Fraction**
  ```python
  set_species_param(config, 'arabiensis', 'Anthropophily', 0.88, overwrite=True)
  set_species_param(config, 'arabiensis', 'Indoor_Feeding_Fraction', 0.5, overwrite=True)
  set_species_param(config, 'funestus', 'Anthropophily', 0.5, overwrite=True)
  set_species_param(config, 'funestus', 'Indoor_Feeding_Fraction', 0.86, overwrite=True)
  set_species_param(config, 'gambiae', 'Anthropophily', 0.74, overwrite=True)
  set_species_param(config, 'gambiae', 'Indoor_Feeding_Fraction', 0.9, overwrite=True)
  ```
  These parameters control the human-biting preference (`Anthropophily`) and indoor feeding behavior (`Indoor_Feeding_Fraction`) for different mosquito species:
  - Adjust these values to simulate changes in mosquito feeding behaviors, which affect malaria transmission dynamics.

## Summary
This document provides an overview of the key configurable parameters for the malaria simulation setup. Adjusting these parameters allows for flexibility in modeling different scenarios and studying their impact on malaria transmission and intervention outcomes. Always ensure that any changes align with the specific goals of your study.

For further assistance, please reach out or refer to the documentation of the specific modules used in the simulation setup.

