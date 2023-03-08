Snorlax
=======

Snorlax is our local server, primarily used for smaller data analyses and data exploration. When you first join the lab, you'll get set up with a snorlax username.

Logging in to snorlax
---------------------

You'll need to be either on the UCSD PROTECTED network, or on VPN, to access snorlax. Then you can log in via ssh from your teminal:

.. code-block:: bash

   ssh $USER@snorlax.ucsd.edu

where $USER should be your username.

Some basic info is provided below. For more detailed info on snorlax, see `snorlax documentation <https://docs.google.com/document/d/1lVrtAVwVkbA9RSY97iB39fzNPEXA6rkOZjtiYU6JfRQ/edit?usp=sharing>`_.

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

Finally, navigate to your web browser: localhost:8888 to use Jupyter Notebook. It may ask you for a "token". When you ran Jupyter notebook from the terminal, it should print the URL including the token to the terminal screen. That URL should start with "localhost:8888" or whatever port you used.

This example uses port 8888 as an example. You'll have to use a different port if that one is in use. If you attempt to run Jupyter notebook with a port that's already in use, Jupyter will often try to choose another port number. e.g. if you chose 8888 but that's in use, it might use 8889. If that happens, you'll get an error message when you try to access the URL on your web browser since that will not be the port you exposed with the :code:`-L` option to your ssh command. You'll have to choose a new port, log out of snorlax, and log back in with the correct port.

You can try ports in the range 1024+. 5000 is typically in use for WebSTR development. Other ports may be reserved for use by other programs or used by labmates. If you know how to check which ports are available, feel free to submit a PR and update this page :). (see https://gymreklabgithubio.readthedocs.io/en/latest/LabWebsite.html).

Use ``screen`` to keep things running even with spotty internet
---------------------------------------------------------------
If your ssh connection dies because of a choppy internet connection, your Jupyter instance will die, too. You can run jupyter from within a ``screen`` session to make it persist. Let's adapt the instructions from above.

1. First, log into snorlax

   .. code-block:: bash

      ssh -L 8888:localhost:8888 $USER@snorlax.ucsd.edu

2. Next, create and enter a ``screen`` session named **jupyter**

   .. code-block:: bash

      screen -S jupyter

3. View a list of running screen sessions

   .. code-block:: bash

      screen -ls

4. Start jupyter on the cluster

   .. code-block:: bash

      jupyter notebook --port 8888

5. In a browser running on your own computer, go to http://localhost:8888
6. If your ssh connection dies, everything in your ``screen`` session will continue running. Just log back in and reattach the session

   .. code-block:: bash

      screen -r jupyter

7. You can also terminate a session while inside of it

   .. code-block:: bash

      exit

   Or, if you’d like to detach from a session (instead of terminating it) while you’re inside of it, press ctrl+a and then the letter d
