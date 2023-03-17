DNANexus & UK Biobank Research Analysis Platform (RAP)
======================================================

Last update: 2023/03/16

Getting access to the RAP
-------------------------

#. `Sign up here <https://bbams.ndph.ox.ac.uk/ams/resProjects>`_
   UKB takes about a week to approve your application.
#. Once that's done, ask Melissa or Jonathan to add you to our UKB application.
   That's instantaneous.
#. Then create an account on `DNANexus's website <https://ukbiobank.dnanexus.com/landing>`_
   This is instantaneous. You must use your UCSD email.
#. Then email Melissa and ask her to add you to our project (currently :code:`UKB_Test`)

Using DNANexus from the command line
------------------------------------
(There's also a GUI, docs TODO)

1. Install the DNA nexus command line tools vended through pip: :code:`pip3 install dxpy`.
2. Run :code:`dx login` and :code:`dx select <project name>`.

Choosing Instance Types
-----------------------
Instances are the virtual machines allocated for each job in the cloud.
The link `here <https://dnanexus.gitbook.io/uk-biobank-rap/working-on-the-research-analysis-platform/billing-and-costs#rates>`__
(under the words `rate card`) describes
the costs of the various instances. At the time of writing, asking for more memory per core doesn't seem to cost more 
(:code:`mem1` vs :code:`mem3`). `Here <https://documentation.dnanexus.com/developer/api/running-analyses/instance-types>`_
is an explanation of the naming conventions for instance types.

Job runtimes
------------
Jobs listed under the `Monitor` tab on the web GUI have a `Status` and a `Duration`. Note that the `Status` will be listed
as `In Progress` and the `Duration` will start counting from when the job is submitted, regardless of whether or not the other
jobs that job is dependent on are all finished. Thus this `Duration` is effectively meaningless. While it's hard to
see this number, the important information is their actual runtime. Actual runtime determines costs. If actual runtime
exceeds one day, you will receive a warning message. As far as I understand, if actual runtime for any one job exceeds two
days, the job will be killed. `Duration` exceeding two days is irrelevant.

Exporting UKB phenotypes as TSVs
--------------------------------

This code runs the 
`table exporter app <https://documentation.dnanexus.com/developer/apps/developing-spark-apps/table-exporter-application#using-the-table-exporter-app>`_
from the command line:

.. code-block:: bash

   dx run table-exporter \
     --folder <output_folder> \
     -ioutput=<output_file_name_without_extension> \
     -idataset_or_cohort_or_dashboard=/app46122_20220823045256.dataset \
     -ioutput_format=TSV \
     -icoding_option=RAW \
     -iheader_style=UKB-FORMAT \
     -ientity=participant \
     -ifield_names=eid \
     ...

You should then append an additional :code:`-ifield_names=<field_name>` for each field you want in the TSV.
The field names of UKB data fields follow the format :code:`p<numeric field ID>_i<instance>_a<array>`
Look up fields in the `UKB data showcase <https://biobank.ndph.ox.ac.uk/showcase/search.cgi>`_
to determine if any given data field has instances or arrays (if it doesn't, you need to omit
that portion of the field name format). As far as I can tell instances are zero indexed but arrays
are one indexed. You need to append :code:`-ifield_names=<field_name>` for each
instance and/or array field you wish to extract. So for example, to extract
a data field with three instances, you could append the following to the command above:

.. code-block:: bash

  $(for i in $(seq 0 2); do echo "-ifield_names=p<field ID>_i${i}" ; done)

