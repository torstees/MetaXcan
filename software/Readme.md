
# MetaXcan

MetaXcan is a set of tools to perform integrative gene mapping studies.
Almost all of the software here is command-line based.

## S-PrediXcan

S-PrediXcan is an extension of [PrediXcan](https://github.com/hakyimlab/PrediXcan), that infers PrediXcan's results using only summary statistics. It is a component of MetaXcan.
A manuscript desribing S-PrediXcan and the MetaXcan framework and application can be found [here](http://www.biorxiv.org/content/early/2017/05/21/045260).

# Software Description

This repository contains software for implementing and/or using MetaXcan method.
Most of it is implemented at [Python](https://www.python.org/) scripts, 
although there are some [R](https://www.r-project.org/) scripts for processing and plotting results.

Most scripts were written as reusable components in a software library,
aimed at building different processing workflows.

Some of them, however, can be executed as command line tools to perform MetaXcan computations.

Check the root [Readme](https://github.com/hakyimlab/MetaXcan) for a sample script usage.

## Prerequisites

The software is developed and tested in Linux and Max OS environments. Should be mostly working on Windows.

To run S-PrediXcan, you need  [Python 2.7](https://www.python.org/), with the following libraries:
* [numpy (>=1.11.1)](http://www.numpy.org/)
* [scipy (>=0.18.1)](http://www.scipy.org/) 
* [pandas (>=0.18.1)](http://pandas.pydata.org/)
* [mock](https://github.com/testing-cabal/mock) and [sqlalchemy](https://www.sqlalchemy.org/) are needed for the unit tests.

To run PrediXcan and MulTiPrediXcan, you also need:
* [patsy (>=0.5.0)](https://patsy.readthedocs.io/en/latest/)
* [statsmodels (>=0.8.0)](https://www.statsmodels.org/stable/index.html)
* [h5py (>=2.7.1)](https://github.com/h5py/h5py)
* [h5py-cache (>=1.0.0)](https://pypi.python.org/pypi/h5py-cache/1.0)

[R](https://www.r-project.org/) with [ggplot](http://ggplot2.org/) and [dplyr](https://cran.r-project.org/web/packages/dplyr/index.html) 
is needed for some optional statistics and charts.

# Overview

S-PrediXcan is concerned with obtaining gene-level association tests from ordinary GWAS data.

Ordinarily, a user would need to obtain/download support data sets comprising of:
- A Transcriptome Prediction Model database (an example is [here](https://s3.amazonaws.com/imlab-open/Data/MetaXcan/sample_data/DGN-WB_0.5.db))
- A file with the covariance matrices of the SNPs within each gene model (such as [this one](https://s3.amazonaws.com/imlab-open/Data/MetaXcan/sample_data/covariance.DGN-WB_0.5.txt.gz))

And use them to run S-PrediXcan analysis on:
- GWAS results (such as [these](https://s3.amazonaws.com/imlab-open/Data/MetaXcan/sample_data/GWAS.tar.gz), from  a randomly generated phenotype)

However, if you have access to interesting data,
you can build your own Transcriptome Prediction Model database 
(using tools such as [PredictDB](http://predictdb.org)), 
and your own covariance (using tools from this repository).

More detailed information and usage documentation can be found at the [Wiki](https://github.com/hakyimlab/MetaXcan/wiki),
where more specific coverage is given to the tools contained here.

# Brief Tour

Scripts **M00_prerequisites.py**, **M01_covariances_correlations.py**, **M02_variances.py**, **M03_betas.py**
and **M04_zscores.py** are steps in a MetaXcan pipeline, although tipically a user will only need to
use the last two. For ease of use, there is a script that performs these last two steps at once, **MetaXcan.py**.
There is also a GUI app, **MetaXcanUI.py**, that offers a simplified operation for **MetaXcan.py**'s functionality.

All of these runable scripts take different number of command line parameters. Run them with
**--help** option to see the options, or check the [Wiki](https://github.com/hakyimlab/MetaXcan/wiki) for examples.

## M00_prerequisites.py

This script is considered deprecated and left here for future reference only.

This script takes extensive genotype data sets such as Thousand Genomes Haplotypes 
([here](https://mathgen.stats.ox.ac.uk/impute/1000GP%20Phase%203%20haplotypes%206%20October%202014.html))
allows for some filtering based on population or ID, and selects those entries
present in hapmap2 biallelic SNP's.

It supports input and output into a custom format that we call internally "PrediXcan Format". It is composed of
gzip compressed textfiles, without headers, where each row contains:

```bash
chromosome SNP_id SNP_position reference_allele effect_allele allele_frequency ...
# "..." stands for allele dosage data, one value per sample individual, value in [0, 2]
```

It is assumed that a "samples file" is provided, describing each sample individual's metadata, which is a plain text file
that looks like:

```bash
ID POP GROUP SEX
HG00096 GBR EUR male
HG00097 GBR EUR female
HG00099 GBR EUR female
HG00100 GBR EUR female
HG00101 GBR EUR male
HG00102 GBR EUR female
...
```
This script is hardly ever necessary. You might use it occasionally to reduce file sizes
used at the covariance script step (**M01_covariances_correlations.py**).

## M01_covariances_correlations.py

This script is considered deprecated and left here for future reference only. This script builds the covariance matrices needed at **M04_zscores.py**.
You will run it once in a while, if ever.
It takes input from **M00_prerequisites.py**'s output, and a genetic expression model database, such as
[this one](https://s3.amazonaws.com/imlab-open/Data/MetaXcan/sample_data/DGN-WB_0.5.db).

It will build the correlation matrix between SNP's in a same gene's model, for each gene, and save them
in a gzip-compressed text file.

## M03_betas.py

This is an optional script that takes GWAS results and converts them into a format that **M04_zscores.py**
can understand, and filters down snps by removing those excluded from the genetic expression.
If you use tools such as [Plink](https://www.cog-genomics.org/plink2), you will need to do this.

This supports both compressed and uncompressed text files as input. When running this script,
you need to specify the column names of expected data. We recommend files that include SNP's GWAS results for
(*beta* or *odd ratio* or *sign of beta*) and p-value. So that, for example, you provide **-or_column** and **--pvalue_column**
when running it.

## M04_zscores.py

This performs the actual S-PrediXcan calculation. It needs a genetic expression model database,
the covariance matrices from this model measured on a specific population, and GWAS data
as in the output of **M03_betas.py**.

If you have the model database and covariance matrix, you can just call this at the end
of your gwas pipeline (optionally using **M03_betas.py**).

## MetaXcan.py

Thin wrapper over **M03_betas.py** and **M04_zscores.py**; it is merely a convenience for running the most frequent S-PrediXcan's steps in a single command.

## MetaXcanUI.py

This script launches a desktop app that processes GWASinput and produces MetaXcan association results.
It is basically a friendlier way to use functionality at **M03_betas.py** and **M04_zscores.py**.
It is deprecated and doesn't support all of the options in the previous scripts (that as a matter of fact haven't been covered in this readme)
but might be friendlier to non technical users.

## MetaMany.py

MetaMany is a script that serially performs S-PrediXcan analysis on a GWAS data set using on multiple tissue models in a single command.

## PrediXcan.py

This is a minimalistic implementation of PrediXcan method. It takes gene expression values (from a plain-text file or HDF5) and computes association to a phenotype vector. 
It supports naive correction for arbitrary covariates (taken from a plain text file or HDF5). there are tools to generate these files [in the main PrediXcan repository](https://github.com/hakyim/PrediXcan).

## MulTiXcan.py

This is the main implementation of the Multi-Tissue association method. 
It takes a collection of files containing gene expression (each file assumed to be a single tissue/study) and computes joint association to a vector phenotype. [The main PrediXcan repository](https://github.com/hakyim/PrediXcan) contaisn tools to generate gene expression files as used by this script.

## SMulTiXcan.py

This is the summary-data based implementation of MulTiXcan. 
You need to run `MetaXcan.py` first for the collection of tissues you are interested in, and a reference LD file such as [this one for v6p models](https://s3.amazonaws.com/imlab-open/Data/MetaXcan/multixcan/snp_covariance.txt.gz)

## Useful Data

We make available several GTEx tissue models, covariances and 1000 Genomes covariances [here](http://predictdb.hakyimlab.org).
<!-- old box https://app.box.com/s/gujt4m6njqjfqqc9tu0oqgtjvtz9860w  -->
These files should be enough for running **MetaXcan.py**, **MulTiXcan.py** and **SMulTiXcan.py** on practically any GWAS study.

## The Rest

The other scripts in this repository fall into two categories:
* components to be used by runnable scripts (such as the ones just described) 
* Support data processing and analysis

# What do I do with this stuff?

It depends on your needs and means.

Ordinarily, most users would need to download some genetic expression databases,
some covariance matrices, and run MetaXcan on GWAS studies.
The **GUI**,  **M03_betas.py**, **M04_zscores.py** and **MetaXcan.py**
can be used for that.

A more sophisticated application could include using MetaXcan
with custom expression model databases, and different population's covariance matrices.
**M00_prerequisites.py** and **M01_covariances_correlations.py** can help with that.

A yet more sophisticated use would imply using and modifying the source code
to build custom processing pipelines.

If curious, check the [Wiki](https://github.com/hakyimlab/MetaXcan/wiki) for further information.

