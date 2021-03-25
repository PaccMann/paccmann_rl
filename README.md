[![Build Status](https://travis-ci.org/PaccMann/paccmann_rl.svg?branch=master)](https://travis-ci.org/PaccMann/paccmann_rl)
# paccmann_rl

Pipeline to reproduce the results of the [PaccMann<sup>RL</sup> paper](https://www.cell.com/iscience/fulltext/S2589-0042(21)00237-6) published in _iScience_.

## Description

In the repo we provide a conda environment and instructions to reproduce the pipeline described in the manuscript:

1. Train a multimodal drug sensitivity predictor ([source code](https://github.com/PaccMann/paccmann_predictor))
2. Train a generative model for omic profiles, also known as the PVAE ([source code](https://github.com/PaccMann/paccmann_omics))
3. Train a generative model for molecules, also known as the SVAE ([source code](https://github.com/PaccMann/paccmann_chemistry))
4. Train PaccMann^RL ([source code](https://github.com/PaccMann/paccmann_generator))

## Requirements

- `conda>=3.7`
- The following data from [here](https://ibm.ent.box.com/v/paccmann-pytoda-data):
  - The processed splitted data from the folder `splitted_data`
  - The processed gene expression data from [GDSC](https://www.cancerrxgene.org/): `data/gene_expression/gdsc-rnaseq_gene-expression.csv`
  - The processed SMILES from the drugs from [GDSC](https://www.cancerrxgene.org/): `data/smiles/gdsc.smi`
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
├── gdsc-rnaseq_gene-expression.csv
├── gdsc.smi
├── gdsc_transcriptomics_for_conditional_generation.pkl
├── smiles_language_chembl_gdsc_ccle.pkl
└── splitted_data
    ├── gdsc_cell_line_ic50_test_fraction_0.1_id_997_seed_42.csv
    ├── gdsc_cell_line_ic50_train_fraction_0.9_id_997_seed_42.csv
    ├── tcga_rnaseq_test_fraction_0.1_id_242870585127480531622270373503581547167_seed_42.csv
    ├── tcga_rnaseq_train_fraction_0.9_id_242870585127480531622270373503581547167_seed_42.csv
    ├── test_chembl_22_clean_1576904_sorted_std_final.smi
    └── train_chembl_22_clean_1576904_sorted_std_final.smi

1 directory, 11 files
```

**NOTE:** no worries, the `data` folder is in the [.gitignore](./.gitignore).

### Clone the repos

To get the scripts to run each of the component create a `code` folder and clone the repos. Simply type this:

```sh
mkdir code && cd code && \
  git clone --branch 0.0.1 https://github.com/PaccMann/paccmann_predictor && \ 
  git clone --branch 0.0.1 https://github.com/PaccMann/paccmann_omics && \ 
  git clone --branch 0.0.1 https://github.com/PaccMann/paccmann_chemistry && \ 
  git clone --branch 0.0.1 https://github.com/PaccMann/paccmann_generator && \
  cd ..
```

**NOTE:** no worries, the `code` folder is in the [.gitignore](./.gitignore).

## Pipeline

Now it's all set to run the full pipeline.

**NOTE:** the workload required to run the full pipeline is intesive and might not be straightforward to run all the steps on a desktop laptop. For this reason, we also provide [pretrained models](https://ibm.ent.box.com/v/paccmann-pytoda-data/folder/91897885403) that can be downloaded and used to run the different steps.

**NOTE:** in the following, we assume a folder `models` has been created in the root of the repository. No worries, the `models` folder is in the [.gitignore](./.gitignore).

### Multimodal drug sensitivity predictor

```console
(paccmann_rl) $ python ./code/paccmann_predictor/examples/train_paccmann.py \
    ./data/splitted_data/gdsc_cell_line_ic50_train_fraction_0.9_id_997_seed_42.csv \
    ./data/splitted_data/gdsc_cell_line_ic50_test_fraction_0.1_id_997_seed_42.csv \
    ./data/gdsc-rnaseq_gene-expression.csv \
    ./data/gdsc.smi \
    ./data/2128_genes.pkl \
    ./data/smiles_language_chembl_gdsc_ccle.pkl \
    ./models/ \
    ./code/paccmann_predictor/examples/example_params.json paccmann
```

### PVAE

``` console
(paccmann_rl) $ python ./code/paccmann_omics/examples/train_vae.py \
    ./data/splitted_data/tcga_rnaseq_train_fraction_0.9_id_242870585127480531622270373503581547167_seed_42.csv \
    ./data/splitted_data/tcga_rnaseq_test_fraction_0.1_id_242870585127480531622270373503581547167_seed_42.csv \
    ./data/2128_genes.pkl \
    ./models/ \
    ./code/paccmann_omics/examples/example_params.json pvae
```

### SVAE

``` console
(paccmann_rl) $ python ./code/paccmann_chemistry/examples/train_vae.py \
    ./data/splitted_data/train_chembl_22_clean_1576904_sorted_std_final.smi \
    ./data/splitted_data/test_chembl_22_clean_1576904_sorted_std_final.smi \
    ./data/smiles_language_chembl_gdsc_ccle.pkl \
    ./models/ \
    ./code/paccmann_chemistry/examples/example_params.json svae
```

### PaccMann^RL

``` console
(paccmann_rl) $ python ./code/paccmann_generator/examples/train_paccmann_rl.py \
    ./models/svae \
    ./models/pvae \
    ./models/paccmann \
    ./data/smiles_language_chembl_gdsc_ccle.pkl \
    ./data/gdsc_transcriptomics_for_conditional_generation.pkl \
    ./code/paccmann_generator/examples/example_params.json \
    paccmann_rl breast
```

**NOTE:** this will create a `biased_model` folder containing the conditional generator and the baseline SMILES generator used. In this case: `breast_paccmann_rl` and `baseline`. No worries, the `biased_models` folder is in the [.gitignore](./.gitignore).

## References

If you use `paccmann_rl` in your projects, please cite the following:

```bib
@article{born2021paccmannrl,
    title = {PaccMann^{RL}: De novo generation of hit-like anticancer molecules from transcriptomic data via reinforcement learning},
    journal = {iScience},
    volume = {24},
    number = {4},
    pages = {102269},
    year = {2021},
    issn = {2589-0042},
    doi = {https://doi.org/10.1016/j.isci.2021.102269},
    url = {https://www.cell.com/iscience/fulltext/S2589-0042(21)00237-6},
    author = {Jannis Born and Matteo Manica and Ali Oskooei and Joris Cadow and Greta Markert and María {Rodríguez Martínez}},
    keywords = {Complex System Biology, Systems Biology, Transcriptomics, Computer Science}
}
```
