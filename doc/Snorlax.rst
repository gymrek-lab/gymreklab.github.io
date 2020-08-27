Snorlax
=======

Snorlax is our local server, primarily used for smaller data analyses and data exploration. When you first join the lab, you'll get set up with a snorlax username.

Logging in to snorlax
---------------------

You'll need to be either on the UCSD PROTECTED network, or on VPN, to access snorlax. Then you can log in via ssh from your teminal:

.. code-block:: bash

   ssh $USER@snorlax.ucsd.edu

where $USER should be your username.

Important directories
---------------------

You'll get two personal directories on snorlax:

* :code:`/home/$USER/`: This your home directory. There is very limited space in :code:`/home/`. So please try not to actually put anything there!

* :code:`/storage/$USER/`: This the directory where you should be storing your code, data, etc. Large shared datasets will go in other directories.

The following shared directories contain lab resources:

* :code:`/storage/resources/source`: contains source code for tools installed for shared use

* :code:`/storage/resources/dbase/`: contains reference genomes for many species

* :code:`/storage/resources/datasetes/`: contains many shared datasets, such as the 1000 Genomes Project data.


Commonly used reference genomes can be found at:

* hg19: :code:`/storage/resources/dbase/human/hg19/hg19.fa`

* GRCh37: :code:`/storage/resources/dbase/human/hs37d5/hs37d5.fa`

* hg38: :code:`/storage/resources/dbase/human/GRCh38/GRCh38_full_analysis_set_plus_decoy_hla.fa`

Running Jupyter Notebook on snorlax
-----------------------------------

To run Jupyter Notebook (or other applications requiring access through the web browser) from snorlax, first when you log in you'll need to expose a port. e.g.:

.. code-block:: bash

   ssh -L 8888:localhost:8888 $USER@snorlax.ucsd.edu

Then, when you run Jupyter notebook, tell it to use that port:

.. code-block:: bash

   jupyter notebook --port 8888

Finally, navigate to your web browser: localhost:8888 to use Jupyter Notebook.

This example uses port 8888 as an example. You'll have to use a different port if that one is in use. 
