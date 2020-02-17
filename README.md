
<!-- README.md is generated from README.Rmd. Please edit that file -->

# FacileBiocData

<!-- 
badges: start
checkout https://lazappi.github.io/clustree/ package for some badge-inspiration

[![Travis build status](https://travis-ci.org/facilebio/FacileBiocData.svg?branch=master)](https://travis-ci.org/facilebio/FacileBiocData)
[![Codecov test coverage](https://codecov.io/gh/facilebio/FacileBioc/branch/master/graph/badge.svg)](https://codecov.io/gh/facilebio/FacileBiocData?branch=master)

badges: end -->

[![Project
Status](http://www.repostatus.org/badges/latest/active.svg)](http://www.repostatus.org/#active)

[![Lifecycle:
Experimental](https://img.shields.io/badge/lifecycle-experimental-orange.svg)](https://www.tidyverse.org/lifecycle/#experimental)

**NOTE**: This package is being actively developed and not yet feature
complete.

The `FacileBiocData` package enables the use of Bioconductor-standard
data containers, like a `SummarizedExperiment`, `DGEList`,
`DESeqDataSet`, etc. as “first-class” data-providers within the facile
ecosystem.

## Example Usage

The user simply needs to call the `facilitate` function on their data
container in order to enable its use in the facile ecosystem.

``` r
library(FacileBiocData)
y <- example_bioc_data("DGEList") # Materialize an edgeR::DGEList
yf <- facilitate(y)               # wrap it for facile exploration
```

We can now use `yf` within the facile framework, ie. we can analyze it
using the [FacileAnalysis](https://facilebio.github.io/FacileAnalysis/)
package:

``` r
library(FacileAnalysis)
dge.facile <- yf %>% 
  flm_def(group = "sample_type", numer = "tumor", denom = "normal") %>% 
  fdge(method = "edgeR-qlf")
```

``` r
shine(dge.facile)
```

The “facilitated” DGEList can still be used as a normal DGEList, so that
you never have to leave the standard bioconductor ecosystem:

``` r
library(edgeR)
genes <- features(dge.facile)$feature_id # restrict to genes used in fdge()
yf <- calcNormFactors(yf[genes,,keep.lib.sizes = FALSE])
yf <- estimateDisp(yf, model.matrix(~ sample_type, data = yf$samples))
fit <- glmQLFit(yf, yf$design, robust = TRUE)
results <- glmQLFTest(fit, coef = "sample_typetumor")
dge.standard <- topTags(results, n = Inf, sort.by = "none")
```

And these two results are equivalent

``` r
all.equal(tidy(dge.facile)$logFC, dge.standard$logFC)
```
