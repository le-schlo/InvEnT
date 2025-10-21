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
- _**Custom alerts**_ <br />
    The custom alerts component can be used to penalize molecules containing unwanted substructures.
  - `name`: Set to `Unwanted SMARTS`
  - `weight`: weight to fine-tune the relevance of this component
  - `params.smarts`: List of SMARTS strings defining unwanted substructures.


- _**Triplet energy prediction**_ <br />
    The [EnTdecker](https://github.com/le-schlo/EnTdecker) model is used to predict the triplet energy of generated molecules as described in this [paper](https://pubs.acs.org/doi/10.1021/jacs.4c01352).
  - `name`: Set to name for scoring component, _e.g._, `"EnTdecker"`
  - `weight`: set weight of the component in the overall score.
  - `params.checkpoint_dir`: Path to the directory containing the pretrained EnTdecker model. A downloaded model can be found in `Models/triplet_energy/model_42.pt` 
  - `params.rdkit_2d_normalized`: Set to `true` to use normalized 2D descriptors. Required for the EnTdecker model.
  - `params.target_column`: Set to `"e_t"`. Required for the EnTdecker model.
- _**ML-predicted absorption wavelength prediction**_ <br />
  The multi-fidelity model described by Greenman et al. [paper](https://doi.org/10.1039/D1SC05677H) is used to predict the maximum absorption wavelength of generated molecules.
  - `name`: Set to name for scoring component, _e.g._, `"ChemProp_uvvis"`
  - `weight`: set weight of the component in the overall score.
  - `params.checkpoint4featuregen`: Path to ensemble of ChemProp models used for predicting S1 excitation energy. The models used in this work can be found in `Models/uvvis/lambda_max_abs_wb97xd3/chemprop/all_wb97xd3/production/fold_0`
  - `params.checkpoint_dir`: Path to the ChemProp model checkpoint used for the low-fidelity prediction. The model used in this work can be found in `Models/uvvis/lambda_max_abs/chemprop_tddft/combined/production/fold_0`
  - `params.tmp_dir`: Path to a temporary directory for storing intermediate files.
  - `params.target_column`: Set to `"peakwavs_max"`.
  - `params.rdkit_2d_normalized`: Set to `false`.
- _**Semi-empirical absorption wavelength prediction**_ <br />
  The semi-empirical excited state calculation component uses xtb and stda to calculate the maximum absorption wavelength of generated molecules.
  - `name`: Set to name for scoring component, _e.g._, `"SQM_lambda_max"`
  - `weight`: set weight of the component in the overall score.
  - `params.tmp_dir`: Path to a temporary directory for storing intermediate files.
  - `params.path_to_xtb`: Path to the directory containing the xtb binary.
  - `params.path_to_stda`: Path to the directory containing the xtb4stda binary.
  - `params.maximum_waiting_time`: Maximum waiting time for the geometry optimization in seconds.
  - `params.use_stddft`: Set to `false` to use stda instead of stddft for excited state calculations.
  - `params.use_gfnff`: Set to `true` to use gfn-ff for geometry optimization instead of gfn2.
  - `params.target_property`: Set to `lambda_max`.
- _**Excited state character**_ <br />
    Computes the HOMO-LUMO overlap and estimates nature of excited state (charge-transfer or local excitation) using xtb and Multiwfn   
  - `name`: Set to name for scoring component, _e.g._, `"FMO analysis"`
  - `weight`: set weight of the component in the overall score.
  - `params.dir4tempfiles`: Path to a temporary directory for storing intermediate files.
  - `params.path_to_multiwfn`: Path to the Multiwfn binary.
  - `params.calculation_mode`: Choose between `multiwfn` and `multiwfn_quick`. In `multiwfn` both singlet and triplet states are optimized for computing the FMOs, while in `multiwfn_quick` only the singlet geometry is used to compute the FMOs.
  - `params.use_gfn2`: Set to `true` to optimize geometries with GFN2-xTB, if set to `false` optimization is performed with GFN-FF. 
  - `params.aggregation_mode`: Choose between `formula` and `threshold`.
    - `formula`:
       The `score` is the sum of the Singlet part ($S_{\text{part}}$) and the Triplet part ($T_{\text{part}}$):

      $$
      score = S_{\text{part}} + T_{\text{part}}
      $$

      Where the parts are defined as:

      $$
      S_{\text{part}} = w_{\text{singlet}} \cdot ((S_{\text{overlap_factor}} \cdot O_{S_1}) + (S_{\text{distance_factor}} \cdot D_{S_1}))
      $$
            
      $$
      T_{\text{part}} = w_{\text{triplet}} \cdot ((T_{\text{overlap_factor}} \cdot O_{T_1}) + (T_{\text{distance_factor}} \cdot D_{T_1}))
      $$
            
      | Variable                                   | Parameter in config file                       | Description                                                                 |
      |:-------------------------------------------|------------------------------------------------|:----------------------------------------------------------------------------|
      | $w_{\text{singlet}}$, $w_{\text{triplet}}$ | `params.Singlet_param`, `params.Triplet_param` | Weights for the overall Singlet and Triplet contributions.                  |
      | $S_{\text{overlap_factor}}$, $D_{S_1}$     | `params.S_overlap`, `params.S_distance`        | Weights for the singlet overlap value and distance of HOMO und LUMO center. |
      | $T_{\text{overlap_factor}}$, $D_{T_1}$     | `params.T_overlap`, `params.T_distance`        | Weights for the triplet overlap value and distance of HOMO und LUMO center. |

    - `threshold`:
      The `score` is a weighted sum of two binary components ($S_{\text{part}}$ and $T_{\text{part}}$), which are either **0** or **1**:
      $$
      \text{score} = w_{\text{singlet}} \cdot S_{\text{part}} + w_{\text{triplet}} \cdot T_{\text{part}}
      $$

      The binary parts are determined by the following conditions:

      | Component         | Condition                                                    | Character Represented    |
      |:------------------|:-------------------------------------------------------------|:-------------------------|
      | $S_{\text{part}}$ | **1** if $O_{S_1} < $`params.S_overlap` <br>**0** otherwise. | **CT** (Charge Transfer) |
      | $T_{\text{part}}$ | **1** if $O_{T_1} > $`params.T_overlap` <br>**0** otherwise. | **LE** (Locally excited) |

- _**Conjugation**_ <br />
    Computes conjugation in molecules 
  - `name`: Set to name for scoring component, _e.g._, `"Conjugation"`
  - `weight`: set weight of the component in the overall score.
  - `params.mode`: Choose between `fraction` and `largest_conjugated_fragment`. 
    - `fraction`: Score is computed as the fraction of conjugated atoms over total number of atoms.
    - `largest_conjugated_fragment`: Score is computed based on the size of the largest conjugated fragment.
  - `params.exclude_split_system`: Set to `true` to exclude molecules with disconnected conjugated structures.
