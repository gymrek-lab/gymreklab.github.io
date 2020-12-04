Obtaining gene annotations
==========================

We often need to perform analyses where we restrict to certain gene regions, such as us coding regions, UTRs, introns, etc.

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
