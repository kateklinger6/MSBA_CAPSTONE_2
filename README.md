# MSBA_CAPSTONE_2

PBS Utah donor-prediction capstone (IS 6813, Fall 2026).

## Contents
- `notebooks/EDA_PBS_Utah_Donors.qmd`: exploratory data analysis (Quarto source, Python)
- `notebooks/EDA_PBS_Utah_Donors.html`: rendered, self-contained report with all outputs

## Data
The course data (synthetic) lives at https://github.com/jefftwebb/donor_prediction_capstone_project and is not committed here. Large tables use Git LFS:

```bash
git clone https://github.com/jefftwebb/donor_prediction_capstone_project.git
cd donor_prediction_capstone_project && git lfs pull
```

Copy or symlink its `tables/` folder to `data/tables/` in this repo, or edit `DATA_DIR` in the report.

## Rendering
Requires [Quarto](https://quarto.org) and Python with `pandas numpy matplotlib seaborn scipy jupyter` (about 10 GB of RAM):

```bash
cd notebooks
quarto render EDA_PBS_Utah_Donors.qmd
```
