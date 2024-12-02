# config_param: Setting Up Malaria Simulation with EmodPy-Malaria

This README explains the components of the Python code used to set up and configure a malaria simulation experiment. The code is organized to create various configurations for interventions, load data, and prepare the suite of simulations. Let's go through it step-by-step.

## 1. **Importing Required Libraries**

```python
import os
from pathlib import Path
import pandas as pd
import manifest
from emodpy_malaria.vector_config import set_species_param
from snt.hbhi.set_up_general import initialize_config
from snt.hbhi.set_up_interventions import InterventionSuite
```

### Explanation:
- **`os` and `Path`**: Used for handling paths and directories in the operating system.
- **`pandas`**: For reading and manipulating data (CSV files).
- **`manifest`**: Used for path configurations, assumed to be related to the project.
- **`emodpy_malaria.vector_config`**: Imports functions for setting species parameters.
- **`initialize_config`** and **`InterventionSuite`**: Modules that help set up the simulation configuration and intervention strategies.

## 2. **Specifying Paths to Input Files**

```python
larval_hab_csv = 'simulation_inputs/larval_habitats/monthly_habitats5.csv'
master_csv = 'guinea_DS_pop.csv'
rel_abund_csv = 'simulation_inputs/DS_vector_rel_abundance.csv'
samp_csv = 'simulation_priors/selected_particles_v1.csv'
scenariopath = os.path.join(manifest.IO_DIR, 'simulation_inputs/_scenarios_2023')
scenario_csv = 'scenarios_2023-2029.csv'
```

### Explanation:
- Defines paths to several CSV files containing input data, such as larval habitats, population data, vector abundance, selected samples, and scenario definitions.
- **`os.path.join`** is used to create a file path by combining the base directory (`manifest.IO_DIR`) with the subdirectory and filename.

## 3. **Naming the Experiment and Suite**

```python
homepath = os.path.expanduser('~')
user = Path(homepath).name
expname = f'{user}_gin_2023-2029_3x'
suitename = f'{user}_gin_2023-2029_3x'
```

### Explanation:
- Retrieves the user's home directory using **`os.path.expanduser('~')`** and then gets the username.
- **`expname`** and **`suitename`** are set based on the username to uniquely name the experiment and suite.

## 4. **Defining Simulation Settings**

```python
num_seeds = 15
ser_num_seeds = 15
years = 7
ser_date = 18 * 365
serialize = False
pull_from_serialization = True
use_arch_burnin = False
```

### Explanation:
- **`num_seeds`** and **`ser_num_seeds`**: Define the number of simulation seeds.
- **`years`**: Length of the simulation in years.
- **`ser_date`**: Serialization date set in days (18 years).
- **`serialize`** and **`pull_from_serialization`**: Flags to control serialization behavior.

## 5. **Reporting Settings**

```python
filtered_report = years
yr_plusone = True
event_reporter = False
add_event_report = True
burnin_id = "kbt4040_gin_2005-2022_3x_2023_09_27_09_56"
```

### Explanation:
- **`filtered_report`**: Set to the number of years for reporting.
- **`yr_plusone`**: Additional year indicator.
- **`event_reporter`** and **`add_event_report`**: Toggle flags for event reporting.
- **`burnin_id`**: An identifier for the burn-in phase, likely used to specify a saved state.

## 6. **Demographic and Climate Settings**

```python
demographic_suffix = '_wSMC_risk_wIP'
climate_suffix = ''
climate_prefix = False
use_arch_input = False
```

### Explanation:
- **`demographic_suffix`**: Suffix used for demographic data.
- **`climate_suffix`** and **`climate_prefix`**: Variables to handle climate data.

## 7. **Loading DataFrames**

```python
scen_df = pd.read_csv(os.path.join(scenariopath, scenario_csv)).set_index('scen')
master_df = pd.read_csv(os.path.join(manifest.IO_DIR, master_csv))
master_df = master_df.set_index('DS_Name')
samp_df = pd.read_csv(os.path.join(manifest.IO_DIR, samp_csv))
ds_list = samp_df.DS_Name.unique()
```

### Explanation:
- Loads data from CSV files into Pandas DataFrames.
- Sets indices for easy lookups and manipulations.
- **`ds_list`**: Extracts a list of unique district names from the `samp_df` DataFrame.

## 8. **Setting Up Interventions**

```python
int_suite = InterventionSuite()
# HS (Health Services) settings
int_suite.hs_ds_col = 'DS_Name'
int_suite.hs_duration = 365
int_suite.hs_coverage_age = {
    'u5_coverage': [0, 5],
    'adult_coverage': [5, 100]
}
```

### Explanation:
- **`InterventionSuite`**: Initializes a new intervention suite for managing health service (HS), insecticide-treated nets (ITN), and other interventions.
- Sets health service coverage for different age groups, covering children under 5 and adults.

## 9. **Configuring ITN (Insecticide-Treated Nets)**

```python
int_suite.itn_ds_col = 'DS_Name'
int_suite.itn_discard_distribution = 'weibull'
int_suite.itn_cov_cols = ['U05', 'U10', 'U20', 'A20']
int_suite.itn_cov_age_bin = [0, 5, 10, 20]
int_suite.itn_retention_in_yr = 1.69
int_suite.itn_seasonal_months = [0, 91, 182, 274]
int_suite.itn_seasonal_values = [0.739, 0.501, 0.682, 1]
```

### Explanation:
- Defines ITN intervention settings, including discard distribution, coverage columns, age bins, and retention.
- Sets seasonal months and values for the ITN intervention.

## 10. **Configuring SMC, PMC, and RTS,S**

```python
# SMC
int_suite.smc_adherence = True
int_suite.smc_coverage_col = ['coverage_high_access_u5', 'coverage_low_access_u5',
                              'coverage_high_access_o5', 'coverage_low_access_o5']
int_suite.smc_access = ['High', 'Low', 'High', 'Low']
int_suite.smc_agemins = [0.25, 0.25, 5, 5]
int_suite.smc_agemax_type = ['fixed', 'fixed', 'fixed', 'fixed']
int_suite.smc_agemaxs = [5, 5, 10, 10]
int_suite.smc_leakage = False
```

### Explanation:
- **SMC (Seasonal Malaria Chemoprevention)**: Sets adherence, access types, age ranges, and coverage columns.

```python
# PMC (Perennial Malaria Chemoprevention)
int_suite.pmc_touchpoint_col = 'pmc_age_days'
int_suite.pmc_start_col = 'simday'
int_suite.pmc_coverage_col = 'pmc_coverage'
```

### Explanation:
- **PMC**: Defines touchpoints, start days, and coverage columns for PMC.

```python
# RTS,S
int_suite.rtss_auto_changeips = True
```

### Explanation:
- **RTS,S**: Automatically changes intervention properties (IPs) for the RTS,S malaria vaccine.

## 11. **Setting Up Configuration Function**

```python
def set_config(config):
    initialize_config(config, manifest, years, serialize)

    config.parameters.x_Temporary_Larval_Habitat = 3  # Package default is 0.2
    config.parameters.x_Base_Population = 3
    config.parameters.x_Birth = 3

    config.parameters.Report_Event_Recorder = 1
    config.parameters.Report_Event_Recorder_Individual_Properties = []
    config.parameters.Report_Event_Recorder_Events = ['Received_NMF_Treatment',
                                                      'Received_Severe_Treatment',
                                                      'Received_Treatment',
                                                      'NewClinicalCase']
    config.parameters.Report_Event_Recorder_Ignore_Events_In_List = 0

    set_species_param(config, 'arabiensis', 'Anthropophily', 0.88, overwrite=True)
    set_species_param(config, 'arabiensis', 'Indoor_Feeding_Fraction', 0.5,
                      overwrite=True)
    set_species_param(config, 'funestus', 'Anthropophily', 0.5, overwrite=True)
    set_species_param(config, 'funestus', 'Indoor_Feeding_Fraction', 0.86,
                      overwrite=True)
    set_species_param(config, 'gambiae', 'Anthropophily', 0.74, overwrite=True)
    set_species_param(config, 'gambiae', 'Indoor_Feeding_Fraction', 0.9, overwrite=True)

    return config
```

### Explanation:
- **`set_config()`**: A function that sets up the simulation configuration.
- **`initialize_config()`**: Initializes general settings, including the number of years and whether serialization is used.
- Sets parameters for larval habitat, population, birth rate, and event recording.
- **`set_species_param()`**: Configures species-specific parameters, such as **Anthropophily** (preference for biting humans) and **Indoor_Feeding_Fraction**.

## Summary
- The code aims to set up malaria intervention simulations using various interventions (e.g., ITN, SMC, PMC).
- Each section of the code handles a specific part of the setup, from loading data to setting up configurations for species and interventions.

Feel free to adapt these explanations to fit your intended audience or application! If you have further questions, let me know.


