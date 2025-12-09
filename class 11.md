# Class 11: Structural Bioinformatics Pt 2
Laura Lu (PID: A17844089)

## AlphaFold Data Base (AFDB)

The EBI maintains the largest database of AlphaFold structure prediction
models at: https://alphafold.ebi.ac.uk

From last class (before Halloween) we saw that PDB had 244,290 (Oct
2025)

The total number of protein sequences in UniProtKB is 199,579,901

> Key Point: This is a tiny fraction of sequence space that has
> structural coverage (0.12%)

``` r
244290/199579901 * 100 
```

    [1] 0.1224021

AFDB is attempting to address this gap…

There are two “Quality Scores” from AlphaFold one for residues (i.e each
amino acid) called **pLDDT** score. The other **PAE** score measures the
confidence in the relative position of two residues (i.e. a score for
every pair of residues).

## Generating your own structure predictions

Figure of 5 generated HIV-PR models

![](HIVPR_DIMER_23119_HIVPR_DIMER_23119_.png)

and the top ranked model colored by chain.

![](HIVPR_Dimer_topmodel.png)

pLDDT score for model 1

![](HIVPR_DIMER_2copy.png)

and model 5

![](HIVPR_pLDDT_m5.png)

# Custom analysis of resulting models in R

Read key result files into R. The first thing I need to know is what my
results directory/folder is called (i.e. it name is different for every
AlphaFold run/job)

``` r
results_dir <- "hivprdimer_23119:"
# File names for all PDB models
pdb_files <- list.files(path=results_dir,
                        pattern=".pdb",
                        full.names = TRUE)

# Print our PDB file names
basename(pdb_files)
```

    character(0)
