# Roadmap Model

The ICCT’s Roadmap model is a global transportation emissions model covering all on-road vehicle activity in over 190 countries. The Roadmap model is intended to help policymakers worldwide to identify and understand trends in the transportation sector, assess emission impacts of different policy options, and frame plans to effectively reduce emissions of both greenhouse gases (GHGs) and local air pollutants. It is designed to allow transparent, customizable estimation of transportation emissions for a broad range of policy cases.

Originally developed for the project [_Global progress toward soot-free diesel vehicles_](https://theicct.org/sites/default/files/publications/Global_progress_sootfree_diesel_2019_20190920.pdf) (Global Roadmap PCA4), the Python Roadmap model is the successor to the legacy Excel Roadmap. Since July 2020, this model has been exclusively used for global and regional analyses, and will continue to be updated with new data and features.

To cite the Roadmap model, reference the model documentation ([coming soon](https://github.com/joshicct/globalprogress/issues/41)).


Table of Contents
=================

  * [Installation](#installation)
     * [Requirements](#requirements)
     * [Setup](#setup)
  * [Scope](#scope)
  * [User guide](#user-guide)
     * [Configuration file](#configuration-file)
        * [Main options](#main-options)
        * [Inputs section](#inputs-section)
        * [Outputs section](#outputs-section)
     * [Model inputs](#model-inputs)
        * [Sales fuel shares](#sales-fuel-shares)
        * [Sales](#sales)
        * [Energy intensity](#energy-intensity)
        * [Fuel sulfur content](#fuel-sulfur-content)
        * [New vehicle emission standards](#new-vehicle-emission-standards)
        * [Used vehicle import restrictions](#used-vehicle-import-restrictions)
        * [Scrappage policies](#scrappage-policies)
        * [Electric grid carbon intensity](#electric-grid-carbon-intensity)
        * [Alternate fleet data](#alternate-fleet-data)
     * [Model outputs](#model-outputs)
     * [Tutorials](#tutorials)
        * [Running Roadmap from a Python script](#running-roadmap-from-a-python-script)
        * [Activity calibration](#activity-calibration)
        * [Stock calibration](#stock-calibration)
  * [Contributing](#contributing)
     * [Tips](#tips)
     * [Collaborating using git and GitHub](#collaborating-using-git-and-github)
     * [Style guide](#style-guide)
        * [Spacing](#spacing)
        * [Case](#case)
        * [String formatting](#string-formatting)
        * [Vectorization in pandas](#vectorization-in-pandas)
        * [DataFrame bracket accessor](#dataframe-bracket-accessor)
        * [DataFrame methods](#dataframe-methods)
        * [Avoid "inplace" operations](#avoid-inplace-operations)
        * [Explicitly use "rows" or "columns" operations over the non-default axis](#explicitly-use-rows-or-columns-operations-over-the-non-default-axis)

## Installation

### Requirements

- Python 3.7 or newer
- `scipy>=1.2.0`
- `pandas>=1.00`
- `numpy>=1.15`
- `configobj>=5.0.0`
- `openpyxl>=3.0.0`

The Roadmap model was developed and tested for macOS and is not guaranteed to work on other operating systems. 

### Setup

1. Download the model
    * If you just need a stable version, download the source code of a published [release](https://github.com/joshicct/globalprogress/releases)
    * If you want the latest version or intend to update the model in the future, it is recommended that you [clone this repository](https://docs.github.com/en/free-pro-team@latest/github/creating-cloning-and-archiving-repositories/cloning-a-repository)
2. Add the MoMo database input Excel file (DataBaseFinal-2020-06-09.xlsm) to inputs/raw
    * Ask a member of the Modeling Team for this file or download it from Egnyte ([here](https://theicct.egnyte.com/navigate/file/d53cc583-4263-4902-bfb4-40205a0a7f11))
3. Navigate into the model directory (in Terminal)
    * `cd globalprogress`
4. Install requirements (optional)
    * If using conda, install the prerequisite packages with `conda install scipy pandas configobj openpyxl` (numpy is a dependency of scipy and will be installed with it)
    * Otherwise, the required packages will be automatically installed with pip in the next step
5. Install the model
    * Run `python -m pip install -e .`
6. Build the data inputs
    * Type `make`
    * _Note_: if you get an error here, try running `xcode-select --install` and trying again
    * _Note_: if you get the error `make: *** No targets specified and no makefile found.  Stop.` check to ensure you are running make from the same directory containing the Makefile
7. The model is now ready to run! Try running the example with `python roadmap/model.py example/example_config.cfg`

## Scope

The Python Roadmap model covers:
* 190+ countries
* on-road vehicles
* historical and projected
* 15 local air pollutant and CO<sub>2</sub> emissions

Vehicle categories:

| Roadmap Name | Vehicle Category | Description |
| ------------ | ---------------- | ----------- |
| HDT          | HDV              | Heavy-duty trucks (GVW > 15 tonnes) |
| MDT          | HDV              | Medium-duty trucks (GVW 3.5-15 tonnes) |
| Bus          | HDV              | Buses |
| LCV          | LDV              | Light commercial vehicles (GVW < 3.5 tonnes) |
| PC           | LDV              | Passenger cars |
| MC           | MC               | Two and three wheelers |

Fuel categories:

| Roadmap Name |  Description |
| ------------ |  ----------- |
| Diesel       | Diesel and biodiesel |
| Gasoline     | Gasoline and ethanol blends |
| Gas          | Natural gas and LPG |
| Elec         | Zero-emission vehicles, including battery electric and hydrogen fuel cell |

## User guide

The Roadmap model is intended to be used to explore the implications of national and international policies. Without any adjustments the model runs with currently adopted policies and estimates future emissions assuming no further developments. This user guide is aimed at users who wish to provide inputs to update baseline assumptions or define alternate scenarios. For more detail on Roadmap inputs and methods, refer to the official documentation ([coming soon](https://github.com/joshicct/globalprogress/issues/41)).

### Configuration file

The configuration file instructs the model on runtime settings, output options, and any user inputs that need to read in. The configuration file format is an [expanded version of INI file syntax](https://configobj.readthedocs.io/en/latest/configobj.html#introduction). The basic pattern used to define configuration options is `keyword = value` (see the ConfigjObj [documentation](https://configobj.readthedocs.io/en/latest/configobj.html#the-config-file-format) for more details). A valid configuration file must be specified to run the model. The configuration file is divided in three parts:

| Section | Description |
| ------- | ----------- |
| Main | Sets options for scenario name, scope, and various run toggles. |
| Inputs | Directs the model to any user-provided scenario data. This section is optional; if empty or not provided Roadmap will run with baseline assumptions. |
| Outputs | Sets options for units, emissions, and years to write out. |

Example configuration files can be found in the _example/_ directory.

#### Main options

The main section defines basic scenario run options. All options are required.

| Option | Description |
| ------- | ----------- |
| scenario | Name of the scenario. |
| countries | List of countries to run, specified using their [ISO 3166-1 alpha-3 country codes](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-3) (referred to as "ISO" throughout the documentation).|
| biodiesel_effects | Boolean; adjust NOx and PM emissions for select ISOs based on biodiesel blend? |
| used_ei_as_entry_year | Boolean; Base energy intensity for used vehicles off of entry year, rather than the inferred model year? If False, energy intensity of used vehicles defaults to the intensity for vehicles of the inferred model year in the importing country. |
| mileage_degredation | Boolean; adjust mileage based on vehicle age? |
| diagnostic | Boolean; write intermediate files to a _diagnostic/_ output folder? |
| write_fleet_data | Boolean; write out complete fleet data before emissions calculations? |
| write | Boolean; write additional detailed scenario results to disk? Detailed results can be up to .5GB per scenario if running many ISOs, so it may be beneficial to set to `False` if running multiple scenarios. If `False`, summary results are still written to disk. |
| calibrate_energy | Boolean; calibrate energy consumption to IEA World Energy Balances? |
| calibrate_stock | Boolean; calibrate from-age-zero survival curves to stock data? |
| calibrate_activity | Boolean; calibrate sales growth rates to activity growth rates? |

See the [tutorials](#tutorials) below for more information on running stock and activity calibrations.

#### Inputs section

The main effort in building a scenario is typically in compiling the required scenario input files. This table gives an overview of the scenario inputs accepted by Roadmap. For details on structuring these inputs, see the [Model inputs](#model-inputs) section below.

| Option | Description |
| ----------- | ------- |
| sales_shares | Sales fuel shares |
| sales | New vehicle sales by ISO |
| energy_intensity | Energy intensity as share of baseline energy intensity |
| fuel_sulfur | Fuel sulfur timeline |
| new_std | Emission standard timeline for new vehicles |
| used_std | Emission standard timeline for used vehicles |
| scrappage | Scrappage policies (annual scrappage rates by vehicle and technology) |
| carbon_intensity | Electric grid carbon intensity |
| fleet | Entire fleet dataset including sales, stock, and activity |

#### Outputs section

The Outputs section controls how and where results are written. Note that limiting the scope of the outputs does not change the scope of the model run. For example, all model years will be run regardless of the values of min_year and max_year, but only those years will be included in the final output.

| Option | Description |
| ------ | ----------- |
| output_folder | Optional, defaults to _outputs/_. Directory to write all model outputs. If running a batch, this value is taken from the first configuration of the batch. |
| em_unit | Kilotonnes (kt). Alternatives not implemented. |
| energy_unit | Petajoules (PJ). Alternatives not implemented. |
| ems | Boolean; output emissions results? |
| pm_components | Boolean; output speciated PM emissions? If `True`, added species are BC, OC, NCOM, MetalsElements, NO3, NH4, PMOTHER, and NonECPM. |
| co2 | Scope of CO<sub>2</sub> emissions to output. Should be a subset of {`TTW`, `WTT`, `WTW`} or `None`. CO<sub>2</sub> emissions of the selected scopes will be output in megatonnes as columns, e.g. "TTW Mt CO2". |
| min_year | Minimum year to include in outputs |
| max_year | Maximum year to include in outputs |


### Model inputs

Policy scenarios are defined by specifying the appropriate inputs in the model configuration file. All scenario inputs are expected as paths to .csv files containing the relevant data. The sections below describe the structure and format of each supported input.

Scenario inputs do not have to cover the full scope of the model and can be given for a subset of countries, years, vehicles, or fuels unless noted otherwise. Where not specified by the user, default data are used.

Users must be careful that their inputs match the specifications below exactly. Roadmap allows two exceptions:

- All inputs also allow the addition of a `Scenario` column to designate multiple scenarios within one input. (The `scrappage` input is unique and does not accept this.)
- Any input that requires `CY` or `MY` as a column in the input also accepts the years pivoted wide (e.g. `2020`, `2021`, etc.).

#### Sales fuel shares

:gear: Configuration setting: `sales_shares`  
:white_check_mark: Required columns:  `ISO`, `Vehicle`, `Fuel`, `CY`, `Share`

The sales fuel shares input allows users to model policies that drive sales of a particular vehicle technology, usually EVs. This input will not adjust total sales of a vehicle type but shifts the portion of sales that are a specific technology.

It is not necessary to provide the shares of every fuel type. For example, to model a policy requiring that 50% of PC sales are EVs in 2030, one row of a user's input might be the following:

| ISO | Vehicle | Fuel | CY  | Share |
| --- | ------- | ---- | --- | ----- |
| MEX | PC | Elec | 2030 | .5 |

The model would take the default sales in 2030 and distribute shares of other sales proportionally. So, if the original sales were:

| ISO | Vehicle | Fuel | CY  | Sales |
| --- | ------- | ---- | --- | ----- |
| MEX | PC | Elec | 2030 | 0 |
| MEX | PC | Diesel | 2030 | 20 |
| MEX | PC | Gasoline | 2030 | 80 |
| MEX | PC | Gas | 2030 | 0 |

The final modeled sales would become:

| ISO | Vehicle | Fuel | CY  | Sales |
| --- | ------- | ---- | --- | ----- |
| MEX | PC | Elec | 2030 | 50 |
| MEX | PC | Diesel | 2030 | 10 |
| MEX | PC | Gasoline | 2030 | 40 |
| MEX | PC | Gas | 2030 | 0 |


#### Sales

:gear: Configuration setting: `sales`  
:white_check_mark: Required columns: `ISO`, `Vehicle`, `CY`, `Sales`

This input overwrites absolute historical total vehicle sales by ISO. Any sales given after the model base year will be dropped (absolute numbers of future sales are projected from the model base year and cannot be directly modified).

#### Energy intensity

:gear: Configuration setting: `energy_intensity`  
:white_check_mark: Required columns: `ISO`, `Vehicle`, `Fuel`, `MY`, `Unit`, `Value`

Alternative values for energy intensity can be specified in two ways:

1. As absolute energy intensity (unit: `MJpKM`)
2. As share of baseline energy intensity (unit: `share`)

A mix of the two can be given in a single input by specifying the corresponding unit in the `Unit` column. In the case where energy intensity is supplied directly, just the values provided are used. The second case, however, has a few nuances. For example, consider the following input table:

| ISO | Vehicle | Fuel | Unit | 2018 | 2020 | 2023 |
| --- | ------- | ---- | ---- | ---- | ---- | ---- |
| MEX | PC | Gasoline | share | 1 | .98 | .95 |
| MEX | HDT | Diesel | share |   | .95 | |

First, remember that any input that requires `CY` or `MY` can be given with the years as columns, so this is still a valid input. Although only 3 years were specified here, all years from 2018 until the model end year (2050) will be derived from this input. The first `MY` provided is the baseline year to which all further adjustments are applied. Shares for years not given explicitly are assigned values equal to the previous year with a value. In the end, this input would be interpreted identically to the one below:

| ISO | Vehicle | Fuel | Unit | MY | Value |
| --- | ------- | ---- | ---- | ---- | ---- |
| MEX | PC | Gasoline | share | 2018 | 1 |
| MEX | PC | Gasoline | share | 2019 | 1 |
| MEX | PC | Gasoline | share | 2020 | .98 |
| MEX | PC | Gasoline | share | 2021 | .98 |
| MEX | PC | Gasoline | share | 2022 | .98 |
| MEX | PC | Gasoline | share | 2023 | .95 |
| MEX | HDT | Diesel  | share | 2018 | 1 |
| MEX | HDT | Diesel  | share | 2019 | 1 |
| MEX | HDT | Diesel  | share | 2020 | .95 |
| MEX | HDT | Diesel  | share | 2021 | .95 |
| MEX | HDT | Diesel  | share | 2022 | .95 |
| MEX | HDT | Diesel  | share | 2023 | .95 |

The value of the final year (.95 for PC Gasoline and .98 for HDT Diesel) is extended to 2050. Finally, Roadmap applies these shares to the default energy intensity in 2018 for each vehicle and fuel type. So, for example, if HDT Diesel energy intensity were 10 MJ/km in 2018, this input would set it to 9.5 MJ/km for years 2020-2050.

#### Fuel sulfur content

:gear: Configuration setting: `fuel_sulfur`  
:white_check_mark: Required columns:  `ISO`, `CY`, `Diesel`, `Gasoline`

The timeline of the fuel sulfur content (in ppm) of diesel and gasoline; sulfur content of natural gas is not considered. Users are only required to provide the years where the fuel sulfur content changes. For example, if a user provides the following table:


| ISO | CY  | Diesel | Gasoline |
| --- | --- | ------ | -------- |
| MEX | 2015 | 500 | 55 | 
| MEX | 2017 | 161 |    | 
| MEX | 2019 | 15  |    | 
| MEX | 2021 |     | 15 | 

the model will fill the rest in as follows:

| ISO | CY  | Diesel | Gasoline |
| --- | --- | ------ | -------- |
| MEX | 2015 | 500 | 55 | 
| MEX | 2016 | 500 | 55 |
| MEX | 2017 | 161 | 55 |
| MEX | 2018 | 161 | 55 |
| MEX | 2019 | 15  | 55 | 
| MEX | 2020 | 15  | 55 |
| MEX | 2021 | 15  | 15 |
| ... | ... | ...  | ... | 
| MEX | 2050 | 15  | 15 | 

#### New vehicle emission standards

:gear: Configuration setting: `new_std`  
:white_check_mark: Required columns: `ISO`, `Fuel`, `VehCat`, `MY`, `NewVehCtrl`

New vehicle emission standards are defined by more aggregate vehicle categories. The `VehCat` column should contain only the groupings in the "Vehicle Category" column of the table [above](#scope). Provide the `NewVehCtrl` value (e.g. "Euro 6") for just the `MY` that the new policy takes effect.

#### Used vehicle import restrictions

:gear: Configuration setting: `import_restrictions`  
:white_check_mark: Required columns: `ISO`, `CY`, `Vehicle`, `Ban`, `AgeRest`, `StdRest`

Roadmap supports policies restricting used vehicle imports based on either age or emission standard. The three cases Roadmap models are:

1.	A complete ban of used vehicle imports
2.	Restrictions based on vehicle's age
3.	Restrictions based on vehicle's standard

For example, consider this subset of the default input (_inputs/final/import_restrictions.csv_):

| ISO | CY   | Vehicle | Ban   | AgeRest | StdRest
| --- | ---- | ------- | ----- | ------- | -------
| USA | 1970 | PC      | True  |         |
| MEX | 2011 | PC      | False | 10      |
| CRI | 2018 | PC      | False |         | Euro 4

This gets interpreted:

- USA bans all used PC imports from 1970 onwards
- MEX bans all used PC imports that are older than 10 years old in 2011
- CRI bans all used PC imports that are not Euro 4 in 2018

Note that in the two cases allowing imports, we conservativley assume used imports are as old possible while still complying with the restriction. The emissions performance of used vehicle imports is always equivalent to or worse than new vehicles.


#### Scrappage policies

:gear: Configuration setting: `scrappage`  
:white_check_mark: Required columns: `ISO`, `Vehicle`, `Fuel`, `Filter`, `Replacement_tech`, `Replacement_MY`, `Priority`, `[years]`

This input is used to model policies such as scrappage programs that are designed to shift vehicle activity from an older vehicle technology to a newer one faster than would result due to natural fleet turnover. For example, if major cities within a country implemented low-emission zones, a user may want to model accelerated turnover for a share of the country's vehicles. For each year, a user can specify a fraction of vehicles matching a filtering criteria that the policy applies to. Note that values are extended forwards and backwards for years outside the range specified.

The input table defines which vehicles the scrappage policy applies to using the `ISO`, `Vehicle`, `Fuel`, and `Filter` columns. Policies apply to one vehicle type at a time, however users can set the `Fuel` value to `All` to cover all fuel types. The `Filter` value is used to further specify a subset of vehicles, typically based on model year or age. The filter must be written using [_pandas_ `query()` syntax](https://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#indexing-query) and should apply to the `MY` or `Age` variables (although technically a filter could be written based on `EntryAge`, `AgeSinceEntry`, `Sales`, `Stock`, `VKTpVeh`, `VKT`, or `MJpKM` as well).

The `Replacement_tech` and `Replacement_MY` columns allow the user to define the vehicles that should replace those affected by the policy. `Replacement_tech` should specify either a single fuel or `All`. The year given for `Replacement_MY` defines the model year of the replacement vehicles unless `Replacement_MY` is greater than the year of the policy in which case the smaller year is used.

For example, consider the following input table:

| ISO | Vehicle | Fuel | Filter | Replacement_tech | Replacement_MY | Priority | 2020 | 2021 | 2022 | 2023 | 2024 | 2025 |
| --- | ------- | ---- | ------ | ---------------- | -------------- | -------- | ---- | ---- | ---- | ---- | ---- | ---- |
| MEX | Bus | Diesel | MY <= 2019 | Elec | 2025 | 1 | 0 | .1 | .2 | .3 | .4 | .5 |

This table defines a scrappage policy affecting diesel buses in Mexico, starting in 2021. For each year after 2020, a share of diesel HDTs model year 2019 or older are "scrapped", modeled as a shift in vehicle activity.

As a simple example, assume that in 2021 diesel buses traveled a total of 1,000,000 km. Of those total km, let’s assume that 90% was attributed to buses made before 2020 that are therefore subject to this policy. Because the input specifies that 10% of these vehicles are to be scrapped, the total “banned” activity is 90,000 km. Total VKT is always preserved, so all 90,000 km are assigned to new electric buses. 

From 2021 to 2024, the `Replacement_MY` is in the future, so all activity is shifted to age zero vehicles. The same holds for 2025. In 2026 (remember that scrappage fractions are extended forward) however, all banned activity is assigned to age 1 (model year 2025) vehicles.

The `Priority` column can be used to specify the order to apply multiple overlapping policies (smaller numbers are given higher priority).


#### Electric grid carbon intensity

:gear: Configuration setting: `carbon_intensity`  
:white_check_mark: Required columns: `ISO`, `Fuel`, `CY`, `ci` 

Roadmap uses default values for the carbon intensity of electricity sourced from the IEA’s World Energy Outlook (WEO) 2019 report. The report estimates electricity carbon intensity for 32 global regions in the years 2018, 2020, 2025, 2030, 2035, 2040, and 2050. Users can use this configuration setting to specify alternate grid decarbonization pathways.

#### Alternate fleet data

:gear: Configuration setting: `fleet`  
:white_check_mark: Required columns: `MoMo_DB`, `ISO`, `Vehicle`, `Fuel`, `NewUsed`, `CY`, `MY`  
:grey_question: Optional columns: `Sales`, `Stock`, `VKTpVeh`, `VKT`, `MJpKM`

In cases where users wish to run the emissions calculations with a custom fleet inventory, this setting can be used to point to an alternate dataset. This dataset must be mapped to the Roadmap region, vehicle, and fuel identifiers. Any of the optional columns may be given, however only `VKT` and `MJpKM` are used for the emissions calculations. Note that this input does not need to cover all vehicle types, fuel types, years, etc., but that default data will be used for any category not specified.


### Model outputs

A model run with `write = True` will write out results for each scenario to the configuration's `output_folder`. Each scenario has two outputs written:

1. \[Scenario\]_fleet_detail.csv – the most detailed output dataset with all results broken down by age, control technology, and new/used status
2. \[Scenario\]_summary.csv – the same set of results, but without detail by age

A third output, _results_all_scenarios.csv_ is also written with the combined results of multiple scenarios (if only a single scenario was run, this file is named _results\_\[Scenario\]_). This output is less detailed, reporting results by country, vehicle, fuel, and year.


### Tutorials

#### Running Roadmap from a Python script

Because Roadmap is a Python package, you can import it to a custom script just like any other package. You can run the model by calling the main function, `roadmap.main` and passing it a configuration file. This can either be a filepath to a configuration file or a ConfigObj object.

For example, assume you had several configuration files in the directory _path/to/configs_. You could run these scenarios from a script as follows:

```python
import roadmap.model as roadmap
import glob

# Create list of all configuration files in path/to/configs directory
my_configs = glob.glob("path/to/configs/*.cfg")

# Run Roadmap
roadmap.main(my_configs)
```

The above script is equivalent to running from the command line:

```bash
python roadmap/model.py path/to/configs/*.cfg
```

While it may be easier to run the example above from the command line, it can be useful to run Roadmap from a script if you need to do pre/post-processing or if you want more control over how the model is run. For example, you could write a wrapper script to run multiple scenarios in [parallel](https://docs.python.org/3/library/multiprocessing.html). This might look something like this:

```python
from multiprocessing import Pool
import roadmap.model as roadmap
import glob

# Create list of all configuration files in path/to/configs directory
my_configs = glob.glob("path/to/configs/*.cfg")

# Chop the list of configuration files into 4 chunks and run each on a separate process
with Pool(4) as p:
    p.map(roadmap.main, my_configs)
```

This flexibility can be helpful in a number of cases, and is especially useful for further processing of results.


#### Activity calibration

This tutorial goes over the steps to calibrate the model to exogenous activity estimates.

To calibrate the model to activity estimates, you must first provide an `activity_calibration_target` input. The target defines the scope (ISOs and vehicles) of the optimization. An input defining activity growth rates over one period for the USA could look like this:

| MoMo_Region | ISO | Vehicle | Range | CAGR |
| ----------- | --- | ------- | ----- | ---- |
| USA | USA | LCV | 2018-2050 | 0.0056 |
| USA | USA | PC | 2018-2050 | 0.0056 |
| USA | USA | MC | 2018-2050 | 0.0056 |
| USA | USA | HDT | 2018-2050 | 0.0105 |
| USA | USA | MDT | 2018-2050 | 0.0105 |
| USA | USA | Bus | 2018-2050 | 0.0032 |

Next, you need to give the structure of the sales growth periods to calibrate to:

| MoMo_Region | Vehicle | Range | CAGR |
| ----------- | ------- | ----- | ---- |
| USA | LCV | 2018-2050 | 0.01 |
| USA | PC | 2018-2050 | 0.01 |
| USA | MC | 2018-2050 | 0.01 |
| USA | HDT | 2018-2050 | 0.01 |
| USA | MDT | 2018-2050 | 0.01 |
| USA | Bus | 2018-2050 | 0.01 |

You can provide multiple ranges here, indicating that the activity growth period(s) should calibrate that many sales CAGRs ranges. The value in the "CAGR" column can be any number, although it is used as an initial guess for the calibration so it is recommended to keep it small.

Now build a configuration file for the activity calibration. Set `calibrate_activity` to `True` and all other toggles to `False`.

In the `[Inputs]` section of the config, add the inputs `activity_calibration_target` and `activity_calibration_guess`. Set their values equal to the paths to csv files of the tables created in the steps above.

Finally, the calibration is ready to be run. Running the model with the configuration file you created will run in activity calibration mode. It may take several hours to complete, depending on how many ISOs are being calibrated. When finished, three additional outputs will be written out:

* activity_calibration_\[iso\].csv - Reports sales, stock, and VKT pre- and post-calibrating
* activity_calibration_cagrs_\[iso\].csv - Reports calibrated sales CAGRs. To use these CAGRs in the model, update inputs/final/sales_cagr.csv and remake.
* activity_calibration_act_cagrs_\[iso\].csv - Reports calibrated activity CAGRs. A successful calibration will result in very little difference between the "post_calibration" and "target_calibration" values.

Note that the "\[iso\]" in the filename will be the final ISO calibrated and that the file will contain all the ISOs run.

#### Stock calibration

This tutorial goes over the steps to calibrate the model's survival curves based on real-world stock and sales data.

1. Set `calibrate_stock = True`
2. Run the model. Calibration is done to stock for the model base year.
3. Check the outputs:
   - The output _optimized_avg_ret_age.csv_ contains the calibrated T<sub>m</sub> values.
   - The output _calibration_report_ gives total stock before and after calibration.
4. Update the model input _Model-inputs-global-2019.xlsx_ tab _AvgRetAge_ with the calibrated values. (Note: this input likely to change.)


## Contributing

### Tips
* Install a good editor such as PyCharm, Sublime text 2, or Atom
* Make sure your text editor is the default for git. You can also use the default. For example, for Sublime text 2, it is `/Applications/Sublime\ Text\ 2.app/Contents/SharedSupport/bin/subl`.  If it works, you will do the following in your home directory to use `subl` in the terminal instead of the full directory.
    1. open `~/.bash_profile`
    2. add `alias subl="[Directory of Sublime, as the example above]"` in your bash_profile
    4. `source ~/.bash_profile`
    5. git config –global core.editor `subl -n -w`. If an error shows up saying that subl command not found when you run git commands, try `git config --global core.editor "/Applications/Sublime\ Text\ 2.app/Contents/SharedSupport/bin/subl"`. You should replace the text in quotes with your sublime directory.
* Delete merged branches from pull requests on Github
* Pull and push small and often
* Always run `git pull` before `git push`
* Tag final versions used for major projects/publications

### Collaborating using git and GitHub
Whenever contributing updates the Roadmap model, track your changes with git. This creates a clear record of how the model got to its present form and allows anyone to go to any historical version of the model.
1. When making a change, first create a branch in your local repo.
    * `git checkout -b my-new-branch`
2. Then, make your changes, committing as you go. Suppose you're changing two features of the file `a.py`. Make your first change, then commit it to your local repository.
    * `git add a.py`
    * `git commit -m "a descriptive message"`
3. Commit messages should focus on _why_ your changes were made, not _what_ specifically the changes were. (The specifics of the change can be seen more clearly using a diff.) Commit messages should follow the conventions detailed [here](https://gist.github.com/robertpainsi/b632364184e70900af4ab688decf6f53) and [here](https://chris.beams.io/posts/git-commit/).
4. When you are done with (or even if you are not done, but are ready to share) your updates, push your changes to the remote repository.
    * `git push origin my-new-branch`
5. To add your changes to the main version of the model, open the repo on GitHub, go to the branch, and make a pull request. Github shows all commits made by this pull request. Repo owner will get a message/email on the request.
6. When creating your pull request, ask relevant people on the Modeling Team to review. The reviewers can comment inline or leave a general comment on the update if changes are necessary.
7. If the pull request shows there would be a conflict merging your branch in, fix it by first pulling `master` (or the branch you are merging into), merging that branch into `my-new-branch`, and resolving conflicts. Then, commit your changes and push again.
8. When both parties are ready, merge and delete your branch. You’ll probably also want to delete your local branch as well to keep it clean (`git branch -d my-new-branch`).


### Style guide
For contributing to the Roadmap model, please refer to this style guide. Roadmap makes extensive use of the _pandas_ which provides flexible and robust data structures for manipulating complex data. The downside of this flexibility is that there can be many ways to accomplish the same result. This style guide covers several common cases and the preferred way to approach them.

#### Spacing
Use the code formatter [Black](https://black.readthedocs.io/en/stable/) (with line length 100 or 120, rather than the default 88). This tool gives a common look and allows you to not worry about spacing while writing code.

#### Case
Use snake_case for variables and column names. Abbreviations, acronyms, or names should generally follow this convention and remain lowercase. Exceptions include units, emission species, or when changing the case changes the meaning of the term.

**Good:**

* total_stock
* momo_sales
* kg_NOx
* Mt_CO2


**Bad:**

* totalStock
* MoMo_sales
* Kg_NOx

Constants should be specified as all uppercase, e.g. `MJ_PER_L_DIESEL`.

#### String formatting

* Use f-strings formatting instead of ‘%’ and ‘.format()’ string formatters
* Concatenate long strings with whitespace and the end of each line:

```python
example_string = (
    "Some long concatenated string, "
    "with good placement of the "
    "whitespaces"
)
```

#### Vectorization in pandas

* If you find yourself writing a for loop, there is likely a more efficient approach using vectorized `pandas` or `numpy` functions.
* Make use of the data structures. Don't [explicity loop over rows](https://engineering.upside.com/a-beginners-guide-to-optimizing-pandas-code-for-speed-c09ef2c6a4d6).

#### DataFrame bracket accessor

Use brackets instead of dot accessors for data access.

For example:

**Good:**
```python
df = pd.DataFrame({"vehicle": ["bus", "car"], "kt_NOx": [.002, .001]})

df["kg_NOx"] = df["kt_NOx"] * 1e6
```

**Bad:**
```python
df = pd.DataFrame({"vehicle": ["bus", "car"], "kt_NOx": [.002, .001]})

df.kg_NOx = df.kt_NOx * 1e6
```

#### DataFrame methods

Call DataFrame methods rather than pandas functions.

For example:

**Good:**
```python
df = pd.DataFrame({"vehicle": ["bus", "car"], "NOx": [2, 1.1], "SO2": [.4, .1]})

df.melt("vehicle", ["NOx", "SO2"], var_name="spec", value_name="tonnes")
```

**Bad:**
```python
df = pd.DataFrame({"vehicle": ["bus", "car"], "NOx": [2, 1.1], "SO2": [.4, .1]})

pd.melt(df, "vehicle", ["NOx", "SO2"], var_name="spec", value_name="tonnes")
```

#### Avoid "inplace" operations

For example:

**Good:**
```python
df = df.reset_index()
```

**Bad:**
```python
df.reset_index(inplace=True)
```

#### Explicitly use "rows" or "columns" operations over the non-default axis

_pandas_ allows axis to be specified by number, keyword, or "rows"/"columns". Be explicit when not using the default.

**Good:**
```python
df = pd.DataFrame({"NOx": [2, 1.1], "SO2": [.4, .1]}, index=["bus", "car"])

# Preferred
df.drop("bus")
df.drop(columns="NOx")

# Okay
df.drop(index="bus")
df.drop("NOx", axis="columns")
```

**Bad:**
```python
df.drop("bus", axis=0)
df.drop("NOx", axis=1)
```


_© 2021 The International Council on Clean Transportation. All rights reserved._
