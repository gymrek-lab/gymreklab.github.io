---
hide: true
layout: default
date: 09 September 2023
title: Associations README
---


## Columns

* chromosome
* base\_pair\_location: beginning of the repeat (hg19, 1-indexed, inclusive)
* alleles:

  lengths alleles in the population measured in number of repeat units. For example, the allele 5 for an
  AC repeat implies the bases "ACACACACAC" (possibly with some impurity).
  Occasionally the repeat unit will be listed as none. This occurs when it was hard to determine the 
  repeat unit from the period as there were multiple repeat units present in the reference allele
  of length equal to period and with similar frequencies.
  In that case, the period of the repeat will still be given, and the length of an allele in base pairs
  can still be calculated by multiplying the allele by the period.

* beta:

  measured effect size of the linear association of the rank-inverse-normalized phenotype against the 
  length-dosages of unnormalized STR genotypes, measured in number of repeat units.
  Phenotypes are measured in unspecified units as they are rank-inverse-normalized, so these betas
  should only be compared to betas from other studies with sufficient reason to believe that such a
  comparison is meaningful. p-values may be more comparable between studies.

* standard\_error: See caveats for beta
* allele\_frequencies
* p\_value: p-values less than 1e-300 exceeded our software's numeric precision and are listed as 0
* ref\_allele: measured in number of repeat units
* repeat\_unit: the standardized repeat unit of this STR, or none if there was no one clear repeat unit
* period: the length of the repeat unit
* end\_pos (hg19): end of the repeat (1-indexed, inclusive)
* start\_pos (hg38)
* end\_pos (hg38)
* n:

  this study worked with imputed calls and no call-level filters, as such n will be equivalent for each
  variant associated with the same phenotype

* number\_of\_common\_alleles:

  the number of alleles in the population with frequency >= 1%

* mean\_{phenotype}\_per\_summed\_gt:

  the mean phenotype value for each sum of allele lengths, where each participant's contribution to the 
  phenotype mean for each length-sum is weighted by the imputed probability of their true genotype sum being
  equal to that length-sum. Can be used for plotting graphs of mean phenotype value vs summed-length.
  Summed gts are measured in number of summed repeat units.

* summed\_0.05\_significance\_CI:

  The 95% symmetric confidence interval for each of the means above

* summed\_5e-8\_significance\_CI:

  The (1 - 5e-8) symmetric confidence interval for each of the means above. I.e. this interval
  is expected to contain the true mean with a probability of 1 - 5e-8, which is very close to one.

* mean\_{phenotype}\_per\_paired\_gt:

  the mean phenotype value for each unordered pair of allele lengths, where each participant's contribution to the 
  phenotype mean for each pair  is weighted by the imputed probability of their true genotype pair being
  equal to that pair. Can be used for plotting graphs of mean phenotype value vs length pair.
  Each gt in each pair is measured in number of repeat units.

* paired\_0.05\_significance\_CI:

  The 95% symmetric confidence interval for each of the means above

* paired\_5e-8\_significance\_CI:

  The (1 - 5e-8) symmetric confidence interval for each of the means above
