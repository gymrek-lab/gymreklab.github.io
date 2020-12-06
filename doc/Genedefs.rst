Obtaining gene annotations
==========================

We often need to perform analyses where we restrict to certain gene regions, such as us coding regions, UTRs, introns, etc.

UCSC
----

One way to obtain BED files (chrom, start, end) with coordinates of certain gene annotations is through the UCSC Genome Browser's Table Browser tool (http://genome.ucsc.edu/cgi-bin/hgTables). This tool allows you to download tons of different annotations, not just gene annotations.

To obtain gene info for your species/genome build of interest:

1. Set clade/genome/assembly to the appropriate values. e.g. Clade=Mammal, genome=Human, assembly=GRCh37/hg19 for the hg19 reference genome.

2. Set group="Genes and gene predictions" and track="UCSC Genes" (or you may be using another gene annotation. Gencode or Refseq are commonly used).

3. Set table="knownGene"

4. Select "genome" for region.

5. (important!) Set output format="custom track". Important for step 7.

6. Set "output file" to the filename you want to save results to.

7. Click "get output". This will bring you to a hidden page where you can select which regions you want included in the output file. e.g. you may select coding, exons, 5'UTR, 3'UTR, etc. Note "exons" includes UTRs. If you only want coding regions, be sure to select "coding exons" only.

8. Click "get custom track in file", which should output the file to your desired filename.

Note: there should also be a way to automate this process by connecting to the UCSC tables using mysql on the command line. e.g. bedtools documentation shows how to do this to get chromosome sizes:

.. code-block:: bash

  mysql --user=genome --host=genome-mysql.cse.ucsc.edu -A -e "select chrom, size from hg19.chromInfo" > hg19.genome

If anyone wants to update this page with mysql instructions, go for it!

Ensembl
-------

Ensembl is a genome browser from The European Bioinformatics Institute (EMBL-EBI). BioMart is a data mining tool from Ensemble for exporting custom datasets. Gene annotations obtained from Ensemble should be nearly identical to those downloaded from UCSC, which serves GENCODE annotations by default. The `UCSC gene FAQ <https://genome.ucsc.edu/FAQ/FAQgenes.html#ens)>`_ discusses this point in addition to other common questions about gene prediction and annotation. 

#. `Explorer BioMart interactively <http://uswest.ensembl.org/biomart/martview>`_
#. `BioMart help docs <http://uswest.ensembl.org/info/data/biomart/index.html>`_

How can one interface with BioMart interactively?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#. `BiomaRt R package <https://bioconductor.org/packages/release/bioc/html/biomaRt.html>`_

This has a bit of a learning curve in my opinion even if one is familiar with R.

#. `BioMart Perl API <http://uswest.ensembl.org/info/data/biomart/biomart_perl_api.html>`_
#. `BioMart RESTful access <http://uswest.ensembl.org/info/data/biomart/biomart_restful.html>`_

Quick and easy. Uses unix ``wget`` command. All you need is a properly formatted XML query. Queries are easy to build using the interactive portal (see above). You can troubleshoot the query interactively and export the XML when you are happy with it.

The `"How to use BioMart" <http://uswest.ensembl.org/info/data/biomart/how_to_use_biomart.html>`_ goes has step-by-step instructions with screenshots on using the interactive interface, so I won't cover this here. Instead please see below some examples of queries for various types of genetic elements in mouse. For each one, I've tried to explain the logic.

RESTful API
~~~~~~~~~~~

Datasets
""""""""

BioMart has four "groups" of datasets at the moment:

#. Ensemble Genes

   * Genes for available vertebrate strains (not just human and mouse).
   * ``Dataset name`` (s) (see below) are fairly predictable for each genus/species: *"mmusculus_gene_ensembl"*, *"hsapiens_gene_ensembl"*, *"rnorvegicus_gene_ensembl"*.

#. Mouse strains

   * Genes for widely used strains of mice.

#. Ensemble Variation

   * SNPs and indels for available vertebrate strains.
   * ``Dataset name`` (s) (see below) are fairly predictable for each genus/species: *"hsapiens_snp"*, *"drerio_structvar"*. Non-SNP datasets are of the form: *"ecaballus_structvar"*.

#. Ensemble Regulation

   * Datasets available for human, mouse and a single one for fruit fly
   * Can query CTCF binding sites, enhancers, promoteres and transcription factor binding sites

Raw data comes from ENCODE, Roadmap Epigenomics and Blueprint. This data is cell-type specific and is mostly ChipSeq using histone methylation, CTCF, and DNaseI assays. Processing involves a segmentation step (ChromHMM) where signal patterns (presence/absence of methylation, DNaseI binding, CTCF binding etc.) are identified across the different cell types. This leads to an assignment of states (ACTIVE, INACTIVE, REPRESSED ...) to each basepair in the genome. The signal data is available under "Regulatory Evidence" datasets. 

In the end of the regulatory build workflow, a consensus is determined and a decision tree is used to assign "feature types". There is a separate transcription binding site annotation to supplement the regularoty regions.

For more information see: `The Ensemble Regulatory Build <http://uswest.ensembl.org/info/genome/funcgen/regulatory_build.html>`_

Queries
"""""""

Each query starts with a ``<Query ... >`` tag, which defines parameters for executing the query. Below are some attributes that can be used inside this tag:

#. ``formatter``: default is TSV (CSV, HTML and XLS are other options).
#. ``header``: boolean to turn on or off.
#. ``uniqueRows``: this is equivalent to ``distinct`` in SQL when you want to get rid of duplicate rows.
#. ``count``: this must simply return the row count and not actual data.

The ``<Query ...>`` tag has a nested ``<Dataset ...>`` where the name of the dataset is defined. 

``<Attribute ...>`` tags are nested within ``<Dataset ...>`` and define the columns to be retrieved from the database. A ``<Filter ...>`` (s) tag may also be specified to limite query results.

To execute a query simply submit it to the BioMart url and download the response with ``wget``.

.. code-block:: bash

    #!/bin/bash
    h_string='http://www.ensembl.org/biomart/martservice?query='
    query='<?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE Query>
    <Query  ...
    ...
    </Query>
    '
    wget -O gene_info.tsv "${h_string}${query}"

Genes
"""""

Here we are querying the "mmusculus_gene_ensembl" dataset and requesting some common information like the unique Ensemble identifier, the genomic position of the gene, it's colloquial name and description. 

Because most vertebrate genes are represented by multiple transcripts, the precise definition of a "gene" is not straightforward. For an information discussion about this see the `UCSC gene FAQ <https://genome.ucsc.edu/FAQ/FAQgenes.html#justsingle>`_. Even though for many applications it is unecessary to select a single representative gene, a unique list of genes can simplify certain analyses. 

The query below gives chromosome, start and end positions of the representative transript for each gene.

.. code-block:: xml

   <?xml version="1.0" encoding="UTF-8"?>
   <!DOCTYPE Query>
   <Query  virtualSchemaName = "default" formatter = "TSV" header = "1" uniqueRows = "0" count = "" datasetConfigVersion = "0.6" >
   	    
       <Dataset name = "mmusculus_gene_ensembl" interface = "default" >
   	    <Attribute name = "ensembl_gene_id" />
   	    <Attribute name = "external_gene_name" />
   	    <Attribute name = "gene_biotype" />
   	    <Attribute name = "strand" />
   	    <Attribute name = "chromosome_name" />
   	    <Attribute name = "start_position" />
   	    <Attribute name = "end_position" />
   	    <Attribute name = "description" />
   	    <Attribute name = "gene_biotype" />
       </Dataset>
   </Query>

Transcripts
"""""""""""

Again we are interested in mouse genes. However, now we request the "ensembl_transcript_id" and "ensembl_exon_id" in additions to the "ensembl_gene_id". This query will return many more rows compared to the previous simplified gene query, because every transcript and every exon will be returned for each gene. 

This is the "longest" version of the table, since each transcript can contain multiple exons. Notice that BioMart has figured out that we no longer want the "representative" transcript and handles all the (likely required) table joins under the hood.

In addition to ids, we also request start and stop positions for transcripts/exons and we can get the 5'-UTR and 3'-UTR locations as well. 

"Rank" refers to the ranking applied by Ensemble to determine the "representative" transcript.

The selected attributes are ones that I have found as essential, but many other are available from BioMart if one explores the interactive interface.

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE Query>
    <Query  virtualSchemaName = "default" formatter = "TSV" header = "1" uniqueRows = "0" count = "" datasetConfigVersion = "0.6" >
    	    
        <Dataset name = "mmusculus_gene_ensembl" interface = "default" >
    	<Attribute name = "ensembl_gene_id" />
    	<Attribute name = "ensembl_transcript_id" />
    	<Attribute name = "ensembl_exon_id" />
    	<Attribute name = "rank" />
    	<Attribute name = "chromosome_name" />
    	<Attribute name = "transcript_start" />
    	<Attribute name = "transcript_end" />
    	<Attribute name = "exon_chrom_start" />
    	<Attribute name = "exon_chrom_end" />
    	<Attribute name = "5_utr_start" />
    	<Attribute name = "5_utr_end" />
    	<Attribute name = "3_utr_start" />
    	<Attribute name = "3_utr_end" />
        </Dataset>
    </Query>


Phenotypes
""""""""""

This query may not be applicable to less well studied vertebrates, but can be usefull when working with human or mouse data. 

Here we request all known phenotypes for mouse genes. Multiple rows will be returned for each. Genes without known phenotypes will not be returned. 

This phenotype column can be a helpful "quick-look" at possible gene function, which searching specific protein databases
more extensively.

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE Query>
    <Query  virtualSchemaName = "default" formatter = "TSV" header = "1" uniqueRows = "0" count = "" datasetConfigVersion = "0.6" >
    	    
        <Dataset name = "mmusculus_gene_ensembl" interface = "default" >
    	    <Attribute name = "ensembl_gene_id" />
    	    <Attribute name = "external_gene_name" />
    	    <Attribute name = "phenotype_description" />
        </Dataset>
    </Query>

Regulatory features
"""""""""""""""""""

This is a simple query for regulatory elements in the mouse genome. The feature position within the genome and the feature type are requested. Additionall, the tissue type will be reported under "epigenome_name". In this example, we have chosen to filter on "regulatory_feature_type_name", since we are only interested in some regulatory features and not others.

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE Query>
    <Query  virtualSchemaName = "default" formatter = "TSV" header = "1" uniqueRows = "0" count = "" datasetConfigVersion = "0.6" >
    	    
        <Dataset name = "mmusculus_regulatory_feature" interface = "default" >
    	    <Filter name = "regulatory_feature_type_name" value = "CTCF Binding Site,Enhancer,Promoter,TF binding site"/>
    	    <Attribute name = "regulatory_stable_id" />
    	    <Attribute name = "chromosome_name" />
    	    <Attribute name = "chromosome_start" />
    	    <Attribute name = "chromosome_end" />
    	    <Attribute name = "feature_type_name" />
            <Attribute name = "epigenome_name" />
        </Dataset>
    </Query>'
