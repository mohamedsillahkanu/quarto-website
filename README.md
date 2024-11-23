### Analyzer

```python
import os
import datetime
import pandas as pd
import numpy as np
import sys
import re
import random
from idmtools.entities import IAnalyzer
from idmtools.entities.simulation import Simulation
import manifest

## For plotting
import matplotlib.pyplot as plt
import matplotlib as mpl
import matplotlib.dates as mdates
```
- **`import os`**: Provides functions to interact with the operating system, such as file creation and directory handling.
- **`import datetime`**: Used for working with dates and times, essential for creating timestamps.
- **`import pandas as pd`**: Imports pandas for data analysis and manipulation; `pd` is the standard alias.
- **`import numpy as np`**: Imports numpy for numerical operations; commonly used for mathematical calculations.
- **`import sys`, `import re`, `import random`**: Imports basic Python libraries for system functions (`sys`), regular expressions (`re`), and generating random numbers (`random`).
- **`from idmtools.entities import IAnalyzer`**: Imports the `IAnalyzer` base class from idmtools, which is used to create custom analyzers for the simulations.
- **`from idmtools.entities.simulation import Simulation`**: Imports `Simulation` class to handle simulation-related operations.
- **`import manifest`**: Imports a `manifest` file that probably contains configuration and paths for files.
- **`import matplotlib.pyplot as plt`**: Imports `matplotlib` for plotting graphs, with `plt` as the alias.
- **`import matplotlib as mpl`**: Imports the base `matplotlib` module to access lower-level functions.
- **`import matplotlib.dates as mdates`**: For formatting date axes in plots.

### Define the `InsetChartAnalyzer` Class
```python
class InsetChartAnalyzer(IAnalyzer):
```
- **`class InsetChartAnalyzer(IAnalyzer):`**: Creates a class called `InsetChartAnalyzer`, inheriting from `IAnalyzer`. This class will be used to analyze simulation outputs.

#### `monthparser` Class Method
```python
    @classmethod
    def monthparser(self, x):
        if x == 0:
            return 12
        else:
            return datetime.datetime.strptime(str(x), '%j').month
```
- **`@classmethod`**: Declares `monthparser` as a class method that can be called on the class itself rather than an instance.
- **Purpose**: **Converts day-of-year (`%j`) into a month number**. If `x` is `0`, it returns `12` (December).
- **Reason**: Helps interpret time data, specifically day-of-year, in the context of months.

#### `__init__` Method for Initialization
```python
    def __init__(self, expt_name, sweep_variables=None, channels=None, working_dir=".", start_year=0):
        super(InsetChartAnalyzer, self).__init__(working_dir=working_dir, filenames=["output/InsetChart.json"])
        self.sweep_variables = sweep_variables or ["Run_Number"]
        self.inset_channels = channels or ['Statistical Population', 'New Clinical Cases', 'Blood Smear Parasite Prevalence', 'Infectious Vectors']
        self.expt_name = expt_name
        self.start_year = start_year
```
- **`def __init__(...)`**: Initializes an instance of `InsetChartAnalyzer`.
- **`super(...)`**: Calls the base class (`IAnalyzer`) constructor to initialize common parameters.
- **`working_dir`**: Sets the directory where the output will be stored.
- **`filenames=["output/InsetChart.json"]`**: Specifies the file to be analyzed.
- **`self.sweep_variables`**: Defines the sweep variables (e.g., parameters over which to vary simulation runs). Defaults to `Run_Number` if not specified.
- **`self.inset_channels`**: Sets the channels to analyze (e.g., metrics from the simulation output).
- **Purpose**: Sets up key parameters like output location, data sources, and sweep variables.

#### `map` Method to Extract Data
```python
    def map(self, data, simulation: Simulation):
        simdata = pd.DataFrame({x: data[self.filenames[0]]['Channels'][x]['Data'] for x in self.inset_channels})
        simdata['Time'] = simdata.index
        simdata['Day'] = simdata['Time'] % 365
        simdata['Year'] = simdata['Time'].apply(lambda x: int(x / 365) + self.start_year)
        simdata['date'] = simdata.apply(lambda x: datetime.date(int(x['Year']), 1, 1) + datetime.timedelta(int(x['Day']) - 1), axis=1)
```
- **`def map(self, data, simulation: Simulation):`**: Maps simulation data into a more usable form (i.e., a DataFrame).
- **`simdata = pd.DataFrame(...)`**: Extracts selected channels (`self.inset_channels`) from the data dictionary and creates a DataFrame.
- **`simdata['Time']`**: Adds a `Time` column based on the index, representing the simulation timestep.
- **`simdata['Day']`**: Calculates the day-of-year by using `Time % 365`.
- **`simdata['Year']`**: Converts the time into a year, accounting for `start_year`.
- **Purpose**: Converts time data into more human-readable formats like day and year.

#### `reduce` Method to Concatenate Data
```python
    def reduce(self, all_data):
        selected = [data for sim, data in all_data.items()]
        if len(selected) == 0:
            print("No data have been returned... Exiting...")
            return

        if not os.path.exists(os.path.join(self.working_dir, self.expt_name)):
            os.mkdir(os.path.join(self.working_dir, self.expt_name))

        adf = pd.concat(selected).reset_index(drop=True)
        adf.to_csv(os.path.join(self.working_dir, self.expt_name, 'All_Age_InsetChart.csv'), index=False)
```
- **`def reduce(...)`**: Reduces the mapped data across all simulations into a single file.
- **`selected = [data for sim, data in all_data.items()]`**: Collects data for all simulations.
- **`if len(selected) == 0:`**: Checks if there is any data to process; prints a warning if none.
- **Creates output directory**: If it doesn't already exist, create a folder for storing results.
- **`adf.to_csv(...)`**: Saves the concatenated data to a CSV file.
- **Purpose**: Collects data from multiple simulations, saves it for later use.

### Define `MonthlyPfPRAnalyzer` Class
This class performs more detailed analysis of malaria prevalence by month.

#### Initialization
```python
    def __init__(self, expt_name, sweep_variables=None, working_dir='./', start_year=0, burnin=None, filter_exists=False):
        super(MonthlyPfPRAnalyzer, self).__init__(working_dir=working_dir, filenames=["output/MalariaSummaryReport_monthly.json"])
        ...
```
- Similar to **`InsetChartAnalyzer`**, this sets parameters needed for the analysis, specifying a different JSON file (`MalariaSummaryReport_monthly.json`).

#### `filter` Method
```python
    def filter(self, simulation: Simulation):
        if self.filter_exists:
            file = os.path.join(simulation.get_path(), self.filenames[0])
            return os.path.exists(file)
        else:
            return True
```
- **`def filter(...)`**: Checks whether the specified file exists for the simulation.
- **Purpose**: Filters out simulations that do not have the necessary data files.

#### `map` Method
```python
    def map(self, data, simulation: Simulation):
        adf = pd.DataFrame()
        fname = self.filenames[0]
        age_bins = data[self.filenames[0]]['Metadata']['Age Bins']
        
        for age in range(len(age_bins)):
            d = data[fname]['DataByTimeAndAgeBins']['PfPR by Age Bin'][:-1]
            pfpr = [x[age] for x in d]
            ...
```
- **Extracts data for each age bin** from the JSON report, including various metrics like `PfPR`, `Annual Clinical Incidence`, etc.
- **Creates DataFrame** `simdata` for each metric and concatenates them into `adf`.
- **Purpose**: Extracts malaria prevalence and other metrics for different age groups, returning a consolidated DataFrame.

### Reduce Method for Monthly Analyzer
```python
    def reduce(self, all_data):
        selected = [data for sim, data in all_data.items()]
        if len(selected) == 0:
            print("\nWarning: No data have been returned... Exiting...")
            return

        if not os.path.exists(os.path.join(self.working_dir, self.expt_name)):
            os.mkdir(os.path.join(self.working_dir, self.expt_name))

        adf = pd.concat(selected).reset_index(drop=True)
        adf.to_csv(os.path.join(self.working_dir, self.expt_name, 'PfPR_ClinicalIncidence_monthly.csv'), index=False)
```
- **Concatenates all extracted data** and saves it as a CSV.
- **Purpose**: Generates a single consolidated CSV file containing malaria prevalence data for all simulations.

### Main Execution Section (`if __name__ == "__main__":`)
```python
if __name__ == "__main__":
```
- Runs only when the script is executed directly.

#### Set Experiment Variables and Directories
```python
    expts = {'week2_outputs' : '92e86035-aaf6-4cbf-be6d-6369586e6a2c'}
    jdir = manifest.job_directory
    wdir = os.path.join(jdir, 'my_outputs')
    if not os.path.exists(wdir):
        os.mkdir(wdir)
```
- **`expts`**: A dictionary mapping experiment names to IDs.
- **`jdir`** and **`wdir`**: Define job and working directories. Creates output directories if they do not exist.

#### Set Up Platform Context and Analyzers
```python
    with Platform('SLURM_LOCAL',job_directory=jdir) as platform:
        for expt_name, exp_id in expts.items():
            analyzer = [
                InsetChartAnalyzer(expt_name=expt_name, channels=channels_inset_chart, sweep_variables=sweep_variables, start_year=2023, working_dir=wdir),
                MonthlyPfPRAnalyzer(expt_name=expt_name, sweep_variables=sweep_variables, start_year=2023, working_dir=wdir)
            ]
```
- **`Platform('SLURM_LOCAL', job_directory=jdir)`**: Sets up a platform context (local execution on SLURM).
- **`analyzer` list**: Creates instances of both analyzers to process the experiment.

#### Analysis Manager
```python
            manager = AnalyzeManager(configuration={}, ids=[(exp_id, ItemType.EXPERIMENT)], analyzers=analyzer, partial_analyze_ok=True)
            manager.analyze()
```
- **`AnalyzeManager`**: Manages the analysis process for the specified analyzers and experiment IDs.
- **`manager.analyze()`**: Runs the analysis for all configured analyzers.

### Read and Plot Results
```python
    df = pd.read_csv(os.path.join(wdir, expt_name, 'All_Age_InsetChart.csv'))
    df['date'] = pd.to_datetime(df['date'])
    df = df.groupby(['date'] + sweep_variables)[channels_inset_chart].agg(np.mean).reset_index()
```
- **Reads the CSV output** from the analyzer.
- **Groups data by date and sweep variables** and calculates the average for each group.

#### Plotting Setup
```python
    fig1 = plt.figure('InsetChart', figsize=(12, 6))
    fig1.subplots_adjust(hspace=0.5, left=0.08, right=0.97)
    fig1.suptitle(f'Analyzer: InsetChartAnalyzer')
    axes = [fig1.add_subplot(2, 3, x + 1) for x in range(6)]
    for ch, channel in enumerate(channels_inset_chart):
        ax = axes[ch]
        for p, pdf in df.groupby(sweep_variables):
            ax.plot(pdf['date'], pdf[channel], '-', linewidth=0.8, label=p)
        ax.set_title(channel)
        ax.set_ylabel(channel)
        ax.xaxis.set_major_locator(mdates.MonthLocator(interval=12))
        ax.xaxis.set_major_formatter(mdates.DateFormatter('%Y'))
    if len(sweep_variables) > 0:
        axes[-1].legend(title=sweep_variables)
    fig1.savefig(os.path.join(wdir, expt_name, 'InsetChart.png'))
```
- **Sets up the plot** for each channel in `channels_inset_chart`.
- **Subplots**: Arranges each channel on a different subplot.
- **Plots each group's data** for each channel.
- **Formats x-axis** to show years instead of dates for better readability.
- **Saves the figure** in the output directory.

### Summary
- The script defines analyzers to process specific simulation outputs, convert them into a human-readable format, and save them.
- Analyzers are **executed through an `AnalyzeManager`**, which manages the workflow.
- The results are **saved as CSV** files and visualized using `matplotlib`.
- The script helps in **automating the analysis** of large-scale simulation data, enabling better insights.
