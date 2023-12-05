Expanse
=======

Last update: 2023/01/10

Expanse uses SLURM to schedule and run jobs

`Expanse user guide <https://www.sdsc.edu/support/user_guides/expanse.html>`_

Getting an account, logging in and setting up
---------------------------------------------

Ask Melissa for her permission. Then email support@sdsc.edu asking for an account that's part of the
gymreklab-group (and also part of the gymreklab-ukb group if applicable and you've already been granted
access to the UKB data via UKB's access management system). CC Melissa on that email.

Then ssh into :code:`login.expanse.sdsc.edu` . That'll send you to either the node login01 or login02. If you want
to make sure you get back to the same login node (because you've got some process there that
persists after you disconnect), then ssh into :code:`login0X.expanse.sdsc.edu`

Your folder will be at :code:`/expanse/projects/gymreklab/<username>` and shared data
is at :code:`/expanse/projects/gymreklab/resources/datasets/`

You can use :code:`module load` to load preconfigured files. I prefer to install everything myself.
I use miniconda to do this when possible. Miniconda can be installed into your home directory.

Running jobs
------------

Only use the login nodes for small tasks.
Any large tasks should be done in a job, either interactively or noninteractively.
To encourage this, Expanse staff has disabled some array/plotting libraries 
on the login nodes, which will cause weird error messages if you try to use them there.

First: :code:`module load slurm`. I like to put this in my `.bashrc`

.. _getting_an_interactive_node_on_expanse:

Grabbing an interactive node
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

   srun --partition=ind-shared --pty --nodes=1 --ntasks-per-node=1 --mem=50G -t 24:00:00 --wait=0 --account=ddp268 /bin/bash

``--pty`` is what specifically makes this treated as an interactive session

Running a script noninteractively with SLURM
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

First add special :code:`SBATCH` flags to the script

.. code-block:: bash

  #!/usr/bin/env bash
  #SBATCH --partition=ind-shared
  #SBATCH --account=ddp268
  #SBATCH --job-name=<...>
  #SBATCH --nodes=1
  #SBATCH --tasks-per-node=1
  #SBATCH --time=<...>
  #SBATCH --output=<...>/logs/%x_%j.out
  #SBATCH --mail-type=FAIL
  #SBATCH --mail-user=<...>

  # an example input variable
  if [ -z "$TARGET" ] ; then echo "TARGET variable is unset" ; exit 1 ; fi
  # write script inputs to log files
  echo "TARGET $TARGET"
  echo "TARGET $TARGET" >&2

  # your script which does stuff with $TARGET
  ...

then run

.. code-block:: bash

   sbatch --export all script.sh

where :code:`--export all` will pass all current environment variables to the running script, so you can use e.g. :code:`$TARGET` in the script.

To submit many jobs, either use a workflow language (e.g. WDL), or I submit them in a for loop with different environment variables.
I don't use SLURM's batch system, but you could look into that. You could also grab more than one node at once (not the same
as multiple processors on the same node) and then do inter-node submissions with :code:`srun`, but I feel that's more complicated
than necessary.

Some notes:

* The output flag determines the file that stdout is written to. This must be a file, not a directory.
  You can use some placeholders in the output location such as `%x` for job name and `%j` for job id.
* Use the error flag to choose stderr's output location, if not specified goes to the output location.
* There may be an optional shebang line at the start of the file, but no blank or other lines
  between the beginning  and the :code:`#SBATCH` lines
* SLURM does not support using environment variables in :code:`#SBATCH` lines in scripts. If you wish to use
  environment variables to set such values, you must pass them to the :code:`sbatch` command directly
  (e.g. :code:`sbatch --output=$SOMEWHERE/out slurm_script.sh`) 

.. _increasing_job_runtime_up_to_one_week:

Increasing job runtime up to one week
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You should be able to submit jobs/grab interactive nodes for up to two days. If you want to be able to do so for up to a week,
* first ask the Expanse team for :code:`ind-shared-oneweek` permissions
* then add :code:`--qos ind-shared-oneweek` to your interactive node/noninteractive job submission and increase the time you're requesting for that node/job.

Managing noninteractive jobs
----------------------------

* :code:`squeue -u <user>` - look at your jobs
* :code:`-p <partition>` - look at a specific partition
* :code:`-j <joblist>` - look for named jobs instead
* :code:`-t <statelist>` 
* :code:`scancel <jobid>`

Use :code:`seff` to figure out job run statistics. Use :code:`sacct` to look at job statistics.

Looking at account balances:

.. code-block:: bash

  module load sdsc
  expanse-client user -p

.. _Using_Singularity_to_run_Docker_containers:

Using Singularity to run Docker containers
------------------------------------------
Docker is insecure (it requires root access) and so isn't compatible
with cluster computing where multiple scheduled process from different
users share a single node. Instead, Singularity is used to run Docker
containers in a secure manner on cluster computers.

Terminology:

* `SingularityCE <https://docs.sylabs.io/guides/3.10/user-guide/index.html>`_ is open source
* Sylabs is the company that owns SingularityPro which is just
  a supported version of singularity

To make singularity work, I add the following to my :code:`.bashrc`:

.. code-block:: bash

  module load singularitypro
  export SINGULARITY_CACHEDIR=/expanse/projects/gymreklab/<username>/.singularity_cache`
  if [[ -n "$SLURM_JOB_ID" ]] ; then
    export SINGULARITY_TMPDIR="/scratch/$USER/job_$SLURM_JOB_ID"
  fi

The general idea is, first grab an interactive node (or put this in a script that you submit) and then:

.. code-block:: bash

  singularity exec --containall docker://<docker_image_url> <command>

You'll notice the first time you run a new docker image Singularity takes a while (~10min) building
it into a singularity image. They are cached at :code:`$SINGUALRITY_CACHEDIR` if that's set
or :code:`~/.singularity/cache` otherwise. For Expanse, IIRC the home directory is
slower than the project folder so I set :code:`$SINGUALRITY_CACHEDIR` to somewhere
in my project space

Any calls to :code:`singularity exec|shell|pull` will cache the image. I wouldn't
trust that the cache is thread-safe, so if you're going to kick off a bunch
of jobs, either cache the image before hand, or have them all check.

To cache the image beforehand:

.. code-block:: bash
   
   singularity exec docker://<docker_image_url> /bin/bash -c "echo pulled the image"

or, to check in a synchronized manner:

.. code-block:: bash
  
   if [ -z $SINGULARITY_CACHEDIR ];
     then CACHE_DIR=$HOME/.singularity/cache
     else CACHE_DIR=$SINGULARITY_CACHEDIR
   fi
   mkdir -p $CACHE_DIR
   LOCK_FILE=$CACHE_DIR/singularity_pull_flock
   flock --verbose --exclusive --timeout 900 $LOCK_FILE \
   SINGULARITY_TMPDIR=/scratch/$USER/job_$SLURM_JOB_ID singularity exec --containall docker://<docker_image_url> echo "successfully pulled image"


Singularity run tips
^^^^^^^^^^^^^^^^^^^^

* To run a shell interactively in a container:
  :code:`singularity shell --containall docker://<docker_image_url>`
  or :code:`singularity exec --containall docker://<docker_image_url> /bin/bash -l`
  (this starts bash in login mode, see below)
* To run a command:
  :code:`singularity exec --containall docker://<docker_image_url> <command>`
* To run a shell script:
  :code:`singularity exec --containall docker://<docker_image_url> /bin/bash -c "<script>"`
* Use :code:`--containall` to not bring in any information from the outside
  environment into the container (e.g. unwanted mount points like `$HOME`,
  env variables, etc.) This makes runs actually reproducible.
* Use :code:`--bind <outsider_location>:<inside_location>` to mount files/directories.
  Add this flag multiple times to mount multiple files/directories
* Use :code:`--env VAR=value` to pass environment variables to the run
* Note: :code:`singularity run` instead of :code:`singularity exec` to run the default
  command of the container instead of the command you've specified.
  This is the same as the difference between run and exec in Docker.
* To run singularity with a docker image that's been saved as an archive (:code:`.tar`), just use
  :code:`singularity exec|shell|run docker-archive://<path_to_archive>`

Singularity build tips
^^^^^^^^^^^^^^^^^^^^^^

* Singularity may not respect anything Docker installs in the user's home. So to install
  your own software manually, you need to put it somewhere else. I'm not sure what the
  cleanest solution is, but I put it in :code:`/container_install` and modified the path
  accordingly.


