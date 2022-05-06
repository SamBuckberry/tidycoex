# coex: An R package for co-expression network analyses

Correlation networks are have become a common way to analyse gene expression data. In this area,  weighted gene co-expression network analysis (WGCNA) is a popular method for analysing the correlation patterns among genes across samples.

The `coex` R package is a collection of eassy to use R functions for performing co-expression analyses, primarily from RNA-seq data. The package includes functions for normalising RNA-seq count data for co-expression analyses, batch effect detection and correction, network construction, module detection, visualisation, among other features.

`coex` features include:  
* An S4 class object designed specifically for co-expression analyses that extends the `SummarizedExperiment` class.
* Data filtering and normalisation functions for RNA-seq count data
    * CTF normalisation as implimented in [Johnson & Krishnan 2022](https://genomebiology.biomedcentral.com/articles/10.1186/s13059-021-02568-9)  
* Optimised wrappers for core `WGCNA` functions  
* Function for CLR adjacency calculations as implimented in [Johnson & Krishnan 2022](https://genomebiology.biomedcentral.com/articles/10.1186/s13059-021-02568-9)  
* Very fast clustering functions derived from the `fastcluster` package  
* Plotting functions that output ``ggplot2`` objects

## How to install the `coex` R package


### GitHub installation
First install the bioconductor dependencies
```{r}
if (!require("BiocManager", quietly = TRUE))
    install.packages("BiocManager")

BiocManager::install(c("impute", "preprocessCore",
    "GO.db", "AnnotationDbi", "edgeR", "SummarizedExperiment",
    "minet")
```

Then use `devtools` to install `coex` 
```{r}
install.packages("devtools")
library(devtools)

devtools::install_github("SamBuckberry/coex")
```

## Testing the `coex` installation

Executing the following lines of code should return a `coexList` object.
```{r}
library(coex)
edat <- matrix(rpois(5000 * 16, lambda=5), nrow=5000)
coexList(counts = edat)
```

## A `coex` WGCNA analysis in 10 lines of code

Analyses are based on using the `coexList` object. Most functions continually add data and results to the `coexList` object in succession throughout the analysis, as the below example demonstrates.

```{r}
counts <- matrix(rpois(1000 * 16, lambda=5), nrow=1000)
cl <- coexList(counts) 
cl <- normCounts(cl, normMethod = "CPM")
cl <- calcSoftPower(cl)
cl <- calcAdjacency(cl, method = "wgcna") 
cl <- calcTOM(cl)
cl <- calcTree(cl) 
plotModuleTree(cl) 
cl <- calcModuleEigengenes(cl, deepSplit=2) 
heatmapModules(cl) 
```

## A `coex` workflow can be piped together

As the `coexList` object is used for the input and output of most functions, they can be piped together in succession.
```{r}
counts <- matrix(rpois(1000 * 16, lambda=5), nrow=1000)

cl <- coexList(counts) %>% normCounts() %>%
    calcAdjacency(softPower=6) %>% calcTOM() %>% 
    calcTree() %>% plotModuleTree()
    
calcModuleEigengene(cl, deepSplit = 2) %>% heatmapModules()
```


### Notes and further reading

[Measures such as Pearson correlation automatically include a standardization of gene expression across samples, which could explain why additional library size correction may not be needed. This implication is also supported by the Counts workflow outperforming within-sample normalization workflows](https://genomebiology.biomedcentral.com/articles/10.1186/s13059-021-02568-9#Sec9) 

__Signed networks are the default in tidycoex__  
For a nice explanation of why signed networks are usually preferable, see the article [Signed or unsigned: which network type is preferable?](https://peterlangfelder.com/2018/11/25/signed-or-unsigned-which-network-type-is-preferable/) by Peter Langfelder.

### Further reading
- [Robust normalization and transformation techniques for constructing gene coexpression networks from RNA-seq data](https://genomebiology.biomedcentral.com/articles/10.1186/s13059-021-02568-9), [Data + more plots](https://krishnanlab.github.io/RNAseq_coexpression/gtex_plots.html)

- [Addressing confounding artifacts in reconstruction of gene co-expression networks](https://genomebiology.biomedcentral.com/articles/10.1186/s13059-019-1700-9)

- [wTO: weighted topological overlap and a consensus network R package](https://bmcbioinformatics.biomedcentral.com/articles/10.1186/s12859-018-2351-7), [Tutorial](https://static-content.springer.com/esm/art%3A10.1186%2Fs13059-019-1700-9/MediaObjects/13059_2019_1700_MOESM4_ESM.html)

- Spatial quantile normalisation for co-expression analysis [bioc](https://bioconductor.org/packages/release/bioc/vignettes/spqn/inst/doc/spqn.html#References), [biorxiv](https://www.biorxiv.org/content/10.1101/2020.02.13.944777v1.full.pdf)
