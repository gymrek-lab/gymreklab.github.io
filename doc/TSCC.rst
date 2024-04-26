TSCC
====

Last update: 2024/01/25

Official docs
-------------
* The `tscc user guide <https://www.sdsc.edu/support/user_guides/tscc.html>`_
* The `tscc description <https://www.sdsc.edu/services/hpc/hpc_systems.html#tscc>`_
* The `tscc 2.0 transitional workshow video <https://youtu.be/U_JGz-sQoV4?si=vFXfDWSIribuTLzd>`_

Logging in
----------
:code:`ssh <user>@login.tscc.sdsc.edu`

* This will put you on a node such as `login1.tscc.sdsc.edu` or `login11.tscc.sdsc.edu` or `login2.tscc.sdsc.edu`.
  You can also ssh into those nodes directly (e.g. if you have :code:`tmux` sessions saved on one of them)

The login nodes are often quite slow because there are too many users on them, and you're not supposed to run code that's
at all computationally burdensome there. So if you want to use tscc as a workstation, you should immediately try to grab an
interactive session. I like to add the following to my `~/.bashrc` file.

.. code-block:: bash

    qsubi(){
        # arg1: the number of hours requested (defaults to 4)
        # arg2: the partition (defaults to condo)
        srun --partition=${2:-condo} --account=ddp268 --pty --nodes=1 --ntasks 1 --cpus-per-task=4 -t ${1:-4}:00:00 --wait=0 --qos=${2:-condo} --export=ALL /bin/bash
    }

Then, when I need a interactive node, I just execute :code:`qsubi <n>` where :code:`<n>` is the max
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

If you need some extra space just for a few months, consider using your Lustre *scratch* directory (:code:`/tscc/lustre/ddn/scratch/$USER`). Files here are deleted automatically after 90 days but there is more than 2 PB available, shared over all of the users of TSCC. Otherwise, if you simply need some extra space just until your job finishes running, you can refer to :code:`/scratch/$USER/job_$SLURM_JOBID` within your jobscript. This storage will be deleted once your job dies, but it's better than Lustre scratch for I/O intensive jobs.

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
  #SBATCH --account ddp268
  #SBATCH --qos <partition>
  #SBATCH --job-name <job_title>
  #SBATCH --nodes 1
  #SBATCH --ntasks 1
  #SBATCH --cpus-per-task 2
  #SBATCH --time <hours>:00:00
  #SBATCH --output slurm-%j.out-%N
  #SBATCH --output slurm-%j.err-%N             # Optional, for separating standard error
  
  # ... do something ... 

Google "SLURM" to look up more information about these flags. In terms of naming conventions:
tscc uses the job scheduler called SLURM and `sbatch` is the name of the command to submit a job to `SLURM`.

The general workflow is to submit many jobs using the same SLURM file, each with slightly different environment variable inputs
telling them to work on different input files. See below.

Notes:

* Aside from the first shebang line, SLURM will stop looking for settings after the first line that does not start with :code:`#SBATCH`.
  This includes blank lines and lines with comments.
* The value for :code:`--account` is specific to our lab. If you aren't in our lab, you can use :code:`sacctmgr show assoc user=$USER format=account` to determine your lab's account.
* If you don't use the "--mem" option to specify how much memory you need, your job will be allocated 1 GB of memory per core.
  So, for example, if you ask for 4 CPU cores in your job but don't specify the memory, then by default you will get 4 GB of memory.
  If you want more memory, you can either request more processors (ex: :code:`--cpus-per-task 4`) or explicitly specify the memory (ex: :code:`--mem 2G`).
  Note that the lab will be charged according to both the number of processors and amount of memory that you request, so it's best to request as few of both resources as you need.
  For more details about job charging, refer to the `TSCC website <https://www.sdsc.edu/support/user_guides/tscc.html#condo_job_charging>`__.
* Don't request more than one node per job. That means you would be managing inter-node inter-process communication yourself. (e.g. message 
  passing). Instead, just submit more jobs
* If :code:`<log_dir>` is mistyped, the job will not run. Double check that location before you submit.
* None of the SLURM settings can access environment variables. If you want to set a value (e.g. the log directory) dynamically, you'll
  need to dynamically generate the SLURM file.

Partitions
^^^^^^^^^^
..
  TODO: check whether we still have a home parition

We have access to two partitions: :code:`condo` and :code:`hotel`. There are two types of hotel nodes: (1) 36 cores, 192 GB of memory; (2) 28 cores, 128 GB of memory. Nodes on :code:`condo` have varying specifications.

Note: TSCC 1.0 had a :code:`home` partition that was accessible by only members of our lab. On TSCC 2.0, this has been removed. You should use :code:`condo` instead.

First consider :code:`condo`

* We have a large number of compute hours here, and they are cheap
* Jobs may be `preempted <https://slurm.schedmd.com/preempt.html>`_ after 8 hrs but can run for up to 14 days
* The architectures of condo nodes vary wildly - if you might hit the mem/core or cores/node limit, go to hotel where (last I checked) you always get at least 4.57 GB memory/node and at least up to 28 cores/node.

If you need more than 8 hours, consider :code:`hotel`:

* Compute hours are more expensive here than on :code:`condo`
* Max walltime is 7 days (168 hours)
* If your job(s) need many processors or a lot of memory on :code:`hotel`, please send a message in the :code:`#computing` channel of our Slack to give everyone a heads up. At any given time, members of our lab cannot **collectively** use more than 36 processors and 192 GB of memory on :code:`hotel`. To check whether these limits have changed, you can run the following.

.. code-block:: bash

    sacctmgr show qos format=Name%20,priority,gracetime,PreemptExemptTime,maxwall,MaxTRES%30,GrpTRES%30 where qos=hcg-ddp268

So if you start a 36-core / 192GB memory job (or multiple jobs that use either a total of 36 cores OR a total of 192GB memory), then everyone else in our lab who submits to the :code:`hotel` partition will see their jobs wait in the queue until yours are finished. These limits are set according to the number of nodes that our lab has contributed to the :code:`hotel` partition. Jobs submitted to the :code:`condo` partition are not subject to this group limit.

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
Listing current jobs: :code:`squeue -u <user>`. To look at a single job, use :code:`squeue -j <jobid>`.
To list maximum information about a job, use :code:`squeue -l -j <jobid>`

* States are Q for queued, R for running, C for cancelled, and D for done. (if I recall correctly)

If your jobs are called :code:`22409804.tscc-mgr7.local` then :code:`22409804` is the job ID.

To look at the stdout of a currently running job: :code:`qpeek <jobID>`. To look at the stderr
:code:`qpeek -e <jobID>`. Once the jobs finish the stdout and stderr will be written to the files
:code:`<log_dir>/<jobName>.o<jobID>` and :code:`<log_dir>/<jobName>.e<jobID>` respectively and 
:code:`qpeek` will no longer work.

To delete a running or queued job: :code:`scancel <jobID>`. To delete all running or queued jobs:
:code:`scancel -u $USER`

To figure out why a job is queued use :code:`scontrol show job <your_job_number>`

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

Using snakemake
---------------
To integrate Snakemake with SLURM, you must first install the SLURM snakemake executor along with Snakemake. Create a new environment with both packages:

.. code-block:: bash
  conda create -y -n snakemake -c conda-forge -c bioconda snakemake-executor-plugin-slurm 'snakemake>=8'
  conda activate snakemake

When structuring your Snakemake project, please consider using `the official recommended directory structure <https://snakemake.readthedocs.io/en/stable/snakefiles/deployment.html#distribution-and-reproducibility>`_ and `template <https://github.com/snakemake-workflows/snakemake-workflow-template>`_.

Within the top level directory of the project (where the :code:`config/` and :code:`workflow/` directories are located), I recommend creating a :code:`profile/` directory. Inside that folder, create another directory called :code:`slurm` and a file named :code:`config.yaml`.
You can store default arguments/options to :code:`snakemake` in the :code:`config.yaml` file. For SLURM, I suggest including the following lines:

.. code-block::
  jobs: 32
  cores: 32
  use-conda: true
  latency-wait: 60
  keep-going: true
  conda-frontend: conda

  executor: slurm
  default-resources:
    slurm_account: ddp268
    slurm_partition: condo
    runtime: 30
    nodes: 1
    slurm_extra: "'--qos=condo'"

This will configure Snakemake to automatically submit the steps of your workflow as SLURM jobs. It will ensure that at most 32 jobs are running simultaneously and at most 32 CPUs are in use simultaneously. You can increase these values if you'd like, but please be mindful of requesting too many resources at once so that you're not impacting the work of others in our lab.

By default, this configuration will submit jobs to the :code:`condo` queue and allocate 30 minutes for each job. But you can override any of the values in the :code:`default-resources` section on a per-rule basis by specifying them as resources in the `resources directive <https://snakemake.readthedocs.io/en/stable/snakefiles/rules.html#resources>`_ of a rule.

Please note that if you try to run Snakemake from a login node, it will simply hang indefinitely. For this reason, I recommend creating a :code:`.slurm` batch script for running Snakemake according to the instructions above. You can also run it from an interactive node.
