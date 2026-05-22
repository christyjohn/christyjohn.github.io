---
title: "Data Science Environment setup"
date: 2026-04-09
draft: false
comments: true
categories: ["AI / ML / Data Science"]
tags: ["environemt", "setup"]
---

This is how you will be setting up an environment (Windows machine for now), so that we can work with Python, Jupyter Notebooks, NumPy, pandas, matplotlib, scipy, scikit-learn, statsmodels etc.

We need a Python environment to work with, and the general idea is to use a Conda environment, either through Anaconda or Miniconda. We will go with the Miniconda as it is a lean environment compared to the exhaustive, all-in-one environment of Anaconda, comprising of all the A.I, ML, and Data Science libraries involved. Miniconda helps us with a small footprint, on which we can add on as we need libraries.

1.  [Install Miniconda on Windows using the GUI installer](https://www.anaconda.com/docs/getting-started/miniconda/install/windows-gui-install)
2.  Create a working directory at your desired location.

```bash
mkdir workspace/sample\_project cd workspace/sample\_project
```

3.  Create a custom environment in the project folder.

```bash
conda create --prefix ./env pandas numpy matplotlib scikit-learn
```

(Nb: We will install Jupyter Notebook separately to learn how to install conda libraries.)

4.  Activate the environment.

```bash
$ conda activate C:\\workspace\\sample\_project\\env
```

To deactivate, use

```bash
$ conda deactivate
```

5.  Install and use Jupyter Notebook (after activating the environment from the project folder)

```bash
conda install jupyter jupyter notebook
```

6.  \[Exporting the environment\] (https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#sharing-an-environment)

```bash
conda env export --prefix C:\\workspace\\sample\_project\\env > environment.yml
```

7.  \[Create an environment from the file\] (https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#creating-an-environment-from-an-environment-yml-file)

```bash
conda env create --file environment.yml --name env\_from\_file
```
