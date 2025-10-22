# Inverse design of organic photocatalysts for energy transfer catalysis

This repository contains the code for the inverse design of organic photocatalysts for energy transfer catalysis as described in this paper: TBA [paper](TBA)

<p align="center">
  <img src="image.png" width="60%" />
</p>

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
An example config file with all run parameters can be found in `examples/config.toml`. A detailed description of all relevant parameters can be found in `examples/README.md`. The results will be saved to a csv file as specified in the config file under `summary_csv_prefix`.

## Data
The `data/` directory contains datasets for validating the triplet energy prediction, absorption wavelength prediction, and the ISC quantum yield. Moreover, the output of the generative models can be found in `data/generated_molecules/`.

## Citation
```
TBA
```
