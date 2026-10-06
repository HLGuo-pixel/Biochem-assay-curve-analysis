# Assay Curve Analysis

MATLAB scripts for analysing biochemical assay data — 4-parameter logistic (4PL)
dose-response fitting, IC50 determination, time-course and kinetics analysis, and
luminescence readout statistics.

These scripts were written for day-to-day analysis of plate-reader data (HTRF,
NanoBRET, luminescence and absorbance assays). They are **standalone scripts**, not a
packaged toolbox: most expect input variables to already exist in the MATLAB
workspace. See [Usage](#usage).

## Contents

| Script | Purpose |
|---|---|
| `plotIC50.m` | **Function.** Fits a 4PL model to dose-response data and returns IC50, Hill slope and full parameters. Requires the Statistics Toolbox (`nlinfit`). |
| `IC50_curve_fix_end.m` | **Script.** Same 4PL fit, but using `fminsearch` with parameter penalties — **no Statistics Toolbox required**. Adds biological plausibility constraints. |
| `compound_kinetic_single_run.m` | Plots a time-course absorbance dataset with an interactive axis-control GUI. Exports a 300 dpi PNG. |
| `compound_kinetic_mutiple.m` | Averages replicate time-course datasets with SD error bars, plus a transformed **-ln(Ax/A0)** view for pseudo-first-order kinetics. Two interactive GUIs, two 300 dpi exports. |
| `Lumi_mean_bar_with_YUI.m` | Bar chart of mean luminescence per sample with SD error bars and jittered individual points, plus an interactive Y-axis control. Supports percentage normalisation (DMSO = 100%). |
| `DataLayoutTransfer.m` | Converts a flat numeric column into a multi-sheet Excel layout (8 values in C1:J1, 50 values in A2:J6). **Discards any remainder**, with a warning. |
| `DataLayoutTransfer_extend.m` | Same as above but **handles partial blocks by NaN-padding** — use this version. |

## Requirements

- MATLAB (developed on R2019b or later — needs `exportgraphics`, `uifigure`, `stack`, `groupsummary`)
- **Statistics and Machine Learning Toolbox** — only for `plotIC50.m` (`nlinfit`, `statset`).
  Use `IC50_curve_fix_end.m` if you don't have it.

## Usage

These are scripts, so the required variable must exist in your workspace first.

### IC50 / dose-response

`plotIC50.m` is callable directly:

```matlab
% data: table where column 1 = concentration, columns 2:end = replicates
[IC50, hillSlope, params] = plotIC50(data, 'Compound A', 'nM');
```

`IC50_curve_fix_end.m` runs on a workspace variable named `data` with the same layout:

```matlab
% import your table as `data`, then run the script
IC50_curve_fix_end
```

Both scripts:

- ignore zero-concentration (DMSO) wells when fitting
- fit the 4PL model `bottom + (top - bottom) / (1 + (x/IC50)^hill)`
- plot raw means ± SD with the fitted curve on a log x-axis
- report bottom, top, IC50 and Hill slope

`IC50_curve_fix_end.m` additionally:

- initialises parameters from the data (asymptotes from the extreme concentrations,
  IC50 from the geometric mean of concentrations)
- **penalises unphysical fits** during optimisation (bottom below 0, top above ~110%)
  and clamps the final parameters
- exports `IC50_Results.png`

### Time-course / kinetics

```matlab
% rename the variable on line 2 to match your data
compound_kinetic_single_run
```

Expected input: a table where column 1 is time and columns 2:end are groups.
Output: a line plot with an interactive window for adjusting axis limits and tick
intervals, exported to `Time_vs_Abs_Plot.png` at 300 dpi.

For replicate datasets, `compound_kinetic_mutiple.m` averages across up to N datasets
named `Rep1`, `Rep2`, … and adds standard-deviation error bars. It also produces a
transformed plot of **-ln(Ax/A0)** against time, which linearises pseudo-first-order
kinetics such as thiol-reactivity labelling.

### Luminescence readouts

```matlab
% expects a workspace variable named `Lumidata` (wide format: one column per sample)
Lumi_mean_bar_with_YUI
```

Produces a mean ± SD bar chart with individual data points overlaid (jittered), an
optional percentage normalisation with DMSO as 100%, and an interactive Y-axis control.

### Excel layout conversion

Both layout scripts expect a workspace variable named `list` (numeric, cell or table)
and write `special_tables.xlsx`:

```matlab
DataLayoutTransfer          % full blocks only; remainder is discarded
DataLayoutTransfer_extend   % full + partial blocks (NaN-padded) — recommended
```

Each sheet `Table_NN` receives 8 values in `C1:J1` and 50 values in `A2:J6`.

## Notes and limitations

- **Scripts, not a toolbox.** There is no `addpath`-and-call interface for most of these;
  they read variables by name from the base workspace. Renaming the input variable near
  the top of each script is intended (the comments point this out).
- `DataLayoutTransfer.m` **silently loses data** if the input length is not a multiple of
  58. Use `_extend` unless you specifically want that behaviour.
- Axis settings in the kinetics and luminescence GUIs distinguish **DATA range** (what is
  plotted) from **DISPLAY ticks** (what is labelled) — this is deliberate, so a curve can be
  shown at full extent while labelling only the region of interest.
- Chinese comments/UI text may appear in places. 

## License

MIT — see [LICENSE](LICENSE).
