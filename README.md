# TD_GWAS

This directory contains course materials for TD GWAS.

## Required Softwares
* R version 4.0.3 (https://pbil.univ-lyon1.fr/CRAN/)
* Rstudio version 1.3 (https://rstudio.com/products/rstudio/download/)
* PLINK version 1.90 beta (https://www.cog-genomics.org/plink2)

## Required R Packages
* dplyr, tidyr, scales, data.table, ggplot2

## Toy Data
The toy data is a subset of the freely [HapMap](https://www.sanger.ac.uk/resources/downloads/human/hapmap3.html) data, containing only CEU samples, a binary trait was simulated for only founders.

Can be downloaded here: https://share-good.egid.fr/fop/VZfxQvAD/TD_GWAS_data.zip

* Raw HapMap CEU samples PLINK format data (build 36): `hapmap_ceu.bim`, `hapmap_ceu.bed`, `hapmap_ceu.fam`
* Samples included in population structure check: `hapmap_1kg.fam`, `hapmap_ceu_pca_1kg_adj_clean.fam`
* PCA eigenvalues and eigenvector for population structure check: `hapmap_1kg.eigenval`, `hapmap_1kg.eigenvec`
* Covariables sex, PC1 and PC2: `covars.txt`
* Logistic regression results: `res_glm_covar_chr22.assoc.logistic`
* Some R functions for data visualization: `viz_functions.R`
* Temporary file for figures: `figs/`
* TD supports: `TD_GWAS.Rmd` and `TD_GWAS.html`
* HapMap sample population: `relationships_w_pops_041510.txt` (ftp://ftp-trace.ncbi.nih.gov/1000genomes/ftp/release/20100804/20100804.ALL.panel)
* 1,000 genomes population: `20100804.ALL.panel` (ftp://ftp-trace.ncbi.nih.gov/1000genomes/ftp/release/20100804/20100804.ALL.panel)
* 1,000 genomes population and super-population: `integrated_call_samples_v3.20130502.ALL.panel` (ftp://ftp-trace.ncbi.nih.gov/1000genomes/ftp/release/20130502/integrated_call_samples_v3.20130502.ALL.panel)

## HapMap CEU Sample Extraction
The CEU samples were extracted using following commands:

```bash
## HapMap plink format data
wget https://ftp.ncbi.nlm.nih.gov/hapmap/genotypes/hapmap3_r3/plink_format/hapmap3_r3_b36_fwd.consensus.qc.poly.map.gz
wget https://ftp.ncbi.nlm.nih.gov/hapmap/genotypes/hapmap3_r3/plink_format/hapmap3_r3_b36_fwd.consensus.qc.poly.ped.gz
gunzip hapmap3_r3_b36_fwd.consensus.qc.poly.ped.gz
gunzip hapmap3_r3_b36_fwd.consensus.qc.poly.map.gz

## HapMap phenotype
wget https://ftp.ncbi.nlm.nih.gov/hapmap/genotypes/hapmap3_r3/relationships_w_pops_041510.txt
awk '($7=="CEU") {print$1,$2}' relationships_w_pops_041510.txt > hapmap_ceu_sample.txt

## Extraction
plink1.9 --file hapmap3_r3_b36_fwd.consensus.qc.poly --keep hapmap_ceu_sample.txt --make-bed --out hapmap_ceu
```

```r
## Binary trait simulation
pop <- data.table::fread("relationships_w_pops_041510.txt", header = TRUE)[population == "CEU", ]
set.seed(20201106)
sim_trait <- sample(1:2, size = nrow(pop[dad == mom]), prob = c(2/3, 1/3), replace = TRUE)
fam_trait <- pop[, 1:5][dad == mom, pheno := sim_trait]
fam_trait$pheno[is.na(fam_trait$pheno)] <- -9
write.table(
  fam_trait, 
  file = "hapmap_ceu.fam", 
  sep = "\t", 
  col.names = FALSE, 
  row.names = FALSE,
  quote = FALSE
)
```
