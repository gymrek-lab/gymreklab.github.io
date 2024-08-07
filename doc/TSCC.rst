TSCC
====

Last update: 2024/05/13

Official docs
-------------
* The `tscc user guide <https://www.sdsc.edu/support/user_guides/tscc.html>`_
* The `tscc description <https://www.sdsc.edu/services/hpc/hpc_systems.html#tscc>`_
* The `tscc 2.0 transitional workshow video <https://youtu.be/U_JGz-sQoV4?si=vFXfDWSIribuTLzd>`_

Getting access
--------------
Email tscc-support AT sdsc DOT edu from your UCSD email and CC Melissa. You can include the following in your email.

  Hello TSCC Support,

  I'm a new member of the Gymrek lab. Is there any chance that you can create a TSCC account for me and add me to the Gymrek Lab group (gymreklab-group:\*:11136)?

Logging in
----------
.. code-block:: bash

  ssh <user>@login.tscc.sdsc.edu

* This will put you on a node such as `login1.tscc.sdsc.edu` or `login11.tscc.sdsc.edu` or `login2.tscc.sdsc.edu`.
  You can also ssh into those nodes directly (e.g. if you have :code:`tmux` sessions saved on one of them)

* To configure ssh for expedited access, consider following the directions under the section *Linux or Mac* on `the TSCC user guide <https://www.sdsc.edu/support/user_guides/tscc.html#Log_in>`_ to add an entry to your :code:`~/.ssh/config`. Here's an example. Remember to replace :code:`YOUR_USERNAME_GOES_HERE`! Afterwards, you should be able to log in with a simple: :code:`ssh tscc` command.

.. code-block:: text

  Host *
      ControlMaster auto
      ControlPath ~/.ssh/ssh_mux_%h_%p_%r
      ControlPersist 1
      ServerAliveInterval 100
  
  Host tscc
      HostName login1.tscc.sdsc.edu
      ForwardX11 yes
      User YOUR_USERNAME_GOES_HERE


* If you are running Windows, you can use the `Windows Subsystem for Linux <https://learn.microsoft.com/en-us/windows/wsl/install#install-wsl-command>`_ to acquire a Linux terminal with SSH

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
storage directory is :code:`/tscc/projects/ps-gymreklab/<user>`. (If this directory doesn't yet exist, feel free to create it with the :code:`mkdir` command.)

You can check the available storage in the shared mount with the following command.

.. code-block:: bash

    df -h | grep -E '^Filesystem|gymreklab' | column -t

Your home directory for config and the like is :code:`/tscc/nfs/home/<user>`, but don't store any large files there, since you'll only get 100 GB there.

If you need some extra space just for a few months, consider using your personal Lustre *scratch* directory (:code:`/tscc/lustre/ddn/scratch/$USER`). Files here are deleted automatically after 90 days but there is more than 2 PB available, shared over all of the users of TSCC. Otherwise, if you simply need some extra space just until your job finishes running, you can refer to :code:`/scratch/$USER/job_$SLURM_JOBID` within your jobscript. This storage will be deleted once your job dies, but it's better than Lustre scratch for I/O intensive jobs.

Communal lab resources are in :code:`/tscc/projects/ps-gymreklab/resources/`. Feel free to contribute to these as appropriate.

* :code:`/tscc/projects/ps-gymreklab/resources/source` contains downloaded software (though I (Jonathan) personally recommend
  you create a conda environment which you personally manage and ensure the stability of).
* :code:`/tscc/projects/ps-gymreklab/resources/dbase` contains reference genome builds for humans and mice and other
  non-project-specific datasets
* :code:`/tscc/projects/ps-gymreklab/resources/datasets` contains project-specific datasets that are shared across the lab.
* :code:`/tscc/projects/ps-gymreklab/resources/datasets/ukbiobank` contains our local copy of the UK Biobank. You must have the proper Unix permissions to read these files. First, create an account `here <https://ams.ukbiobank.ac.uk/ams>`_ and then once that's approved, ask Melissa to add you on the UK Biobank portal and the :code:`gymreklab-ukb` Unix group.
* :code:`/tscc/projects/ps-gymreklab/resources/datasets/1000Genomes` contains files for the 1000 Genomes dataset
* :code:`/tscc/projects/ps-gymreklab/resources/datasets/gtex` contains the GTEX dataset
* :code:`/tscc/projects/ps-gymreklab/resources/datasets/pangenome` contains pangenome files

Access TSCC files locally on your computer
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
You can upload and download files from TSCC using the `scp` command. Assuming you've configured a host in your `~/.ssh/config` named `tscc`, you would download chrM of the hg19 reference genome like this, for example.

.. code-block:: bash

    scp -r tscc:/tscc/projects/ps-gymreklab/resources/dbase/human_by_chrom/hg19/chrM.fa .

However, if you would like to download many files from TSCC or edit files on TSCC in real time, you may opt to mount TSCC as a network drive, instead. A program called `sshfs` will allow you to view and edit TSCC files on your computer and keep them synced with TSCC.

To set up :code:`sshfs`, you must first download and install it. With `homebrew <https://docs.brew.sh/Installation>`_ on MacOS, you can do :code:`brew install sshfs` or on Ubuntu or `Ubuntu <https://apps.microsoft.com/detail/9pdxgncfsczv>`_ on `Windows Subsystem for Linux <https://learn.microsoft.com/en-us/windows/wsl/install#install-wsl-command>`_, you can just do :code:`sudo apt install sshfs`. Next, simply add the following snippet to your :code:`~/.bashrc`:

.. code-block:: bash

    # mount a remote drive over ssh
    # arg 1: the hostname of the server, as specified in your ssh config
    # arg 2 (optional): the mount directory; defaults to arg1 in the current directory
    sshopen() {
        # perform validation checks, first
        command -v sshfs >/dev/null 2>&1 || { echo >&2 "error: sshfs is not installed"; return 1; }
        grep -q '^user_allow_other' /etc/fuse.conf || { echo >&2 "error: please uncomment the 'user_allow_other' option in /etc/fuse.conf"; return 1; }
        ssh -q "$1" exit >/dev/null || { echo >&2 "error: cannot connect to '$1' via ssh; check that '$1' is in your ~/.ssh/config"; return 1; }
        [ -d "${2:-$1}" ] && { ls -1qA "${2:-$1}" | grep -q .; } >/dev/null 2>&1 && { echo >&2 "error: '${2:-$1}' is not an empty directory; is it already mounted?"; return 1; }
        # set up a trap to exit the mount before attempting to create it
        trap "cd \"$PWD\" && { fusermount -u \"${2:-$1}\"; rmdir \"${2:-$1}\"; }" EXIT && mkdir -p "${2:-$1}" && {
            # ServerAlive settings prevent the ssh connection from dying unexpectedly
            # cache_timeout controls the number of seconds before sshfs retrieves new files from the server
            sshfs -o allow_other,reconnect,ServerAliveInterval=15,ServerAliveCountMax=3,cache_timeout=900,follow_symlinks "$1": "${2:-$1}"
        } || {
            # if the sshfs command didn't work, store the exit code, clean up the dir and the trap, and then return the exit code
            local exit_code=$?
            rmdir "${2:-$1}" && trap - EXIT
            return $exit_code
        }
    }

After sourcing your :code:`~/.bashrc` you should now be able to run :code:`sshopen tscc`! This will create a folder in your working directory with all of your files from TSCC. The network mount will be automatically disconnected when you close your terminal.

Some notes on usage:

* Depending on your network connection, :code:`sshopen` might choke on large files. Consider using :code:`scp` for such files, instead.
* In order to reduce network usage, sshopen will only retrieve new files from the server every 15 minutes. If you want this to happen more frequently, just change the :code:`cache_timeout` setting in the sshfs command.
* The unmount will fail if any processes are still utilizing files in the mount, so you should close your File Explorer or any other applications before you close your terminal window. If the unmount fails, you can always unmount manually: :code:`pkill sshfs && rmdir tscc` will kill the :code:`sshfs` command and delete the mounted folder.

Syncing TSCC files with Google Drive or OneDrive
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Ever wanted to share your plots with a collaborator or your PI? But you have too many and they're updated too often to use :code:`scp` to download and reupload each time?

Consider using :code:`rclone` to automatically sync your files with a cloud storage provider! You can install :code:`rclone` `using conda <https://anaconda.org/conda-forge/rclone>`_ and then configure it according to the instructions for `Google Drive <https://rclone.org/drive>`_ or for `OneDrive <https://rclone.org/onedrive>`_.

When configuring :code:`rclone`, you should answer **No** to the question *Use web browser to automatically authenticate rclone with remote?*. You can instead follow their directions to install :code:`rclone` on your laptop or personal computer to get the appropriate token. Or, if that doesn't work, you can try using the (less secure) `SSH tunneling approach <https://rclone.org/remote_setup/#configuring-using-ssh-tunnel>`_.

Sharing files with Snorlax
^^^^^^^^^^^^^^^^^^^^^^^^^^

If you ssh into snorlax, you can access :code:`/tscc/projects/ps-gymreklab` (on TSCC) as :code:`/gymreklab-tscc` on Snorlax.
You cannot access Snorlax files from TSCC, so if you want to move files to/from Snorlax you'll need to be logged in to Snorlax.
There are some wonky permissions issues - if you write files into the tscc drive while on Snorlax, your user on tscc may not
be able to modify those files.

.. _tscc-submitting-jobs:

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
TSCC uses the job scheduler called SLURM and `sbatch` is the name of the command to submit a job to `SLURM`

The general workflow is to submit many jobs using the same SLURM file, each with slightly different environment variable inputs
telling them to work on different input files. See below.

Notes:

* Aside from the first shebang line, SLURM will stop looking for settings after the first line that does not start with :code:`#SBATCH`.
  This includes blank lines and lines with comments.
* The value for :code:`--account` is specific to our lab. If you aren't in our lab, you can use :code:`sacctmgr show assoc user=$USER format=account` to determine your lab's account.
* If you don't use the :code:`--mem` option to specify how much memory you need, your job will be allocated 1 GB of memory per core.
  So, for example, if you ask for 4 CPU cores in your job but don't specify the memory, then by default you will get 4 GB of memory.
  If you want more memory, you can either request more processors (ex: :code:`--cpus-per-task 4`) or explicitly specify the memory (ex: :code:`--mem 2G`).
  Note that the lab will be charged according to both the number of processors and amount of memory that you request, so it's best to request as few of both resources as you need.
  For more details about job charging, refer to the `TSCC website <https://www.sdsc.edu/support/user_guides/tscc.html#condo_job_charging>`__.
* Don't request more than one node per job. That means you would be managing inter-node inter-process communication yourself. (e.g. message 
  passing). Instead, just submit more jobs
* If :code:`<log_dir>` is mistyped, the job will not run. Double check that location before you submit.
* There may be an optional shebang line at the start of the file, but no blank or other lines between the beginning and the :code:`#SBATCH` lines
* None of the SLURM settings can access environment variables. If you want to set a value (e.g. the log directory) dynamically, you'll
  need to dynamically generate the SLURM file.
* SLURM does not support using environment variables in :code:`#SBATCH` lines in scripts. If you wish to use
  environment variables to set such values, you must pass them to the :code:`sbatch` command directly
  (e.g. :code:`sbatch --output=$SOMEWHERE/out slurm_script.sh`) 

Partitions
^^^^^^^^^^
We have access to two partitions: :code:`condo` and :code:`hotel`. There are two types of hotel nodes: (1) 36 cores, 192 GB of memory; (2) 28 cores, 128 GB of memory. Nodes on :code:`condo` have varying specifications.

Note: TSCC 1.0 had a :code:`home` partition that was accessible by only members of our lab. On TSCC 2.0, this has been removed. You should use :code:`condo` instead.

First consider :code:`condo`

* We have a large number of compute hours here, and they are cheap
* Jobs may be `preempted <https://slurm.schedmd.com/preempt.html>`_ after 8 hrs but can run for up to 14 days
* The architectures of condo nodes vary wildly - if you might hit the mem/core or cores/node limit, go to hotel where (last I checked) you always get at least 4.57 GB memory/node and at least up to 28 cores/node.

.. warning::
  As of the migration to TSCC 2.0 (in Jan 2024), our lab no longer has a hotel allocation!
  But we will continue to include the :code:`hotel` documentation below in case we ever obtain an allocation again.

If you need more than 8 hours, consider :code:`hotel`:

* Compute hours are more expensive here than on :code:`condo`
* Max walltime is 7 days (168 hours)
* If your job(s) need many processors or a lot of memory on :code:`hotel`, please send a message in the :code:`#computing` channel of our Slack to give everyone a heads up. At any given time, members of our lab cannot **collectively** use more than 36 processors and 192 GB of memory on :code:`hotel`. To check whether these limits have changed, you can run the following.

.. code-block:: bash

    sacctmgr show qos format=Name%20,priority,gracetime,PreemptExemptTime,maxwall,MaxTRES%30,GrpTRES%30 where qos=hcg-ddp268

So if you start a 36-core / 192GB memory job (or multiple jobs that use either a total of 36 cores OR a total of 192GB memory), then everyone else in our lab who submits to the :code:`hotel` partition will see their jobs wait in the queue until yours are finished. These limits are set according to the number of nodes that our lab has contributed to the :code:`hotel` partition. Jobs submitted to the :code:`condo` partition are not subject to this group limit. For more information about account limits, including info about viewing your account usage, read `the section of the TSCC docs titled "Managing Your User Account" <https://sdsc.edu/support/user_guides/tscc.html#tscc_client>`_. For example, you can get a lot of information by using the `tscc_client`:

.. code-block:: bash

    module load sdsc
    tscc_client -A ddp268

Env Variables and Submitting Many Jobs
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
To pass an environment variable to a job, make sure the :code:`#SBATCH --export ALL` flag is set in the SLURM file or run

.. code-block:: bash

    sbatch <file>.slurm --export "<var1>=<value1>,<var2>=<value2>,..."

You should then be able to access those values in the script using :code:`$var1` and so on.

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

The output flag determines the file that stdout is written to. This must be a file, not a directory.
You can use some placeholders in the output location such as `%x` for job name and `%j` for job id.

Use the error flag to choose stderr's output location. If not specifie, it will go to the output location.

To delete a running or queued job: :code:`scancel <jobID>`. To delete all running or queued jobs:
:code:`scancel -u $USER`

To figure out why a job is queued use :code:`scontrol show job <your_job_number>`

Debugging jobs the OS killed
^^^^^^^^^^^^^^^^^^^^^^^^^^^^
#. Look at the standard output and standard error files. Any error messages should be there.
#. ssh into the node. You can do this to any node, but if you run a large process the OS will kill you because you have not been scheduled to that node. You can figure out the name of the node assigned to your job using :code:`squeue` once the status of the job is "RUNNING".
#. Scan the os logs for a killed process :code:`dmesg -T | grep <jobid>`
#. If there are any messages stating that your job was "Killed", its usually a sign that you ran out of memory. You can request more memory by resubmitting the job with the :code:`--mem` parameter. For ex: :code:`--mem 8G`

Get Slack notifications when your jobs finish
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
1. Add `Slack's Incoming Webhooks App <https://slack.com/apps/A0F7XDUAZ-incoming-webhooks>`_ to your workspace and during the set up, make the app post to your own personal channel (ex: :code:`@arya`).
2. Once you've added the app, make note of the webhook URL it provides.
3. Execute the following command to define a command named :code:`slack` in your :code:`~/.bashrc` file, making sure to replace :code:`WEBHOOK_URL` with the webhook URL from step 2.

  .. code-block:: bash

    echo 'slack(){ curl -X POST --data-urlencode "payload={\"text\": \"$1\"}" WEBHOOK_URL; } && export -f slack' >> ~/.bashrc

4. Close and re-open your terminal / ssh connection or run :code:`source ~/.bashrc`. You should now be able to send yourself a Slack message by typing :code:`slack 'hello world'`
5. Create your job script and make sure to specify :code:`#SBATCH --export ALL` at the top. At the end of your job script, add something like the following.

  .. code-block:: bash

    slack "your job terminated with exit status $?"

Installing software
-------------------
The best practice is for each user of TSCC to use conda to install their own software. Run these commands to download, install, and configure conda properly on TSCC:

.. code-block:: bash

  wget https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-x86_64.sh
  bash Miniforge3-Linux-x86_64.sh -b -u
  rm Miniforge3-Linux-x86_64.sh
  source ~/miniforge3/bin/activate
  conda init bash
  conda config --add channels nodefaults
  conda config --add channels bioconda
  conda config --add channels conda-forge
  conda config --set channel_priority strict
  conda update -y --all

.. note::
    Make sure to never install software with conda on a login node! It will take a long time and slow down the login node for other TSCC users.

If you are feeling lazy, you can also use the :code:`module` system to load preconfigured software tools.
Refer to `the TSCC documentation <https://www.sdsc.edu/support/user_guides/tscc.html#env_modules>`_ for more information.

.. warning::
  Software available through the module system is usually out of date and cannot be easily updated.
  It's also unlikely that collaborators/reviewers will be able to run your code once you're ready to share it with them, since,
  unlike with conda, the module system doesn't offer a way to share your software environment with non-TSCC users.
  For these reasons, we do not recommend using the :code:`module` system.

Using containers
----------------
You can also load software via containers. Unfortunately, Docker is not available on TSCC and cannot be installed. Instead, you can use singularity (which was recently renamed to apptainer). First, run :code:`module load singularity` to make the :code:`singularity` command available. Refer to `the apptainer documentation <https://apptainer.org/documentation>`_ for usage information.

For example, to grab a bash shell with TRTools:

.. code-block:: bash

  singularity shell --bind /tscc docker://quay.io/biocontainers/trtools:6.0.1--pyhdfd78af_0

Or, to run the :code:`dumpSTR --help` command, for example:

.. code-block:: bash

  singularity exec --bind /tscc docker://quay.io/biocontainers/trtools:6.0.1--pyhdfd78af_0 dumpSTR --help

You can find containers for all Bioconda packages on `the Biocontainers registry <https://biocontainers.pro/registry>`_.

.. warning::
  You must provide :code:`--bind /tscc` if you want to have access to files in the :code:`/tscc` directory within the container.

Managing funds
--------------
.. code-block:: bash

  /cm/shared/apps/sdsc/1.0/bin/tscc_client.sh -A ddp268

Refer to `this page of the TSCC docs <https://www.sdsc.edu/support/user_guides/tscc.html#tscc_client>`_ for more info.

Using Jupyter
-------------
Looking for a way to edit code that you've stored on TSCC?

Before considering Jupyter, you may want to try `VSCode's Remote Development Extension <https://code.visualstudio.com/docs/remote/ssh>`_, which is usually easier to set up. You can also edit Jupyter notebooks with VSCode.

Otherwise, you can follow `these instructions to set up and run Jupyter from TSCC <https://bioinfo-ucsd-wiki.readthedocs.io/docs/jupyter_setup.html>`_.
Make sure to perform any :code:`conda` installations on an interactive node. Also, please note that you will need to perform a few extra steps to use :code:`jupyter` on TSCC, as described in the section `Usage on an HPC <https://bioinfo-ucsd-wiki.readthedocs.io/docs/jupyter_setup.html#usage-on-an-hpc>`_

Using graphical applications
----------------------------
It's easy to execute applications with graphics (like IGV or matplotlib) on TSCC!
Graphical applications typically rely on a port number defined in an environment variabled called :code:`$DISPLAY`.
When you run IGV, it will attach itself to this port and send you graphical messages according to a standard called *X11*.
On the receiving end, your laptop or local computer interprets these messages through the port using an application called *an X11 client*.
The X11 client will use the messages to figure out how to display your IGV window on your computer.

..
  TODO: figure out how to set up an X11 client on Macs

1. First, you'll need to install and set up an X11 client on your laptop. Windows users relying on Windows Subsystem Linux can skip this step, since `WSL has a built-in X11 client <https://learn.microsoft.com/en-us/windows/wsl/tutorials/gui-apps>`_.
2. When ssh-ing into TSCC, make sure to forward the :code:`$DISPLAY` variable through the tunnel by passing the :code:`-X` parameter to TSCC.

.. code-block:: bash

  ssh -X <user>@login.tscc.sdsc.edu

3. When grabbing an interactive node, make sure to forward the :code:`$DISPLAY` variable to the node by passing the :code:`--x11` parameter to :code:`srun`.

.. code-block:: bash

  srun --x11 ...

4. Now, just run your graphical application on the interactive node! A window should pop up when your display is ready.

Using Snakemake
---------------
To integrate Snakemake with SLURM, you must first install the SLURM Snakemake executor along with Snakemake.
Create a new environment with both packages:

.. code-block:: bash

  conda create -y -n snakemake -c conda-forge -c bioconda snakemake-executor-plugin-slurm 'snakemake>=8'
  conda activate snakemake

When structuring your Snakemake project, please consider using `the official recommended directory structure <https://snakemake.readthedocs.io/en/stable/snakefiles/deployment.html#distribution-and-reproducibility>`_ and `template <https://github.com/snakemake-workflows/snakemake-workflow-template>`_.

Within the top level directory of the project (where the :code:`config/` and :code:`workflow/` directories are located), I recommend creating a :code:`profile/` directory.
Inside that folder, create another directory called :code:`slurm` and a file within it :code:`profile/slurm/config.yaml`.
When executing Snakemake, you can specify the path to this profile via :code:`--workflow-profile profile/slurm`

You should store default arguments/options to :code:`snakemake` in the :code:`config.yaml` file.
For SLURM, I suggest including the following lines:

.. code-block::

  jobs: 16
  cores: 16
  use-conda: true
  latency-wait: 60
  keep-going: true
  conda-frontend: conda

  executor: slurm
  default-resources:
    nodes: 1
    runtime: 10
    slurm_account: ddp268
    slurm_partition: condo
    slurm_extra: "'--qos=condo'"

This will configure Snakemake to automatically submit the steps of your workflow as SLURM jobs.
It will ensure that at most 16 jobs are running simultaneously and at most 16 CPUs are in use simultaneously.
You can increase these values if you'd like, but please be mindful of requesting too many resources at once so that you're not impacting the work of others in our lab.

By default, this configuration will submit jobs to the :code:`condo` queue and allocate 10 minutes for each job.
But you can override any of the values in the :code:`default-resources` section on a per-rule basis by specifying them in the `resources directive <https://snakemake.readthedocs.io/en/stable/snakefiles/rules.html#resources>`_ of a rule.
Each step in the workflow will be allocated 1 CPU by default unless you request additonal CPUs via `the threads directive <https://snakemake.readthedocs.io/en/stable/snakefiles/rules.html#threads>`_

Please note that if you try to run Snakemake from a login node, it will simply hang indefinitely.
For this reason, I recommend running Snakemake from an interactive node or creating a :code:`.slurm` batch script for running Snakemake according to :ref:`the instructions above <tscc-submitting-jobs>`.
Here's an example of one.

.. code-block:: bash

  #!/usr/bin/env bash
  #SBATCH --export ALL
  #SBATCH --partition condo
  #SBATCH --account ddp268
  #SBATCH --qos condo
  #SBATCH --job-name smk
  #SBATCH --nodes 1
  #SBATCH --ntasks 1
  #SBATCH --cpus-per-task 1
  #SBATCH --mem 2G
  #SBATCH --time 1:00:00
  #SBATCH --output /dev/null

  # An example bash script demonstrating how to run the entire snakemake pipeline
  # This script creates a log file in the execution directory

  # clear anything left over in the log file
  echo ""> log

  # try to find and activate the snakemake conda env if we need it
  if ! command -v 'snakemake' &>/dev/null && \
    command -v 'conda' &>/dev/null && \
    [ "$CONDA_DEFAULT_ENV" != "snakemake" ] && \
    conda info --envs | grep "$CONDA_ROOT/snakemake" &>/dev/null; then
          echo "Snakemake not detected. Attempting to switch to snakemake environment." >> log
          eval "$(conda shell.bash hook)"
          conda activate snakemake
  fi

  # Pass any parameters to this script as additional arguments to snakemake via "$@"
  # For example, to execute a dry-run: 'sbatch smk.slurm -np' instead of 'sbatch smk.slurm'
  snakemake \
  --workflow-profile profile/slurm \
  --rerun-trigger {mtime,params,input} \
  "$@" &>log

  exit_code="$?"
  if command -v 'slack' &>/dev/null; then
      if [ "$exit_code" -eq 0 ]; then
          slack "snakemake finished successfully" &>/dev/null
      else
          slack "snakemake failed" &>/dev/null
          slack "$(tail -n4 log)" &>/dev/null
      fi
  fi
  exit "$exit_code"

Let's assume that you name the file :code:`run.bash` and mark it as executable with :code:`chmod u+x run.bash`.
Then you can run it on an interactive node with:

.. code-block:: bash

  ./run.bash

Or on a login node with:

.. code-block:: bash

  sbatch run.bash

You can override the default :code:`sbatch` parameters or :code:`snakemake` profile values directly from the command-line. For example, you can perform `a dry-run <https://snakemake.readthedocs.io/en/stable/executing/cli.html#useful-command-line-arguments>`_ of the workflow like this:

.. code-block:: bash

  sbatch --time 0:10:00 run.bash -np
