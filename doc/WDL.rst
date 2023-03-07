WDL
===

Last update: 2023/01/10

You'll want to read the "Constraints on how you write your WDL" sections
for each runtime environment you plan to execute your WDL in before
beginning to write your WDL

* `WDL 1.0 spec <https://github.com/openwdl/wdl/blob/main/versions/1.0/SPEC.md>`_
  (it's quite readable!)
* `differences between WDL versions <https://github.com/openwdl/wdl/blob/main/versions/Differences.md>`_

WDL by itself is just configuration, it needs an executor to run it. If you're using Expanse or All of Us,
you'll use Cromwell. If you're using DNANexus, you'll use dxCompiler. You'll need to understand
the ins and outs of those in addition to how to write WDL.

TODO intro to writing WDL - all the below assumes you can write basic WDL and is about running it
or making it runnable on certain platforms

Containers
----------
You'll likely want to specify a container with the :code:`docker` runtime flag as it's
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

WDL with Cromwell on Expanse (or other clusters)
------------------------------------------------

Constraints on how you write your WDL
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Cromwell only supports WDL 1.0, not 1.1 or development (2.0)

Running
^^^^^^^

Requires Java. Download the JAR file from `here <https://github.com/broadinstitute/cromwell/releases>`__.
(Can ignore womtool)

Cromwell will require configuration before working well (see below), but just as an intro:

#. Grab an interactive node.
#. Copy the config below to a location you want and modify it as necessary, and get your WDL workflow.
#. Stand up the MySQL server (see below) 
#. Have singularity cache whatever containers you plan to use (see the Singularity section of the Expanse notes)
#. To run the WDL in cromwell on the interactive node, run the command :code:`java -Dconfig.file=<path_to_config> -jar utilities/cromwell-84.jar run <path_to_WDL>`
#. If you want to submit jobs for each task instead of running them directly on the interactive node,
   change which :code:`backend.default` is commented out in the config.

Either way, you need to be running cromwell on an interactive node.
If you want more logs from cromwell for debugging purposes (you don't), prepend the flag :code:`-DLOG_LEVEL=DEBUG`

Cromwell has a server mode where you stand it up and can inspect running jobs through a web interface. I haven't
learned how to use that, so I'm not documenting it here.

Inputs and outputs 
^^^^^^^^^^^^^^^^^^

If you're using a container then you're inputs cannot contain symlinks.i

* You'll get something like file does not exist errors.
* It's possible symlinks to files underneath the root of the run will work, but not to files outside of the run root. I'd just avoid.
*  Use hardlinks instead.

Cromwell will dump its outputs to :code:`cromwell-executions/<workflow_name>/<workflow_run_id>/call-<task_alias>/execution`
That folder can also be used to inspect the stdout and stderr of that task for debugging.
Worfklow run ids are unhelpful randomly generated strings. To figure out which belongs to your
most recent run, you can look at the logs on the terminal for that run, or use
:code:`ls -t` to sort them by recency, e.g. :code:`cd cromwell-executions/<workflow_name> | ls -t | head -1`.
To check a task's inputs, looks at :code:`cromwell-executions/<workflow_name>/<workflow_run_id>/call-<task_alias>/inputs/<arbitrary_number>/<input_file>`
If you use subworkflows in your WDL then those workflows will be represented by nested folders between
the base workflow and the end task leaf. If your task has multiple inputs, then you'll have to look
at all the input folders with arbitrary numbers to determine which is the input you're looking for.
If you move task outputs from those folders they will no longer be available for call caching (see below),
so don't do that. I would instead hard link or copy them if you want the output in a more memorable location.

Cromwell's outputs will keep growing as you keep running it if you don't delete them. And due to randomized workflow run IDs it'll be very
hard to track which workflows have results important to caching and which errored out or are no longer needed.
No clue how to make managing that easier.

Configuration
^^^^^^^^^^^^^

I've saved my configuration as :code:`cromwell.conf`. I've copied it below, and then will explain it. 
Here's the `example config <https://github.com/broadinstitute/cromwell/blob/develop/cromwell.example.backends/cromwell.examples.conf>`_
from Cromwell's docs if you want to take a look, but it doesn't explain everything or have every option

.. code-block:: text

  # See https://cromwell.readthedocs.io/en/stable/Configuring/
  # this configuration only accepts double quotes! not singule quotes
  include required(classpath("application"))

  system {
    abort-jobs-on-terminate = true
    io {
      number-of-requests = 30
      per = 1 second
    }
    file-hash-cache = true
  }

  # necessary for call result caching
  # will need to stand up the MySQL server each time before running cromwell
  # stand it up on the same node that's running cromwell
  database {
    profile = "slick.jdbc.MySQLProfile$"
    db {
      driver = "com.mysql.cj.jdbc.Driver"
      url = "jdbc:mysql://localhost/cromwell?rewriteBatchedStatements=true"
      user = "root"
      password = "pass"
      connectionTimeout = 5000
    }
  }

  ### file based persistent database
  # the implementation here proved to be poorly designed and so much too slow
  #database {
  #  profile = "slick.jdbc.HsqldbProfile$"
  #  db {
  #    driver = "org.hsqldb.jdbcDriver"
  #    url = """
  #    jdbc:hsqldb:file:cromwell-executions/cromwell-db/cromwell-db;
  #    shutdown=false;
  #    hsqldb.default_table_type=cached;hsqldb.tx=mvcc;
  #    hsqldb.result_max_memory_rows=10000;
  #    hsqldb.large_data=true;
  #    hsqldb.applog=1;
  #    hsqldb.lob_compressed=true;
  #    hsqldb.script_format=3
  #    """
  #    connectionTimeout = 120000
  #    numThreads = 1
  #   }
  #}

  call-caching {
    enabled = true
    invalidate-bad-cache-results = true
  }

  docker {
    hash-lookup {
      enabled = true
      method = "remote"
    }
  }

  backend {
    # which backend do you want to use?
    # Right now I don't know how to choose this via command line, only here
    #default = "Local" # For running jobs on an interactive node
    default = "SLURM" # For running jobs by submitting them from an interactive node to the cluster
    providers {  
      # For running jobs on an interactive node
      Local {
        actor-factory = "cromwell.backend.impl.sfs.config.ConfigBackendLifecycleActorFactory"
        config {
          concurrent-job-limit = 10
          run-in-background = true
          root = "cromwell-executions"
          dockerRoot = "/cromwell-executions"
          runtime-attributes = """
            String? docker
          """
          submit = "/usr/bin/env bash ${script}"

          # We're asking bash-within-singularity to run the script, but the script's location on the machine
          # is different then the location its mounted to in the container, so need to change the path with sed
          submit-docker = """
            module load singularitypro && \
            singularity exec --containall --bind ${cwd}:${docker_cwd} docker://${docker} bash \
                 "$(echo ${script} | sed -e 's@.*cromwell-executions@/cromwell-executions@')"
          """
          filesystems {
            local {
              localization: ["hard-link"]
              caching {
                duplication-strategy: ["hard-link"]
                hasing-strategy: "fingerprint"
                check-sibling-md5: true
                fingerprint-size: 1048576 # 1 MB 
              }
            }
          }
        }
      }
      # For running jobs by submitting them from an interactive node to the cluster
      SLURM {
        actor-factory = "cromwell.backend.impl.sfs.config.ConfigBackendLifecycleActorFactory"
        config {
          concurrent-job-limit = 500
          root = "cromwell-executions"
          dockerRoot = "/cromwell-executions"

          runtime-attributes = """
            Int cpus = 1
            String mem = "2g"
            String dx_timeout
            String? docker
          """
          check-alive = "squeue -j ${job_id}"
          exit-code-timeout-seconds = 500
          job-id-regex = "Submitted batch job (\\d+).*"

          submit = """
            sbatch \
              --account ddp268 \
              --partition ind-shared \
              --nodes 1 \
              --job-name=${job_name} \
              -o ${out} -e ${err}  \
              --mail-type FAIL --mail-user <your_email> \
              --ntasks-per-node=${cpus} \
              --mem=${mem} \
              -c ${cpus} \
              --time=$(echo ${dx_timeout} | sed -e 's/m/:00/' -e 's/h/:00:00/' -e 's/ //g') \
              --chdir ${cwd} \
              --wrap "/bin/bash ${script}"
          """
          kill = "scancel ${job_id}"

          # We're asking bash-within-singularity to run the script, but the script's location on the machine
          # is different then the location its mounted to in the container, so need to change the path with sed
          submit-docker = """
            sbatch \
              --account ddp268 \
              --partition ind-shared \
              --nodes 1 \
              --job-name=${job_name} \
              -o ${out} -e ${err}  \
              --mail-type FAIL --mail-user <your_email> \
              --ntasks-per-node=${cpus} \
              --mem=${mem} \
              -c ${cpus} \
              --time=$(echo ${dx_timeout} | sed -e 's/m/:00/' -e 's/h/:00:00/' -e 's/ //g') \
              --chdir ${cwd} \
              --wrap "
                module load singularitypro && \
                singularity exec --containall --bind ${cwd}:${docker_cwd} docker://${docker} bash \
                     \"$(echo ${script} | sed -e 's@.*cromwell-executions@/cromwell-executions@')\"
              "
          """
          kill-docker = "scancel ${job_id}"

          filesystems {
            local {
              localization: ["hard-link"]
              caching {
                duplication-strategy: ["hard-link"]
                check-sibling-md5: true
                hasing-strategy: "fingerprint"
                fingerprint-size: 1048576 # 1 MB 
              }
            }
          }

        }
      }
  }}

Before using the configuration you'll need to insert your email address where specified.

Note that

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

Call caching with Cromwell
^^^^^^^^^^^^^^^^^^^^^^^^^^
Call caching allows you to reuse results of an old call in place of rerunning it if they have
the same inputs. This is generally necessary for developing most large workflows. (In general
these tasks may have different runtime-attributes and still be equivalent for call-caching,
docker is the main exception, see below)

You need to configure Cromwell with a database to store the cache results. Cromwell has a
simple to use database but unfortunately it's slow so I'd avoid it (if you want to use it anyway,
uncomment the section :code:`### file based persistent database` in the config above and
remove the MySQL database section)

Instead, use the MySQL database. Unfortuantely, this requires a running MySQL server. From the node which
you plan to execute cromwell from, run:

.. code-block:: bash

   singularity run --containall --env MYSQL_ROOT_PASSWORD=pass --bind <path1>:/var/lib/mysql --bind <path2>:/var/run/mysqld docker://mysql > <path3> 2>&1 &

This uses the default mysql docker continaer from DockerHub to start a mysql server. Here :code:`<path1>` should
be an absolute path to the directory where you want to store the MySQL database, :code:`<path2>` should be an absolute
path to a directory where MySQL can store some working files (I have it as be a sibling directory to :code:`<path1>`), and :code:`<path3>`
should be a path to a file where you want MySQL to write its log for the current session (for debugging if necessary).
So, for example

.. code-block:: bash

   singularity run --containall --env MYSQL_ROOT_PASSWORD=pass --bind ${PWD}/cromwell-executions/mysqldb:/var/lib/mysql --bind ${PWD}/cromwell-executions/mysql_var_run_mysqld:/var/run/mysqld docker://mysql > cromwell-executions/mysql.run.log 2>&1 &

To take down the MySQL server, just kill the process from that command.
   
Note: I've configured the MySQL database with a dummy user and password (user = root, password = pass)
which is not secure. I'm just assuming the Expanse nodes are secure enough already and no one
malicious is on them. Also, this uses the default MySQL port (3306). You may need to change that
if someone's already taken that port.

If cromwell doesn't shut down cleanly the MySQL server may remain locked and uninteractable with the next
cromwell session. To fix this, run:

.. code-block:: bash

   mysql -h localhost -P <your_port> --protocol tcp -u root -ppass cromwell \
   < <(echo "update DATABASECHANGELOGLOCK set locked=0, lockgranted=null, lockedby=null where id=1;" )

To check this has worked, you can run:

.. code-block:: bash

   mysql -h localhost -P <your_port> --protocol tcp -u root -ppass cromwell \
   < <(echo "select * from DATABASECHANGELOGLOCK;")

that should return output something like:

..

  ID      LOCKED  LOCKGRANTED     LOCKEDBY
  1       \0      NULL    NULL


Opening an interactive session with the MySQL server for debugging purposes:

.. code-block:: bash

   mysql -h localhost -P <your_port> --protocol tcp -u root -ppass cromwell

Notice there is no space between the -p and the password, unlike all the other flags.

Unexpected call caching behaviors
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
If you set the docker runtime attribute for a task
then for call caching Cromwell insists on trying to find
the corresponding docker image and using its digest (i.e. hash code)
as one of the keys for caching that task (not just the docker string
itself) (see `here <https://github.com/broadinstitute/cromwell/issues/2048>`__). If cromwell can't figure out how to locate the docker image
then it simply refuses to try to load the call from cache.
Cromwell's log method of telling you this is very unclear, I think
it's something like "task not eligible for call caching".
Because of this design choice, I'm not sure if you can get Cromwell
call caching to work with local docker image tarballs. 

Another unexpected input to call caching seems to be the backend 
(though I've not seen this confirmed in the docs), so for instance
if you run your job sometimes with SLURM and sometimes on an interactive
node, I can't seem to use the results of one in the other.

Other call caching optimizations
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Even with the above, my caching was quite slow, I think one of these options sped it up.
Not 100% sure which. They both have some details that might be worth knowing.

* :code:`backend.SLURM.filesystems.local.caching.check-sibling-md5: true`. In theory
  this means that if your input file is `foo.txt.` and you have `foo.txt.md5` in the same directory
  then instead of hashing the entirety of `foo.txt` you just read the md5 from the nearby file.
  This can be used to avoid hashing large input files more than once. Just use
  :code:`md5sum $file | awk '{print $1}'> ${file}.md5` to write the md5 checksum.
* :code:`backend.SLURM.filesystems.local.caching.fingerprint-size: true`. This isn't documented
  anywhere that I saw, but does exist in the `code <https://github.com/broadinstitute/cromwell/blob/32d5d0cbf07e46f56d3d070f457eaff0138478d5/supportedBackends/sfs/src/main/scala/cromwell/backend/impl/sfs/config/ConfigHashingStrategy.scala>`_
  This reduces the amount of file that's read by the hashing strategy. Note that this means that two files
  with the first MB of data identical and the sam mod time will be treated as identical, even if the 
  remaining MBs differ

Disabling call caching
~~~~~~~~~~~~~~~~~~~~~~

Add

.. code-block: text

  meta {
    volatile: true
  }

to a task definition to prevent it from being cached.

WDL with dxCompiler on DNANexus/UKB Research Analysis Platform
--------------------------------------------------------------

Constraints on how you write your WDL
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
You'll want your tasks' custom runtime attribute that denotes their timelimits
to be called :code:`dx_timeout`. (Cromwell is agnostic to what attribute you
use for denoting time limits, if any, but dxCompiler requires this specific attribute)

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

