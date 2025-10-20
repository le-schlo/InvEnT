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
An example config file with all run parameters can be found in `examples/config.toml`.

### Structure in config.toml
The config file is structured in different sections. The parameters relevant for this work are described. Information about other parameters and options can be found in the repository of [REINVENT4](https://github.com/MolecularAI/REINVENT4/tree/main).
#### General settings
- `run_type`: Set to `staged_learning` to run the inverse design for molecular generation guided by reward function and `transfer_learning` for training an agent to generate molecules according to a set of input molecules (see section prior training).
- `device`: Set to `cpu` or `cuda:0` depending on the available hardware.

#### Parameters section
- `summary_csv_prefix` : Prefix for the summary csv file containing the results of the run.
- `prior_file`: Path to a pretrained prior model.
- `agent_file`: Path to the pretrained agent model. Generally, prior and agent refer to the same model, however, prior is a model that is trained to generate molecules according to some training distribution and agent is a fine-tuned model that is trained to generate molecules according to some property distribution defined by the reward function.
- `batch_size`: Number of molecules generated per batch.

#### Diversity filter section
This set of parameters is optional and can be used to increase the diversity of generated molecules.
- `type`: Choose between `IdenticalTopologicalScaffold`, `ScaffoldSimilarity`, `PenalizeSameSmiles`, and `IdenticalMurckoScaffold`. In this work, `ScaffoldSimilarity` was used exclusively.
- `bucket_size`: Number of molecules with the same scaffold/similarity that can be accepted before penalizing further molecules.
- `minscore`: Minimum score a molecule must have to be registered in the diversity filter.
- `minsimilarity`: Minimum similarity threshold for scaffold similarity.
- `penalty_multiplier`: penalty factor for PenalizeSameSmiles

#### Scoring settings
- `chkpt_file`: Name of the checkpoint file to store the agent model after the run.
- `max_score`: Termination criterion for the run.
- `min_steps`: Minimum number of steps to run.
- `max_steps`: Maximum number of steps to run.

- `type`: Choose the aggregation method for the individual components of the reward function. Available options are `custom_product`, `custom_sum`, `hypervolume`, and `prod_plus_hypervolume`. For the aggregation methods using the hypervolume its important that the individual scoring components are scaled between 0 and 1 and that all components have the same weight.

##### Scoring components
###### **Custom alerts**
The custom alerts component can be used to penalize molecules containing unwanted substructures.
- `name`: Set to `Unwanted SMARTS`
- `weight`: weight to fine-tune the relevance of this component
- `params.smarts`: List of SMARTS strings defining unwanted substructures.
###### Triplet energy prediction
The [EnTdecker](https://github.com/le-schlo/EnTdecker) model is used to predict the triplet energy of generated molecules as described in this [paper](https://pubs.acs.org/doi/10.1021/jacs.4c01352).
- `name`: Set to `EnTdecker`
- `weight`: set weight of the component in the overall score.
- `params.checkpoint_dir`: Path to the directory containing the pretrained EnTdecker model. A downloaded model can be found in `Models/triplet_energy/model_42.pt` 
- `params.rdkit_2d_normalized`: Set to `True` to use normalized 2D descriptors. Required for the EnTdecker model.
- `params.target_column`: Set to `e_t`. Required for the EnTdecker model.
###### ML-predicted absorption wavelength prediction




#ToDo: Describe transform type functions and params