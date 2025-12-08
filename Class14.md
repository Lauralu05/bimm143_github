# Class 14: RNASeq Mini Project
Laura Lu (PID: A17844089)

- [Background](#background)
- [Data Import](#data-import)
  - [Remove zero count genes](#remove-zero-count-genes)
- [DESeq analysis](#deseq-analysis)
- [Data Visualization](#data-visualization)
- [Add Annotation](#add-annotation)
- [Pathway Analysis](#pathway-analysis)
  - [GO terms](#go-terms)
  - [Reactome](#reactome)
- [Save Our Results](#save-our-results)

## Background

Here we work through a complete RNASeq analysis project. The input data
ceoms from a knock-down experiment of a HOX gene

## Data Import

Reading the `counts` and `metadata` CSV files

``` r
counts <- read.csv("GSE37704_featurecounts.csv", row.names = 1)
metadata <- read.csv("GSE37704_metadata.csv") 
```

Check on data structure

``` r
head(counts)
```

                    length SRR493366 SRR493367 SRR493368 SRR493369 SRR493370
    ENSG00000186092    918         0         0         0         0         0
    ENSG00000279928    718         0         0         0         0         0
    ENSG00000279457   1982        23        28        29        29        28
    ENSG00000278566    939         0         0         0         0         0
    ENSG00000273547    939         0         0         0         0         0
    ENSG00000187634   3214       124       123       205       207       212
                    SRR493371
    ENSG00000186092         0
    ENSG00000279928         0
    ENSG00000279457        46
    ENSG00000278566         0
    ENSG00000273547         0
    ENSG00000187634       258

``` r
metadata
```

             id     condition
    1 SRR493366 control_sirna
    2 SRR493367 control_sirna
    3 SRR493368 control_sirna
    4 SRR493369      hoxa1_kd
    5 SRR493370      hoxa1_kd
    6 SRR493371      hoxa1_kd

Some book-keeping is required as there looks to be a mis-match between
metadata and counts columns

``` r
ncol(counts)
```

    [1] 7

``` r
nrow(metadata) 
```

    [1] 6

Looks like we need to get rid of the first “length” column of our
`counts` object.

``` r
cleancounts <- counts[ , -1]
```

``` r
colnames(cleancounts) 
```

    [1] "SRR493366" "SRR493367" "SRR493368" "SRR493369" "SRR493370" "SRR493371"

``` r
metadata$id
```

    [1] "SRR493366" "SRR493367" "SRR493368" "SRR493369" "SRR493370" "SRR493371"

``` r
all( colnames(cleancounts) == metadata$id ) 
```

    [1] TRUE

### Remove zero count genes

There are lots of genes with zero counts. We can remove these.

``` r
head(cleancounts)
```

                    SRR493366 SRR493367 SRR493368 SRR493369 SRR493370 SRR493371
    ENSG00000186092         0         0         0         0         0         0
    ENSG00000279928         0         0         0         0         0         0
    ENSG00000279457        23        28        29        29        28        46
    ENSG00000278566         0         0         0         0         0         0
    ENSG00000273547         0         0         0         0         0         0
    ENSG00000187634       124       123       205       207       212       258

``` r
to.keep.inds <- rowSums(cleancounts) > 0 
nonzero_counts <- cleancounts[to.keep.inds,]
```

## DESeq analysis

Load the package

``` r
library(DESeq2) 
```

    Warning: package 'DESeq2' was built under R version 4.5.2

Setup DESeq object

``` r
dds <- DESeqDataSetFromMatrix(countData = nonzero_counts, 
                              colData = metadata, 
                              design = ~condition) 
```

    Warning in DESeqDataSet(se, design = design, ignoreRank): some variables in
    design formula are characters, converting to factors

run DESeq

``` r
dds <- DESeq(dds)
```

    estimating size factors

    estimating dispersions

    gene-wise dispersion estimates

    mean-dispersion relationship

    final dispersion estimates

    fitting model and testing

get results

``` r
res <- results(dds) 
head(res) 
```

    log2 fold change (MLE): condition hoxa1 kd vs control sirna 
    Wald test p-value: condition hoxa1 kd vs control sirna 
    DataFrame with 6 rows and 6 columns
                     baseMean log2FoldChange     lfcSE       stat      pvalue
                    <numeric>      <numeric> <numeric>  <numeric>   <numeric>
    ENSG00000279457   29.9136      0.1792571 0.3248216   0.551863 5.81042e-01
    ENSG00000187634  183.2296      0.4264571 0.1402658   3.040350 2.36304e-03
    ENSG00000188976 1651.1881     -0.6927205 0.0548465 -12.630158 1.43990e-36
    ENSG00000187961  209.6379      0.7297556 0.1318599   5.534326 3.12428e-08
    ENSG00000187583   47.2551      0.0405765 0.2718928   0.149237 8.81366e-01
    ENSG00000187642   11.9798      0.5428105 0.5215598   1.040744 2.97994e-01
                           padj
                      <numeric>
    ENSG00000279457 6.86555e-01
    ENSG00000187634 5.15718e-03
    ENSG00000188976 1.76549e-35
    ENSG00000187961 1.13413e-07
    ENSG00000187583 9.19031e-01
    ENSG00000187642 4.03379e-01

## Data Visualization

Volcano Plot

``` r
dds <- DESeq(dds)
```

    using pre-existing size factors

    estimating dispersions

    found already estimated dispersions, replacing these

    gene-wise dispersion estimates

    mean-dispersion relationship

    final dispersion estimates

    fitting model and testing

``` r
res <- results(dds)
```

``` r
library(ggplot2) 

ggplot(res) + 
  aes(log2FoldChange, -log(padj) ) + 
  geom_point()
```

    Warning: Removed 1237 rows containing missing values or values outside the scale range
    (`geom_point()`).

![](Class14_files/figure-commonmark/unnamed-chunk-17-1.png)

Add threshold lines for fold-change and P-value and color our subset of
genes that make these threshold cut-offs in the plot

## Add Annotation

Add gene symbols and entrez id

``` r
BiocManager::install("org.Hs.eg.db", force = TRUE)
```

    Bioconductor version 3.22 (BiocManager 1.30.27), R 4.5.1 (2025-06-13)

    Installing package(s) 'org.Hs.eg.db'

    installing the source package 'org.Hs.eg.db'

    Warning in install.packages(...): installation of package 'org.Hs.eg.db' had
    non-zero exit status

``` r
library(AnnotationDbi)
library(org.Hs.eg.db)
```

``` r
res$symbol <- mapIds(
  x = org.Hs.eg.db, 
  keys = row.names(res),
  keytype = "ENSEMBL",
  column = "SYMBOL"
)
```

    'select()' returned 1:many mapping between keys and columns

``` r
res$entrez <- mapIds(
  x = org.Hs.eg.db, 
  keys = row.names(res),
  keytype = "ENSEMBL",
  column = "ENTREZID"
)
```

    'select()' returned 1:many mapping between keys and columns

## Pathway Analysis

Run gage analysis

``` r
library(gage)
library(gageData) 
library(pathview) 
```

We need a named vector of fold-change values as input for gage

``` r
foldchanges = res$log2FoldChange
names(foldchanges) = res$entrez
head(foldchanges)
```

           <NA>      148398       26155      339451       84069       84808 
     0.17925708  0.42645712 -0.69272046  0.72975561  0.04057653  0.54281049 

``` r
data(kegg.sets.hs) 

keggres = gage(foldchanges, gsets=kegg.sets.hs)
```

``` r
head(keggres$less, 2) 
```

                                p.geomean stat.mean        p.val       q.val
    hsa04110 Cell cycle      8.995727e-06 -4.378644 8.995727e-06 0.001889103
    hsa03030 DNA replication 9.424076e-05 -3.951803 9.424076e-05 0.009841047
                             set.size         exp1
    hsa04110 Cell cycle           121 8.995727e-06
    hsa03030 DNA replication       36 9.424076e-05

``` r
pathview(pathway.id = "hsa04110", gene.data = foldchanges) 
```

    'select()' returned 1:1 mapping between keys and columns

    Info: Working in directory /Users/lauralu/Desktop/untitled folder/lecture 4/BIMM 143/Class 14

    Info: Writing image file hsa04110.pathview.png

    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)
    Warning in cbind(blk.ind, j): number of rows of result is not a multiple of
    vector length (arg 2)

![](hsa04110.png) ![](hsa0303.png)

### GO terms

Same analysis but using GO genesets rather than KEGG

``` r
data(go.sets.hs) 
data(go.subs.hs) 

# Focus on Biological Process subset of GO 

gobpsets = go.sets.hs[go.subs.hs$BP] 

gobpres = gage(foldchanges, gsets=gobpsets) 
```

``` r
head(gobpres$less) 
```

                                                p.geomean stat.mean        p.val
    GO:0048285 organelle fission             1.536227e-15 -8.063910 1.536227e-15
    GO:0000280 nuclear division              4.286961e-15 -7.939217 4.286961e-15
    GO:0007067 mitosis                       4.286961e-15 -7.939217 4.286961e-15
    GO:0000087 M phase of mitotic cell cycle 1.169934e-14 -7.797496 1.169934e-14
    GO:0007059 chromosome segregation        2.028624e-11 -6.878340 2.028624e-11
    GO:0000236 mitotic prometaphase          1.729553e-10 -6.695966 1.729553e-10
                                                    q.val set.size         exp1
    GO:0048285 organelle fission             5.841698e-12      376 1.536227e-15
    GO:0000280 nuclear division              5.841698e-12      352 4.286961e-15
    GO:0007067 mitosis                       5.841698e-12      352 4.286961e-15
    GO:0000087 M phase of mitotic cell cycle 1.195672e-11      362 1.169934e-14
    GO:0007059 chromosome segregation        1.658603e-08      142 2.028624e-11
    GO:0000236 mitotic prometaphase          1.178402e-07       84 1.729553e-10

### Reactome

Lots of folks like the reactome web interface. You can also run this as
an R function but lets look at the website first \<
https://reactome.org/ \>

The website wants a text file with one gene symbol per line of the genes
you want to map to pathways.

``` r
sig_genes <- res[res$padj <= 0.05 & !is.na(res$padj), ]$symbol 
head(sig_genes) 
```

    ENSG00000187634 ENSG00000188976 ENSG00000187961 ENSG00000188290 ENSG00000187608 
           "SAMD11"         "NOC2L"        "KLHL17"          "HES4"         "ISG15" 
    ENSG00000188157 
             "AGRN" 

``` r
#res$symbol
```

and write out to a file:

``` r
write.table(sig_genes, file="significant_genes.txt",
            row.names=FALSE, col.names=FALSE, quote=FALSE)
```

## Save Our Results

``` r
write.csv(res, file="myresults.csv") 
```
