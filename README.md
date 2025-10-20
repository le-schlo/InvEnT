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

Optional dependencies:
- [stda](https://github.com/grimme-lab/std2/tree/v1.6.1) and [xtb4stda](https://github.com/grimme-lab/xtb4stda) <br />
  Tested with stda v1.6.1 and xtb4stda v1.0
  Statically linked binaries can be downloaded from [https://github.com/grimme-lab/xtb4stda/releases/tag/v1.0](https://github.com/grimme-lab/xtb4stda/releases/tag/v1.0)
  
- [Multiwfn](https://doi.org/10.1063/5.0216272) <br />
  Tested with Multiwfn v3.7 on Linux without GUI which can be downloaded from this [source](http://sobereva.com/multiwfn/misc/Multiwfn_3.7_bin_Linux_noGUI.zip)

## Run generative model
To run the model execute 
```
reinvent -l logging.log config.toml
```
An example config file with all the run parameters can be found in `examples/config.toml`.

### Parameters in config.toml


After termination the results can be found as a csv file in the base diretory with a name as specified in the `config.toml` under the variable `summary_csv_prefix`.


