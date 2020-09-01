TSCC
====

Last update: 2020/08/31

Official docs
-------------
* The `tscc user guide <https://www.sdsc.edu/support/user_guides/tscc.html>`_
* The `tscc description <https://www.sdsc.edu/services/hpc/hpc_systems.html#tscc>`_

Logging in
----------
:code:`ssh <user>@tscc.sdsc.edu`

* This will put you on a node such as `tscc-login1.sdsc.edu` or `tscc-login11.sdsc.edu` or `tscc-login2.sdsc.edu`.
  You can also ssh into  those nodes directly (e.g. if you have :code:`tmux` sessions saved on one of them)

The login nodes are often quite slow because there are too many users on them, and you're not supposed to run code that's
at all computationally burdensome there. So if you want to use tscc as a workstation, you should immediately try to grab an
interactive session. :code:`qsub -I -l walltime=<n>:00:00 -l nodes=1:ppn=1 -q home` where :code:`<n>` is the max
number of hours you plan to work. That will wait till it can find a slot on our home node and then log you into
that node. (Note: :code:`qsub`'s user interface sucks, if you mistype that command it will hang forever without
telling you that's what happened. You should see :code:`qsub: waiting for job 22409571.tscc-mgr7.local to start`
if you typed the command correctly)

In the unlikely event that our home node is full of work, this will hang till space opens up (up to 8 hours).

Any time your internet connection gets disrupted (depending on your settings, when your computer falls asleep) the 
interactive session will be killed along with any jobs you were running. AFAIK there is no way to preserve processes
between interactive sessions (i.e. saving sessions with :code:`tmux` won't work). You can ssh directly back in to 
the node which your interactive session was on, but the scheduler will kill processes you start, so that's not a soltuion.

Filesystem locations
--------------------
We have 100TB of space in :code:`/projects/ps-gymreklab`, which is where all of our files are stored. Your personal
storage directory is :code:`/projects/ps-gymreklab/<user>`. Your home directory for config and the like is
:code:`/home/<user>`, don't store any large files there.

Communal lab resources are in :code:`/projects/ps-gymreklab/resources/`. Feel free to contribute to these as appropriate.

* :code:`/projects/ps-gymreklab/resources/source` contains downloaded software (though I (Jonathan) personally recommend
  you create a conda environment which you personally manage and ensure the stability of).
* :code:`/projects/ps-gymreklab/resources/dbase` contains reference genome builds for humans and mice and other
  non-project-specific datasets
* :code:`/projects/ps-gymreklab/resources/datasets` contains project-specific datasets that are shared across the lab.

Sharing files with Snorlax
^^^^^^^^^^^^^^^^^^^^^^^^^^

If you ssh into snorlax, you can access :code:`/projects/ps-gymreklab` (on TSCC) as :code:`/gymreklab-tscc` on Snorlax.
You cannot access Snorlax files from TSCC, so if you want to move files to/from Snorlax you'll need to be logged in to Snorlax.
There are some wonky permissions issues - if you write files into the tscc drive while on Snorlax, your user on tscc may not
be able to modify those files.

Submitting jobs
---------------
Jobs are scripts that the cluster runs for you. 

To submit a job, write a :code:`*.pbs` file and then run :code:`qsub <file>.pbs`.
PBS files are bash script files with PBS specific comments at the top.
Example:

.. code-block:: bash

  #!/bin/bash
  #PBS -q <queue>
  #PBS -N <job_title>
  #PBS -l nodes=1:ppn=1
  #PBS -l walltime=<hours>:00:00
  #PBS -o <log_dir>
  #PBS -e <log_dir>
  #PBS -V
  #PBS -M <your email>
  #PBS -m a
  
  # ... do something ... 

Google PBS to look up more information about these flags. Note that the PBS implementation on TSCC is likely to be slightly
different than the implementation elsewhere, so online information won't be 100% accurate. In terms of naming conventions:
tscc uses the job scheduler called `Torque`, `qsub` is the name of the command to submit a job to `Torque` and `PBS` is the 
file format that `qsub` expects.

The general workflow is to submit many jobs using the same PBS file, each with slightly different environment variable inputs
telling them to work on different input files. See below.

Notes:

* As above, :code:`qsub`'s user interface sucks. If you mistype the file it may hang forever without telling you. If successful
  qsub should immediately return having printed out the name of your job, e.g. :code:`22409804.tscc-mgr7.local` telling you 
  that's what happened. 
* Aside from the first shebang line, PBS will stop looking for settings after the first line that does not start with :code:`#PBS`.
  This includes blank lines and lines with comments.
* Your job will be allocated memory in proportion to the number of processors your request. If you request 1 core on a node with 64GB
  of memory and 16 cores (a hotel node), your process will be allocated 64GB/16 = 4GB of memory and will be killed it if it exceeds
  that limit. If you want more memory, request more processors per node accordingly. (e.g. :code:`ppn=4`)
* Don't request more than one node per job. That means you would be managing inter-node inter-process communication yourself. (e.g. message 
  passing). Instead, just submit more jobs
* You can have max 1500 jobs in the queues at once.
* If :code:`<log_dir>` is mistyped, the job will not run. Double check that location before you submit.
* None of the PBS settings can access environment variables. If you want to set a value (e.g. the log directory) dynamically, you'll
  need to dynamically generate the PBS file.

Queues
^^^^^^
We have access to three queues: :code:`condo`, :code:`hotel` and :code:`home`. Nodes on :code:`hotel` have 16 cores and 4GB memory/core.
I do not know about the specs of the other nodes.

First consider :code:`condo`

* We have a large number of compute hours here, and they are cheap
* Jobs are limited to 8 hrs.

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
To pass an environment variable to a job, make sure the :code:`#PBS -V` flag is set in the PBS file and run
:code:`qsub <file>.pbs -V "<var1>=<value1>,<var2>=<value2>,..."`. You should then be able to access those
values in the script using :code:`$var1` and so on.

Here's an example for how to submit many jobs. Suppose your current directory is::

  process-vcf.pbs
  vcfs_dir/
    vcf1.vcf.gz
    vcf2.vcf.gz
    ...

:code:`process-vcf.pbs`:

.. code-block:: bash

  #!/bin/bash
  #PBS -V
  #PBS other settings
  #PBS ...
  
  # echo the input args so you can distinguish betweeen jobs from their log files
  echo "Working on VCF $VCF" 
  >&2 echo "Working on VCF $VCF"

  # ... do something with a vcf ... 
  process $VCF

To launch the jobs::

  for vcf in vcfs_dir ; do qsub process-vcf.pbs -V "VCF=$vcf" ; done

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
