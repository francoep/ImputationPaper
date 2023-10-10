# This git repository contains everything necessary to reproduce the results of the paper.

## Downloading CrossDocked2020v1.3 molcaches

This paper utilizes version 1.3 of CrossDocked2020. If you want to download all of the raw training/testing data you can follow instructions [here](https://github.com/gnina/models/tree/master/data/CrossDocked2020).
For reproducing the model training here you will need to download the molcache2 files and put them in the models directory as defined below.
Additionally, we utilized the dataset provided by [Tosstorff et al](https://link.springer.com/article/10.1007/s10822-022-00478-x).
We refer to this as the Roche dataset, and have provided molcache2 files for it as well.

```
wget http://bits.csb.pitt.edu/files/crossdock2020/crossdock2020_1.3_lig.molcache2
wget http://bits.csb.pitt.edu/files/crossdock2020/crossdock2020_1.3_rec.molcache2
wget http://bits.csb.pitt.edu/files/imputation_paper/roche_set_crystal_lig.molcache2
wget http://bits.csb.pitt.edu/files/imputation_paper/roche_set_crystal_rec.molcache2
wget http://bits.csb.pitt.edu/files/imputation_paper/roche_set_docked_lig.molcache2
wget http://bits.csb.pitt.edu/files/imputation_paper/roche_set_docked_rec.molcache2
mv *.molcache2 models
```

## Downloading the types files utilized for this paper

Each round of training utilizes a types file format. All of these types files utilize the following format:

```
<label> <pK> <RMSD to crystal> <Receptor> <Ligand> # <Autodock Vina score>
```

The label is 1 if the RMSD to the crystal pose is <=2, and 0 otherwise. 
The pK is initially calculated by taking the negative log (base 10) of the Kd/Ki/IC50 given for that ligand in the PDBbind.
Ligands with unknown affinity are labeled as 0.
Notably, this pK labeling is only true for the initial types file, for the imputed files all the 0 values are replaced with the imputed number.
We additionally label the pK column as negative if the label is 0.
The receptor and ligand columns correspond to the filenames of the raw data file (and are utilized with our molcache files).

The types files need to be downloaded and extracted:

```
wget http://bits.csb.pitt.edu/files/imputation_paper/typesfiles.tar.gz
tar -xzf typesfiles.tar.gz
```

This will create a types subdirectory containing each of the types files utilized by our training and evaluation scripts.
There are 288 types files, used across our experiments.
They are organized as follows:

```
it2_tt_v1.3_0_t*                               -- Initial types files for CrossDocked2020 utilized for the first round of training to get an initial model.
it2_cd_retrain_DEF2018_*                       -- Types files for experiment 1 in the paper. These are imputed files utilizing the results of the 5 models trained with the above types files.
it2_cd_imputation_DEF2018_*                    -- Series of 5 rounds of imputed types files from Experiment 1.
it2_cd_imputation_DEF2018_mean*                -- Series of 6 rounds of imputed types files from Experiment 2 (where we took the ensemble mean per pose).
it2_cd_imputation_1<max, median, min>_*        -- The types files used for Experiments 3,5,7. The imputation is selecting a singular value for all pocket-ligand poses
it2_cd_imputation_goodonly_1<max,median,min>_* -- Types files for Experiemnts 4,6,8. Same as above, but only using the pool of <2 rmsd poses for all pocket-ligand pairs.
it2_cd_imputaition_<20,40,60,80>pa_goodonly*   -- Types files for gradually adding more imputed values in increments of 20%.
roche_set_crystal.types                        -- Types files for the Roche dataset.
```

## Generating the Roche dataset

We provide a processed version of the Roche dataset, which includes the original data, our gninatypes files used for libmolgrid, and the processed receptor (where we removed the water present), and the gnina docked poses and corresponding gninatypes files.
You can download and extract all of these files:

```
wget http://bits.csb.pitt.edu/files/imputation_paper/roche_dataset.tar.gz
tar -xzf roche_dataset.tar.gz
```

## Running the training and evaluations

Each of the commands we utilized to train our models are available at results/training_commands.txt.
Notably, this is a compliation of all of the jobs we ran, but there are several sections that need to be run in stages.
The evaluation commands are available at results/evaluation_commands.txt.

The way to tell which commands to run is to run the training_commands lines that share an output directory, then run the corresponding evaluations that utilize the same output directory.
The training and evaluation require a working version of [gnina](https://github.com/gnina/gnina) and the [train.py](https://github.com/gnina/scripts) script and [evaluate_cross.py](https://github.com/gnina/scripts/tree/master/affinity_search) script available [here](https://github.com/gnina/scripts).

## Downloading our training set predictions

The prediction files resulting from the training and evaluations are massive.
We provide our files to compare with what you can generate.

```
mkdir paper_predictions
cd paper_predictions
wget http://bits.csb.pitt.edu/files/imputation_paper/trainingset_predictions.tar.gz
tar -xzf trainingset_predictions.tar.gz
```

## Downloading our test set predictions

We also provide all of our test set predictions.
These were utilized to calculate coefficient of determinations.
These test set predictions were utilized with the [results/calc_r2_scores.ipynb](https://github.com/francoep/ImputationPaper/blob/master/results/calc_r2_scores.ipynb) file to calculate the coefficients of determination for each model trained.
These coefficients of determination are the `*.r2.results` files in each directory of [results](https://github.com/francoep/ImputationPaper/blob/master/results/)

```
cd results
wget http://bits.csb.pitt.edu/files/imputation_paper/testset_predictions.tar.gz
tar -xzf testset_predictions.tar.gz
```

## Generating the figures utilized in the paper.

The provided generate_figures.ipynb is the notebook used to generate the information present for all of the tables and figures in the publication.
