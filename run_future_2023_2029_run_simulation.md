# README: Running Malaria Simulation Experiment with Idmtools

This README explains how to set up and run malaria simulation experiments using the provided Python script. The script uses various libraries to create, configure, and execute simulations using EMOD tools and Idmtools. Let's go through the code step-by-step.

## 1. **Importing Required Libraries**

```python
import argparse
from datetime import datetime as dt
import config_params as par
import emod_api.schema_to_class as s2c
from idmtools.core.platform_factory import Platform
from idmtools.entities import Suite
from idmtools.entities.experiment import Experiment
from idmtools.entities.templated_simulation import TemplatedSimulations
from task_and_builders import get_sweep_builders, get_task
import manifest
```

### Explanation:
- **`argparse`**: For parsing command-line arguments.
- **`datetime`**: Used to get the current timestamp to uniquely identify experiments.
- **`config_params`** and **`manifest`**: Custom modules to configure parameters and access manifest variables.
- **`emod_api.schema_to_class`**: Converts schema to Python classes for easy manipulation of parameters.
- **`idmtools`**: A suite for managing simulations using platforms like SLURM.

## 2. **Suppress Warnings**

```python
s2c.show_warnings = False
```

### Explanation:
- Suppresses warnings from **`schema_to_class`** module for a cleaner output.

## 3. **Parsing Command-Line Arguments**

```python
def parse_args():
    parser = argparse.ArgumentParser()
    parser.add_argument('-sc', dest='scen', type=str, required=True)

    return parser.parse_args()
```

### Explanation:
- Defines the **`parse_args()`** function to accept a scenario (`-sc`) argument that must be passed when executing the script.

## 4. **Printing Configuration Parameters**

```python
def _print_params():
    """
    Just a useful convenient function for the user.
    """
    print("test_run: ", par.test_run)
    print("iopath: ", par.iopath)
    print("expname: ", par.expname)
    print("homepath: ", par.homepath)
    print("user: ", par.user)
    print("serialize: ", par.serialize)
    print("num_seeds: ", par.num_seeds)
    print("years: ", par.years)
    print("pull_from_serialization: ", par.pull_from_serialization)
```

### Explanation:
- **`_print_params()`** is a utility function to display key configuration parameters, making it easier to verify settings before running the experiment.

## 5. **Post-Run Function**

```python
def _post_run(experiment: Experiment, **kwargs):
    """
    Add extra work after run experiment.
    Args:
        experiment: idmtools Experiment
        kwargs: additional parameters
    Return:
    """
    pass
```

### Explanation:
- **`_post_run()`** is a placeholder function to handle any actions needed after running an experiment. Currently, it doesn't perform any operations.

## 6. **Configuring the Experiment**

```python
def _config_experiment(**kwargs):
    """
    Build experiment from task and builder. task is EMODTask. builder is
    SimulationBuilder used for config parameter sweeping.
    Return:
        experiment
    """
    scen = kwargs.get('scen')
    expname = f'{par.expname}_{scen}'
    builders = get_sweep_builders(**kwargs)
    task = get_task(**kwargs)

    if manifest.SIF_PATH:
        task.sif_path = manifest.SIF_PATH

    ts = TemplatedSimulations(base_task=task, builders=builders)
    experiment = Experiment.from_template(ts, name=expname)

    suite = Suite(name=par.suitename)
    suite.uid = par.suitename

    now = dt.now()
    now_str = now.strftime("%Y_%m_%d_%H_%M")
    expid = f'{expname}_{now_str}'
    experiment.uid = expid
    suite.add_experiment(experiment)

    return experiment
```

### Explanation:
- **`_config_experiment()`** creates an experiment:
  - **`get_sweep_builders()`** and **`get_task()`** are functions to get simulation builders and tasks, respectively.
  - **`TemplatedSimulations`**: Combines tasks and builders to generate simulations.
  - **`Experiment`**: Created from the template with a unique experiment name.
  - **`Suite`**: Created to group experiments and associate with a unique ID.
  - Adds a timestamp to ensure that each experiment ID is unique.

## 7. **Running the Experiment**

```python
def run_experiment(**kwargs):
    """
    Get configured calibration and run
    Args:
        kwargs: user inputs

    Returns: None

    """
    # make sure pass platform through
    kwargs['platform'] = platform

    experiment = _config_experiment(**kwargs)
    experiment.run(wait_until_done=False, wait_on_done=False)
    _post_run(experiment, **kwargs)
```

### Explanation:
- **`run_experiment()`** runs the configured experiment:
  - Passes in the platform configuration.
  - Calls **`_config_experiment()`** to set up the experiment.
  - Runs the experiment asynchronously, allowing other tasks to continue.
  - Calls **`_post_run()`** to handle any post-run actions.

## 8. **Main Execution Block**

```python
if __name__ == "__main__":
    args = parse_args()

    # To use Slurm Platform: Specify job directory
    platform = Platform('SLURM_LOCAL', job_directory=manifest.job_dir,
                        partition=manifest.partition,
                        account=manifest.account,
                        modules=['singularity'],
                        time='3:00:00',
                        mem_per_cpu='3G',
                        max_running_jobs=manifest.max_running_jobs,
                        run_on_slurm=False)

    # If you don't have Eradication, un-comment out the following to download it
    # import emod_malaria.bootstrap as dtk
    #
    # dtk.setup(pathlib.Path(manifest.eradication_path).parent)
    # os.chdir(os.path.dirname(__file__))
    # print("...done.")

    run_experiment(scen=args.scen)
```

### Explanation:
- **`if __name__ == "__main__":`**: Indicates that this block should only be executed if the script is run directly.
- **`args = parse_args()`**: Parses command-line arguments.
- **`Platform()`**: Configures the SLURM platform with necessary settings such as partition, account, job directory, and resources.
- **`run_experiment()`**: Executes the experiment based on the scenario provided as input.

## Summary
- The script sets up a malaria simulation using EMOD tools and Idmtools.
- It handles configuration, creation, and execution of experiments.
- Command-line arguments are used to specify the scenario to be simulated.
- The SLURM platform is used for job submission.

This script provides a systematic approach to creating, running, and managing malaria simulations. If you need more information on individual functions or configurations, feel free to explore the referenced modules or contact the authors.

