---
layout: default
title:  'Supplementary Datasets for Margoliash et al. "Polymorphic short tandem repeats make widespread contributions to blood and serum traits"'
tags: science STRs
categories: science
date: 08 September 2023
---

# Supplementary Datasets for Margoliash et al. 2023

This page contains links to the supplementary dataset files for 

#### Margoliash, Jonathan, et al. "Polymorphic short tandem repeats make widespread contributions to blood and serum traits." bioRxiv (2022): 2022-08.

There supplementary datasets are:

## STR association tests performed in the White British population, by phenotype
## STR association tests performed in the other populations, by phenotype and population

The README for these association files is [here](https://gymreklab.com/2023/Margoliash/et/al/GWAS/README.html).
Association statistics referred to in the text are from the White British population unless otherwise specified. Association tests
in the White British population were performed genome-wide, while STR association tests in the other populations were performed in 
fine-mapping regions identified by the White British association tests.

URLs are of the form: https://margoliash-et-al-2023.s3.amazonaws.com/associations/{population}\_{phenotype}\_str\_gwas\_results.tab.gz

Populations are:

* white\_british
* black
* south\_asian
* chinese
* irish
* white\_other

Phenotypes are written exactly as listed in the Supplementary Table 1.

We filtered STRs with total minor allele dosage less than 20. This amounted to very few STRs for each population, phenotype combination.
A list of those STRs (if any) is available at the URL: 

https://margoliash-et-al-2023.s3.amazonaws.com/associations/{population}\_{phenotype}\_str\_gwas\_results.filtered.tab

## Tarballs of all fine-mapping outputs across all fine-mapping regions and conditions, by phenotype

Each tarball contains two summary tables: finemapping\_first\_pass.tab and finemapping\_followup.tab, describing the fine-mapping results
from the standard fine-mapping runs and followup fine-mapping runs under alternate conditions, respectively. Additionally,
each tarball contains all the raw fine-mapping outputs for each fine-mapping run for both SuSiE and FINEMAP. Note that
fine-mapping was only performed in the White British population.

The README for these files is [here](https://gymreklab.com/2023/Margoliash/et/al/finemapping/README.html).

URLs are of the form: https://margoliash-et-al-2023.s3.amazonaws.com/finemapping/finemapping\_{phenotype}.tgz
