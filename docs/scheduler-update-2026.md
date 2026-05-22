# Tursa scheduler configuration update 2026

!!! note "Last update"
    This information was last updated on 22 May 2026.

In June 2026 the Tursa scheduler configuration will undergo major changes to make
the resource more flexible. This page provides an overview of the planned changes
and what it means for users.

## Summary of user impact

- Strict topology blocking will no longer be enforced on the system
- Users that wish to continue to request strict topology blocking will need to
  use the `--switches` option to `sbatch`/`srun`/`salloc` to request this
- Jobs queued during the migration will retain their strict topology blocking
  requests that were the default at time of submission. Users that want to 
  remove this constraint will need to delete jobs and resubmit

## Overview of Tursa interconnect topology

The current configuration of the scheduler is designed to match onto the
interconnect topology as initial use cases for Tursa were critically dependent
on best interconnect performance which can only by achieved by matching job
layout to interconnect topology. 

Many of the upcoming changes are based on relaxing topology restrictions so we provide
a very brief description of the topology layout here as useful context for the 
following descriptions.

Tursa is composed of 181 compute nodes, all but five of which are contained within blocks
connected to a set of 4 L1 switches. These are divided into

- 14 blocks of 8 nodes of A100-40
- 6 blocks of 8 nodes of A100-80 nodes
- 4 blocks of 4 nodes of A100-80 nodes

All blocks are connected via 20 L2 switches in a fat tree topology.

## What is changing?

### Removal of enforced topology blocking

Currently, all jobs on Tursa are subject to enforced topology blocking. For multi-node
jobs of 8 nodes or less, all the nodes in the job must come from a single block that share
an L1 switch. For jobs larger than 8 nodes, the scheduler is further configured to allocate
jobs into predefined 16, 32 and 64 node blocks based on tested performance between groups of
individual 8 node blocks.

For users, the effect of these restrictions is that single node failures in a block can render
larger jobs very difficult to place and lead to long queue times. If the application being run
has a critical performance dependence on interconnect performance then the blocking makes sense
but many HPC applications do not have their performance limited in such a way.

**This change will remove this enforced blocking so that jobs, by default, can be assigned nodes
from anywhere in the interconnect topology.** This will allow for more flexibility in job placement and
reduce the impact of single node failures on availability of resources.

Users can recover the strict blocking behaviour by using
[the Slurm `--switches` option](https://slurm.schedmd.com/sbatch.html#OPT_switches). See 
below for specific examples of how to use this.

### Removal of power of two job size restriction

Another consequence of the strict interconnect blocking is that jobs are currently restricted
to power of two sizes (to match onto the topology layout).

**This change will remove the restriction on job sizes to power of two.** This will allow for
more flexibility in job placement.

### Changes to priority formula

Along with changes to blocking restrictions, we will be implementing some changes to the job
priority setup:

- **Enable allocation-tied fairshare**: at the moment, all projects have the same number of
  shares on the service. This change will link the number of shares on the system to the size
  of the project's current GPUh allocation. This change will help ensure that projects get
  priorities that allow them to use the level of allocation they have been granted on the service.
- **Update priority weights**: To support the change in allocation-tied fairshare, we will update
  weights for different parts of the priority formula to increase the weight associated with the
  fairshare priority component relative to other components.

## Slurm `--switches` option

Users can use the Slurm `--switches` option to partially recover the job topology blocking 
behaviour from before the change.

!!! note "Static 16-node block or greater setup cannot be recovered"
    In the current configuration, the blocks in the topology that make up 16, 32, 64 or 128
    node blocks are statically defined. In the updated configuration, any combination of 8 node
    blocks can make up larger jobs with strict blocking defined by `--switches` options as 
    described below.

### Default behaviour

In the updated configuration, no interconnect blocking is enforced by default. **If you do not supply
any additional options to your Slurm jobs the scheduler is free to pick nodes from anywhere on the
system.**

### Enforcing blocking: 8-nodes or less

A single interconnect block on the Tursa topology (where all nodes share L1 switches) contains 
8 nodes. If you are running jobs of 8-nodes and less and want to ensure all nodes are in a 
single block (i.e. share a L1 switch) you should add the `--switches=1` option to your Slurm 
submission commands. Most commonly, this would mean adding the following to your batch
submission script:

```
#SBATCH --switches=1
```

!!! important "Adding blocking is likely to increase queue time"
    Adding the additional constraint around interconnect blocking with the `switches` option
    will likely have a detrimental effect on queue time.

### Enforced blocking: 16-nodes or more

Beyond 8-nodes, to ensure strict interconnect blocking, you need to increase the number of
switches along with node count:

| Node count | `switches` option | Notes |
|--:|---|---|
| 16 | `switches=2` | | 
| 32 | `switches=4` | | 
| 64 | `switches=8` | Not available for A100-80 |
| 128 | `switches=16` | Not available for A100-80. Only available when A100-40 and A100-80 nodes are mixed in the job. |

!!! note "Power of 2 job sizes make sense for strict blocking"
    While there will no longer be a restriction to power of two size jobs in the updated
    configuration, it will usually make sense to stick to these sizes if you are wanting
    to request strict interconnect topology blocking.