UKB Expanse SNP/indel/STR GWAS
==============================

This guide will show you how to run a GWAS against a UK Biobank phenotype on Expanse.
The GWAS will include both SNPs, indels and STRs. This uses the WDL and scripts pipeline
written for the UKB blood-traits imputed STRs paper.

Setting up the GWAS and WDL inputs
----------------------------------

First, choose a phenotype you want to perform the GWAS against.
You can explore UKB phenotypes `here <https://biobank.ndph.ox.ac.uk/showcase/index.cgi>`__.
You'll need the data field ID of the phenotype, and the data field IDs of any fields
you wish to use as categorical covariates.

Caveats:

* Currently, only continuous phenotypes are supported.
* Those continuous phenotypes must have been measured at the UKB assessment visits
  (otherwise age calculations will be thrown off or may crash)
* Currently, only categorical covariates are supported

Create a json file for input:

.. code-block:: json

  {
    "expanse_gwas.script_dir": "...",
    "expanse_gwas.phenotype_name": "your_phenotype_name",
    "expanse_gwas.phenotype_id": "its_ID",
    "expanse_gwas.categorical_covariate_names": ["a_list_of_categorical_covariates"],
    "expanse_gwas.categorical_covariate_ids": ["their_IDs"]
  }

Create a json options file specifying where you want your output to be written:

.. code-block:: json

  {
    "final_workflow_outputs_dir": "your_output_directory"
  }


Running the GWAS
----------------

Then, get set up with :ref:`WDL_with_Cromwell_on_Expanse`, including the bit about Singularity.
The docker container you'll want to cache with Singularity is :code:`quay.io/thedevilinthedetails/work/ukb_strs:v1.3`

In the cromwell.conf file you create, add this:

And then for the two lines that says :code:`root = "cromwell-executions`, change them to an
absolute path to the location you want all of your Cromwell run's work to be stored in.

Once you're ready to run WDL

.. code-block:: bash

  # you need to be in this directory for the WDL config to find the scripts
  # appropriately, but all the work, outputs and logs will be written to locations
  # you've specified and not this directory
  cd /expanse/projects/gymreklab/jmargoli/ukbiobank

  java \
    -Dconfig.file=<path_to_your_cromwell.conf_file> \
    -jar <path_to_the_cromwell_jar_you_downloaded> \
    run \
    -i <path_to_your_input_file> \
    -o <path_to_your_options_file> \
    /expanse/projects/gymreklab/jmargoli/ukbiobank/workflow/expanse_targets/expanse_gwas_workflow.wdl \
    > <path_to_your_output.log>

Then you can follow along in another window with :code:`tail -f <your_output.log>`

What the GWAS does
------------------

* Gets a sample list of QCed, unrelated white brits that has the specified phenotype and each specified covariate
* Includes age at time of measurement, genetic PCs and sex as additional covariates.
* Rank quantile normalizes the phenotype (this in theory gives better power to work with non-normal phenotype data,
  but makes output effect sizes and standard errors uninterpretable and uncomparable with those from other runs or studies)
* Performs a GWAS for each imputed SNP, indel and STR of the transformed phenotype against the genotype of that variant
  and all the covariates.
* Calculates the peaks of the signals across the genome.
* Gets a sample list of participants in the same manner as white brits, but for the five ethnicities:
  [black, south_asian, chinese, irish, white_other]
* Runs the GWAS for STRs ONLY in those populations on the subset of regions containing a variant with p<5e-8 in the White Brits.

Output files
------------

Will all be located in :code:`your_output_dir/expanse_gwas`. Unfortunately, they paths to them
will also include IDs which are random alphanumeric strings with dashes in them.

* PLINK GWAS output for imputed SNPs and indels in white_brits :code:`workflow_ID/call-gwas/gwas/subworkflow_ID/call-plink_snp_association/execution/out.tab`
* associaTR GWAS output for imputed STRs in white_brits :code:`workflow_id/call-gwas/gwas/subworkflow_id/call-my_str_gwas_/execution/out.tab`
* associaTR GWAS output for imputed STRs in the other ethnicities:
  :code:`workflow_ID/call-gwas/gwas/subworkflow_ID/call-ethnic_my_str_gwas_/shard_X/execution/out.tab` where X in shard_X is a number from 0 to 4 indicating
  the index of the ethnicity in the list of ethnicities above
* List of GWAS peaks across all variant types in white brits: :code:`workflow_id/call-gwas/gwas/subworkflow_id/call-generate_peaks/execution/peaks.tab`
* Other intermediate outputs will also be there if you want to look at those.

Running on a subpopulation
--------------------------

If you wish to restrict the GWAS to a certain subset of the population, just write that subset
of sample IDs into a file, one per line, with the first line having the header 'ID'. Then add

.. code-block:: json

  "expanse_gwas.subpop_sample_list": "your_sample_file"

to the json input file.

This subpopulation file must contain all samples of all ethnicities that you want included
(so any samples not included will be omitted).

* Samples that fail QC will still be removed.
* Analyses will still be split per ethnicity.
* Each ethnicity's sample list will still be shrunk to remove related participants
* You should include some samples from each ethnicity or the workflow will probably fail
  - you'll still likely get GWAS results from the ethnicities you included, but you'll have to dig for those
  instead of getting them put into the output location you asked for.

You may find the files at :code:`/expanse/projects/gymreklab/jmargoli/ukbiobank/sample_qc/runs/<ethnicity>/no_phenotype/combined.sample`
helpful for building your subpopulation - those location contains the QCed (but not yet unrelated) samples for the six ethincities used in the imputed UKB STRs paper.
