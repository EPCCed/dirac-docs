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

### GPUh

Time used on Tursa nodes is measured in GPUh.  
1 GPUh = 1 GPU for 1 hour. So a Tursa compute node with 4 GPUs would cost
4 GPUh per hour.

!!! note
    The minimum resource request on Tursa is one full node which is charged 
    at a rate of 4 GPUh per hour.

### Checking available budget

You can check in [SAFE](https://safe.epcc.ed.ac.uk) by selecting `Login accounts` from the menu, select the login account you want to query.

Under `Login account details` you will see each of the budget codes you have access to listed e.g.
`dp123 resources` and then under Resource Pool to the right of this, a note of the remaining budgets. 

When logged in to the machine you can also use the command 

    sacctmgr show assoc where user=$LOGNAME format=account,user,maxtresmins%75

This will list all the budget codes that you have access to e.g.

```
Account       User                                                                 MaxTRESMins 
---------- ---------- --------------------------------------------------------------------------- 
       t01   dc-user1                           gres/cpu-low=0,gres/cpu-standard=0,gres/gpu-low=0 
       z01   dc-user1   
```

This shows that `dc-user1` is a member of budgets `t01` and `z01`.  However, the `gres/cpu-low=0,gres/cpu-standard=0,gres/gpu-low=0` indicates that the `t01` budget can only run GPU jobs in standard (charged) partitions (all other options are disabled, indicated by `=0` for CPU standard, CPU low and GPU low).  This user can also submit jobs to any partition using the `z01` budget.

To see the number of coreh or GPUh remaining you must check in [SAFE](https://safe.epcc.ed.ac.uk/dirac).

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
cpu          up 2-00:00:00      4  alloc tu-c0r0n[66-69]
cpu          up 2-00:00:00      2   idle tu-c0r0n[70-71]
gpu          up 2-00:00:00      1   plnd tu-c0r2n93
gpu          up 2-00:00:00     11  drain tu-c0r0n75,tu-c0r5n[48,51,54,57],tu-c0r6n[48,51,54,57],tu-c0r7n[00,48]
gpu          up 2-00:00:00    112    mix tu-c0r0n[00,03,06,09,12,15,18,21,24,27,30,33,36,39,42,45,72,87,90],tu-c0r1n[00,03,06,09,12,15,18,21,24,27,30,33,60,63,66,69,72,75,78,81,84,87,90,93],tu-c0r2n[00,03,06,09,12,15,18,21,24,27,30,33,60,63,66,69,72,75,78,81,84,87,90],tu-c0r3n[00,03,06,09,12,15,18,21,24,27,30,33,60,63,66,69,72,75,78,81,84,90,93],tu-c0r4n[00,03,06,09,12,15,18,21,24,27,30,33,60,63,66,69,72,75,81,84,87,90,93]
gpu          up 2-00:00:00     56   resv tu-c0r0n93,tu-c0r4n78,tu-c0r5n[00,03,06,09,12,15,18,21,24,27,30,33,36,39,42,45],tu-c0r6n[00,03,06,09,12,15,18,21,24,27,30,33,36,39,42,45,60,63,66,69],tu-c0r7n[03,06,09,12,15,18,21,24,27,30,33,36,39,42,45,51,54,57]
gpu          up 2-00:00:00      1   idle tu-c0r3n87
```

* `alloc` nodes are those that are running jobs
* `idle` nodes are empty
* `drain`, `down`, `maint` nodes are unavailable to users
* `plnd` nodes are reserved for future jobs

### `sbatch`: submitting jobs

`sbatch` is used to submit a job script to the job submission system.
The script will typically contain one or more `mpirun` commands to launch
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
output to just your jobs by adding the `--me` option:

    squeue --me

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
    You will not generally have access to the full amount of memory resource
    on the the node as some is retained for running the operating system and
    other system processes.

### Partitions

On Tursa, compute nodes are grouped into partitions. You will have to
specify a partition using the `--partition` option in your Slurm
submission script. The following table has a list of active partitions
on Tursa.

| Partition | Description                                                 | Max nodes available |
| --------- | ----------------------------------------------------------- | ------------------- |
| cpu  | CPU nodes with AMD EPYC 48-core processor &times; 2    | 6               |
| gpu  | GPU nodes with AMD EPYC 48-core processor and NVIDIA A100 GPU &times; 4 (this includes both A100-40 and A100-80 GPU)  | 181                |
| gpu-a100-40  | GPU nodes with 2 AMD EPYC 16-core processors and NVIDIA A100-40 GPU &times; 4  | 114                |
| gpu-a100-80  | GPU nodes with 2 AMD EPYC 24-core processor (3 nodes have 2 AMD EPYC 16-core processors) and NVIDIA A100-80 GPU &times; 4  | 67                |

You can list the active partitions by running `sinfo`.

!!! tip
    You may not have access to all the available partitions.

### Quality of Service (QoS)

On Tursa, job limits are defined by the requested Quality of Service
(QoS), as specified by the `--qos` Slurm directive. The following table
lists the active QoS on Tursa.

| QoS        | Max Nodes Per Job | Max Walltime | Jobs Queued | Jobs Running | Partition(s) | Notes |
| ---------- | ----------------- | ------------ | ----------- | ------------ | ------------ | ------|
| standard   | 64                | 48 hrs       | 32          | 16           | gpu, gpu-a100-40, gpu-a100-80, cpu     | Only jobs sizes that are powers of 2 nodes are allowed (i.e. 1, 2, 4, 8, 16, 32, 64 nodes), only available when your budget is positive. |
| low        | 64                | 24 hrs       | 4           | 4            | gpu, gpu-a100-40, gpu-a100-40, cpu     | Only jobs sizes that are powers of 2 nodes are allowed (i.e. 1, 2, 4, 8, 16, 32, 64 nodes), only available when your budget is zero or negative |
| dev        | 2                | 4 hrs       | 2           | 1            | gpu     | For faster turnaround for development jobs and interactive sessions, only available when your budget is positive. The dev QoS must be used with the `gpu-a100-40` (1-node maximum) or `gpu-a100-80` (2-node maximum) partitions.  |

You can find out the QoS that you can use by running the following
command:

    sacctmgr show assoc user=$USER cluster=tursa format=cluster,account,user,qos%50

As long as you have a positive budget, you should use the `standard` QoS. Once you have exhausted your
budget you can use the `low` QoS to continue to run jobs at a lower priority than jobs in the 
`standard` QoS.

!!! hint
    If you have needs which do not fit within the current QoS, please
    [contact the Service
    Desk](https://www.archer2.ac.uk/support-access/servicedesk.html) and we
    can discuss how to accommodate your requirements.

!!! important
    Only jobs sizes that are powers of 2 nodes
    are allowed. i.e. 1, 2, 4, 8, 16, 32, 64 nodes on the `gpu` partition and 
    1, 2, 4 nodes on the `cpu` partition.

### Priority

Job priority on Tursa depends on a number of different factors:

 - The QoS your job has specified
 - The amount of time you have been queuing for
 - Your current fairshare factor

Each of these factors is normalised to a value between 0 and 1, is multiplied
with a weight and the resulting values combined to produce a priority for the job. 
The current job priority formula on Tursa is:

```
Priority = [10000 * P(QoS)] + [500 * P(Age)] + [300 * P(Fairshare)]
```

The priority factors are:

- P(QoS) - The QoS priority normalised to a value between 0 and 1. The maximum raw
  value is 10000 and the minimum is 0. `standard` QoS has a value of 5000 and `low`
  QoS a value of 1.
- P(Age) - The priority based on the job age normalised to a value between 0 and 1.
  The maximum raw value is 14 days (where P(Age) = 1).
- P(Fairshare) - The fairshare priority normalised to a value between 0 and 1. Your
  fairshare priority is determined by a combination of your budget code fairshare 
  value and your user fairshare value within that budget code. The more use that 
  the budget code you are using has made of the system recently relative to other 
  budget codes on the system, the lower the budget code fairshare value will be; and the more
  use you have made of the system recently relative to other users within your
  budget code, the lower your user fairshare value will be. The decay half life 
  for fairshare on Tursa is set to 14 days. [More information on the Slurm fairshare
  algorithm](https://slurm.schedmd.com/fair_tree.html).

You can view the priorities for current queued jobs on the system with the `sprio`
command:

```
[dc-user1@tursa-login1 ~]$ sprio 
          JOBID PARTITION   PRIORITY       SITE        AGE  FAIRSHARE        QOS
          43963 gpu             5055          0         51          5       5000
          43975 gpu             5061          0         41         20       5000
          43976 gpu             5061          0         41         20       5000
          43982 gpu             5046          0         26         20       5000
          43986 gpu             5011          0          6          5       5000
          43996 gpu             5020          0          0         20       5000
          43997 gpu             5020          0          0         20       5000
```

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
     processes (e.g. MPI ranks) per node. For Grid on GPU nodes this will
     typically be 4 to give 1 MPI process per GPU. The CPU nodes have 128
     cores per node.
   - `--cpus-per-task=<stride between processes>` for Grid jobs on GPU nodes
     where you typically use 1 MPI process per GPU, 4 per node, this will
     usually be 12 (so that the 48 cores on a node are evenly divided between
     the 4 MPI processes)
   - `--gres=gpu:4` the number of GPU to use per node. This will almost always
     be 4 to use all GPUs on a node. (This option should not be specified for
     jobs on the CPU nodes.)

If you are happy to have any GPU type for your job (A100-40 or A100-80) then you
select the `gpu` partition:

   - `--partition=gpu`

If you wish to use just the A100-80 GPU nodes which have higher memory, you add the
following option:

   - `--partition=gpu-a100-80` request the job is placed on nodes with high-memory
   (80 GB) GPUs - there are 64 high memory GPU nodes on the system. 

To just use the A100-40 GPU nodes:

   - `--partition=gpu-a100-40` request the job is placed on nodes with standard memory
   (40 GB) GPUs.

If you do not specfy a partition, the scheduler may use any available node types for 
the job (equivalent of `--partition=gpu`).

!!! note
    For parallel jobs, Tursa operates in a *node exclusive* way. This
    means that you are assigned resources in the units of full compute nodes
    for your jobs (*i.e.* 32 cores and 4 GPU on GPU A100-40 nodes, 48 cores and 4 GPU on A100-80 nodes, 128 cores on CPU nodes)
    and that no other user can share those compute nodes with you. Hence,
    the minimum amount of resource you can request for a parallel job is 1 node
    (or 32 cores and 4 GPU on GPU A100-40 nodes, 48 cores and 4 GPU on A100-80 nodes, 128 cores on CPU nodes).

To prevent the behaviour of batch scripts being dependent on the user
environment at the point of submission, the option

   - `--export=none` prevents the user environment from being exported
     to the batch system.

Using the `--export=none` means that the behaviour of batch submissions
should be repeatable. We strongly recommend its use, although see
[the following section](scheduler.md#using-modules-in-the-batch-system-the-epcc-job-env-module)
to enable access to the usual modules.

### GPU frequency

!!! important
    The default GPU frequency on Tursa compute nodes was changed from 1410 MHz
    to 1040 MHz on Thursday 15 Dec 2022 to improve the energy efficiency of the
    service.

Users can control the GPU frequency in their job submission scripts:

   - `--gpu-freq=<desired GPU freq in MHz>` allows users to set the GPU frequency 
     on a per job basis. The frequency can be set in the range 210 - 1410 MHz in steps
     of 15 MHz.

!!! bug
    When setting the GPU frequency you will see an error in the output from the job 
    that says `control disabled`. This is an incorrect message due to an issue with 
    how Slurm sets the GPU frequency and can be safely ignored.

## `srun`: Launching parallel jobs

If you are running parallel jobs, your job submission script should contain one or
more srun commands to launch the parallel executable across the compute nodes. In
most cases you will want to add the following options to `srun`:

- `--nodes=[number of nodes]` - Set the number of compute nodes for this job step
- `--ntasks-per-node=[MPI processes per node]` - This will usually be `4` for GPU jobs 
  as you usually have 1 MPI process per GPU
- `--cpus-per-task=[stride between MPI processes]` - This will usually be either `8`
   (for A100-40 nodes) or `12` (for A100-80 nodes). If you are using the `gpu` QoS
   where you can get any type of GPU node, you will usually se this to `8`.
- `--distribution=block:block` - do not use hyperthreads/SMP
- `--hint=nomultithread`- the first `block` means use a block distribution
   of processes across nodes (i.e. fill nodes before moving onto the next one) and
   the second `block` means use a block distribution of processes across "sockets"
   within a node (i.e. fill a "socket" before moving on to the next one).

!!! important
    The Slurm definition of a "socket" does not usually correspond to a physical CPU socket.
    On Tursa GPU nodes it corresponds to half the cores on a socket as the GPU nodes
    are configured with NPS2.

    On the Tursa CPU nodes, the Slurm definition of a scoket does correspond to a physical
    CPU socket (64 cores) as the COU nodes are configured with NPS1.

## Example job submission scripts

The typical strategy for submitting josb on Tursa is for the batch script to 
request full nodes with no process/thread pinning and then the individual 
`srun` commands set the correct options for dividing up processes and threads
across nodes.

### Example: job submission script for a parallel job using CUDA

A job submission script for a parallel job that uses 4 compute nodes, 4 MPI
processes per node and 4 GPUs per node. It does not restrict what type of
GPU the job can run on so both A100-40 and A100-80 can be used.

```slurm
#!/bin/bash

# Slurm job options
#SBATCH --job-name=Example_MPI_job
#SBATCH --time=12:0:0
#SBATCH --partition=gpu
#SBATCH --qos=gpu
# Replace [budget code] below with your budget code (e.g. t01)
#SBATCH --account=[budget code]  

# Request right number of full nodes (32 cores by node fits any GPU compute nodes))
#SBATCH --nodes=4
#SBATCH --ntasks-per-node=32
#SBATCH --cpus-per-task=1
#SBATCH --gres=gpu:4

# Load the correct modules
module load /home/y07/shared/tursa-modules/setup-env
module load gcc/9.3.0
module load cuda/12.3
module load openmpi/4.1.5-cuda12.3 

export OMP_NUM_THREADS=8
export OMP_PLACES=cores

# These will need to be changed to match the actual application you are running
application="my_mpi_openmp_app.x"
options="arg 1 arg2"

# We have reserved the full nodes, now distribute the processes as
# required: 4 MPI processes per node, stride of 8 cores between 
# MPI processes
# 
# Note use of gpu_launch.sh wrapper script for GPU and NIC pinning 
srun --nodes=4 --ntasks-per-node=4 --cpus-per-task=8 \
     --hint=nomultithread --distribution=block:block \
     gpu_launch.sh \
     ${application} ${options}
```

This will run your executable "my_mpi_opnemp_app.x" in parallel usimg 16
MPI processes on 4 nodes. 4 GPUs will be used per node.

!!! important
    You must use the `gpu_launch.sh` wrapper script to get the correct biniding
    of GPU to MPI processes and of network interface to GPU and MPI process.
    This script is described in more detail below.

### `gpu_launch.sh` wrapper script

The `gpu_launch.sh` wrapper script is required to set the correct binding of
GPU to MPI processes and the correct binding of interconnect interfaces to 
MPI process and GPU. We provide this centrally for convenience but its contents
are simple:

```
#!/bin/bash

# Compute the raw process ID for binding to GPU and NIC
lrank=$((SLURM_PROCID % SLURM_NTASKS_PER_NODE))
 
# Bind the process to the correct GPU and NIC
export CUDA_VISIBLE_DEVICES=${lrank}
export UCX_NET_DEVICES=mlx5_${lrank}:1
 
$@
```

## Using the `dev` QoS

The `dev` QoS is designed for faster turnaround of short jobs than is usually available through
the production QoS. It is subject to a number of restrictions:

* 4 hour maximum walltime
* Maximum job size:
    * 2 nodes for `gpu-a100-80` partition
    * 1 node for `gpu-a100-40` partition
* Maximum 1 job running per user
* Maximum 2 jobs queued per user
* Only available to projects with a positive budget

In addtion, you *must* specify either the `gpu-a100-80` or `gpu-a100-40` partitions when using the
`dev` QoS.

!!! tip
    The generic `gpu` partition will not work consistently when using the `dev` QoS.

Here is an example job submission script for a 2-node job in the `dev` QoS using the `gpu-a100-80` 
partition. Note the use of the `gpu_launch.sh` wrapper script to get correct GPU and NIC
binding.

```slurm
#!/bin/bash

# Slurm job options
#SBATCH --job-name=Example_MPI_job
#SBATCH --time=12:0:0
#SBATCH --partition=gpu-a100-80
#SBATCH --qos=dev
# Replace [budget code] below with your budget code (e.g. t01)
#SBATCH --account=[budget code]  

# Request right number of full nodes (48 cores by node for A100-80 GPU nodes))
#SBATCH --nodes=4
#SBATCH --ntasks-per-node=48
#SBATCH --cpus-per-task=1
#SBATCH --gres=gpu:4

export OMP_NUM_THREADS=1
export OMP_PLACES=cores

# Load the correct modules
module load /home/y07/shared/tursa-modules/setup-env
module load gcc/9.3.0
module load cuda/12.3
module load openmpi/4.1.5-cuda12.3 

# These will need to be changed to match the actual application you are running
application="my_mpi_openmp_app.x"
options="arg 1 arg2"

# We have reserved the full nodes, now distribute the processes as
# required: 4 MPI processes per node, stride of 12 cores between 
# MPI processes
# 
# Note use of gpu_launch.sh wrapper script for GPU and NIC pinning 
srun --nodes=4 --ntasks-per-node=4 --cpus-per-task=12 \
     --hint=nomultithread --distribution=block:block \
     gpu_launch.sh \
     ${application} ${options}
```
