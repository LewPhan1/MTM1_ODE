# MTM1 / Snx17 ODE model

This repository contains the ODE models used to explore how MTM1 and Snx17 affect PI(3)P on membranes. The main model follows binding of Snx17, MTM1 and a PI(3)P probe, formation of membrane complexes, and conversion of PI(3)P to PI by MTM1. There are also notebooks for parameter fitting, comparing simulations with and without Snx17, and adding VPS34-mediated PI(3)P production.

The most basic notebook `ODE_v0.ipynb`.

## Downloading and running the notebooks

The notebooks are standard Jupyter Notebook (`.ipynb`) files. These were run through Anaconda.

### 1. Install Anaconda

Download and install Anaconda from:

https://www.anaconda.com/download

Most of the Python packages used here are included with Anaconda. The main dependencies are:

- Python
- Jupyter Notebook
- NumPy
- SciPy
- pandas
- Matplotlib
- PyYAML

If PyYAML or another package is missing, open Anaconda Prompt and run:

```bash
conda install numpy scipy pandas matplotlib pyyaml
```

### 2. Download the repository

The simplest option is to open the repository on GitHub:

https://github.com/LewPhan1/MTM1_ODE

Click **Code -> Download ZIP**, then extract the folder somewhere on your computer.

Alternatively, with Git installed:

```bash
git clone https://github.com/LewPhan1/MTM1_ODE.git
```

### 3. Open the notebooks

Open **Anaconda Navigator**, launch **Jupyter Notebook**, and navigate to the downloaded repository folder. Click on any `.ipynb` file to open it.

The notebooks expect their YAML (where parameters values are stored) and experimental data files (for fitting or graphical overlays) to be in the same folder. Once a notebook is open, use **Run -> Run All Cells**.

If a value in a YAML file is changed, restart the kernel and run the notebook again from the beginning. This makes sure the new parameter values are loaded correctly.

You can also start Jupyter from Anaconda Prompt:

```bash
cd path/to/MTM1_ODE
jupyter notebook
```

## ODE model structure

The current model contains 15 states.

There are three main parts to the model. `Sa`, `Ma` and `Pa` are undelivered Snx17, MTM1 and probe. These move into their free solution states (`Sf`, `Mf` and `Pf`) with the first-order delivery rate `ka`. The proteins can then bind membrane PI(3)P, PI or each other to form the membrane-bound states.

The basic state order is:

```text
Sa, Ma, Pa,
Sf, Mf, Pf,
SbA, MbA, PbA, PbB, SMb, SbC,
A, B, C
```

Binding reactions use the general form:

```text
binding rate = kon * free species * free binding site - koff * bound species
```

For the 2D membrane reactions, concentrations are converted between nM and molecules/um2 using the configured reaction volume and total membrane surface area.

MTM1 converts PI(3)P to PI. In the main model this is calculated from both PI(3)P-bound MTM1 and the membrane Snx17-MTM1 complex:

```text
v_MTM1 = keff_M * MbA * free_PI3P
       + keff_SM * SMb * free_PI3P
```

The standard model therefore has:

```text
dPI3P/dt = -v_MTM1
dPI/dt   =  v_MTM1
```

The VPS34 version adds PI to PI(3)P conversion in the opposite direction.

## Key to model terms

| Term | Meaning |
| --- | --- |
| `S` | Snx17 |
| `M` | MTM1 |
| `P` | PI(3)P probe |
| `A` | Total PI(3)P pool |
| `B` | Total PI pool |
| `C` | Total NPxY cargo pool |
| `Sa`, `Ma`, `Pa` | Undelivered Snx17, MTM1 and probe |
| `Sf`, `Mf`, `Pf` | Free Snx17, MTM1 and probe in solution |
| `SbA` | PI(3)P-bound Snx17 |
| `MbA` | PI(3)P-bound MTM1 |
| `PbA` | Probe bound to PI(3)P |
| `PbB` | Probe bound to PI |
| `SMb` | Membrane-bound Snx17-MTM1 complex |
| `SbC` | Membrane-bound Snx17-cargo complex |
| `A_bind` | Free PI(3)P available for binding/catalysis after subtracting occupied PI(3)P |
| `B_bind` | Free PI available for binding |
| `C_bind` | Free cargo available for binding |
| `Kd_XY` | Dissociation constant for the indicated interaction |
| `kon_XY` | Association rate constant |
| `koff_XY` | Dissociation rate constant |
| `ka` | Delivery rate from the undelivered to solution state |
| `keff_M` | Effective catalytic coefficient for PI(3)P-bound MTM1 |
| `keff_SM` | Effective catalytic coefficient for the Snx17-MTM1 membrane complex |
| `PC_concentration` | Lipid concentration used to scale the starting membrane lipid and cargo fractions |

Membrane-bound MTM1 is calculated as:

```text
MbA + SMb
```

because each of these states contains one MTM1 molecule.

## Files and dependencies

The main notebooks use YAML files to keep the model parameters separate from the Python code. Experimental CSV filenames and column names are also set in the YAML.

```text
ODE_v0.ipynb
    -> ode_parameters.yaml
    -> experimental CSV specified in ode_parameters.yaml

ODE_Snx17_AUCR_v0.ipynb
    -> ode_parameters.yaml
    -> experimental CSV specified in ode_parameters.yaml

ODE_fitting_v0.ipynb
    -> ode_parameters.yaml
    -> experimental CSV specified in ode_parameters.yaml

ODE_with_VPS34.ipynb
    -> ode_parameters_VPS34.yaml
```

`ode_parameters.yaml` contains the standard model conditions, binding affinities, rate constants, membrane conditions, solver settings and file names.

`ode_parameters_VPS34.yaml` contains the same main model information, together with the VPS34 feedback and dynamic-cargo parameters used by `ODE_with_VPS34.ipynb`.

The experimental CSV is used for plotting or fitting experimental data. The exact filename is set under `files -> experimental_csv` in the YAML, so this can be changed without editing the main model code.

## `ODE_v0.ipynb`

This is the simplest version of the model and is the best notebook to use first.

It runs one equilibration and one assay using the conditions given directly in `ode_parameters.yaml`. The equilibrated state is carried into the assay and the proteins listed under the YAML `assay` section are then added.

The notebook plots:

- the equilibration trajectory;
- normalised PI(3)P depletion;
- normalised PI(3)P-probe signal loss together with the experimental data;
- total membrane-bound MTM1 (`MbA + SMb`).

The two assay trajectories are also exported as CSV files at 1 second intervals:

```text
results_normalised_signals.csv
results_membrane_bound_MTM1.csv
```

The output filename stem comes from `files -> results_csv` in the YAML.

## `ODE_Snx17_AUCR_v0.ipynb`

This notebook compares the model with and without Snx17.

It independently equilibrates and runs assays for 0 nM and 300 nM Snx17. It then calculates the area under the curve (AUC) for the assay trajectories and calculates the AUCR as:

```text
AUCR = AUC at 300 nM Snx17 / AUC at 0 nM Snx17
```

The primary comparison is the normalised loss of PI(3)P-probe signal. The notebook also compares PI(3)P depletion, raw bound probe and total PI(3)P.

It plots the two assay conditions, membrane-bound MTM1 and the experimental data. The final section carries out a simple sensitivity analysis by changing model parameters to 0.5x and 2x their baseline values and measuring the effect on the two AUCs and the AUCR.

The main outputs include separate trajectories for the two Snx17 conditions, an AUC comparison CSV and a sensitivity-analysis CSV.

## `ODE_fitting_v0.ipynb`

This notebook is used to fit model parameters against an experimental time course.

The fitting settings are near the top of the notebook. `FIT_PARAMETERS` controls which model parameters are fitted and their lower and upper bounds. The current example fits `Kd_SM` and `ka`, although other parameters can be added or removed from this list.

The notebook runs a complete equilibration and assay for every parameter set tested by the optimiser. Fitting uses bounded `scipy.optimize.least_squares` in log10 parameter space. By default the fitted observable is the normalised loss of PI(3)P-probe signal.

The notebook plots the initial model, fitted model, experimental data, fitting residuals and membrane-bound MTM1.

The main outputs are:

```text
fit_parameters.csv
fit_predictions.csv
fit_parameters.yaml
```

The fitted YAML is kept separate from the starting YAML so the original parameter file is not overwritten.

## `ODE_with_VPS34.ipynb`

This is the extended model containing PI(3)P production by VPS34.

VPS34 is not included as a separate concentration/state. Instead, PI to PI(3)P conversion is treated as a pseudo-first-order process in free PI. The effective VPS34 activity is controlled by instantaneous PI(3)P-dependent negative feedback using a Hill function:

```text
VPS34_activity = basal_fraction + (1 - basal_fraction) /
                 (1 + (PI3P / K_feedback)^hill_n)

v_VPS34 = k_max * VPS34_activity * free_PI
```

This changes the lipid equations to:

```text
dPI3P/dt = v_VPS34 - v_MTM1
dPI/dt   = v_MTM1 - v_VPS34
```

The notebook also contains a dynamic-cargo simulation. Cargo starts at the fraction specified in `dynamic_cargo_assay`, decays exponentially during the assay, and is then reset at the start of the next cycle. The current notebook runs three consecutive cargo cycles. All other model states continue between cycles rather than being reset.

The output plots include PI(3)P, cargo, membrane-bound MTM1 and the instantaneous VPS34 activity. The repeated simulation is exported to a CSV at 1 second intervals.

## Changing parameters

Most parameters should be changed in the YAML rather than directly in the notebooks. This includes protein concentrations, PI(3)P fraction, cargo fraction, binding affinities, catalytic coefficients, assay time and solver settings.

After changing a YAML file, restart the Jupyter kernel and run the notebook again from the first cell.
