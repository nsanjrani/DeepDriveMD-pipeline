# DeepDriveMD-F (DeepDriveMD-pipeline)

DeepDriveMD-F: Deep-Learning Driven Adaptive Molecular Simulations (file-based continual learning loop)

[![Documentation Status](https://readthedocs.org/projects/deepdrivemd-pipeline/badge/?version=latest)](https://deepdrivemd-pipeline.readthedocs.io/en/latest/?badge=latest)

Details can be found in the [ducumentation](https://deepdrivemd-pipeline.readthedocs.io/en/latest/). For more information, please see our [website](https://deepdrivemd.github.io/).

## How to run

### Summit HPC pre-existing environments on RHEL8

Copy pre-existing conda environments into new ones. Two different conda environments are needed, one for the MD stage, and the other for the ML and other stages. 

```
conda create --clone /gpfs/alpine/world-shared/chm155/hrlee/conda/openmm --name conda-ddmd
conda create --clone /gpfs/alpine/world-shared/chm155/hrlee/conda/pytorch-1.6.0_cudnn-8.0.2.39_nccl-2.7.8-1_static_mlperf --name conda-pytorch
```

If more changes are made to any packages (`DeepDriveMD` or `MD-Tools`), simply pull these changes using `git pull` and run `pip install -e .` in the conda environment, as described below. 

### Manual setup

Install `deepdrivemd` into a virtualenv with:

```
python3 -m venv env
source env/bin/activate
pip install --upgrade pip setuptools wheel
pip install -e .
```

Then, install pre-commit hooks: this will auto-format and auto-lint _on commit_ to enforce consistent code style:

```
pre-commit install
pre-commit autoupdate
```

In some places, DeepDriveMD relies on external libraries to configure MD simulations and import specific ML models.

For MD, install the `mdtools` package found here: https://github.com/braceal/MD-tools

For ML (specifically the AAE model), install the `molecules` package found here: https://github.com/braceal/molecules/tree/main

### Generating a YAML input spec:

First, run this command to get a _sample_ YAML config file:

```
python -m deepdrivemd.config
```

This will write a file named `deepdrivemd_template.yaml` which should be adapted for the experiment at hand. You should configure the `molecular_dynamics_stage`, `aggregation_stage`, `machine_learning_stage`, `model_selection_stage` and `agent_stage` sections to use the appropriate run commands and environment setups.

### Running an experiment

Then, launch an experiment with:

```
python -m deepdrivemd.deepdrivemd -c <experiment_config.yaml>
```

This experiment should be launched

### Note on input data

The input PDB and topology files should have the following structure:

```
ls data/sys*

data/sys1:
comp.pdb comp.top

data/sys2:
comp.pdb comp.top
```
Where the topology files are optional and only used when `molecular_dynamics_stage.task_config.solvent_type` is "explicit". Only one system directory is needed but an arbitrary number are supported. Also note that the system directory names are arbitrary. The path to the `data` directory should be passed into the config via `molecular_dynamics_stage.initial_pdb_dir`.


## Running DDMD for protein-ligand interactions
Clone `DeepDriveMD` from this fork then change to the GitHub branch `ligand_protein` and pull these changes, this should give access to the `heavy_atom_contacts` functionality.

A test system of the RQN ligand in 3CLPro is included under the `3clpro_unbound` folder, with the ligand bound pdb (`complex_rqn_bound.pdb`) and the ligand unbound (`system/complex_rqn_unbound.pdb`). This should be run on Summit.

Alternatively, another test system of the biotin ligand bound in streptavidin is included under the `protlig_exp` folder with a corresponding yaml file (`deepdrivemd_bridges.yaml`) to be run on the Bridges-2 cluster.

Additionally, a yaml file is included outside the folder (`deepdrivemd_rqn.yaml`) which can be run on Summit using the same commands as above:

```
python -m deepdrivemd.deepdrivemd -c deepdrivemd_rqn.yaml
```

Ensure you are in the `DeepDriveMD-pipeline` cloned git repository when launching this, and that the `conda-ddmd` environment is activated.

### Resource for creating AMBER input files

Biotin in streptavidin system:
https://ringo.ams.stonybrook.edu/index.php/2012_AMBER_Tutorial_with_Biotin_and_Streptavidin#Biotin_and_Streptavidin

Building systems in explicit solvent:
https://ambermd.org/tutorials/basic/tutorial7/index.php#Use_LEaP_to_Build_a_Protein_System_in_Explicit_Solvent