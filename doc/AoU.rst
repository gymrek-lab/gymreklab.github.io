All of Us
=========

Getting access
--------------
Please note that you need to complete many trainings before you can work with any of the data. If your project will work with genotype or phenotype data, you will need access to the Controlled Tier and will need to complete the trainings for that too.

Creating a new workspace
------------------------

1. First, make a copy of `this document <https://docs.google.com/document/d/1RfQB2rTLQh6cyRs8mSMUVDMUC9ttn-WVXj647HDRgxo/edit?tab=t.0#bookmark=id.2nesmcih5hz4>`_ and fill out the workspace name, summary, and description.
    If your project works with genotype or phenotype data (or anything in the Controlled Tier, including All by All tables), you should also fill out `the section "Getting access to the CDR data" <https://docs.google.com/document/d/1RfQB2rTLQh6cyRs8mSMUVDMUC9ttn-WVXj647HDRgxo/edit?tab=t.0#bookmark=id.6ahr7fb1hf9>`_. Otherwise, please delete this section.
2. Once you've completed the document, send it to Melissa and Yang for approval. They may make suggestions. Yang will create a workspace for you.
3. Find your workspace in `Verily <https://workbench.verily.com>`_. Go to the Resources tab, then click "+ Data from catalog". Select the data that you need: either the registered tier or controlled tier. Select the version of the data that you'd like to use (or just the most recent version if you're not sure) and the type of data resource (probably at least :code:`vwb-aou-datasets-controlled`). Then click "Next" and fill out the rest of the questions with your answers from the Google Doc.
4. You can now link the workspace to existing storage buckets or create new ones. Step 3 will add special permissions to your workspace allowing you to link buckets from other workspaces with similar permissions.
    For example, you can link to the CAST v8 workspace bucket by navigating to the Resources tab, clicking "+ New resource", and then "Reference Cloud Storage bucket".

    .. figure:: ../images/AoULinkingStorageBucketExample.png
        :alt: Example of linking to an existing storage bucket
        :align: center
        :width: 400px
