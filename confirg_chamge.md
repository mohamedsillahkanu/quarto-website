# Files
larval_hab_csv = 'simulation_inputs/larval_habitats/monthly_habitats5.csv'
master_csv = 'guinea_DS_pop.csv'
rel_abund_csv = 'simulation_inputs/DS_vector_rel_abundance.csv'
samp_csv = 'simulation_priors/selected_particles_v1.csv'
scenariopath = os.path.join(manifest.IO_DIR, 'simulation_inputs/_scenarios_2023')
scenario_csv = 'scenarios_2023-2029.csv'

# Suite and experiment name
homepath = os.path.expanduser('~')
user = Path(homepath).name
expname = f'{user}_gin_2023-2029_3x'
suitename = f'{user}_gin_2023-2029_3x'

# Simulations settings
[34mnum_seeds = 15[0m
[34mser_num_seeds = 15[0m
[34myears = 7[0m
[34mser_date = 18 * 365[0m
[34mserialize = False[0m
[34mpull_from_serialization = True[0m
use_arch_burnin = False

filtered_report = years
yr_plusone = True
[34mevent_reporter = False[0m
[34madd_event_report = True[0m

burnin_id = "kbt4040_gin_2005-2022_3x_2023_09_27_09_56"

# DS Settings
[34mdemographic_suffix = '_wSMC_risk_wIP'[0m
climate_suffix = ''
climate_prefix = False
use_arch_input = False

# load dfs
scen_df = pd.read_csv(os.path.join(scenariopath, scenario_csv)).set_index('scen')
master_df = pd.read_csv(os.path.join(manifest.IO_DIR, master_csv))
master_df = master_df.set_index('DS_Name')
samp_df = pd.read_csv(os.path.join(manifest.IO_DIR, samp_csv))
ds_list = samp_df.DS_Name.unique()
#ds_list = ['Kerouane', 'Kissidougou']

# INTERVENTIONS
## Intervention Suite
int_suite =  InterventionSuite()
# hs unchanged
int_suite.hs_ds_col = 'DS_Name'
[34mint_suite.hs_duration = 365[0m
int_suite.hs_coverage_age = {
    'u5_coverage': [0, 5],
    'adult_coverage': [5, 100]
}
# itn
int_suite.itn_ds_col = 'DS_Name'
int_suite.itn_discard_distribution = 'weibull'
#int_suite.itn_discard_lambda = 2.39
int_suite.itn_cov_cols = ['U05', 'U10', 'U20', 'A20']
int_suite.itn_cov_age_bin = [0, 5, 10, 20]
[34mint_suite.itn_retention_in_yr = 1.69[0m
int_suite.itn_seasonal_months = [0, 91, 182, 274]
int_suite.itn_seasonal_values = [0.739, 0.501, 0.682, 1]
# smc
int_suite.smc_adherence = True
int_suite.smc_coverage_col = ['coverage_high_access_u5', 'coverage_low_access_u5',
                              'coverage_high_access_o5', 'coverage_low_access_o5']
int_suite.smc_access = ['High', 'Low', 'High', 'Low']
int_suite.smc_agemins = [0.25, 0.25, 5, 5]
int_suite.smc_agemax_type = ['fixed', 'fixed', 'fixed', 'fixed']
int_suite.smc_agemaxs = [5, 5, 10, 10]
int_suite.smc_leakage = False
# pmc
int_suite.pmc_touchpoint_col = 'pmc_age_days'
int_suite.pmc_start_col = 'simday'
int_suite.pmc_coverage_col = 'pmc_coverage'
# rtss
int_suite.rtss_auto_changeips = True

# Config prefix
def set_config(config):
    initialize_config(config, manifest, years, serialize)

    [34mconfig.parameters.x_Temporary_Larval_Habitat = 3[0m
    [34mconfig.parameters.x_Base_Population = 3[0m
    [34mconfig.parameters.x_Birth = 3[0m

    config.parameters.Report_Event_Recorder = 1
    config.parameters.Report_Event_Recorder_Individual_Properties = []
    config.parameters.Report_Event_Recorder_Events = ['Received_NMF_Treatment', 
                                                      'Received_Severe_Treatment',
                                                      'Received_Treatment',
                                                      'NewClinicalCase']
    config.parameters.Report_Event_Recorder_Ignore_Events_In_List = 0

    [34mset_species_param(config, 'arabiensis', 'Anthropophily', 0.88, overwrite=True)[0m
    [34mset_species_param(config, 'arabiensis', 'Indoor_Feeding_Fraction', 0.5, overwrite=True)[0m
    [34mset_species_param(config, 'funestus', 'Anthropophily', 0.5, overwrite=True)[0m
    [34mset_species_param(config, 'funestus', 'Indoor_Feeding_Fraction', 0.86, overwrite=True)[0m
    [34mset_species_param(config, 'gambiae', 'Anthropophily', 0.74, overwrite=True)[0m
    [34mset_species_param(config, 'gambiae', 'Indoor_Feeding_Fraction', 0.9, overwrite=True)[0m

    return config

