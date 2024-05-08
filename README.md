
<!-- README.md is generated from README.Rmd. Please edit that file -->

# vcfppR: rapid manipulation of the VCF/BCF file

<!-- badges: start -->

<a href="https://github.com/Zilong-Li/random/blob/main/vcfppR.png"><img src="https://raw.githubusercontent.com/Zilong-Li/random/main/vcfppR.png" width="120" align="right" /></a>
![R-CMD-check](https://github.com/Zilong-Li/vcfppR/actions/workflows/check-release.yaml/badge.svg)
[![CRAN
status](https://www.r-pkg.org/badges/version/vcfppR)](https://CRAN.R-project.org/package=vcfppR)
![<https://github.com/Zilong-Li/vcfppR/releases/latest>](https://img.shields.io/github/v/release/Zilong-Li/vcfppR.svg)
[![codecov](https://codecov.io/github/Zilong-Li/vcfppR/graph/badge.svg?token=QE1UFVRH98)](https://app.codecov.io/github/Zilong-Li/vcfppR)
[![Downloads](https://cranlogs.r-pkg.org/badges/vcfppR?color=blue)](https://CRAN.R-project.org/package=vcfppR)
<!-- badges: end -->

The vcfppR package implements various powerful functions for fast
genomics analyses with VCF/BCF files using the C++ API of
[vcfpp.h](https://github.com/Zilong-Li/vcfpp).

  - Load/save content of VCF/BCF into R objects with highly customizable
    options
  - Visualize and chracterize variants
  - Compare two VCF/BCF files and report various statistics
  - Streaming read/write VCF/BCF files with fine control of everything
  - [Paper](https://doi.org/10.1093/bioinformatics/btae049) shows vcfppR
    is 20x faster than `vcfR`. Also, much faster than `cyvcf2`

## Installation

``` r
## install.package("vcfppR") ## from CRAN
remotes::install_github("Zilong-Li/vcfppR") ## from latest github
```

## Cite the work

If you find it useful, please cite the
[paper](https://doi.org/10.1093/bioinformatics/btae049)

``` r
library(vcfppR)
citation("vcfppR")
```

## URL as filename

All functions in vcfppR support URL as filename of VCF/BCF files.

``` r
phasedvcf <- "https://ftp.1000genomes.ebi.ac.uk/vol1/ftp/data_collections/1000G_2504_high_coverage/working/20220422_3202_phased_SNV_INDEL_SV/1kGP_high_coverage_Illumina.chr21.filtered.SNV_INDEL_SV_phased_panel.vcf.gz"
rawvcf <- "https://ftp.1000genomes.ebi.ac.uk/vol1/ftp/data_collections/1000G_2504_high_coverage/working/20201028_3202_raw_GT_with_annot/20201028_CCDG_14151_B01_GRM_WGS_2020-08-05_chr21.recalibrated_variants.vcf.gz"
svfile <- "https://ftp.1000genomes.ebi.ac.uk/vol1/ftp/data_collections/1000G_2504_high_coverage/working/20210124.SV_Illumina_Integration/1KGP_3202.gatksv_svtools_novelins.freeze_V3.wAF.vcf.gz"
popfile <- "https://ftp.1000genomes.ebi.ac.uk/vol1/ftp/data_collections/1000G_2504_high_coverage/20130606_g1k_3202_samples_ped_population.txt"
```

## `vcfcomp`: compare two VCF files and report concordance

Want to investigate the concordance between two VCF files? `vcfcomp` is
the utility function you need\! For example, in benchmarkings, we intend
to calculate the genotype correlation between the test and the truth.

``` r
library(vcfppR)
res <- vcfcomp(test = rawvcf, truth = phasedvcf,
               stats = "r2", region = "chr21:1-5100000", 
               formats = c("GT","GT"))
par(mar=c(5,5,2,2), cex.lab = 2)
vcfplot(res, what = "r2", col = 2, cex = 2, lwd = 4, type = "b")
#> input is an object with vcfcomp class
```

<img src="man/figures/README-r2-1.png" width="100%" />

``` r
res <- vcfcomp(test = rawvcf, truth = phasedvcf,
               stats = "pse",
               region = "chr21:5000000-6000000",
               samples = "HG00673,NA10840",
               return_pse_sites = TRUE)
#> stats F1 or NRC or PSE only uses GT format
vcfplot(res, what = "pse", which=1:2, main = "Phasing switch error", ylab = "HG00673,NA10840")
#> input is an object with vcfcomp class
```

<img src="man/figures/README-pse-1.png" width="100%" />

Check out the [vignettes](https://zilong-li.github.io/vcfppR/articles/)
for more\!

## `vcfsummary`: variants characterization

Want to summarize variants discovered by genotype caller e.g. GATK?
`vcfsummary` is the utility function you need\!

**Small variants**

``` r
res <- vcfsummary(rawvcf,"chr21:10000000-10010000")
vcfplot(res, pop = popfile, col = 1:5, main = "Number of SNP & INDEL variants per population")
#> input is an object with vcfsummary class
```

<img src="man/figures/README-summary_sm-1.png" width="100%" />

**Complex structure variants**

``` r
res <- vcfsummary(svfile, svtype = TRUE, region = "chr20")
vcfplot(res, main = "Structure Variant Counts", col = 1:7)
#> input is an object with vcfsummary class
```

<img src="man/figures/README-summary_sv-1.png" width="100%" />

## `vcftable`: read VCF as tabular data

`vcftable` gives you fine control over what you want to extract from
VCF/BCF files.

**Read SNP variants with GT format and two samples**

``` r
res <- vcftable(phasedvcf, "chr21:1-5100000", samples = "HG00673,NA10840", vartype = "snps")
str(res)
#> List of 10
#>  $ samples: chr [1:2] "HG00673" "NA10840"
#>  $ chr    : chr [1:194] "chr21" "chr21" "chr21" "chr21" ...
#>  $ pos    : int [1:194] 5030578 5030588 5030596 5030673 5030957 5030960 5031004 5031031 5031194 5031224 ...
#>  $ id     : chr [1:194] "21:5030578:C:T" "21:5030588:T:C" "21:5030596:A:G" "21:5030673:G:A" ...
#>  $ ref    : chr [1:194] "C" "T" "A" "G" ...
#>  $ alt    : chr [1:194] "T" "C" "G" "A" ...
#>  $ qual   : num [1:194] 2.14e+09 2.14e+09 2.14e+09 2.14e+09 2.14e+09 ...
#>  $ filter : chr [1:194] "." "." "." "." ...
#>  $ info   : chr [1:194] "AC=74;AF=0.0115553;CM=0;AN=6404;AN_EAS=1170;AN_AMR=980;AN_EUR=1266;AN_AFR=1786;AN_SAS=1202;AN_EUR_unrel=1006;AN"| __truncated__ "AC=53;AF=0.00827608;CM=1.78789e-05;AN=6404;AN_EAS=1170;AN_AMR=980;AN_EUR=1266;AN_AFR=1786;AN_SAS=1202;AN_EUR_un"| __truncated__ "AC=2;AF=0.000312305;CM=3.21821e-05;AN=6404;AN_EAS=1170;AN_AMR=980;AN_EUR=1266;AN_AFR=1786;AN_SAS=1202;AN_EUR_un"| __truncated__ "AC=2;AF=0.000312305;CM=0.00016985;AN=6404;AN_EAS=1170;AN_AMR=980;AN_EUR=1266;AN_AFR=1786;AN_SAS=1202;AN_EUR_unr"| __truncated__ ...
#>  $ gt     : int [1:194, 1:2] 0 0 0 0 0 0 0 0 0 0 ...
#>  - attr(*, "class")= chr "vcftable"
```

**Read SNP variants with PL format and drop the INFO column**

``` r
res <- vcftable(rawvcf, "chr21:1-5100000", samples = "HG00673,NA10840", format = "PL", info = FALSE)
#> Warning in rbind(c(0L, 3L, 35L, 0L, 9L, 104L), c(32L, 3L, 0L, 95L, 9L, 0L:
#> number of columns of result is not a multiple of vector length (arg 7)
str(res)
#> List of 10
#>  $ samples: chr [1:2] "HG00673" "NA10840"
#>  $ chr    : chr [1:1682] "chr21" "chr21" "chr21" "chr21" ...
#>  $ pos    : int [1:1682] 5030082 5030088 5030105 5030240 5030253 5030278 5030319 5030347 5030356 5030357 ...
#>  $ id     : chr [1:1682] "." "." "." "." ...
#>  $ ref    : chr [1:1682] "G" "C" "C" "AC" ...
#>  $ alt    : chr [1:1682] "A" "T" "A" "A" ...
#>  $ qual   : num [1:1682] 70.1 2773.1 3897.8 6872.4 102.6 ...
#>  $ filter : chr [1:1682] "VQSRTrancheSNP99.80to100.00" "VQSRTrancheSNP99.80to100.00" "VQSRTrancheSNP99.80to100.00" "VQSRTrancheINDEL99.00to100.00" ...
#>  $ info   : chr(0) 
#>  $ PL     : int [1:1682, 1:42] 0 32 66 0 0 0 0 0 0 0 ...
#>  - attr(*, "class")= chr "vcftable"
```

**Read INDEL variants with DP format and QUAL\>100**

``` r
res <- vcftable(rawvcf, "chr21:1-5100000", vartype = "indels", format = "DP", qual=100)
str(res)
#> List of 10
#>  $ samples: chr [1:3202] "HG00096" "HG00097" "HG00099" "HG00100" ...
#>  $ chr    : chr [1:180] "chr21" "chr21" "chr21" "chr21" ...
#>  $ pos    : int [1:180] 5030240 5030912 5030937 5031018 5031125 5031147 5031747 5031881 5031919 5031984 ...
#>  $ id     : chr [1:180] "." "." "." "." ...
#>  $ ref    : chr [1:180] "AC" "CA" "T" "C" ...
#>  $ alt    : chr [1:180] "A" "C" "TGGTGCACGCCTGCAGTCCCGGC" "CCTATGATCACACCGT" ...
#>  $ qual   : num [1:180] 6872 321 233 270 58361 ...
#>  $ filter : chr [1:180] "VQSRTrancheINDEL99.00to100.00" "PASS" "PASS" "PASS" ...
#>  $ info   : chr [1:180] "AC=82;AF=0.0136439;AN=6010;BaseQRankSum=-0.55;ClippingRankSum=0;DP=18067;FS=0;InbreedingCoeff=0.0664;MLEAC=76;M"| __truncated__ "AC=19;AF=0.0030586;AN=6212;BaseQRankSum=0.358;ClippingRankSum=-0.594;DP=30725;FS=44.58;InbreedingCoeff=-1.1608;"| __truncated__ "AC=1;AF=0.00015753;AN=6348;BaseQRankSum=-1.793;ClippingRankSum=-0.48;DP=32486;FS=27.337;InbreedingCoeff=-0.0212"| __truncated__ "AC=2;AF=0.000314268;AN=6364;BaseQRankSum=1.59;ClippingRankSum=0.358;DP=44916;FS=0;InbreedingCoeff=-0.0092;MLEAC"| __truncated__ ...
#>  $ DP     : int [1:180, 1:3202] 3 15 13 16 19 20 5 34 29 24 ...
#>  - attr(*, "class")= chr "vcftable"
```

## R API of vcfpp.h

There are two classes i.e. ***vcfreader*** and ***vcfwriter*** offering
the full R-bindings of vcfpp.h. Check out the examples in the
[tests](tests/testthat) folder or refer to the manual.

``` r
?vcfppR::vcfreader
```
