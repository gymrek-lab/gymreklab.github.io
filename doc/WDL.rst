WDL
===

Last update: 2023/12/04

WDL is a configuration language. You use an executor to run it. If you're using Expanse or All of Us,
you'll use Cromwell as the executor. If you're using DNANexus, you'll use dxCompiler.

The first sections below are about getting set up with those executors on the platform you're using.
You should read those whether you're planning on writing your own WDL or running someone else's. 

After that there's some information if you're planning to learn to write your own WDL workflows.
Note that each executor has different constraints on the WDL you write, so if you're writing your own WDL,
first figure out what platforms you want it to run on and then read the "Constraints" sections
for those executors/platforms before beginning to write your WDL.

.. _WDL_with_Cromwell_on_Expanse:

WDL with Cromwell on Expanse (or other clusters)
------------------------------------------------

Constraints (important if you're writing your own WDL)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Cromwell only supports WDL 1.0, not 1.1 or development (2.0)

Getting Cromwell
^^^^^^^^^^^^^^^^

First, install java.

Jonathan compiled Cromwell from source with two changes to make it run better on Expanse. You can access that JAR 
at :code:`/expanse/projects/gymreklab/jmargoli/ukbiobank/utilities/cromwell-86-90af36d-SNAP-600ReadWaitTimeout-FingerprintNoTimestamp.jar`.

Alternatively, you can download Cromwell's official JAR file from `here <https://github.com/broadinstitute/cromwell/releases>`__. You can
ignore the womtool JAR at that location.

Specifying inputs to WDL workflows with Cromwell
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Cromwell passes inputs to WDL workflows via a JSON file which looks like:

.. code-block:: json

  {
    "workflow_name.input1_name": "value1",
    "workflow_name.input2_name": "value2",
    ...
  }

Add `-i <input_file>.json` to your Cromwell run command to pass in the input file.

If the WDL workflow you're running uses containers (e.g. Docker) then file inputs to the workflow cannot be symlinks.
If the files are symlinks, you'll get something like "file does not exist" errors. Instead of symlinks, use hardlinks.

Specifying Cromwell output locations
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

:ref:`_Cromwells_execution_directory` is confusingly organized, so it's hard to find the outputs of the final tasks(s) in a Cromwell run
if you don't tell it to put them anywhere specific. Instead, add an options JSON file to your Cromwell run with `-o <option_file>.json`
and tell it to put the workflow's outputs in the location you'd like:

.. code-block:: json

  {
    "final_workflow_outputs_dir": "<your_output_directory>"
  }

Running
^^^^^^^

Here are the steps for running it with Cromwell:

#. See :ref:`_cromwell_configuration` below for setting up your cromwell configuration.
#. If you're running with Docker containers, see :ref:`_Using_Singularity_to_run_Docker_containers` for setting up your :code:`.bashrc` file to make singularity work on Expanse,
   and then cache the singularity images before you start your job (also documented there).
#. Start by :ref:`_getting_an_interactive_node_on_Expanse`. You should set that to last for as long as the entire WDL workflow you are running with Cromwell.
   Depending on how long it will take, consider :ref:`_increasing_job_runtime_up_to_one_week`.
#. To enable :ref:`call-caching <_call_caching_with_Cromwell>`, (create the necessary directories, if this is your first time) and stand up the MySQL server on the interactive node (and
   then create the cromwell database, if this is your first time)
#. From the interactive node, execute the command :code:`java -Dconfig.file=<path_to_config> -jar <path_to_cromwell>.jar run -i <input_file>.json -o <options_file>.json <path_to_WDL> | tee <your_logfile>.txt` 
   to run the WDL using Cromwell. Feel free to omit the input and options flags if you're not using them.

Note: Cromwell has a server mode where you stand it up and can inspect running jobs through a web interface. As I (Jonathan) haven't
learned how to use that, so I'm not documenting it here.

If you need help debugging, start by looking at Cromwell's log file, which will be written to the log file you specified at the end of the command above.
If the workflow completed successfully, the lines toward the end of the log should tell you where it put the workflow's outputs (if you didn't specify an output location above).
If a task failed and you want to inspect its intermediate inputs/outputs for debugging, see :ref:`_Cromwells_execution_directory`.

.. _cromwell_configuration:

Configuration
^^^^^^^^^^^^^

I (Jonathan) recommend you make a copy of my config `here <https://github.com/LiterallyUniqueLogin/ukbiobank_strs/blob/master/workflow/cromwell.conf>`.
Another reference is the `example config <https://github.com/broadinstitute/cromwell/blob/develop/cromwell.example.backends/cromwell.examples.conf>`_
from Cromwell's docs, but it doesn't explain everything or have every option you might want.

After copying my config, you will need to:

* swap my email address for yours
* Either set up :ref:`_call_caching_with_Cromwell`, or set :code:`call-caching.enabled = False`.
  If you disable it, then every time you run a job it will be run again from the beginning instead of reusing intermediate results that finished successfully.
* When running jobs, if you want to run them all on the cluster, make sure under backend that :code:`default = "SLURM"`. If you only have a small number of jobs and 
  you'd rather run them on your local node for debugging purposes or because the Expanse queue is backed up right now, instead change that to :code:`default = "Local"`

If you want to understand the config file
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: text

  foo {
    bar {
      baz = "bop"
    }
  }

is equivalent to :code:`foo.bar.baz = "bop"`

* :code:`backends.providers.<backend>.config.submit` and :code:`submit-docker` are what control
  how tasks are submitted as jobs.
* :code:`backends.providers.<backend>.config.runtime-attributes` is where you configure which
  attributes from the :code:`runtime-attributes` section of a WDL task are actually used when
  submitting the job corresponding to that task. Any runtime attributes in the WDL but not in the config
  are ignored. Runtime attributes with :code:`?` or that have defaults :code:`= <default>` are optional,
  runtime attributes that are just declared (e.g. :code:`String dx_timeout`) are required.

.. _call_caching_with_cromwell:

Call caching with Cromwell
^^^^^^^^^^^^^^^^^^^^^^^^^^
Call caching allows you to reuse results of a successful previous run of a WDL task in place of rerunning that task.
Note that the task being reused must have had the exact same inputs and docker file as the task being replaced.

Call caching is generally helpful for large workflows where you might find an error halfway through your workflow run
and want to restart the workflow without having to rerun everything from the beginning. Unfortunately, this requires configuring Cromwell with a database to store the cache results
which is unpleasantly complex, as it requires running a MySQL server.

To enable call caching, you will need to do the following once:
* make sure you've set up your :code:`.bashrc` to handle :ref:`Using_Singularity_to_run_Docker_containers`
* :code:`cd` into the directory you want to launch cromwell from and make the following directories:

.. code-block:: bash

     mkdir -p cromwell-executions/mysql_var_run_mysqld
     mkdir -p cromwell-executions/mysqldb

Then, each time you want to run Cromwell, after logging in to the interactive node but before running Cromwell, run

.. code-block:: bash

   singularity run --containall --env MYSQL_ROOT_PASSWORD=pass --bind ${PWD}/cromwell-executions/mysqldb:/var/lib/mysql --bind ${PWD}/cromwell-executions/mysql_var_run_mysqld:/var/run/mysqld docker://mysql > cromwell-executions/mysql.run.log 2>&1 &

This starts a MySQL server running on the interactive node by using singularity to run the the default MySQL docker.
This command stores the MySQL log at :code:`cromwell-executions/mysql.run.log` if you need it for debuging. 

The first time you stand up MySQL, you'll need to run the following:

.. code-block:: bash

   # start an interactive my sql session
   mysql -h localhost -P 3306 --protocol tcp -u root -ppass cromwell
   # from within the mysql prompt
   create database cromwell;
   exit;

You should now (finally!) be good to go with call caching.

Debugging MySQL issues
~~~~~~~~~~~~~~~~~~~~~~

To take down the MySQL server, just kill the process spawned by that command.
   
Note: I've configured the MySQL database with a dummy user and password (user = root, password = pass)
which is not secure. I'm just assuming the Expanse nodes are secure enough already and no one
malicious is on them. Also, this uses the default MySQL port (3306). You may need to change that
(I don't know how) if someone's already taken that port.

*Debugging tip if cromwell hangs at*  :code:`[info] Running with database db.url = jdbc:mysql://localhost/cromwell?rewriteBatchedStatements=true`:

If the previous cromwell execution didn't shut down cleanly (say, you kill it because it's hanging) then the MySQL server may remain locked and
uninteractable, causing the next cromwell session to hang. To fix this, run:

.. code-block:: bash

   mysql -h localhost -P 3306 --protocol tcp -u root -ppass cromwell \
   < <(echo "update DATABASECHANGELOGLOCK set locked=0, lockgranted=null, lockedby=null where id=1;" )
   mysql -h localhost -P 3306 --protocol tcp -u root -ppass cromwell \
   < <(echo "update SQLMETADATADATABASECHANGELOGLOCK set locked=0, lockgranted=null, lockedby=null where id=1;" )

To check this has worked, you can run:

.. code-block:: bash

   mysql -h localhost -P 3306 --protocol tcp -u root -ppass cromwell \
   < <(echo "select * from DATABASECHANGELOGLOCK;")
   mysql -h localhost -P 3306 --protocol tcp -u root -ppass cromwell \
   < <(echo "select * from SQLMETADATADATABASECHANGELOGLOCK;")

that should return output something like:

..

  ID      LOCKED  LOCKGRANTED     LOCKEDBY
  1       \0      NULL    NULL
  ID      LOCKED  LOCKGRANTED     LOCKEDBY
  1       \0      NULL    NULL

*Debugging tip if the mysql log at path3 says* :code:`another process is using this socket`

Delete the lock files at `<path2>/*lock`, kill the mysql server and then restart it and it should work.

*Debugging tip*: Opening an interactive session with the MySQL server for debugging purposes:

.. code-block:: bash

   mysql -h localhost -P 3306 --protocol tcp -u root -ppass cromwell

Notice there is no space between the -p and the password, unlike all the other flags.

Unexpected call caching behaviors
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
If you set the docker runtime attribute for a task
then Cromwell insists on looking up the 
corresponding docker image and using its digest (i.e. hash code) 
as one of the keys for caching that task. This is unintuitive because it's not just using the string
in the runtime attribute as the cache key (see `here <https://github.com/broadinstitute/cromwell/issues/2048>`__).
Moreover, if cromwell can't figure out how to locate the docker image's digest during this process,
then it simply refuses to try to load the call from cache at all, with a very inspecific
log message to the effect of "task not eligible for call caching".
Because of this design choice, I'm not sure if you can get Cromwell
call caching to work with local docker image tarballs, which cause the image digest lookup step to fail. 

Another surprising behavior is that call caching seems to be backend specific
(though I've not seen this confirmed in the docs), so for instance
if you run your job sometimes with SLURM and sometimes locally on an interactive
node, I can't seem to use the cached results of one for the other.

Disabling call caching for a task
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Add

.. code-block:: text

  meta {
    volatile: true
  }

to a task definition to prevent it from being cached.

.. _Cromwells_execution_directory:

Cromwell's execution directory
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Cromwell runs its executions (including task inputs and outputs) in :code:`cromwell-executions/<workflow_name>/<workflow_run_id>`
Worfklow run ids are unhelpful randomly generated strings. To figure out which belongs to your
most recent run, you can look at the logs on the terminal for that run, or use
:code:`ls -t` to sort them by recency, e.g. :code:`cd cromwell-executions/<workflow_name> | ls -t | head -1`.
Once you're in the your workflow run's folder, you should see one folder named `call-<task_alias>`
for each task called in the workflow. The task folder will contain two important directories :code:`inputs` and :code:`executions`.
:code:`inputs` contains a bunch of subfolders with random numbers, each of which contain one or more input files (input files
originally stored in the same directory will be put into the same inputs subdirectory). Note that input files will be named
by their original filenames, not by the variable names they were referred to in the task, so it can be hard to match which inputs
in this directory correspond to which inputs in the task. :code:`executions` contains a number of useful files for debugging:

* :code:`rc` contains the return code of the task (if it completed)
* :code:`script.submit` is the script used to submit the task to SLURM (not sure if this is present on local runs)
* :code:`stdout.submit` and :code:`stderr.submit` are the stdout/err for the job submission to SLURM.
* :code:`script` contains the script that Cromwell executed to run this task on a SLRUM node (which is the command section of the task wrapped in 
  some autogenerated code)
* :code:`stdout` and :code:`stderr` are the stdout/err for the actual run of the task (if you didn't capture them inside 
  WDL with :code:`stdout()` or :code:`stderr()`).
* All the output files generated by the task should be in this folder as well.
  If you move task outputs from this folders they will no longer be available for call caching,
  so don't do that. Instead, hard or symlink them to another location.

If the task was call cached, then instead `call-<task_alias>` will contain `cacheCopy/execution` as a subdirectory
and there will be no inputs folder you can cross reference against (which can make debugging harder).

If the workflow you called in turn called subworkflows, those workflows will be represented by nested folders between
the base workflow and the end task leaf, looking something like:
:code:`cromwell-executions/<workflow_name>/<workflow_run_id>/call-<subworkflow_alias>/<subworkflow_name>/<subworkflow_run_id>/call-...`
If a task or subworkflow is called in a scatter block, then between the `call-<alias>` folder and its
usual contents there will be a bunch of `shard-<number>` folders which contain each of the scattered subcalls. All this nesting
can get a bit overwhelming when you're trying to debug.

Cromwell's outputs will keep growing as you keep running it if you don't delete them. And due to randomized workflow run IDs it'll be very
hard to track which workflows have results important to caching and which errored out or are no longer needed.
No clue how to make managing that easier.

WDL with dxCompiler on DNANexus/UKB Research Analysis Platform
--------------------------------------------------------------

Constraints (important if you're writing your own WDL)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Unlike Cromwell, dxCompiler supports WDL 1.1. So if you don't need your WDL to be cross-platform,
you can use those features.

dxCompiler's implementation of WDL has a few limitations, read them `here <https://github.com/dnanexus/dxCompiler#Limitations>`_.

Additionally, you'll want your tasks' custom runtime attribute that denotes their timelimits
to be called :code:`dx_timeout`. (Cromwell is agnostic to what attribute you
use for denoting time limits, if any, but dxCompiler requires this specific attribute)

From personal correspondence with Rylie Yeakley from ukbiobank-support@dnanexus.com on 2023/01/25,
you currently cannot access record objects (e.g. the UKBiobank phenotype database) from within
WDL. Neither writing a python script to access those records and calling that from WDL nor calling
the existing table_exporter app from WDL will work. So instead, you'll need to extract all data fields
from that dataset (presumably to a TSV) using the GUI, JupyterLab, or the command line before
running your WDL pipeline. See the docs we've written about DNANexus for info on how to do that on the command line.

dxCompiler only seems to run commands
directly in the container (it does not seem to support any setup after container start before
running the command, such as specified by ENTRYPOINT or SHELL in a Dockerfile) so
you'll want your commands to be compatible with that. This is specifically a problem
with conda as you need to run a shell, activate your conda env, and then execute
the command from that shell in order to get access to your conda environment. To
get around this, I've written the following script:

.. code-block:: bash
  
  #!/bin/bash
  #filename: envsetup

  source /etc/profile.d/conda.sh
  conda activate ukb

  # run the command passed as arguments on the command line
  "$@"

and I include it in my container with the following Dockerfile commands:

.. code-block:: docker

  RUN mkdir /container_install
  COPY envsetup /container_install/envsetup
  RUN chmod a+rx /container_install/envsetup
 
and then in the command sections of my WDL tasks I simply write 

.. code-block:: text
    
  command <<<
    envsetup <mycommand> <arg1> ...
  >>>

(`This Dockerfile <https://github.com/fritzsedlazeck/parliament2/blob/master/Dockerfile>`_
suggests an alternative by mucking directly with env variables to simulate
a conda activation, but that seems like a bad idea)

Running
^^^^^^^

1. Install the DNA nexus command line tools vended through pip: :code:`pip3 install dxpy`.
2. Run :code:`dx login` and :code:`dx select <project name>`.
3. Download :code:`dxCompiler` from the releases section of its `github page <https://github.com/dnanexus/dxCompiler>`_.
   A detailed breakdown of its features is hidden at `this hard to find page <https://github.com/dnanexus/dxCompiler/blob/develop/doc/ExpertOptions.md>`_
4. Compiling a WDL file for UKB RAP: 
   :code:`java -jar dxCompiler-2.10.4.jar compile <yourfile.wdl> -project <project-name> -folder <DNANexus directory to put the compiled workflow in>`
5. Running the file: :code:`dx run <workflow directory>/<workflow name>`

Use :code:`dx://<project_name>:<path_to_file>` for :code:`File` inputs to your WDL tasks that are hosted on DNANexus.

Misc:

* Uploading files to DNANexus: :code:`dx upload --path <directory> <file>`

WDL with Cromwell on All of Us (hosted on TerraBio)
---------------------------------------------------

TODO

Constraints on how you write your WDL
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Cromwell only supports WDL 1.0, not 1.1 or development (2.0)


Learning WDL
------------

I recommend these links for learning WDL. There are also good tutorials you can find for parts of the spec you're confused by.

* `WDL 1.0 spec <https://github.com/openwdl/wdl/blob/main/versions/1.0/SPEC.md>`_
  (it's quite readable!)
* `differences between WDL versions <https://github.com/openwdl/wdl/blob/main/versions/Differences.md>`_

WDL Gotchas
^^^^^^^^^^^

(I'm unclear if these gotchas only exist for Cromwell running WDL 1.0 or for all versions of WDL and also for dxCompiler)

* There are no :code:`else` statements to pair with :code:`if` statements. Instead
  write :code:`if (x) {}`, then :code:`if (!x) {}`, and then use :code:`select_first()`
  to condense the results of both branches to single variables.
* For whatever reason, trying :code:`my_array[x+1]` will fail at compile time. Instead, write
  :code:`Int x_plus_one = x + 1` and then :code:`my_array[x_plus_one]`.
* There is no array slicing. If you want to scatter over :code:`item in my_array[1:]`, instead
  scatter over :code:`idx in range(length(my_array)-1)` and manually access the array at
  `Int idx_plus_one = idx + 1`
* If you want to create an array literal that's easier to specify via a list comprehension than to type it all out,
  do so by writing out the expression inside a scatter block in a worfklow. There's no way to get list comprehensions to work
  anywhere in tasks or within the input or output sections of a workflow.
* The :code:`glob()` library function can only be used within tasks, not within workflows.
  It will not error out at language examination time but at runtime if used within a workflow.
* The :code:`write_XXX()` functions will fail in weird ways if used in a workflow and not a task.
* The :code:`write_XXX()` functions will not accept :code:`Array[X?]`, only :code:`Array[X]`.

These gotchas I know only apply to WDL 1.0 (but perhaps to both Cromwell and dxCompiler?)

* The :code:`write_objects()` function will crash when passed an empty array of structs
  instead of writing a header line and no content rows.
* The :code:`write_objects()` function will crash at runtime when passed a struct with a member
  that is a compound type (struct, map, array, object).
* While structs can contain members of multiple types, maps cannot, and so to create such a struct
  it must be assigned from an object literal and not a map literal.

Using Docker containers from WDL
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You'll likely want to specify a container within each tasks' :code:`docker` runtime flag as that's
necessary to execute your WDL on cloud platforms. (Cromwell doesn't support the 
equivalent :code:`container` flag).

Constraints imposed by runtime environments:

* If running All of Us, seems like you'll need to host on Google Container Registry? (not tested)
* If running with Cromwell on Expanse, will need to either store the image locally, or host
  on one of the following supported environments: quay.io, dockerhub, google container registry (GCR)
  or google artifact registry (GAR). I'm not sure storing locally will work though,
  as I'm not sure you can get call caching to work with that - haven't tried.
* No constraints for UKB RAP as far as I know - you can upload the docker container to DNA Nexus,
  or pull from an cloud container registry.

quay.io is my cloud container registry of choice. Terminology:

* quay.io - Red Hat's cloud container registry
* Red Hat Quay - Red Hat's private deployment container registry service
* Project quay - an open source version of Red Hat Quay where you can
  deploy and stand up your own private container registry

It's my container registry of choice because it has free accounts 
(though this isn't super clear from their pricing docs), doesn't charge
for public containers, and because at least
so far I haven't found any pull restrictions. If you do run into issues,
I'd recommend moving to GCR. Yang has tried Dockerhub, but that has really
restrictive pull limits if you're using the free account. The paid account
isn't such an issue (only $7/mo.) but Yang couldn't figure out how to get
the authentication to work on UKB RAP so that you could log in from each task
before pulling the docker container so as to circumvent the pull limit.

Repositories in quay.io start as private, even on the free account 
which in theory hasn't paid for private repos (not sure why?).
After pushing to them for the first time,
sign into the web interface, select the repo, click on the wheel icon
on the left (settings) and click Make Public.

To push to quay.io after building your docker image, do

.. code-block:: bash

  docker login --username <user_name> quay.io
  docker tag <existing_image_name>:<existing_image_tag> quay.io/<user_name>/<container_repository_name>:<tag>
  docker push quay.io/<user_name>/<container_repository_name>:<tag>

depending on how you configured docker, you may need to run those commands with sudo.

Tips on building a container with conda
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* Use :code:`continuumio/miniconda3` as the base container.
* Put :code:`RUN conda init --system bash` in your Dockerfile
* See the section about conda and dxCompiler below to get
  a script for activating conda. Then either configure that to run
  automatically with the Dockerfile commands ENTRYPOINT
  or SHELL if you're running the container with run or shell, or make sure
  to call that script manually as part of the container exec invocation.

