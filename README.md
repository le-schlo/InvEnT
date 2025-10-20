# Inverse design of organic photocatalysts for energy transfer catalysis

This repository contains the code for the inverse design of organic photocatalysts for energy transfer catalysis as described in this paper: TBA [paper](TBA)

## Installation
For installation run
```
git clone --recurse-submodules https://github.com/le-schlo/InvEnT.git

conda install -c conda-forge xtb==6.7.1 morfeus-ml==0.7.2
cd REINVENT4/
pip install -r requirements-linux-64.lock
pip install --no-deps .
```
The installation on a standard Linux machine takes approx. 2-3 minutes.

## Run generative model
To run the model execute 
```
reinvent -l logging.log config.toml
```
An example config file with all the run parameters can be found in `examples/config.toml`.

### Set parameters

After termination the results can be found as a csv file in the base diretory with a name as specified in the `config.toml` under the variable `summary_csv_prefix`.


