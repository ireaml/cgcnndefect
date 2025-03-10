# Personal modifications to CGCNN-defect

Small modifications to [cgcnndefect](https://github.com/ireaml/cgcnndefect), developed by [Matthew Witman](https://scholar.google.com/citations?hl=en&user=bb7EKzIAAAAJ&view_op=list_works&sortby=pubdate), which iself is based on the
[CGCNN model](https://github.com/txie-93/cgcnn) developed by [Tian Xie](http://txie.me) and [Prof. Jeffrey Grossman](https://dmse.mit.edu/faculty/profile/grossman).

## Installable package
To install as a package use
```bash
pip install -e .
```
which allows you to then execute training or prediction tasks from anywhere using the command line:
```bash
cgcnn-defect-train $flags
cgcnn-defect-predict $flags
```
For instance, the following
```bash
cgcnn-defect-train root_dir --radius 5.0
```
specifies the folder `root_dir` where the input files are located, as well as the cutoff radius (in A) used to look for the neighbours of each atom.

## Command line arguments
Several command line arguments are available to control target files and facilitate high-throughput execution across different encoding strategies, cross-validation, etc. These include:
- Specify the cutoff radius used for searching neighbours
  ```bash
  --radius 5.0
  ```
- Specify global and/or local features (which should be included as columns in the input dataframe, and their
  column labels should contain the specified strings (e.g. if --crys-spec Global then valid column labels would be Global_BandGap, Global_Whatever...).
  ```bash
  --crys-spec Global
  ```
  and/or
  ```bash
  --atom-spec Local
  ```
- Change element encoding file
  ```bash
  --init-embed-file $your_atom_init.json
  ```
- Change the default df.pkl file of (structures, property, global features) data to df.pkl.your_pkl_ext:
  ```bash
  --pkl-ext .your_pkl_ext
  ```

## Defect modifications
- Pooling function has been hard coded to only extract the feature vector of the node at index i=0 (the atom to be defected). In progress: this will be made a lot more efficient in the future by specifying the index from the CL and not needing multiple CIF files for all unique defects within a given host structure
- For a given structure1.cif, can introduce local node attributes (e.g. oxidation state) contained in structure1.cif.locals at the graph encoding stage via:
```bash
--atom-spec locals
``` 
- For a given structure1.cif, can introduce global features (e.g. compound formation enthalpy) contained in structure1.cif.globals at the graph encoding stage via:
```bash
--crys-spec globals
``` 
- EXPERIMENTAL: can use a local convolution block based on spherical harmonics (at the cost of higher model complexity)

## How to cite

Please cite the following work if you want to use CGCNN and defect modifications.

```
@article{PhysRevLett.120.145301,
  title = {Crystal Graph Convolutional Neural Networks for an Accurate and Interpretable Prediction of Material Properties},
  author = {Xie, Tian and Grossman, Jeffrey C.},
  journal = {Phys. Rev. Lett.},
  volume = {120},
  issue = {14},
  pages = {145301},
  numpages = {6},
  year = {2018},
  month = {Apr},
  publisher = {American Physical Society},
  doi = {10.1103/PhysRevLett.120.145301},
  url = {https://link.aps.org/doi/10.1103/PhysRevLett.120.145301}
}
@article{Witman2022,
  author = {Witman, Matthew D. and Goyal, Anuj and Ogitsu, Tadashi and McDaniel, Anthony H. and Lany, Stephan},
  doi = {10.26434/chemrxiv-2022-frcns},
  journal = {ChemRxiv},
  pages = {10.26434/chemrxiv-2022-frcns},
  title = {{Graph neural network modeling of vacancy formation enthalpy for materials discovery and its application in solar thermochemical water splitting}},
  year = {2022},
  url = {https://chemrxiv.org/engage/chemrxiv/article-details/628bdf9f87d01f60fcefa355}
}
```

# README of Crystal Graph Convolutional Neural Networks

This software package implements the Crystal Graph Convolutional Neural Networks (CGCNN) that takes an arbitary crystal structure to predict material properties. 

The package provides two major functions:

- Train a CGCNN model with a customized dataset.
- Predict material properties of new crystals with a pre-trained CGCNN model.

The following paper describes the details of the CGCNN framework:

[Crystal Graph Convolutional Neural Networks for an Accurate and Interpretable Prediction of Material Properties](https://link.aps.org/doi/10.1103/PhysRevLett.120.145301)

## Table of Contents

- [How to cite](#how-to-cite)
- [Prerequisites](#prerequisites)
- [Usage](#usage)
  - [Define a customized dataset](#define-a-customized-dataset)
  - [Train a CGCNN model](#train-a-cgcnn-model)
  - [Predict material properties with a pre-trained CGCNN model](#predict-material-properties-with-a-pre-trained-cgcnn-model)
- [Data](#data)
- [Authors](#authors)
- [License](#license)

## How to cite

Please cite the following work if you want to use CGCNN.

```
@article{PhysRevLett.120.145301,
  title = {Crystal Graph Convolutional Neural Networks for an Accurate and Interpretable Prediction of Material Properties},
  author = {Xie, Tian and Grossman, Jeffrey C.},
  journal = {Phys. Rev. Lett.},
  volume = {120},
  issue = {14},
  pages = {145301},
  numpages = {6},
  year = {2018},
  month = {Apr},
  publisher = {American Physical Society},
  doi = {10.1103/PhysRevLett.120.145301},
  url = {https://link.aps.org/doi/10.1103/PhysRevLett.120.145301}
}
```

##  Prerequisites

This package requires:

- [PyTorch](http://pytorch.org)
- [scikit-learn](http://scikit-learn.org/stable/)
- [pymatgen](http://pymatgen.org)

If you are new to Python, the easiest way of installing the prerequisites is via [conda](https://conda.io/docs/index.html). After installing [conda](http://conda.pydata.org/), run the following command to create a new [environment](https://conda.io/docs/user-guide/tasks/manage-environments.html) named `cgcnn` and install all prerequisites:

```bash
conda upgrade conda
conda create -n cgcnn python=3 scikit-learn pytorch torchvision pymatgen -c pytorch -c conda-forge
```

*Note: this code is tested for PyTorch v1.0.0+ and is not compatible with versions below v0.4.0 due to some breaking changes.

This creates a conda environment for running CGCNN. Before using CGCNN, activate the environment by:

```bash
source activate cgcnn
```

Then, in directory `cgcnn`, you can test if all the prerequisites are installed properly by running:

```bash
python main.py -h
python predict.py -h
```

This should display the help messages for `main.py` and `predict.py`. If you find no error messages, it means that the prerequisites are installed properly.

After you finished using CGCNN, exit the environment by:

```bash
source deactivate
```

## Usage

### Define a customized dataset 

To input crystal structures to CGCNN, you will need to define a customized dataset. Note that this is required for both training and predicting. 

Before defining a customized dataset, you will need:

- [CIF](https://en.wikipedia.org/wiki/Crystallographic_Information_File) files recording the structure of the crystals that you are interested in
- The target properties for each crystal (not needed for predicting, but you need to put some random numbers in `id_prop.csv`)

You can create a customized dataset by creating a directory `root_dir` with the following files: 

1. `id_prop.csv`: a [CSV](https://en.wikipedia.org/wiki/Comma-separated_values) file with two columns. The first column recodes a unique `ID` for each crystal, and the second column recodes the value of target property. If you want to predict material properties with `predict.py`, you can put any number in the second column. (The second column is still needed.)

2. `atom_init.json`: a [JSON](https://en.wikipedia.org/wiki/JSON) file that stores the initialization vector for each element. An example of `atom_init.json` is `data/sample-regression/atom_init.json`, which should be good for most applications.

3. `ID.cif`: a [CIF](https://en.wikipedia.org/wiki/Crystallographic_Information_File) file that recodes the crystal structure, where `ID` is the unique `ID` for the crystal.

The structure of the `root_dir` should be:

```
root_dir
├── id_prop.csv
├── atom_init.json
├── id0.cif
├── id1.cif
├── ...
```

There are two examples of customized datasets in the repository: `data/sample-regression` for regression and `data/sample-classification` for classification. 

**For advanced PyTorch users**

The above method of creating a customized dataset uses the `CIFData` class in `cgcnn.data`. If you want a more flexible way to input crystal structures, PyTorch has a great [Tutorial](http://pytorch.org/tutorials/beginner/data_loading_tutorial.html#sphx-glr-beginner-data-loading-tutorial-py) for writing your own dataset class.

### Train a CGCNN model

Before training a new CGCNN model, you will need to:

- [Define a customized dataset](#define-a-customized-dataset) at `root_dir` to store the structure-property relations of interest.

Then, in directory `cgcnn`, you can train a CGCNN model for your customized dataset by:

```bash
python main.py root_dir
```

You can set the number of training, validation, and test data with labels `--train-size`, `--val-size`, and `--test-size`. Alternatively, you may use the flags `--train-ratio`, `--val-ratio`, `--test-ratio` instead. Note that the ratio flags cannot be used with the size flags simultaneously. For instance, `data/sample-regression` has 10 data points in total. You can train a model by:

```bash
python main.py --train-size 6 --val-size 2 --test-size 2 data/sample-regression
```
or alternatively
```bash
python main.py --train-ratio 0.6 --val-ratio 0.2 --test-ratio 0.2 data/sample-regression
```

You can also train a classification model with label `--task classification`. For instance, you can use `data/sample-classification` by:

```bash
python main.py --task classification --train-size 5 --val-size 2 --test-size 3 data/sample-classification
```

After training, you will get three files in `cgcnn` directory.

- `model_best.pth.tar`: stores the CGCNN model with the best validation accuracy.
- `checkpoint.pth.tar`: stores the CGCNN model at the last epoch.
- `test_results.csv`: stores the `ID`, target value, and predicted value for each crystal in test set.

### Predict material properties with a pre-trained CGCNN model

Before predicting the material properties, you will need to:

- [Define a customized dataset](#define-a-customized-dataset) at `root_dir` for all the crystal structures that you want to predict.
- Obtain a [pre-trained CGCNN model](pre-trained) named `pre-trained.pth.tar`.

Then, in directory `cgcnn`, you can predict the properties of the crystals in `root_dir`:

```bash
python predict.py pre-trained.pth.tar root_dir
```

For instace, you can predict the formation energies of the crystals in `data/sample-regression`:

```bash
python predict.py pre-trained/formation-energy-per-atom.pth.tar data/sample-regression
```

And you can also predict if the crystals in `data/sample-classification` are metal (1) or semiconductors (0):

```bash
python predict.py pre-trained/semi-metal-classification.pth.tar data/sample-classification
```

Note that for classification, the predicted values in `test_results.csv` is a probability between 0 and 1 that the crystal can be classified as 1 (metal in the above example).

After predicting, you will get one file in `cgcnn` directory:

- `test_results.csv`: stores the `ID`, target value, and predicted value for each crystal in test set. Here the target value is just any number that you set while defining the dataset in `id_prop.csv`, which is not important.

## Data

To reproduce our paper, you can download the corresponding datasets following the [instruction](data/material-data).

## Authors

This software was primarily written by [Tian Xie](http://txie.me) who was advised by [Prof. Jeffrey Grossman](https://dmse.mit.edu/faculty/profile/grossman). 

## License

CGCNN is released under the MIT License.



