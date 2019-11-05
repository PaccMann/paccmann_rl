[![Build Status](https://travis-ci.org/PaccMann/paccmann_rl.svg?branch=master)](https://travis-ci.org/PaccMann/paccmann_rl)
# paccmann_rl

Pipeline to reproduce the results of the [PaccMann^RL paper](https://arxiv.org/abs/1909.05114).

## Description

In the repo we provide a conda environment and instructions to reproduce the pipeline descirbed in the manuscript:

1. Train a multimodal drug sensitivity predictor ([source code](https://github.com/PaccMann/paccmann_predictor))
2. Train a generative model for omic profiles, also known as the PVAE ([source code](https://github.com/PaccMann/paccmann_omics))
3. Train a generative model for molecules, also known as the SVAE ([source code](https://github.com/PaccMann/paccmann_chemistry))
4. Train PaccMann^RL ([source code](https://github.com/PaccMann/paccmann_generator))

## Requirements

- `conda>=3.7`
- The following data from [here](https://ibm.ent.box.com/v/paccmann-pytoda-data):
  - The processed splitted data from the folder `splitted_data`
  - A pickled [SMILESLanguage](https://github.com/PaccMann/paccmann_datasets/blob/master/pytoda/smiles/smiles_language.py) object (`data/smiles_language_chembl_gdsc_ccle.pkl`)
  - A pickled list of genes representing the panel considered in the paper (`data/2128_genes.pkl`)
  - A pickled pandas DataFrame containing expression values and metadata for the cell lines considered in the paper (`data/gdsc_transcriptomics_for_conditional_generation.pkl`)
- The git repos linked in the [previous section](#description)

**NOTE:** please refer to the [README.md](https://ibm.ent.box.com/v/paccmann-pytoda-data/file/548614344106) and to the manuscript for details on the datasets used and the preprocessing applied.

## Setup

### Install the environment

Create a conda environment:

```sh
conda env create -f conda.yml
```

Activate the environment:

```sh
conda activate paccmann_rl
```

### Download data

Download the data reported in the [requirements section](#requirements).
From now on, we will assume that they are stored in the root of the repository in a folder called `data`, following this structure:

```console
data
├── 2128_genes.pkl
├── gdsc_transcriptomics_for_conditional_generation.pkl
├── smiles_language_chembl_gdsc_ccle.pkl
└── gene_expression
    ├── gdsc-rnaseq_gene-expression.csv
└── smiles
    ├── gdsc.smi
└── splitted_data
    ├── gdsc_cell_line_ic50_test_fraction_0.1_id_997_seed_42.csv
    ├── gdsc_cell_line_ic50_train_fraction_0.9_id_997_seed_42.csv
    ├── tcga_rnaseq_test_fraction_0.1_id_242870585127480531622270373503581547167_seed_42.csv
    ├── tcga_rnaseq_train_fraction_0.9_id_242870585127480531622270373503581547167_seed_42.csv
    ├── test_chembl_22_clean_1576904_sorted_std_final.smi
    └── train_chembl_22_clean_1576904_sorted_std_final.smi

1 directory, 9 files
```

**NOTE:** no worries, the `data` folder is in the [.gitignore](./.gitignore).

### Clone the repos

To get the scripts to run each of the component create a `code` folder and clone the repos. Simply type this:

```sh
mkdir code && cd code && \
  git clone https://github.com/PaccMann/paccmann_predictor && \ 
  git clone https://github.com/PaccMann/paccmann_omics && \ 
  git clone https://github.com/PaccMann/paccmann_chemistry && \ 
  git clone https://github.com/PaccMann/paccmann_generator && \
  cd ..
```

**NOTE:** no worries, the `code` folder is in the [.gitignore](./.gitignore).

## Pipeline

Now it's all set to run the full pipeline.

**NOTE:** the workload required to run the full pipeline is intesive and might not be straightforward to run all the steps on a desktop laptop. For this reason, we also provide [pretrained models](https://ibm.ent.box.com/v/paccmann-pytoda-data/folder/91897885403) that can be downloaded and used to run the different steps.

### Multimodal drug sensitivity predictor

python3 code/paccmann_predictor/examples/train_paccmann.py \
data/splitted_data/gdsc_cell_line_ic50_train_fraction_0.9_id_997_seed_42.csv \
data/splitted_data/gdsc_cell_line_ic50_test_fraction_0.1_id_997_seed_42.csv \
data/gene_expression/gdsc-rnaseq_gene-expression.csv \
data/smiles/gdsc.smi \
data/2128_genes.pkl \
data/smiles_language_chembl_gdsc_ccle.pkl \
code/paccmann_predictor/models/ \
code/paccmann_predictor/examples/example_params.json \ 
paccmann_model

### PVAE

python3 code/paccmann_omics/examples/train_vae.py \
data/splitted_data/tcga_rnaseq_train_fraction_0.9_id_242870585127480531622270373503581547167_seed_42.csv \
data/splitted_data/tcga_rnaseq_test_fraction_0.1_id_242870585127480531622270373503581547167_seed_42.csv \
data/2128_genes.pkl \
code/paccmann_omics/models/ \
code/paccmann_omics/examples/example_params.json \ 
pvae


### SVAE

python3 code/paccmann_chemistry/examples/train_vae.py \
data/splitted_data/train_chembl_22_clean_1576904_sorted_std_final.smi \
data/splitted_data/test_chembl_22_clean_1576904_sorted_std_final.smi \
data/smiles_language_chembl_gdsc_ccle.pkl \
code/paccmann_chemistry/models/ \
code/paccmann_chemistry/examples/example_params.json \ 
svae

### PaccMann^RL

python code/paccmann_generator/examples/train_paccmann_rl.py \
code/paccmann_chemistry/models/svae_pretrained \
code/paccmann_omics/models/pvae_pretrained \
code/paccmann_predictor/models/paccmann_pretrained \
data/smiles_language_chembl_gdsc_ccle.pkl \
data/gdsc_transcriptomics_for_conditional_generation.pkl \
code/paccmann_generator/examples/example_params.json \
paccmann_rl \
breast


## References

If you use `paccmann_rl` in your projects, please cite the following:

```bib
@misc{born2019reinforcement,
    title={Reinforcement learning-driven de-novo design of anticancer compounds conditioned on biomolecular profiles},
    author={Jannis Born and Matteo Manica and Ali Oskooei and Maria Rodriguez Martinez},
    year={2019},
    eprint={1909.05114},
    archivePrefix={arXiv},
    primaryClass={q-bio.BM}
}
```
