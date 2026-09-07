# ODE dimerisation: parameter fitting

`ODE_dimerisation_fitting.ipynb` models MTM1/Snx17 membrane binding, MTM1 dimerisation, and PI(3)P turnover. It fits selected parameters to an experimental CSV using bounded least squares in log10 parameter space.

## Setup

Keep these files in the same folder:

- `ODE_dimerisation_fitting.ipynb`
- `ode_parameters.yaml`
- Your experimental CSV

Install the required packages in a notebook cell if needed:

```python
%pip install numpy scipy pandas matplotlib pyyaml
```

Start Jupyter in that folder, or set `CONFIG_PATH` to the full path of your YAML file.

## Select your data

Edit the **Fit settings** cell:

```python
CSV_PATH = CONFIG_PATH.parent / "my_data.csv"
TIME_COLUMN = "time (min)"
TIME_TO_SECONDS = 60  # use 1 for time in seconds
SIGNAL_COLUMN = "WT"
```

The defaults come from the YAML. With the supplied YAML, the selected signal column is `WT`. Only one signal column is fitted per run.

The CSV needs a numeric time column and a numeric signal column. Blank/non-finite rows and observations outside the YAML assay interval are excluded and reported. Duplicate times are retained as replicate observations.

By default, the fitted model output is normalised probe-signal loss:

```text
1 - PbA(t) / max(PbA over the assay)
```

Prepare the experimental signal with a compatible normalisation: the notebook does not normalise the CSV automatically. Set `NORMALISATION = "initial"` to use the model signal at assay start as the reference instead. Other `OBSERVABLE` options are listed in the settings cell.

## Set the experimental conditions

Use the YAML for concentrations, membrane composition, and equilibration/assay times. You can override selected conditions in the notebook:

```python
CONDITION_OVERRIDES = {
    "equilibration.Snx17_conc": 0.0,
    "assay.MTM1_conc": 50.0,
}
```

These values must match the selected experimental trace. CSV column names do not automatically set concentrations.

Each trial runs equilibration, copies its final state, and adds the assay proteins without dilution before running the assay.

## Choose parameters to fit

The default fits `Kd_MA` only. Edit `FIT_PARAMETERS` to choose another parameter or add parameters:

```python
FIT_PARAMETERS = [
    {"path": "affinities.Kd_MA", "initial": None,
     "lower": 1.0, "upper": 1e6},
]
```

- `initial: None` uses the YAML value as the starting guess.
- Bounds and starting values must be positive.
- Supported paths are `affinities.*`, `catalysis.keff_M`, `catalysis.keff_SM`, `catalysis.keff_MM`, and `membrane.membrane_substrate`.
- All unselected parameters remain fixed. Dependent dissociation rates are rebuilt when affinities change.
- Catalytic rates are independent by default. Set `LINK_CATALYTIC_RATES = True` and fit `catalysis.keff_M` to use one shared coefficient for all three catalytic rates.

Start with one parameter: a single experimental trace may not uniquely determine several parameters. Set `SIGMA_COLUMN` to a measurement-SD column for weighted fitting; otherwise each observation has equal weight.

## Run and review

Select **Restart Kernel → Run All** after changing settings.

Review the fitted parameter table, convergence message, SSE/RMSE, fit overlay, and residual plot. The notebook flags bound hits and locally insensitive parameters, and checks the final predictions with tighter numerical settings. Successful optimisation alone does not establish parameter identifiability or biological validity.

## Output files

With the default `OUTPUT_PREFIX`, the export cell writes:

| File | Contents |
| --- | --- |
| `fit_dimerisation_parameters.csv` | Starting/fitted values, bounds, and bound flags |
| `fit_dimerisation_predictions.csv` | Observations, predictions, and residuals |
| `fit_dimerisation_curve.csv` | Fitted signal over the assay |
| `fit_dimerisation_parameters.yaml` | Model configuration containing fitted values |
| `fit_dimerisation_settings.json` | Data selection, fitting settings, and fit summary |

The original YAML is preserved. Re-running exports replaces files with the same output prefix. Change `OUTPUT_PREFIX` when saving a different fit. Export is blocked if the optimiser has not converged.

The notebook was checked using synthetic data for single- and two-parameter recovery. An experimental CSV is required to fit your own data.
