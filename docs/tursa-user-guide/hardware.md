# ARCHER2 hardware

!!! note
    Some of the material in this section is closely based on [information provided by NASA](https://www.nas.nasa.gov/hecc/support/kb/amd-rome-processors_658.html) as part of the documentation for the [Aitkin HPC system](https://www.nas.nasa.gov/hecc/resources/aitken.html).

## System overview

Tursa is a Eviden supercomputing system which has a total of 178 GPU compute nodes. Each GPU compute node has a CPU with 48 cores and 4 NVIDIA A100 GPU. Compute nodes are connected together by an Infiniband interconnect. 

There are additional login nodes, which provide access to the system.

Compute nodes are only accessible via the Slurm job scheduling system.

There is a single file system which is available on login and compute nodes (see [Data management and transfer](data.md)).

The Lustre file system has a capacity of 5.1 PiB.

The interconnect uses a Fat Tree topology.

## Interconnect details

Tursa has a high performance interconnect with 4x 200 Gb/s infiniband interfaces per node. It uses a 2-layer fat tree topology:

- Each node connects to 4 of the 5 L1 (leaf) switches within the same cabinet with 200 Gb/s links
- Within an 8-node block, all nodes share the same 4 switches
- Each L1 switch connects to all 20 L2 switches via 200 Gb/s links - leading maximum of 2 switch to switch hops to get between any 2 nodes
- There are no direct L1 to L1 or L2 to L2 switch connections
- 16-node, 32-node and 64-node blocks are constructed from 8-node blocks that show the required performance on the inter-block links








