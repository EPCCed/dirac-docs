# Running jobs on Tursa

As with most HPC services, Tursa uses a scheduler to manage access to
resources and ensure that the thousands of different users of system are
able to share the system and all get access to the resources they
require. Tursa uses the Slurm software to schedule jobs.

Writing a submission script is typically the most convenient way to
submit your job to the scheduler. Example submission scripts (with
explanations) for the most common job types are provided below.

Interactive jobs are also available and can be particularly useful for
developing and debugging applications. More details are available below.

!!! hint
    If you have any questions on how to run jobs on Tursa do not hesitate
    to contact the [DiRAC Service Desk](mailto:dirac-support@epcc.ed.ac.uk).

You typically interact with Slurm by issuing Slurm commands from the
login nodes (to submit, check and cancel jobs), and by specifying Slurm
directives that describe the resources required for your jobs in job
submission scripts.

## Resources

### coreh

Time used on Tursa CPU nodes is measured in coreh.  
1 coreh = 1 physical core for 1 hour. So a Tursa compute node with 2, 64 core CPUs would cost
128 coreh per hour. 

### GPUh

Time used on Tursa GPU nodes  is measured in GPUh.  
1 GPUh = 1 GPU for 1 hour. So a Tursa compute node with 4 GPUs would cost
4 GPUh per hour. 

### Checking available budget

You can check in [SAFE](https://safe.epcc.ed.ac.uk) by selecting `Login accounts` from the menu, select the login account you want to query.

Under `Login account details` you will see each of the budget codes you have access to listed e.g.
`e123 resources` and then under Resource Pool to the right of this, a note of the remaining budgets. 

When logged in to the machine you can also use the command 

    sacctmgr show assoc where user=$LOGNAME format=account,user,maxtresmins

This will list all the budget codes that you have access to e.g.


       Account       User   MaxTRESMins
    ---------- ---------- -------------
          e123      userx         cpu=0
     e123-test      userx

This shows that `userx` is a member of budgets `e123` and `e123-test`.  However, the `cpu=0` indicates that the `e123` budget is empty or disabled.   This user can submit jobs using the `e123-test` budget.

To see the number of coreh or GPUh remaining you must check in [SAFE](https://safe.epcc.ed.ac.uk).

### Charging

Jobs run on Tursa are charged for the time they use i.e. from the time the job begins to run until the time the job ends (not the full wall time requested).

Jobs are charged for the full number of nodes which are requested, even if they are not all used.

Charging takes place at the time the job ends, and the job is charged in full to the budget which is live at the end time.


## Basic Slurm commands

There are three key commands used to interact with the Slurm on the
command line:

  - `sinfo` - Get information on the partitions and resources available
  - `sbatch jobscript.slurm` - Submit a job submission script (in this
    case called: `jobscript.slurm`) to the scheduler
  - `squeue` - Get the current status of jobs submitted to the scheduler
  - `scancel 12345` - Cancel a job (in this case with the job ID
    `12345`)

We cover each of these commands in more detail below.

### `sinfo`: information on resources

`sinfo` is used to query information about available resources and
partitions. Without any options, `sinfo` lists the status of all
resources and partitions, e.g.

```
[dc-user1@tursa-login1 ~]$ sinfo 
    
PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
cpu          up   infinite      6   idle tu-c0r0n[66-71]
gpu*         up   infinite      2 drain* tu-c0r0n[27,30]
gpu*         up   infinite      1  down* tu-c0r4n33
gpu*         up   infinite      1  drain tu-c0r3n81
gpu*         up   infinite     58  alloc tu-c0r0n[00,03,06,09,12,15,18,21,24,33,36,39,42,45],tu-c0r1n[24,27,30,33,60,63,66,69,72,75,78,81,84,87,90,93],tu-c0r2n[24,27,30,33,60,63,66,69],tu-c0r3n[03,72,75,78,84,87,90,93],tu-c0r4n[00,03,06,12,15,24,27,30,60,63,66,69]
gpu*         up   infinite     50   idle tu-c0r1n[00,03,06,09,12,15,18,21],tu-c0r2n[00,03,06,09,12,15,18,21,72,75,78,81,84,87,90,93],tu-c0r3n[00,06,09,12,15,18,21,24,27,30,33,60,63,66,69],tu-c0r4n[09,18,21,72,75,78,81,84,87,90,93]
rack0        up   infinite      2 drain* tu-c0r0n[27,30]
rack0        up   infinite     14  alloc tu-c0r0n[00,03,06,09,12,15,18,21,24,33,36,39,42,45]
rack1        up   infinite     16  alloc tu-c0r1n[24,27,30,33,60,63,66,69,72,75,78,81,84,87,90,93]
rack1        up   infinite      8   idle tu-c0r1n[00,03,06,09,12,15,18,21]
rack2        up   infinite      8  alloc tu-c0r2n[24,27,30,33,60,63,66,69]
rack2        up   infinite     16   idle tu-c0r2n[00,03,06,09,12,15,18,21,72,75,78,81,84,87,90,93]
rack3        up   infinite      1  drain tu-c0r3n81
rack3        up   infinite      8  alloc tu-c0r3n[03,72,75,78,84,87,90,93]
rack3        up   infinite     15   idle tu-c0r3n[00,06,09,12,15,18,21,24,27,30,33,60,63,66,69]
rack4        up   infinite      1  down* tu-c0r4n33
rack4        up   infinite     12  alloc tu-c0r4n[00,03,06,12,15,24,27,30,60,63,66,69]
rack4        up   infinite     11   idle tu-c0r4n[09,18,21,72,75,78,81,84,87,90,93]
```

* `alloc` nodes are those that are running jobs
* `idle` nodes are empty
* `drain`, `down`, `maint` nodes are unavailable to users

### `sbatch`: submitting jobs

`sbatch` is used to submit a job script to the job submission system.
The script will typically contain one or more `srun` commands to launch
parallel tasks.

When you submit the job, the scheduler provides the job ID, which is
used to identify this job in other Slurm commands and when looking at
resource usage in SAFE.

    sbatch test-job.slurm
    Submitted batch job 12345

### `squeue`: monitoring jobs

`squeue` without any options or arguments shows the current status of
all jobs known to the scheduler. For example:

    squeue

will list all jobs on Tursa.

The output of this is often large. You can restrict the
output to just your jobs by adding the `-u $USER` option:

    squeue -u $USER

### `scancel`: deleting jobs

`scancel` is used to delete a jobs from the scheduler. If the job is
waiting to run it is simply cancelled, if it is a running job then it is
stopped immediately. You need to provide the job ID of the job you wish
to cancel/stop. For example:

    scancel 12345

will cancel (if waiting) or stop (if running) the job with ID `12345`.

## Resource Limits

The Tursa resource limits for any given job are covered by three
separate attributes.

  - The amount of *primary resource* you require, i.e., number of
    compute nodes.
  - The *partition* that you want to use - this specifies the nodes that
    are eligible to run your job.
  - The *Quality of Service (QoS)* that you want to use - this specifies
    the job limits that apply.

### Primary resource

The *primary resource* you can request for your job is the compute node.

!!! information
    The `--exclusive` option is enforced on Tursa which means you will
    always have access to all of the memory on the compute node regardless
    of how many processes are actually running on the node.

!!! note
    You will not generally have access to the full amount of memory resource on the the node as some is retained for running the operating system and other system processes.

### Partitions

On Tursa, compute nodes are grouped into partitions. You will have to
specify a partition using the `--partition` option in your Slurm
submission script. The following table has a list of active partitions
on Tursa.

| Partition | Description                                                 | Max nodes available |
| --------- | ----------------------------------------------------------- | ------------------- |
| cpu  | CPU nodes with AMD EPYC 32-core processor &times; 2    | 6               |
| gpu  | GPU nodes with AMD EPYC 32-core processor and NVIDIA A100 GPU &times; 4  | 114                |

You can list the active partitions by running `sinfo`.

!!! tip
    You may not have access to all the available partitions.

## Troubleshooting

### Slurm error messages

An incorrect submission will cause Slurm to return an error.
Some common problems are listed below, with a suggestion about
the likely cause:

* ``sbatch: unrecognized option <text>``
  
    One of your options is invalid or has a typo. ``man sbatch`` to help.


* ``error: Batch job submission failed: No partition specified or system default partition``
  
    A ``--partition=`` option is missing. You must specify the partition
    (see the list above). This is most often ``--partition=standard``.

* ``error: invalid partition specified: <partition>``

    ``error: Batch job submission failed: Invalid partition name specified``

    Check the partition exists and check the spelling is correct.


*  ``error: Batch job submission failed: Invalid account or account/partition combination specified``

    This probably means an invalid account has been given. Check the
    ``--account=`` options against valid accounts in SAFE.

* ``error: Batch job submission failed: Invalid qos specification``

    A QoS option is either missing or invalid. Check the script has a
    ``--qos=`` option and that the option is a valid one from the
    table above. (Check the spelling of the QoS is correct.)


* ``error: Your job has no time specification (--time=)...``

    Add an option of the form ``--time=hours:minutes:seconds`` to the
    submission script. E.g., ``--time=01:30:00`` gives a time limit of
    90 minutes.

* ``error: QOSMaxWallDurationPerJobLimit``
    ``error: Batch job submission failed: Job violates accounting/QOS policy``
    ``(job submit limit, user's size and/or time limits)``
  
    The script has probably specified a time limit which is too long for
    the corresponding QoS. E.g., the time limit for the short QoS
    is 20 minutes.


### Slurm queued reasons

The ``squeue`` command allows users to view information for jobs managed by Slurm. Jobs
typically go through the following states: PENDING, RUNNING, COMPLETING, and COMPLETED.
The first table provides a description of some job state codes. The second table provides a description
of the reasons that cause a job to be in a state.


| Status        | Code | Description |
|---------------|------|-------------|
| PENDING       | PD   | Job is awaiting resource allocation. |
| RUNNING       | R    | Job currently has an allocation. |
| SUSPENDED     | S    | Job currently has an allocation. |
| COMPLETING    | CG   | Job is in the process of completing. Some processes on some nodes may still be active. |
| COMPLETED     | CD   | Job has terminated all processes on all nodes with an exit code of zero. |
| TIMEOUT       | TO   | Job terminated upon reaching its time limit. |
| STOPPED       | ST   | Job has an allocation, but execution has been stopped with SIGSTOP signal. CPUS have been retained by this job. |
| OUT_OF_MEMORY | OOM  | Job experienced out of memory error. |
| FAILED        | F    | Job terminated with non-zero exit code or other failure condition. |
| NODE_FAIL     | NF   | Job terminated due to failure of one or more allocated nodes. |
| CANCELLED     | CA   | Job was explicitly cancelled by the user or system administrator. The job may or may not have been initiated. |

For a full list of see [Job State Codes](https://slurm.schedmd.com/squeue.html#lbAG).

| Reason | Description |
|--------|-------------|
| Priority | One or more higher priority jobs exist for this partition or advanced reservation. |
| Resources | The job is waiting for resources to become available. |
| BadConstraints | The job's constraints can not be satisfied. |
| BeginTime | The job's earliest start time has not yet been reached. |
| Dependency | This job is waiting for a dependent job to complete. |
| Licenses | The job is waiting for a license. |
| WaitingForScheduling | No reason has been set for this job yet. Waiting for the scheduler to determine the appropriate reason. |
| Prolog | Its PrologSlurmctld program is still running. |
| JobHeldAdmin | The job is held by a system administrator. |
| JobHeldUser | The job is held by the user. |
| JobLaunchFailure | The job could not be launched. This may be due to a file system problem, invalid program name, etc. |
| NonZeroExitCode | The job terminated with a non-zero exit code. |
| InvalidAccount | The job's account is invalid. |
| InvalidQOS | The job's QOS is invalid. |
| QOSUsageThreshold | Required QOS threshold has been breached. |
| QOSJobLimit | The job's QOS has reached its maximum job count. |
| QOSResourceLimit | The job's QOS has reached some resource limit. |
| QOSTimeLimit | The job's QOS has reached its time limit. |
| NodeDown | A node required by the job is down. |
| TimeLimit | The job exhausted its time limit. |
| ReqNodeNotAvail | Some node specifically required by the job is not currently available. The node may currently be in use, reserved for another job, in an advanced reservation, DOWN, DRAINED, or not responding. Nodes which are DOWN, DRAINED, or not responding will be identified as part of the job's "reason" field as "UnavailableNodes". Such nodes will typically require the intervention of a system administrator to make available. |

For a full list of see [Job Reasons](https://slurm.schedmd.com/squeue.html#lbAF).

## Output from Slurm jobs

Slurm places standard output (STDOUT) and standard error (STDERR) for
each job in the file `slurm_<JobID>.out`. This file appears in the job's
working directory once your job starts running.

!!! hint
    Output may be buffered - to enable live output, e.g. for monitoring
	job status, add `--unbuffered` to the `srun` command in your SLURM
	script.

## Specifying resources in job scripts

You specify the resources you require for your job using directives at
the top of your job submission script using lines that start with the
directive `#SBATCH`.

!!! hint
    Most options provided using `#SBATCH` directives can also be specified as command line options to `srun`.

If you do not specify any options, then the default for each option will
be applied. As a minimum, all job submissions must specify the budget
that they wish to charge the job too with the option:

   - `--account=<budgetID>` your budget ID is usually something like
     `t01` or `t01-test`. You can see which budget codes you can charge
     to in SAFE.

Other common options that are used are:

   - `--time=<hh:mm:ss>` the maximum walltime for your job. *e.g.* For
     a 6.5 hour walltime, you would use `--time=6:30:0`.
   - `--job-name=<jobname>` set a name for the job to help identify it
     in

In addition, parallel jobs will also need to specify how many nodes,
parallel processes and threads they require.

   - `--nodes=<nodes>` the number of nodes to use for the job.
   - `--tasks-per-node=<processes per node>` the number of parallel
     processes (e.g. MPI ranks) per node. For Grid this will typically be 4 to give 1 MPI process per GPU
   - `--cpus-per-task=8` for Grid jobs where you typically use 1 MPI process per GPU, 4 per node, this will usually be 8 (so that the 32 cores on a node are evenly divided between the 4 MPI processes)
   - `--gres=gpu:4` the number of GPU to use per node. This will almost always be 4 to use all GPUs on a node

!!! note
    For parallel jobs, Tursa operates in a *node exclusive* way. This
    means that you are assigned resources in the units of full compute nodes
    for your jobs (*i.e.* 32 cores and 4 GPU) and that no other user can share those
    compute nodes with you. Hence, the minimum amount of resource you can
    request for a parallel job is 1 node (or 32 cores and 4 GPU).

To prevent the behaviour of batch scripts being dependent on the user
environment at the point of submission, the option

   - `--export=none` prevents the user environment from being exported
     to the batch system.

Using the `--export=none` means that the behaviour of batch submissions
should be repeatable. We strongly recommend its use, although see
[the following section](scheduler.md#using-modules-in-the-batch-system-the-epcc-job-env-module)
to enable access to the usual modules.

## `mpirun`: Launching parallel jobs

If you are running parallel jobs, your job submission script should
contain one or more `mpirun` commands to launch the parallel executable
across the compute nodes. You will usually add the following options to
`mpirun`:

- `-np <number of MPI processes>`: specify the number of MPI processes
  to launch
- `--map-by-numa`
- `-x $LD_LIBRARY_PATH`: ensure that the libray paths are available to MPI processes
- `--bind-to none`

## Example job submission scripts

A subset of example job submission scripts for the Grid program are included in full below.

### Example: job submission script for Grid parallel job using CUDA

A job submission script for a Grid job that uses 4 compute nodes, 16 MPI
processes per node and 4 GPUs per node:

```
#!/bin/bash

# Slurm job options (job-name, compute nodes, job time)
#SBATCH --job-name=Example_Grid_job
#SBATCH --time=12:0:0
#SBATCH --nodes=4
#SBATCH --tasks-per-node=4
#SBATCH --cpus-per-task=8
#SBATCH --gres=gpu:4

# Replace [budget code] below with your budget code (e.g. t01)
#SBATCH --account=[budget code]             

# Load the correct modules

ACC_THREADS=8

export OMP_NUM_THREADS=8

# Settings for MPI performance
export OMPI_MCA_btl=^uct,openib
export UCX_TLS=rc,rc_x,sm,cuda_copy,cuda_ipc,gdr_copy
export UCX_RNDV_THRESH=16384
export UCX_RNDV_SCHEME=put_zcopy
export UCX_IB_GPU_DIRECT_RDMA=yes
export UCX_MEMTYPE_CACHE=n

export OMPI_MCA_io=romio321
export OMPI_MCA_btl_openib_allow_ib=true
export OMPI_MCA_btl_openib_device_type=infiniband
export OMPI_MCA_btl_openib_if_exclude=mlx5_1,mlx5_2,mlx5_3

options=" --grid @grid@ --mpi @mpi@ --shm 2048 --shm-hugepages --accelerator-threads ${ACC_THREADS} --comms-overlap --decomposition --log Error,Warning,Message"

mpirun -np $SLURM_NTASKS --map-by numa -x LD_LIBRARY_PATH --bind-to none ./wrapper.sh ${application} ${options}"
```


This will run your executable "my\_mpi\_executable.x" in parallel on 512
MPI processes using 4 nodes (128 cores per node, i.e. not using
hyper-threading). Slurm will allocate 4 nodes to your job and srun will
place 128 MPI processes on each node (one per physical core).

See above for a more detailed discussion of the different `sbatch`
options

### Example: job submission script for MPI+OpenMP (mixed mode) parallel job

Mixed mode codes that use both MPI (or another distributed memory
parallel model) and OpenMP should take care to ensure that the shared
memory portion of the process/thread placement does not span more than
one NUMA region. Nodes on Tursa are made up of two sockets each
containing 4 NUMA regions of 16 cores, i.e. there are 8 NUMA regions in
total. Therefore the total number of threads should ideally not be
greater than 16, and also needs to be a factor of 16. Sensible choices
for the number of threads are therefore 1 (single-threaded), 2, 4, 8,
and 16. More information about using OpenMP and MPI+OpenMP can be found
in the Tuning chapter.

To ensure correct placement of MPI processes the number of cpus-per-task
needs to match the number of OpenMP threads, and the number of
tasks-per-node should be set to ensure the entire node is filled with
MPI tasks.

In the example below, we are using 4 nodes for 6 hours. There are 32 MPI
processes in total (8 MPI processes per node) and 16 OpenMP threads per
MPI process. This results in all 128 physical cores per node being used.

!!! hint
    Note the use of the `export OMP_PLACES=cores` environment option to
    generate the correct thread pinning.

```
#!/bin/bash

# Slurm job options (job-name, compute nodes, job time)
#SBATCH --job-name=Example_MPI_Job
#SBATCH --time=0:20:0
#SBATCH --nodes=4
#SBATCH --tasks-per-node=8
#SBATCH --cpus-per-task=16

# Replace [budget code] below with your project code (e.g. t01)
#SBATCH --account=[budget code] 
#SBATCH --partition=standard
#SBATCH --qos=standard

# Setup the job environment (this module needs to be loaded before any other modules)
module load epcc-job-env

# Set the number of threads to 16 and specify placement
#   There are 16 OpenMP threads per MPI process
#   We want one thread per physical core
export OMP_NUM_THREADS=16
export OMP_PLACES=cores

# Launch the parallel job
#   Using 32 MPI processes
#   8 MPI processes per node
#   16 OpenMP threads per MPI process
#   Additional srun options to pin one thread per physical core
srun --hint=nomultithread --distribution=block:block ./my_mixed_executable.x arg1 arg2
```

## Job arrays

The Slurm job scheduling system offers the *job array* concept, for
running collections of almost-identical jobs. For example, running the
same program several times with different arguments or input data.

Each job in a job array is called a *subjob*. The subjobs of a job array
can be submitted and queried as a unit, making it easier and cleaner to
handle the full set, compared to individual jobs.

All subjobs in a job array are started by running the same job script.
The job script also contains information on the number of jobs to be
started, and Slurm provides a subjob index which can be passed to the
individual subjobs or used to select the input data per subjob.

### Job script for a job array

As an example, the following script runs 56 subjobs, with the subjob
index as the only argument to the executable. Each subjob requests a
single node and uses all 128 cores on the node by placing 1 MPI process
per core and specifies 4 hours maximum runtime per subjob:

    #!/bin/bash
    # Slurm job options (job-name, compute nodes, job time)
    #SBATCH --job-name=Example_Array_Job
    #SBATCH --time=04:00:00
    #SBATCH --nodes=1
    #SBATCH --tasks-per-node=128
    #SBATCH --cpus-per-task=1
    #SBATCH --array=0-55

    # Replace [budget code] below with your budget code (e.g. t01)
    #SBATCH --account=[budget code]  
    #SBATCH --partition=standard
    #SBATCH --qos=standard

    # Setup the job environment (this module needs to be loaded before any other modules)
    module load epcc-job-env

    # Set the number of threads to 1
    #   This prevents any threaded system libraries from automatically 
    #   using threading.
    export OMP_NUM_THREADS=1

    srun --distribution=block:block --hint=nomultithread /path/to/exe $SLURM_ARRAY_TASK_ID

### Submitting a job array

Job arrays are submitted using `sbatch` in the same way as for standard
jobs:

    sbatch job_script.pbs 

## Job chaining

Job dependencies can be used to construct complex pipelines or chain
together long simulations requiring multiple steps.

!!! hint
    The `--parsable` option to `sbatch` can simplify working with job
    dependencies. It returns the job ID in a format that can be used as the
    input to other commands.

For example:

    jobid=$(sbatch --parsable first_job.sh)
    sbatch --dependency=afterok:$jobid second_job.sh

or for a longer chain:

    jobid1=$(sbatch --parsable first_job.sh)
    jobid2=$(sbatch --parsable --dependency=afterok:$jobid1 second_job.sh)
    jobid3=$(sbatch --parsable --dependency=afterok:$jobid1 third_job.sh)
    sbatch --dependency=afterok:$jobid2,afterok:$jobid3 last_job.sh

## Using multiple `srun` commands in a single job script

You can use multiple `srun` commands within in a Slurm job submission script
to allow you to use the resource requested more flexibly. For example, you 
could run a collection of smaller jobs within the requested resources or
you could even subdivide nodes if your individual calculations do not scale
up to use all 128 cores on a node.

In this guide we will cover two scenarios:

 1. Subdividing the job into multiple full-node or multi-node subjobs, e.g.
    requesting 100 nodes and running 100, 1-node subjobs or 50, 2-node 
    subjobs.
 2. Subdividing the job into multiple subjobs that each use a fraction of a
    node, e.g. requesting 2 nodes and running 256, 1-core subjobs or 16,
    16-core subjobs.

### Running multiple, full-node subjobs within a larger job

When subdivding a larger job into smaller subjobs you typically need to 
overwrite the `--nodes` option to `srun` and add the `--ntasks` option
to ensure that each subjob runs on the correct number of nodes and that
subjobs are placed correctly onto separate nodes.

For example, we will show how to request 100 nodes and then run 100
separate 1-node jobs, each of which use 128 MPI processes and which
run on a different compute node. We start by showing 
the job script that would achieve this and then explain how this works
and the options used. In our case, we will run 100 copies of the `xthi` 
program that prints the process placement on the node it is running on.

```slurm
#!/bin/bash

# Slurm job options (job-name, compute nodes, job time)
#SBATCH --job-name=multi_xthi
#SBATCH --time=0:20:0
#SBATCH --nodes=100
#SBATCH --tasks-per-node=128
#SBATCH --cpus-per-task=1

# Replace [budget code] below with your budget code (e.g. t01)
#SBATCH --account=[budget code]             
#SBATCH --partition=standard
#SBATCH --qos=standard

# Setup the job environment (this module needs to be loaded before any other modules)
module load epcc-job-env

# Load the xthi module
module load xthi

# Set the number of threads to 1
#   This prevents any threaded system libraries from automatically 
#   using threading.
export OMP_NUM_THREADS=1

# Loop over 100 subjobs starting each of them on a separate node
for i in $(seq 1 100)
do
   # Launch this subjob on 1 node, note nodes and ntasks options and & to place subjob in the background
   srun --nodes=1 --ntasks=128 --distribution=block:block --hint=nomultithread xthi > placement${i}.txt &
done
# Wait for all background subjobs to finish
wait
```

Key points from the example job script:

- The `#SBATCH` options select 100 full nodes in the usual way.
- Each subjob `srun` command sets the following:
    - `--nodes=1` We need override this setting from the main job so that each subjob only uses 1 node
    - `--ntasks=128` For normal jobs, the number of parallel tasks (MPI processes) is calculated from
      the number of nodes you request and the number of tasks per node. We need to explicitly tell `srun`
      how many we require for this subjob.
    - `--distribution=block:block --hint=nomultithread` These options ensure correct placement of
      processes within the compute nodes.
    - `&` Each subjob `srun` command ends with an ampersand to place the process in the background
      and move on to the next loop iteration (and subjob submission). Without this, the script would
      wait for this subjob to complete before moving on to submit the next.
- Finally, there is the `wait` command to tell the script to wait for all the background subjobs
to complete before exiting. If we did not have this in place, the script would exit as soon as the
last subjob was submitted and kill all running subjobs.


## Interactive Jobs

### Using `salloc` to reserve resources

When you are developing or debugging code you often want to run many
short jobs with a small amount of editing the code between runs. This
can be achieved by using the login nodes to run MPI but you may want to
test on the compute nodes (e.g. you may want to test running on multiple
nodes across the high performance interconnect). One of the best ways to
achieve this on Tursa is to use interactive jobs.

An interactive job allows you to issue `srun` commands directly from the
command line without using a job submission script, and to see the
output from your program directly in the terminal.

You use the `salloc` command to reserve compute nodes for interactive
jobs.

To submit a request for an interactive job reserving 8 nodes (1024
physical cores) for 20 minutes on the short queue you would issue the
following command from the command line:

    auser@login01:> salloc --nodes=8 --tasks-per-node=128 --cpus-per-task=1 \
                  --time=00:20:00 --partition=standard --qos=short \
                  --reservation=shortqos --account=[budget code]

When you submit this job your terminal will display something like:

    salloc: Granted job allocation 24236
    salloc: Waiting for resource configuration
    salloc: Nodes nid000002 are ready for job
    auser@login01:>

It may take some time for your interactive job to start. Once it runs
you will enter a standard interactive terminal session (a new shell).
Note that this shell is still on the front end (the prompt has not
change). Whilst the interactive session lasts you will be able to run
parallel jobs on the compute nodes by issuing the `srun
--distribution=block:block --hint=nomultithread` command directly at 
your command prompt using the same syntax as you would inside a job
script. The maximum number of nodes you can use is limited by resources
requested in the `salloc` command.

If you know you will be doing a lot of intensive debugging you may find
it useful to request an interactive session lasting the expected length
of your working session, say a full day.

Your session will end when you hit the requested walltime. If you wish
to finish before this you should use the `exit` command - this will
return you to your prompt before you issued the `salloc` command.

### Using `srun` directly

A second way to run an interactive job is to use `srun` directly in the
following way (here using the "short queue"):

    auser@login01:/work/t01/t01/auser> srun --nodes=1 --exclusive --time=00:20:00 \
                   --partition=standard --qos=short --reservation=shortqos \
                   --pty /bin/bash
    auser@login01:/work/t01/t01/auser> hostname
    nid001261

The `--pty /bin/bash` will cause a new shell to be started on the first
node of a new allocation (note that while the shell prompt has not
changed, we are now on the compute node). This is perhaps closer to what
many people consider an 'interactive' job than the method using `salloc`
appears.

One can now issue shell commands in the usual way. A further invocation
of `srun` is required to launch a parallel job in the allocation.

When finished, type `exit` to relinquish the allocation and control will
be returned to the front end.

By default, the interactive shell will retain the environment of the
parent. If you want a clean shell, remember to specify `--export=none`.
If you need to
[use modules within your job](scheduler.md#using-modules-in-the-batch-system-the-epcc-job-env-module),
you will need to start a login shell by passing the `--login` argument
to `bash`.

## Heterogeneous jobs

The SLURM submissions discussed above involve a single executable image.
However, there are situtions where two or more distinct executables are
coupled and need to be run at the same time. This is most easily handled
via the SLURM heterogeneous job mechanism.

The essential feature of a heterogeneous job is to create a single batch
submission which specifies the resource requirements for the individual
components. Schematically, we would use

```
#!/bin/bash

# SLURM specifications for the first component

#SBATCH --partition=standard

...

#SBATCH hetjob

# SLURM specifications for the second component

#SBATCH --partition=standard

...

```
where new each component beyond the first is introduced by the special
token `#SBATCH hetjob` (note this is not a normal option and is not
`--hetjob`). Each component must specify a partition.

Such a job will appear in the queue system as, e.g.,
```
           50098+0  standard qscript-    user  PD       0:00      1 (None) 
           50098+1  standard qscript-    user  PD       0:00      2 (None) 
```
and counts as (in this case) two separate jobs from the point of
QoS limits.

Two common cases are discussed below: first, a client server model in
which client and server each have a different `MPI_COMM_WORLD`, and second
the case were two or more executables share `MPI_COMM_WORLD`.

### Heterogeneous jobs for a client/server model

Consider a case where we have two executables which may both be parallel (in
that they use MPI), both run at the same time, and communicate with each
other by some means other than MPI. In the following example, we run two
different executables, both of which must finish before the jobs completes.

```
#!/bin/bash

#SBATCH --time=00:20:00
#SBATCH --exclusive
#SBATCH --export=none

#SBATCH --partition=standard
#SBATCH --qos=standard

#SBATCH --nodes=1
#SBATCH --ntasks-per-node=8

#SBATCH hetjob

#SBATCH --partition=standard
#SBATCH --qos=standard

#SBATCH --nodes=2
#SBATCH --ntasks-per-node=4


# Run two execuatables with separate MPI_COMM_WORLD

srun --distribution=block:block --hint=nomultithread --het-group=0 ./xthi-a &
srun --distribution=block:block --hint=nomultithread --het-group=1 ./xthi-b &
wait
```
In this case, each executable is launched with a separate call to
`srun` but specifies a different heterogeneous group via the
`--het-group` option. The first group is `--het-group=0`.
Both are run in the background with `&` and the `wait` is required
to ensure both executables have completed before the job submission
exits.

In this rather artificial example, where each component makes a
simple report about its placement, the output might be
```
Node    0, hostname nid001028, mpi   4, omp   1, executable xthi-b
Node    1, hostname nid001048, mpi   4, omp   1, executable xthi-b
Node    0, rank    0, thread   0, (affinity =    0)
Node    0, rank    1, thread   0, (affinity =    1)
Node    0, rank    2, thread   0, (affinity =    2)
Node    0, rank    3, thread   0, (affinity =    3)
Node    1, rank    4, thread   0, (affinity =    0)
Node    1, rank    5, thread   0, (affinity =    1)
Node    1, rank    6, thread   0, (affinity =    2)
Node    1, rank    7, thread   0, (affinity =    3)
Node    0, hostname nid001027, mpi   8, omp   1, executable xthi-a
Node    0, rank    0, thread   0, (affinity =    0)
Node    0, rank    1, thread   0, (affinity =    1)
Node    0, rank    2, thread   0, (affinity =    2)
Node    0, rank    3, thread   0, (affinity =    3)
Node    0, rank    4, thread   0, (affinity =    4)
Node    0, rank    5, thread   0, (affinity =    5)
Node    0, rank    6, thread   0, (affinity =    6)
Node    0, rank    7, thread   0, (affinity =    7)
```
Here we have the first executable running on one node with
a communicator size 8 (ranks 0-7). The second executable runs on
two nodes also with communicator size 8 (ranks 0-7, 4 ranks per node).
Further examples of placement for heterogenenous jobs are given below.


### Heterogeneous jobs for a shared `MPI_COM_WORLD`

If two or more heterogeneous components need to share a unique
`MPI_COMM_WORLD`, a single `srun` invocation with the differrent
components separated by a colon `:` should be used. For example,

```
#!/bin/bash

#SBATCH --time=00:20:00
#SBATCH --exclusive
#SBATCH --export=none

#SBATCH --partition=standard
#SBATCH --qos=standard

#SBATCH --nodes=1
#SBATCH --ntasks-per-node=8

#SBATCH hetjob

#SBATCH --partition=standard
#SBATCH --qos=standard

#SBATCH --nodes=2
#SBATCH --ntasks-per-node=4

srun --distribution=block:block --hint=nomultithread --het-group=0 ./xthi-a : \
     --distribution=block:block --hint=nomultithread --het-group=1 ./xthi-b
```

The output should confirm we have a single `MPI_COMM_WORLD` with
ranks 0-15.
```
Node    0, hostname nid001027, mpi   8, omp   1, executable xthi-a
Node    1, hostname nid001028, mpi   4, omp   1, executable xthi-b
Node    2, hostname nid001048, mpi   4, omp   1, executable xthi-b
Node    0, rank    0, thread   0, (affinity =    0)
Node    0, rank    1, thread   0, (affinity =    1)
Node    0, rank    2, thread   0, (affinity =    2)
Node    0, rank    3, thread   0, (affinity =    3)
Node    0, rank    4, thread   0, (affinity =    4)
Node    0, rank    5, thread   0, (affinity =    5)
Node    0, rank    6, thread   0, (affinity =    6)
Node    0, rank    7, thread   0, (affinity =    7)
Node    1, rank    8, thread   0, (affinity =    0)
Node    1, rank    9, thread   0, (affinity =    1)
Node    1, rank   10, thread   0, (affinity =    2)
Node    1, rank   11, thread   0, (affinity =    3)
Node    2, rank   12, thread   0, (affinity =    0)
Node    2, rank   13, thread   0, (affinity =    1)
Node    2, rank   14, thread   0, (affinity =    2)
Node    2, rank   15, thread   0, (affinity =    3)
```

### Heterogeneous placement for mixed MPI/OpenMP work

Some care may be required for placement of tasks/threads in heterogeneous
jobs in which the number of threads needs to be specified differently
for different components.

In the following we have two components. The
first component runs 8 MPI tasks each with 16 OpenMP threads.
The second component runs 8 MPI tasks with
one task per NUMA region on one node; each task has one thread.
An appropriate SLURM submission might be:

```
#!/bin/bash

#SBATCH --time=00:20:00
#SBATCH --exclusive
#SBATCH --export=none

# First component 

#SBATCH --partition=standard
#SBATCH --qos=standard

#SBATCH --nodes=1
#SBATCH --ntasks-per-node=8
#SBATCH --cpus-per-task=16
#SBATCH --hint=nomultithread

# Second component

#SBATCH hetjob

#SBATCH --partition=standard

#SBATCH --nodes=1
#SBATCH --ntasks-per-node=8
#SBATCH --cpus-per-task=16

# Do not set OMP_NUM_THREADS in the calling environment

unset OMP_NUM_THREADS
export OMP_PROC_BIND=spread

srun --het-group=0 --export=all,OMP_NUM_THREADS=16 ./xthi-a : \
     --het-group=1 --export=all,OMP_NUM_THREADS=1  ./xthi-b

```
The important point here is that `OMP_NUM_THREADS` must not be set
in the environment that calls `srun` in order that the different
specifications for the separate groups via `--export` on the `srun`
command line take effect. If `OMP_NUM_THREADS` is set in the calling
environment, then that value takes precedence, and each component will
see the same value of `OMP_NUM_THREADS`.

The output would be:
```
Node    0, hostname nid001111, mpi   8, omp  16, executable xthi-a
Node    1, hostname nid001126, mpi   8, omp   1, executable xthi-b
Node    0, rank    0, thread   0, (affinity =    0)
Node    0, rank    0, thread   1, (affinity =    1)
Node    0, rank    0, thread   2, (affinity =    2)
Node    0, rank    0, thread   3, (affinity =    3)
Node    0, rank    0, thread   4, (affinity =    4)
Node    0, rank    0, thread   5, (affinity =    5)
Node    0, rank    0, thread   6, (affinity =    6)
Node    0, rank    0, thread   7, (affinity =    7)
Node    0, rank    0, thread   8, (affinity =    8)
Node    0, rank    0, thread   9, (affinity =    9)
Node    0, rank    0, thread  10, (affinity =   10)
Node    0, rank    0, thread  11, (affinity =   11)
Node    0, rank    0, thread  12, (affinity =   12)
Node    0, rank    0, thread  13, (affinity =   13)
Node    0, rank    0, thread  14, (affinity =   14)
Node    0, rank    0, thread  15, (affinity =   15)
Node    0, rank    1, thread   0, (affinity =   16)
Node    0, rank    1, thread   1, (affinity =   17)
...
Node    0, rank    7, thread  14, (affinity =  126)
Node    0, rank    7, thread  15, (affinity =  127)
Node    1, rank    8, thread   0, (affinity =    0)
Node    1, rank    9, thread   0, (affinity =   16)
Node    1, rank   10, thread   0, (affinity =   32)
Node    1, rank   11, thread   0, (affinity =   48)
Node    1, rank   12, thread   0, (affinity =   64)
Node    1, rank   13, thread   0, (affinity =   80)
Node    1, rank   14, thread   0, (affinity =   96)
Node    1, rank   15, thread   0, (affinity =  112)
```

## Low priority access

Low priority jobs are not charged against your allocation but will only run when
other, higher-priority, jobs cannot be run or there are no higher-priority jobs in
the queue. Although low priority jobs are not charged, you do need a valid, positive
budget to be able to submit and run low priority jobs, i.e. you need at least 1 CU
in your budget.

Low priority access is always available and has the following limits:

- 256 node maximum job size
- 256 nodes maximum in use by any one user
- 512 nodes maximum in use by low priority at any one time
- Maximum 4 low priority jobs in the queue per user
- Maximum 1 low priority job running per user (of the 4 queued)
- Maximum runtime of 3 hours

You submit a low priority job on Tursa by using the `lowpriority` QoS. For example,
you would usually have the following line in your job submission script sbatch 
options:

```
#SBATCH --qos=lowpriority
```

## Reservations

Reservations are available on Tursa. These allow users to reserve a number of nodes
for a specified length of time starting at a particular time on the system.

Reservations require justification. They will only be approved if the request could not
be fulfilled with the standard queues. For instance, you require a job/jobs to run at a
particular time e.g. for a demonstration or course.

!!! note
    Reservation requests must be submitted at least 60 hours in advance of the reservation
    start time. If requesting a reservation for a Monday at 18:00, please ensure this is
    received by the Friday at 12:00 the latest. The same applies over Service Holidays.

!!! note
    Reservations are only valid for standard compute nodes, high memory compute nodes
    and/or PP nodes cannot be included in reservations.

Reservations will be charged at 1.5 times the usual CU rate and our policy is that they
will be charged the full rate for the entire reservation at the time of booking, whether
or not you use the nodes for the full time. In addition, you will not be refunded the
CUs if you fail to use them due to a job crash unless this crash is due to a system failure.

!!! bug
    At the moment, we are only able to charge for jobs in reservations, not for the full 
    reservation itself. Jobs in reservations are charged at 1.5x the standard rate.

To request a reservation please [contact the Tursa Service Desk](mailto:support@tursa.ac.uk).
You need to provide the following:

 - The start time and date of the reservation.
 - The end time and date of the reservation.
 - The project code for the reservation.
 - The number of nodes required.
 - Your justification for the reservation -- this must be provided or the request will be rejected.

Your request will be checked by the Tursa User Administration team and, if approved, you will be provided a reservation ID which can be used on the system. To submit jobs to a reservation, you need to add `--reservation=<reservation ID>` to your job submission script or command.

!!! important
    You must have at least 1 CU in the budget to submit a job on Tursa, even to a pre-paid reservation.

!!! tip
    You can submit jobs to a reservation as soon as the reservation has been set up; jobs will remain queued until the reservation starts.

## Best practices for job submission

This guidance is adapted from [the advice provided by
NERSC](https://docs.nersc.gov/jobs/best-practices/)

### Time Limits

Due to backfill scheduling, short and variable-length jobs generally
start quickly resulting in much better job throughput. You can specify a
minimum time for your job with the `--time-min` option to SBATCH:

    #SBATCH --time-min=<lower_bound>
    #SBATCH --time=<upper_bound>

Within your job script, you can get the time remaining in the job with
`squeue -h -j ${Slurm_JOBID} -o %L` to allow you to deal with
potentially varying runtimes when using this option.

### Long Running Jobs

Simulations which must run for a long period of time achieve the best
throughput when composed of many small jobs using a checkpoint and
restart method chained together (see above for how to chain jobs
together). However, this method does occur a startup and shutdown
overhead for each job as the state is saved and loaded so you should
experiment to find the best balance between runtime (long runtimes
minimise the checkpoint/restart overheads) and throughput (short
runtimes maximise throughput).

### I/O performance

### Large Jobs

Large jobs may take longer to start up. The `sbcast` command is
recommended for large jobs requesting over 1500 MPI tasks. By default,
Slurm reads the executable on the allocated compute nodes from the
location where it is installed; this may take long time when the file
system (where the executable resides) is slow or busy. The `sbcast`
command, the executable can be copied to the `/tmp` directory on each of
the compute nodes. Since `/tmp` is part of the memory on the compute
nodes, it can speed up the job startup time.

```
sbcast --compress=lz4 /path/to/exe /tmp/exe
srun /tmp/exe
```