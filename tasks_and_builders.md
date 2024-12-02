# README: Setting Up Malaria Simulation Task with EMODTools and Idmtools

This README provides a detailed explanation of the code used to set up and configure a malaria simulation task. The script defines the configuration of interventions, input files, reports, and simulation parameters using the EMODTools and Idmtools packages. The goal is to create a fully functional and repeatable simulation experiment. Let's go through the script step-by-step.

## 1. **Importing Required Libraries**

```python
import os
from functools import partial
from pathlib import Path
from typing import List

import config_params as par
import pandas as pd
from emodpy import emod_task
from emodpy_malaria.interventions.outbreak import add_outbreak_individual
from emodpy_malaria.reporters.builtin import add_report_event_counter
from idmtools.builders import SimulationBuilder
from idmtools.entities.simulation import Simulation
from scipy import interpolate
from snt.hbhi.set_up_general import setup_ds
from snt.hbhi.set_up_interventions import add_all_interventions, update_smc_access_ips
from snt.hbhi.utils import (
    add_annual_parasitemia_rep,
    add_monthly_parasitemia_rep_by_year,
    add_nmf_trt,
    tryread_df,
)
from snt.utility.sweeping import CfgFn, ItvFn, set_param
from tqdm import tqdm

import manifest
```

### Explanation:
- **`os`, `partial`, `Path`, `List`**: General utility modules.
- **`config_params as par`**: Custom configuration parameters for the simulation.
- **`pandas`**: For data handling (reading and processing CSV files).
- **`emodpy` and `emodpy_malaria`**: EMODTools modules used for creating simulation tasks and adding malaria-specific interventions.
- **`idmtools`**: Tools for building and managing simulation workflows.
- **`snt.hbhi`**: Custom module used for setting up general configurations, interventions, and utility functions.
- **`tqdm`**: Used for displaying progress bars.

## 2. **Adding Reports**

```python
def _config_reports(task):
    """
    Add reports.
    
    Args:
        task: EMODTask
    
    Returns: None
    """
    from snt.hbhi.set_up_general import initialize_reports
    initialize_reports(task, manifest,
                       par.event_reporter,
                       par.filtered_report,
                       par.years,
                       par.yr_plusone)
    add_monthly_parasitemia_rep_by_year(
        task, manifest, num_year=par.years,
        tot_year=par.years, sim_start_year=2023,
        yr_plusone=par.yr_plusone, prefix='Monthly'
    )
    add_annual_parasitemia_rep(
        task, manifest, num_year=par.years,
        tot_year=par.years, sim_start_year=2023,
        age_bins=[0.25, 1, 2, 5, 10, 15, 30, 50, 125]
    )
```

### Explanation:
- **`_config_reports()`** adds reports to the EMODTask, including monthly and annual parasitemia reports, and initializes various configuration parameters.

## 3. **Adding Input Files**

```python
def add_input_files(task, inputpath, my_ds, demographic_suffix='',
                    clim_subfolder='5yr_consttemp'):
    """
    Add assets corresponding to the filename parameters set in set_input_files.
    
    Args:
        task:
        inputpath:
        my_ds:
        demographic_suffix:
        clim_subfolder:
    
    Returns:
        None
    """
    # Demographics
    if demographic_suffix is not None:
        if not demographic_suffix.startswith('_') and not demographic_suffix == '':
            demographic_suffix = '_' + demographic_suffix
    
    if demographic_suffix is not None:
        demog_path = os.path.join(my_ds,
                                  f'{my_ds}_demographics{demographic_suffix}.json')
        task.common_assets.add_asset(os.path.join(inputpath, demog_path),
                                     relative_path=str(Path(demog_path).parent),
                                     fail_on_duplicate=False)
    
    # Climate
    if clim_subfolder is not None:
        for climate_file in ['air_temperature_daily.bin', 'rainfall_daily.bin', 'relative_humidity_daily.bin']:
            file_path = os.path.join(my_ds, clim_subfolder, climate_file)
            task.common_assets.add_asset(os.path.join(inputpath, file_path),
                                         relative_path=str(Path(file_path).parent.parent),
                                         fail_on_duplicate=False)
            task.common_assets.add_asset(os.path.join(inputpath, f'{file_path}.json'),
                                         relative_path=str(Path(file_path).parent.parent),
                                         fail_on_duplicate=False)
```

### Explanation:
- **`add_input_files()`** adds input files like demographics and climate data to the EMODTask.
- The climate data includes temperature, rainfall, and humidity, which are essential for modeling malaria transmission.

## 4. **Building Campaign**

```python
def build_campaign():
    """
    Adding required interventions common to all

    Returns:
        campaign object
    """
    import emod_api.campaign as campaign
    campaign.schema_path = manifest.schema_file
    
    add_outbreak_individual(campaign, start_day=182,
                            demographic_coverage=0.01,
                            repetitions=-1,
                            timesteps_between_repetitions=365)
    add_nmf_trt(campaign, par.years, 0)
    
    return campaign
```

### Explanation:
- **`build_campaign()`** creates a campaign object that defines interventions.
- Adds outbreak events and treatment interventions to the simulation.

## 5. **Setting Parameters**

```python
def set_param_fn(config):
    """
    Callback to set simulation parameters.
    
    Args:
        config:
    
    Returns:
        configuration settings
    """
    import emodpy_malaria.malaria_config as malaria_config
    config = malaria_config.set_team_defaults(config, manifest)
    par.set_config(config)
    
    return config
```

### Explanation:
- **`set_param_fn()`** sets up the configuration parameters for the simulation, such as malaria-specific settings and defaults.

## 6. **Creating EMODTask**

```python
def get_task(**kwargs):
    global platform
    platform = kwargs.get('platform', None)
    
    # Create EMODTask
    print("Creating EMODTask...")
    task = emod_task.EMODTask.from_default2(
        config_path=None,
        eradication_path=manifest.eradication_path,
        schema_path=manifest.schema_file,
        campaign_builder=build_campaign,
        param_custom_cb=set_param_fn,
        ep4_custom_cb=None,
    )

    # Add input files to the task
    ds_list = par.ds_list
    for my_ds in ds_list:
        add_input_files(task,
                        inputpath=os.path.join(manifest.IO_DIR, 'simulation_inputs',
                                               'DS_inputs_files'),
                        my_ds=my_ds,
                        demographic_suffix=par.demographic_suffix)

    # Add reports to the task
    _config_reports(task)

    return task
```

### Explanation:
- **`get_task()`** creates an EMODTask and configures it with parameters, interventions, input files, and reports.
- Uses **`build_campaign()`** and **`set_param_fn()`** to define campaign and configuration settings.

## 7. **Sweeping Interventions**

```python
def sweep_interventions(simulation: Simulation, func_list: List):
    tags_updated = {}
    for func in func_list:
        tags = func(simulation)
        if tags:
            if isinstance(func, ItvFn):
                fname = func.func.__name__
                if fname == 'add_all_interventions':
                    add_report_event_counter(simulation.task, manifest,
                                             event_trigger_list=tags["events"])
            else:
                tags_updated.update(tags)
    return tags_updated
```

### Explanation:
- **`sweep_interventions()`** applies a list of intervention functions to each simulation and updates event reports if necessary.

## 8. **Builder for Sweep Parameters**

```python
def get_sweep_builders(**kwargs):
    global platform
    platform = kwargs.get('platform', None)
    scen = kwargs.get('scen')

    builder = SimulationBuilder()
    scen_row = par.scen_df.loc[scen, :]
    
    # Reading data files for various interventions (e.g., ITNs, SMC)
    hs_df = tryread_df(os.path.join(par.scenariopath, 'cm', f"{scen_row['CM']}.csv"))
    itn_df = tryread_df(os.path.join(par.scenariopath, 'itn', f"{scen_row['ITN']}.csv"))
    smc_df = tryread_df(os.path.join(par.scenariopath, 'smc', f"{scen_row['SMC']}.csv"))
    
    # Adding interventions and other configurations
    int_suite = par.int_suite
    int_sweeps = []
    for my_ds in tqdm(ds_list):
        samp_ds = samp_df[samp_df.DS_Name == my_ds].copy()
        for r, row in samp_ds.iterrows():
            int_f = ItvFn(add_all_interventions,
                          int_suite=int_suite,
                          my_ds=my_ds,
                          hs_df=hs_df,
                          itn_df=itn_df,
                          smc_df=smc_df,
                          addtl_smc_func=update_smc_access_ips)
            for x in range(par.num_seeds):
                cnf = CfgFn(setup_ds,
                            manifest=manifest,
                            platform=platform,
                            my_ds=my_ds,
                            archetype_ds=master_df.at[my_ds, 'seasonality_archetype_2'],
                            pull_from_serialization=par.pull_from_serialization,
                            burnin_df=burnin_df,
                            ser_date=par.ser_date,
                            rel_abund_df=rel_abund_df,
                            lhdf=lhdf,
                            demographic_suffix=par.demographic_suffix,
                            climate_prefix=par.climate_prefix,
                            climate_suffix=par.climate_suffix,
                            use_arch_burnin=par.use_arch_burnin,
                            use_arch_input=par.use_arch_input,
                            hab_multiplier=row['Habitat_Multiplier'],
                            serialize_match_tag=['Sample_ID', 'Run_Number'],
                            serialize_match_val=[row['id'], row['seed2'] + x % par.ser_num_seeds])

                int_funcs = [cnf, int_f, partial(tagger, param='Sample_ID', value=row['id']),
                             partial(set_param, param='Run_Number', value=row['seed2'] + x)]
                int_sweeps.append(int_funcs)

    builder.add_sweep_definition(sweep_interventions, int_sweeps)
    print(builder.count)

    return [builder]
```

### Explanation:
- **`get_sweep_builders()`** sets up different intervention configurations, generating multiple simulations with varying parameters.
- The **`SimulationBuilder`** object is used to define sweeping parameters for the simulations.

## Summary
- The code sets up and runs a malaria simulation with different intervention strategies and parameters.
- It uses EMOD and Idmtools for building and managing the simulation.
- Key functions define campaigns, add input files, reports, and configure intervention sweeps.

Feel free to adjust the parameters or interventions to match the specific needs of your simulation. Let me know if you have further questions or need more details on any part of this setup!

