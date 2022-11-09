High-throughput computing
=========================

A frequent problem in research is the need to run lots of analysis scripts which may take a long time to run.
These scripts frequently need access to large amounts of system resources and data, and so they're ill-suited to running on a normal user machine.

The solution to this problem comes in the form of *batch processing* or *high throughput computing*.
This is normally done by setting up a machine, or a cluster of networked machines, which can access resources over a period of days, weeks, or even months.


.. sidebar:: High-performance computing

   High-throughput computing constrasts with another paradigm, *high-performance computing*, or HPC.
   While The two share some common elements, HPC is normally assosicated with jobs which are highly parallelised, and which consume very large amounts of resources for a comparitively short amount of time.

.. note:: HTCondor 9

   These notes should be generally relevant to all versions of condor, but they've been written to align with version 9 of htcondor, which at the time of writing was the latest stable version.
   The `documentation <https://htcondor.readthedocs.io/en/feature/>`_ for this version of htcondor are substantially improved over earlier versions, too, which is a major bonus.
   

``htcondor``: Scheduling jobs
-----------------------------

High throughput computing (HTC) facilities are normally in high-demand, and if capacity was planned around the times of day when most people wanted to use them we'd end up with a very expensive facility which went largely unused at night.
To circumnavigate this HTC clusters are designed to queue jobs up, and run them when they have the available resources.
For some jobs this can be frustrating or unsuitable, but for many tasks in the natural sciences, where we don't need instant results, this is fine.

An unfortunate consequence of this is that we need to do a little more work to get a job running, and we need to write something akin to an advertisment for the cluster, so that it knows a job is available to run when it gets a chance.
Initially I'll describe how to get a very simple job running on an htcondor cluster, simply as an illustration, but as you get more familiar with the process you will find that some changes are needed in the way that you write code to take best advantage of the strengths of HTC.

.. sidebar:: HTCondor Clusters

	     Throughout these notes I'll make many reference to "clusters"; this is a term frequently used in HTC parlence, and refers to groupings of computers which share the load of running submitted jobs.
	     Different approaches to constructing such a cluster exist: sometimes all of the machines will be near-identical, running the same operating system, with the same processors.
	     Perhaps more frequently you'll encounter "heterogeneous" clusters, which are composed of a range of different types of machine running different OSes.
	     HTCondor allows the construction of such clusters (they can even be constructed from desktop machines set to run jobs when they're not being used), but allows jobs to specify the kind of machine they must run on, if it's important.
	     HTCondor normally refers to this collection of computers as a "pool"; there are probably some slight differences between what people mean by cluster and what HTCondor means by pool, but for most purposes they're one and the same.

In keeping with the majority of these notes I'm going to use ``python`` to write the demonstration script, but it should work equally well with other interpretted languages.
If you're using a compiled language, especially ``C``, then things might be a little trickier.

The job we're going to schedule will just write out some text to ``stdout``.

.. code-block:: python

		print("Hello htcondor!")

I'll save this script as ``test.py``.
So let's see how we get this to run.

Submitting jobs to condor
-------------------------

Getting a job running on a condor cluster requires a little more preparation than just running it on a local machine via the terminal.
To run the script we created in the last section in a normal terminal we would probably just run

.. code:: bash

	  $ python test.py

Or, if we wanted to be slightly more precise about how we specify the python executable we might do something like this:

.. code:: bash

	  $ /usr/bin/python3 test.py

To run with condor we need to prepare a short file called a "submit description file", or a "subfile".
This file contains the arguments and the executable that you want to run, and tells the cluster where to store the outputs of the script: since you don't have a terminal attached to the program you'll need to store the outputs it would normally direct there somewhere else.

A very simple submit file for the program we wrote earlier might look like this:

::
   
   # This is a comment in this example file
   executable = /usr/bin/python3
   arguments  = test.py

   output = out.txt
   error  = error.txt
   log    = log.txt

   request_cpus   = 1
   request_memory = 1024

   queue

The first two lines after the comment point to the executable, which in this case is python, and the argument for that executable.

.. sidebar::

   If we were running a compiled program, or we'd made the script executable and added a shebang to it we could omit the arguments section, and just use the script as the executable, but normally for running python programs I find this a little bit easier to work with.

You can provide the arguments in the same format you would on the command line.

The next three lines tell htcondor where it should redirect the stdout (``output``), and stderr (``error``) streams to.
These can just be text files.
The ``log`` argument should be another text file where information about how condor processes your job should get written to.

Next, we need to tell htcondor what resources we need.
For this job we're requesting a single CPU core (``request_cpus = 1``) and 1024-MB of RAM (``request_memory = 1024``).
These lines allow htcondor to allocate your job to a machine which is able to run it.

The final line, which just reads ``queue`` ends the description of the job.
It is possible to add an argument after this to repeat the job multiple times, for example, ``queue 5`` would run the specified job five times.
For more details about handling jobs like this it would be worth having a look at example 2 in the `HTCondor documentation <https://htcondor.readthedocs.io/en/latest/users-manual/submitting-a-job.html>`_.

.. warning:: In this example all of the paths have been relative paths, so htcondor will use the files in the working directory from which you submit the job.
	     In some situations this might not be ideal, and it may be wise to use absolute paths to your files and executables.

Let's save our submit file as ``test.sub``.
All that we need to do now is to run ``condor_submit test.sub``:

.. code:: bash

	  $ condor_submit test.sub
	  
	  Submitting job(s).
	  1 job(s) submitted to cluster 74043.

HTcondor will now attempt to match our job with a suitable machine and run it.
You can check-up on the job by running ``condor_q 74043``, where the number is the cluster number the ``condor_submit`` command produced.

Once the job's finished we can look at the outputs.
We used a ``print`` command in the script, so we should expect to see something in the ``out.txt`` file, and, sure enough, its contents are

::

   Hello htcondor!

We can also look at the ``err.txt`` file, which is empty, indicating that the python script didn't write-out anything on ``stderr``.
The ``log.txt`` file contains quite a lot of information explaining which machine ran the job:

::

   000 (74043.000.000) 2022-02-03 16:13:22 Job submitted from host: <130.209.45.81:9618?addrs=130.209.45.81-9618&alias=wiay.astro.gla.ac.uk&noUDP&sock=schedd_3970772_e555>
   ...
   001 (74043.000.000) 2022-02-03 16:13:22 Job executing on host: <130.209.45.81:9618?addrs=130.209.45.81-9618&alias=wiay.astro.gla.ac.uk&noUDP&sock=startd_3970772_e555>
   ...
   006 (74043.000.000) 2022-02-03 16:13:23 Image size of job updated: 5000
	   0  -  MemoryUsage of job (MB)
	   0  -  ResidentSetSize of job (KB)
   ...
   005 (74043.000.000) 2022-02-03 16:13:23 Job terminated.
	   (1) Normal termination (return value 0)
		   Usr 0 00:00:00, Sys 0 00:00:00  -  Run Remote Usage
		   Usr 0 00:00:00, Sys 0 00:00:00  -  Run Local Usage
		   Usr 0 00:00:00, Sys 0 00:00:00  -  Total Remote Usage
		   Usr 0 00:00:00, Sys 0 00:00:00  -  Total Local Usage
	   0  -  Run Bytes Sent By Job
	   0  -  Run Bytes Received By Job
	   0  -  Total Bytes Sent By Job
	   0  -  Total Bytes Received By Job
	   Partitionable Resources :    Usage  Request Allocated 
	      Cpus                 :                 1         1 
	      Disk (KB)            :     5000     5000      6940 
	      GPUs                 :                           0 
	      Memory (MB)          :        0     1024      1024 

	   Job terminated of its own accord at 2022-02-03T16:13:23Z.
   ...

Normally you'll not need to spend too much time looking at these log files, but they can be helpful if something goes wrong.
