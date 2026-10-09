# MSBA_CAPSTONE_2

PBS Utah donor-prediction capstone (IS 6813, Fall 2026).

## Contents
- `notebooks/EDA_PBS_Utah_Donors.ipynb`: exploratory data analysis notebook
- `notebooks/EDA_PBS_Utah_Donors.html`: rendered copy of the notebook with all outputs

## Data
The course data (synthetic) lives at https://github.com/jefftwebb/donor_prediction_capstone_project and is not committed here. Large tables use Git LFS:

```bash
git clone https://github.com/jefftwebb/donor_prediction_capstone_project.git
cd donor_prediction_capstone_project && git lfs pull
```

Copy or symlink its `tables/` folder to `data/tables/` in this repo, or edit `DATA_DIR` in the notebook. The notebook needs `pandas numpy matplotlib seaborn scipy` and about 8 GB of RAM.
