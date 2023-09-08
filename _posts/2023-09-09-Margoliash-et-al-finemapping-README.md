---
hide: true
layout: default
date: 08 September 2023
title: Fine-mapping README
---

# Fine-mapping Supplementary Dataset README

One tarball finemapping\_${trait}.tgz for each trait

The tarball contains two summary files

    finemapping_first_pass.tab
    finemapping_followup.tab

In these files

- positions are hg19 and 1-indexed
- Regions are denoted by ${chrom}\_${start\_pos\_inclusive}\_${end\_pos\_inclusive}
- variant names for SNPS are SNP\_{position}\_{ref}\_{alt}
- variant names for STRs are STR\_{start\_position}
- p\_val, coeff and se are from the association of that variant with the trait being fine-mapped
- susie\_cs values denote which pure cs a variant is present in, or -1 if its in no pure cs
- finemap\_pip and susie\_alpha values were used as CP values in the paper
- susie\_cs values of -1 imply that the susie\_alpha value should be ignored (treated as zero)
- only variants included in fine-mapping are present in these files
- some variants lack susie values if they were not included in the susie fine-mapping (probably because they had too high a p-value)
 The followup files only contain variants from the regions subjected to follow-up fine-mapping conditions.
- traits with no regions that were followed-up on are empty.
- In addition to the columns in the first pass file (except for coeff or se), there are additional FINEMAP or SuSiE columns for each extra condition.
- the best\_guess column corresponds to the use of best guess genotypes from imputation for fine-mapping
- the ratio columns correspond to the prior of favoring SNP over STR causality by a 4-to-1 ratio
- the repeat column corresponds to the repeat FINEMAP run with no changed settings
- the total\_prob column corresponds to the FINEMAP run with a prior of there being 4 total causal variants
- the prior\_std\_derived and prior\_std\_low columns correspond to the priors for the effect sizes of causal variants of 0.05% and 0.0025%, respectively
- the conv\_tol column corresponds to the flag --prob-conv-sss-tol 0.0001
- the mac column corresponds to the non-major allele dosage threshold of 100 
- the p\_thresh column corresponds to the p\_value threshold of 1e-4
- values for the additional columns will be missing for variants which were not fine-mapped in specifically those conditions
  (say, a variant with p-value of 1e-3 in the 1e-4 threshold column)

In addition to the two summary files, the tarballs contain raw fine-mapping output files for each region.  
The files for FINEMAP are ( described at http://christianbenner.com/ ):

    FINEMAP\_first\_pass\_${region}\_finemap\_output.log
    FINEMAP\_first\_pass\_${region}\_finemap\_output.snp
    FINEMAP\_first\_pass\_${region}\_finemap\_output.config
    FINEMAP\_first\_pass\_${region}\_finemap\_output.credX

and the following files for SuSiE:

    SuSiE\_first\_pass\_${region}\_alpha.tab
    SuSiE\_first\_pass\_${region}\_colnames.txt
    SuSiE\_first\_pass\_${region}\_csX.txt
    SuSiE\_first\_pass\_${region}\_lbf.tab
    SuSiE\_first\_pass\_${region}\_lbf\_variable.tab
    SuSiE\_first\_pass\_${region}\_lfsr.tab
    SuSiE\_first\_pass\_${region}\_sigma2.txt
    SuSiE\_first\_pass\_${region}\_V.tab

* Except for the colnames and cs files, these are arrays written from the output fields of the susie() function described at
  https://stephenslab.github.io/susieR/reference/susie.html  
* colnames contains one variant name per line, each line corresponding to one column of the alpha array
* csX contains three rows:

  - the first contains the 1-indexed numbers of the variables included in the credible set, in ascending order
  - the second contains the coverage of the credible set (Always greater than 0.9 which was the requested coverage)
  - the third contains three numbers, the min, mean and median absolute correlations between each variable in the credible set 
    ( see susie\_get\_cs() at https://stephenslab.github.io/susieR/reference/susie\_get\_methods.html )
  - the 1-indexed number in the filename corresponds to the corresponding row of the alpha.tab array

Additionally, for each of the regions we followed up on, this tarball contains the same files as above, but with
the following prefixes, corresponding to the follow-up fine-mapping condition being tested:

    FINEMAP\_derived\_effect\_size\_prior - (effect size prior of 0.05%)
    FINEMAP\_low\_effect\_size\_prior - (effect size prior of 0.0025%)
    FINEMAP\_mac\_threshold\_100 - (non-major allele dosage threshold of 100)
    FINEMAP\_prior\_4\_signals - (prior of 4 causal variants per region)
    FINEMAP\_prior\_snps\_over\_strs - (prior of favoring SNP over STR causality by a 4-to-1 ratio)
    FINEMAP\_pval\_threshold\_1e4 - (p\_value threshold of 1e-4)
    FINEMAP\_stricter\_stopping\_threshold - (using the flag --prob-conv-sss-tol 0.0001)
    SuSiE\_prior\_snps\_over\_strs - (prior of favoring SNP over STR causality by a 4-to-1 ratio)
    SuSiE\_best\_guess\_genotypes - (the use of best guess genotypes from imputation for fine-mapping)

See the paper and Supplementary Note 3 for more details on each follow-up condition.  
We do not have raw output files for FINEMAP\_repeat runs.

Notes:

* For FINEMAP, I have focused on the posterior probabilities in the finemap\_output.snp files
* For SuSiE I focused on the cs files with min absolute correlation > 0.8,
  and then the values in alpha.tab with rows identified by the CS number and columns identified by the variants in the first row of the CS file
* the log files for FINEMAP saying v1.4.1 seem to be buggy, the actual FINEMAP output to the command line indicates that v1.4.2 was run.
