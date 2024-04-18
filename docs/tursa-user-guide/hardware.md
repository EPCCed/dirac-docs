# Tursa hardware

## System overview

Tursa is a Eviden supercomputing system which has a total of 178 GPU compute nodes. Each GPU compute node has a CPU with either 32 or 48 cores and 4 NVIDIA A100 GPU (either A100-40 or A100-80). Compute nodes are connected together by an Infiniband interconnect. The compute nodes are housed in water colled Eviden Sequana XH2000 racks. 

There are additional login nodes, which provide access to the system.

Compute nodes are only accessible via the Slurm job scheduling system.

There is a single Lustre file system which is available on login and compute nodes (see [Data management and transfer](data.md)). The Lustre file system has a capacity of 5.1 PiB. This is supported by a tape storage system which is used for both backup of data from the Lustre file system and as a user controlled archive for storing data that is not in active use. The tape storage has a total capacity of over 20 PiB.

## Compute node details

- 64 GPU nodes with A100-80
     + 4x NVIDIA Ampere A100-80 GPU
     + 2x AMD 7413 EPYC 24c processor
     + NVLink intranode GPU interconnect
     + 4x 200 Gbps NVIDIA Infiniband interfaces
- 114 GPU nodes with A100-40
     + 4x NVIDIA Ampere A100-40 GPU
     + 2x AMD 7302 EPYC 16c processor
     + NVLink intranode GPU interconnect
     + 4x 200 Gbps NVIDIA Infiniband interfaces

## Interconnect details

Tursa has a high performance interconnect with 4x 200 Gb/s infiniband interfaces per node. It uses a 2-layer fat tree topology:

- Each node connects to 4 of the 5 L1 (leaf) switches within the same cabinet with 200 Gb/s links
- Within an 8-node block, all nodes share the same 4 switches
- Each L1 switch connects to all 20 L2 switches via 200 Gb/s links - leading maximum of 2 switch to switch hops to get between any 2 nodes
- There are no direct L1 to L1 or L2 to L2 switch connections
- 16-node, 32-node and 64-node blocks are constructed from 8-node blocks that show the required performance on the inter-block links

### Scheduler configuration

To make sure that jobs have access to the required internode communication performance, jobs are 
constrained to be able to use nodes in coherent 8-node blocks only rather than having free choice of
any nodes on the system. An additional result of this strick blocking is that only job sizes of powers
of 2 are supported on the system. In practice, and due to these additional scheduling constraints, this
means that jobs will tyically wait longer to run and overall utilisation will be lower than on a system
of the same size but with free placement. This also means the system is more sensative to individual node
hardware failures as a single node failure within an 8-node block will make that block unusable for
jobs of 8 nodes or larger (though 1, 2 and 4-node jobs could all potentially run in the degraded 
block).

Resources for job sizes larger than 8-nodes are constructed by combining coherent 8-node blocks. For example,
16-node jobs can only run if there are two intact 8-node blocks available. 








