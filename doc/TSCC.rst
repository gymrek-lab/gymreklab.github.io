TSCC
====

Last update: 2024/01/25

Official docs
-------------
* The `tscc user guide <https://www.sdsc.edu/support/user_guides/tscc_2.html>`_
* The `tscc description <https://www.sdsc.edu/services/hpc/hpc_systems.html#tscc>`_
* The `old tscc 1.0 user guide <https://www.sdsc.edu/support/user_guides/tscc.html>`_
* THe `tscc 2.0 transitional workshow video <https://youtu.be/U_JGz-sQoV4?si=vFXfDWSIribuTLzd>`_

Logging in
----------
:code:`ssh <user>@login.tscc.sdsc.edu`

* This will put you on a node such as `login1.sdsc.edu` or `login11.sdsc.edu` or `login2.sdsc.edu`.
  You can also ssh into those nodes directly (e.g. if you have :code:`tmux` sessions saved on one of them)

The login nodes are often quite slow because there are too many users on them, and you're not supposed to run code that's
at all computationally burdensome there. So if you want to use tscc as a workstation, you should immediately try to grab an
interactive session. :code:`srun --partition=condo --pty --nodes=1 --ntasks-per-node=4 -t <n>:00:00 --wait=0 --qos=condo --export=ALL /bin/bash` where :code:`<n>` is the max
number of hours you plan to work. That will wait till it can find a slot on the condo node and then log you into
that node.

In the unlikely event that our condo node is full of work, this will hang till space opens up.

Any time your internet connection gets disrupted (depending on your settings, when your computer falls asleep) the 
interactive session will be killed along with any jobs you were running. To preserve processes
between interactive sessions, you should use :code:`tmux` or :ref:`screen <snorlax-screen>` on the login node *before* you start the interactive session.

Filesystem locations
--------------------
We have 100TB of space in :code:`/tscc/projects/ps-gymreklab`, which is where all of our files are stored. Your personal
storage directory is :code:`/tscc/projects/ps-gymreklab/<user>`. Your home directory for config and the like is
:code:`/tscc/nfs/home/<user>`, don't store any large files there, since you'll only get 100 GB there.

Communal lab resources are in :code:`/tscc/projects/ps-gymreklab/resources/`. Feel free to contribute to these as appropriate.

* :code:`/tscc/projects/ps-gymreklab/resources/source` contains downloaded software (though I (Jonathan) personally recommend
  you create a conda environment which you personally manage and ensure the stability of).
* :code:`/tscc/projects/ps-gymreklab/resources/dbase` contains reference genome builds for humans and mice and other
  non-project-specific datasets
* :code:`/tscc/projects/ps-gymreklab/resources/datasets` contains project-specific datasets that are shared across the lab.

Sharing files with Snorlax
^^^^^^^^^^^^^^^^^^^^^^^^^^

If you ssh into snorlax, you can access :code:`/tscc/projects/ps-gymreklab` (on TSCC) as :code:`/gymreklab-tscc` on Snorlax.
You cannot access Snorlax files from TSCC, so if you want to move files to/from Snorlax you'll need to be logged in to Snorlax.
There are some wonky permissions issues - if you write files into the tscc drive while on Snorlax, your user on tscc may not
be able to modify those files.

Submitting jobs
---------------
Jobs are scripts that the cluster runs for you. 

To submit a job, write a :code:`*.slurm` file and then run :code:`sbatch <file>.slurm`.
SLURM files are bash script files with SLURM specific comments at the top.
Example:

.. code-block:: bash

  #!/usr/bin/env bash
  #SBATCH --export ALL
  #SBATCH --partition <partition>
  #SBATCH --qos <partition>
  #SBATCH --job-name <job_title>
  #SBATCH --nodes 1
  #SBATCH --ntasks 1
  #SBATCH --cpus-per-task 2
  #SBATCH --time <hours>:00:00
  #SBATCH --output slurm-%j.out-%N
  #SBATCH --output slurm-%j.err-%N             # Optional, for separating standard error
  
  # ... do something ... 

Google "SLURM" to look up more information about these flags. Note that the SLURM implementation on TSCC is likely to be slightly
different than the implementation elsewhere, so online information won't be 100% accurate. In terms of naming conventions:
tscc uses the job scheduler called SLURM and `sbatch` is the name of the command to submit a job to `SLURM`.

The general workflow is to submit many jobs using the same SLURM file, each with slightly different environment variable inputs
telling them to work on different input files. See below.

Notes:

* Aside from the first shebang line, SLURM will stop looking for settings after the first line that does not start with :code:`#SBATCH`.
  This includes blank lines and lines with comments.
* Your job will be allocated memory in proportion to the number of processors your request. If you request 1 core and are assigned a node with 128GB
  of memory and 28 cores (the smaller of the two architectures of hotel nodes), your process will be allocated 128GB/28 = 4.57GB of memory and will
  be killed it if it exceeds that limit. If you want more memory, request more processors per node accordingly. (e.g. :code:`ppn=4`)
* Don't request more than one node per job. That means you would be managing inter-node inter-process communication yourself. (e.g. message 
  passing). Instead, just submit more jobs
* If :code:`<log_dir>` is mistyped, the job will not run. Double check that location before you submit.
* None of the SLURM settings can access environment variables. If you want to set a value (e.g. the log directory) dynamically, you'll
  need to dynamically generate the SLURM file.

Partitions
^^^^^^^^^^
..
  TODO: check whether we still have a home parition
We have access to two partitions: :code:`condo` and :code:`hotel`. Nodes on :code:`hotel` have a minimum of 28 cores and 4.57GB memory/core.
I do not know about the specs of the other nodes.

First consider :code:`condo`

* We have a large number of compute hours here, and they are cheap
* Jobs are limited to 8 hrs.
* The architectures of condo nodes vary wildly - if you might hit the mem/core or cores/node limit, go to hotel where (last I checked) you always get at least 4.57 GB memory/node and at least up to 28 cores/node.

If you have a single long running job, consider :code:`home`

* The node we own.
* People use this for interactive sessions, please do not take all the cores on this node for you processes.
* Jobs have no time limit.
* Jobs are guaranteed to start in 8 hours.

If you need more than 8 hours, consider :code:`hotel`:

* Compute hours are more expensive here than on :code:`condo`
* Max walltime is 1 week (168 hours)

Env Variables and Submitting Many Jobs
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
To pass an environment variable to a job, make sure the :code:`#SBATCH --export ALL` flag is set in the SLURM file or run
:code:`sbatch <file>.slurm --export "<var1>=<value1>,<var2>=<value2>,..."`. You should then be able to access those
values in the script using :code:`$var1` and so on.

Here's an example for how to submit many jobs. Suppose your current directory is::

  process-vcf.slurm
  vcfs_dir/
    vcf1.vcf.gz
    vcf2.vcf.gz
    ...

:code:`process-vcf.slurm`:

.. code-block:: bash

  #!/usr/bin/env bash
  #SBATCH other settings
  #SBATCH ...
  
  # echo the input args so you can distinguish betweeen jobs from their log files
  echo "Working on VCF $VCF" 
  >&2 echo "Working on VCF $VCF"

  # ... do something with a vcf ... 
  process $VCF

To launch the jobs::

  for vcf in vcfs_dir ; do sbatch --export "VCF=$vcf" process-vcf.slurm; done

You can also pass arguments to any :code:`.slurm` script just as you would a regular bash script. Consider the following example.

.. code-block:: bash

  #!/usr/bin/env bash
  #SBATCH other settings
  #SBATCH ...

  # copy the first argument of the script into the "VCF" variable
  VCF="$1"
  
  # echo the input args so you can distinguish betweeen jobs from their log files
  echo "Working on VCF $VCF" 
  >&2 echo "Working on VCF $VCF"

  # ... do something with a vcf ... 
  process $VCF

To launch the jobs::

  for vcf in vcfs_dir ; do sbatch process-vcf.slurm "$vcf"; done

Managing jobs
-------------
Listing current jobs: :code:`qstat -u <user>`. To look at a single job, use :code:`qstat -r <jobid>`.
To list maximum information about a job, use :code:`qstat -f -r <jobid>`

* States are Q for queued, R for running, C for cancelled, and D for done. (if I recall correctly)

If your jobs are called :code:`22409804.tscc-mgr7.local` then :code:`22409804` is the job ID.

To look at the stdout of a currently running job: :code:`qpeek <jobID>`. To look at the stderr
:code:`qpeek -e <jobID>`. Once the jobs finish the stdout and stderr will be written to the files
:code:`<log_dir>/<jobName>.o<jobID>` and :code:`<log_dir>/<jobName>.e<jobID>` respectively and 
:code:`qpeek` will no longer work.

To delete a running or queued job: :code:`qdel <jobID>`. To delete all running or queued jobs:
:code:`qstat -u <user> | cut -f1 | cut -f1 -d | xargs qdel`

To figure out why a job is queued use 'why queued?' :code:`yqd <jobid>`.

Debugging jobs the OS killed
^^^^^^^^^^^^^^^^^^^^^^^^^^^^
#. Look at the output file :code:`<log_dir>/<jobName>.o<jobID>`, the first line should contain the node
   name. (e.g. :code:`Nodes: tscc-5-7`)
#. ssh into the node (you can do this to any node, but if you run a large process the OS will kill you because
   you have not been scheduled to that node)
#. Scan the os logs for a killed process `dmesg -T | grep <jobid>`

The OS normally kills jobs because you ran over your memory limit.

Managing funds
--------------
:code:`gbalance -u <user>` will show the balance for our group, but I don't know how to see the balance on hotel vs condo,
so I'm not actually sure what this output means.
