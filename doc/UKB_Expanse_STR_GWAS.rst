UKB Expanse SNP/indel/STR GWAS and fine-mapping
===============================================

This guide will show you how to run a GWAS against a UK Biobank phenotype on Expanse.
The GWAS will include both SNPs, indels and STRs. This uses the WDL and scripts pipeline
written for the UKB blood-traits imputed STRs paper.

Setting up the GWAS and WDL inputs
----------------------------------

First, check out `my paper's repository <https://github.com/LiterallyUniqueLogin/ukbiobank_strs>`_ into some directory you manage.

Then, choose a phenotype you want to perform the GWAS against.
You can explore UKB phenotypes `here <https://biobank.ndph.ox.ac.uk/showcase/index.cgi>`__.
You'll need the data field ID of the phenotype, and the data field IDs of any fields
you wish to use as categorical covariates.

Caveats:

* Currently, only continuous phenotypes are supported.
* Those continuous phenotypes must have been measured at the UKB assessment visits
  (otherwise age calculations will be thrown off or may crash)
* Currently, only categorical covariates are supported

Create a json file for input, setting script_dir to the root of the git repo you checked out above, and all the others as appropriate:

.. code-block:: json

  {
    "expanse_gwas.script_dir": "repo_source"
    "expanse_gwas.phenotype_name": "your_phenotype_name",
    "expanse_gwas.phenotype_id": "its_data_field_ID",
    "expanse_gwas.categorical_covariate_names": ["a_list_of_categorical_covariates"],
    "expanse_gwas.categorical_covariate_ids": ["their_IDs"]
  }

Running the GWAS
----------------

Then, get set up with :ref:`WDL_with_Cromwell_on_Expanse`, including the bit about Singularity.
The two docker containers you'll want to cache with Singularity prior to your run are 
:code:`quay.io/thedevilinthedetails/work/ukb_strs:v1.3` and
:code:`quay.io/thedevilinthedetails/work/ukb_strs:v1.4`

The WDL workflow file you'll point Cromwell to is in the repo you checked out at :code:`workflow/expanse_wdl/gwas.wdl`. Run
Cromwell as normal using the standard Cromwell instructions with your input and that workflow.

If you want to run the GWAS only in a specific subpopulation of the UKB, see :ref:`running_on_a_subpopulation` below.

What the GWAS does
------------------

The full details are in the methods of the paper `here <https://www.biorxiv.org/content/10.1101/2022.08.01.502370v3>`_. In short, this pipeline:

* Gets a sample list of QCed, unrelated white brits that has the specified phenotype and each specified covariate
* Includes age at time of measurement, genetic PCs and sex as additional covariates.
* Rank quantile normalizes the phenotype (this in theory gives better power to work with non-normal phenotype data,
  but makes output effect sizes and standard errors uninterpretable and uncomparable with those from other runs or studies)
* Performs a GWAS for each imputed SNP, indel and STR of the transformed phenotype against the genotype of that variant
  and all the covariates.
* Calculates the peaks of the signals across the genome.
* Gets a sample list of participants in a similar manner as white brits, but for the five ethnicities:
  [black, south_asian, chinese, irish, white_other]
* Runs the GWAS for STRs in those populations *only* on the regions containing a variant with p<5e-8 in the White Brits.

Output file names
-----------------

Final outputs:

* PLINK GWAS output for imputed SNPs and indels in white_brits :code:`white_brits_snp_gwas.tab`
* associaTR GWAS output for imputed STRs in white_brits :code:`white_brits_str_gwas.tab`
* associaTR GWAS output for imputed STRs in the other ethnicities :code:`<ethnicity>_str_gwas.tab`
* List of GWAS peaks across all variant types, at least 250kb separate, in white brits: :code:`peaks.tab`
* List of regions for followup fine-mapping regions in all variant types, in white brits: :code:`finemapping_regions.tab`

Intermediate outputs potentially useful for debugging:

* Lists of all the participants used in the GWAS after all subsetting, entitled :code:`<ethnicity>.samples`
* The shared covars array :code:`shared_covars.npy` and their names :code:`covar_names.txt`
* The (original) untransformed phenotype data, deposited for your reference, :code:`<ethnicity>_original_pheno.npy`
* The transformed phenotype data used in the regression plus all the covariates you specified :code:`<ethnicity>_pheno.npy`, as well as the names of those covariates :code:`<ethnicity>_pheno_covar_names.txt`

  .. _running_on_a_subpopulation:

Running on a subpopulation
--------------------------

If you wish to restrict the GWAS to a certain subset of the population, just write that subset
of sample IDs into a file, one per line, with the first line having the header 'ID'. Then add

.. code-block:: json

  "gwas.subpop_sample_list": "your_sample_file"

to the json input file.

This subpopulation file must contain all samples of all ethnicities that you want included
(i.e. any samples not included will be omitted).

Note that providing this file doesn't change the pipeline's workflow:

* Samples that fail QC will still be removed.
* Analyses will still be split per ethnicity.
* Each ethnicity's sample list will still be shrunk to remove related participants

Running fine-mapping
--------------------

TODO
