## Dilution Effect Analysis Pipeline ver 1.1

## Authors
**Kristen Beck** and **Keith Bradnam**
[Korf Lab](http://korflab.ucdavis.edu)  
UC Davis [Genome Center](http://genomecenter.ucdavis.edu)
Davis, CA
USA

Email: <korflab@ucdavis.edu>

## Objective
This software is intended to compare RNA-Seq data from two different conditions (each with multiple replicates). The script first determines whether the two sets of data differ significantly in either their mean expression or their overall distributions.

If either test proves significant, the script will then generate a 'dilution adjustment factor' that will be applied to the raw count data of all genes below a certain threshold value.

This threshold value will be determined using a high quantile (defaulting to 0.9995). I.e. the expression of the top 0.05% of genes will be used to boost the expression of the remaining 99.95%. The exact level of adjustment is based on  the degree to which expression is dominated by the 0.05% of highly expressed genes.

The goal of this script is to produce increase the counts of low-abundance transcripts which may normally be regarded as being highly abundant but which are swamped by the extremely high numbers of a very small subset of genes. This adjustment will aid in the subsequent identification of differentially expressed genes.


## Software requirements

1. A working installation of Perl (for the main script)
2. An installed copy of [R](http://www.r-project.org) (for performing statistical tests by the secondary R script that is called by the main Perl script.)


## How to install software
Simply clone the repository to directory of choice from Unix/Linux terminal:

`$ git clone https://github.com/kbeck527/DilutionEffect.git`

Alternatively, download a ZIP file of the repository (see link on right of page). Ideally the directory containing these two scripts (`dilution.pl` and `test_difference_in_average_expression.R`) should be included in your Unix `$PATH` environment variable. Otherwise you must use this software with your data files in the same directory as these scripts.

## Input requirements 

The script expects *two* files containing tab-separated tables of RNA-Seq expression data where the first column contains a gene/transcript identifier, and subsequent columns contain count data. A header line is also expected. E.g.

For example:  
```
GeneID	Rep1	Rep2    Rep3    Rep4
GeneA	37		42		32      19
GeneB	529		544		400     349
GeneC	468		462		567     412
```

It is expected that each file — representing a different experimental condition — will contain multiple replicates (as with all analyses of RNA-Seq data, more replicates will yield stronger statistical interpretations).

The Gene/transcript IDs should be identical and in the same order for both of the two input files. Also these IDs should not be duplicated within a single expression data set.


## Usage

Assuming count data is in two files 'condition_A.tsv' and 'condition_B.tsv'
```
./dilution.pl condition_A.tsv condition_B.tsv
```

## Script output 

For each input file, the dilution.pl script will generate two output files, with names based on the input files. These will take the form:

1. condition_A_adjusted.tsv
2. condition_A_pseudocounted.tsv
3. condition_B_adjusted.tsv
4. condition_B_pseudocounted.tsv

The two adjusted files will contain counts that are increased by the dilution adjustment factor, and then pseudocounted (incremented by 1). The pseudocounted files simply increment the values in the input files by 1.

## Optional arguments

    
    --quantile <float> - specify a different quantile to use rather than the default 0.9995

    --normalize - before comparing statistical tests to compare means and overall distributions, the script will normalize count values by dividing by the total expression level of each replicate

    --verbose - turn on extra information as script runs (including details of dilution adjustment factors for each replicate)

    --help - display help


## Example input and output files

The [Examples](Examples/) subdirectory contains example input files (`test_condition_A.tsv` and `test_condition_B.tsv`) and the output files that would result from applying the dilution adjustment factor to these input files.


## More details ## 

The initial step of comparing the mean expression of both data sets is done by performing a Mann-Whitney-Wilcoxon text to compare the average expression of each gene in both dataset (averaged across all replicates). Then, the overall distribution of expression counts in both datasets is compared using a two sample Kolmogorov-Smirnov test.

If both datasets appear different as a result of one or both of these tests, then the Dilution Adjustment Model is applied. This uses a threshold to separate "high" and "low" abundance genes. Genes with expression values above the index are considered high abundance. Their expression values are summed and used in the calculation of the dilution adjustment factor (formula provided in publication). The expression values of low abundance genes are then multiplied by the adjustment factor to create the adjusted data set. This can be used for subsequent analysis i.e. a pairwise comparison to another physiological state or a comparison to the raw unadjusted expression data.

In some situations, it may be the case that the small number of high-abundance genes (by default this will be just 0.05% of your dataset), can account for more than two-thirds of all of the expression across all genes. In these situations it will be extrememly difficult to spot other upregulated genes.
  
Further details and an application of this model are provided here:  
Beck, K. et al. "Adjusting RNA-Seq data to account for the effect of highly abundant transcripts: a case study in milk production" BMC Bioinformatics. 2014 (submitted).

