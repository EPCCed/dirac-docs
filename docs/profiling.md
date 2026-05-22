# Profiling

This section provides brief documentation on how to use the NVIDIA NSight tools 
to profile an application on Tursa. The process is provided as short example using
a simple application.

For full details, see the NVIDIA Nsight documentation:

 - [NVIDIA Nsight Systems](https://docs.nvidia.com/nsight-systems/UserGuide/index.html)
 - [NVIDIA Nsight Compute](https://docs.nvidia.com/nsight-compute/)

!!! important
    The Nsight GUI is not available on Tursa and you cannot connect a local GUI to
    Tursa over SSH due to limitations in the SSH module in the Nsight GUI. If you want
    to visualise profiles, you must download them from Tursa to your local system 
    where you have installed the GUI.

!!! credit
    Thanks to Paul Graham of NVIDIA for agreeing to share this example.

## Example code

Here is the example CUDA code that will be used for this example. In the rest of the 
exercise, we assume you have saved this to a file called `vector-add.cu` on Tursa.

```
#include <stdio.h>

/*
 * Host function to initialize vector elements. This function
 * simply initializes each element to equal its index in the
 * vector.
 */

void initWith(float num, float *a, int N)
{
  for(int i = 0; i < N; ++i)
  {
    a[i] = num;
  }
}

/*
 * Device kernel stores into `result` the sum of each
 * same-indexed value of `a` and `b`.
 */

__global__
void addVectorsInto(float *result, float *a, float *b, int N)
{
  int index = threadIdx.x + blockIdx.x * blockDim.x;
  int stride = blockDim.x * gridDim.x;

  for(int i = index; i < N; i += stride)
  {
    result[i] = a[i] + b[i];
  }
}

/*
 * Host function to confirm values in `vector`. This function
 * assumes all values are the same `target` value.
 */

void checkElementsAre(float target, float *vector, int N)
{
  for(int i = 0; i < N; i++)
  {
    if(vector[i] != target)
    {
      printf("FAIL: vector[%d] - %0.0f does not equal %0.0f\n", i, vector[i], target);
      exit(1);
    }
  }
  printf("Success! All values calculated correctly.\n");
}

int main()
{

  int deviceId;
  int numberOfSMs;

  cudaGetDevice(&deviceId);
  cudaDeviceGetAttribute(&numberOfSMs, cudaDevAttrMultiProcessorCount, deviceId);
  printf("Device ID: %d\tNumber of SMs: %d\n", deviceId, numberOfSMs);
  
  const int N = 2<<24;
  size_t size = N * sizeof(float);

  float *a;
  float *b;
  float *c;

  cudaMallocManaged(&a, size);
  cudaMallocManaged(&b, size);
  cudaMallocManaged(&c, size);

  initWith(3, a, N);
  initWith(4, b, N);
  initWith(0, c, N);

  size_t threadsPerBlock;
  size_t numberOfBlocks;

  /*
   * nsys should register performance changes when execution configuration
   * is updated.
   */

  threadsPerBlock = 256;
  numberOfBlocks = 32 * numberOfSMs;

  cudaError_t addVectorsErr;
  cudaError_t asyncErr;

  addVectorsInto<<<numberOfBlocks, threadsPerBlock>>>(c, a, b, N);

  addVectorsErr = cudaGetLastError();
  if(addVectorsErr != cudaSuccess) printf("Error: %s\n", cudaGetErrorString(addVectorsErr));

  asyncErr = cudaDeviceSynchronize();
  if(asyncErr != cudaSuccess) printf("Error: %s\n", cudaGetErrorString(asyncErr));

  checkElementsAre(7, c, N);

  cudaFree(a);
  cudaFree(b);
  cudaFree(c);
}
```

## Compile the code

Compile the example code:

```
module load nvhpc/23.5-nompi
module load gcc/12.2.0

nvcc -o vector-add.exe vector-add.cu
```

## Test the example

Create a job submission script to run the example code:

```slurm
#!/bin/bash

#SBATCH --job-name=vector-add
#SBATCH --time=0:5:0
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=32
#SBATCH --cpus-per-task=1
#SBATCH --gres=gpu:4
#SBATCH --partition=gpu-a100-40
#SBATCH --qos=dev

#SBATCH --account=[add your budget code]

# Load the correct modules
module load nvhpc/23.5-nompi
module load gcc/12.2.0

./vector-add.exe
```

When you submit this, you should see the code produce output like:

```
Device ID: 0    Number of SMs: 108
Success! All values calculated correctly.
```

## Use Nsight System to generate a profile

Create a job submission script to get a profile of the example application:

```slurm
#!/bin/bash

#SBATCH --job-name=vector-add
#SBATCH --time=0:5:0
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=32
#SBATCH --cpus-per-task=1
#SBATCH --gres=gpu:4
#SBATCH --partition=gpu-a100-40
#SBATCH --qos=dev

#SBATCH --account=[add your budget code]

# Load the correct modules
module load nvhpc/23.5-nompi
module load gcc/12.2.0

nsys profile --stats=true vector-add.exe
```

This should produce output something like:

```
[1/8] [========================100%] report1.nsys-rep
[2/8] [========================100%] report1.sqlite
SKIPPED: /mnt/lustre/tursafs1/home/t01/t01/dc-user1/report1.sqlite does not contain NV Tools Extension (NVTX) data.
[3/8] Executing 'nvtx_sum' stats report
[4/8] Executing 'osrt_sum' stats report

 Time (%)  Total Time (ns)  Num Calls   Avg (ns)    Med (ns)   Min (ns)   Max (ns)   StdDev (ns)          Name         
 --------  ---------------  ---------  ----------  ----------  --------  ----------  -----------  ---------------------
     84.5       3740154532        120  31167954.4  10103030.0      7054  1413120085  130155802.9  poll                 
      8.5        376952738       1065    353946.2     20882.0      1397    29245539    1359316.7  ioctl                
      5.2        230556453        106   2175060.9   2094310.0      1815    20983399    2645399.3  sem_timedwait        
      0.9         38619004          7   5517000.6      8521.0      1467    20053805    9423347.1  fread                
      0.3         15445783         58    266306.6      5168.0      2794    11102745    1479367.5  fopen                
      0.3         13440698         26    516949.9      6425.5      2305     5145173    1446432.8  mmap                 
      0.2          6658153         10    665815.3      1431.5      1396     6644466    2100683.7  dup                  
      0.0          1609153         42     38313.2      8520.0      6635      937695     143076.8  mmap64               
      0.0           621661          4    155415.3    152814.0    121175      194858      30933.7  pthread_create       
      0.0           536320        102      5258.0      1816.0       978      213296      25541.3  fcntl                
      0.0           533379         52     10257.3      2375.0      1885      260160      40144.9  fclose               
      0.0           485683         83      5851.6      5169.0      2794       22349       2604.6  open64               
      0.0           106926         64      1670.7      1467.0       978        2864        389.8  pthread_mutex_trylock
      0.0            94711         29      3265.9      1397.0       908       58668      10657.4  fgets                
      0.0            73122         14      5223.0      4260.0      1816       16552       3478.6  write                
      0.0            54408         11      4946.2      4679.0      2794        7124       1502.6  munmap               
      0.0            49799          7      7114.1      7054.0      2445       13829       3811.4  open                 
      0.0            47074         17      2769.1      2794.0      1815        5168        868.5  read                 
      0.0            23257          3      7752.3      8521.0      4260       10476       3178.5  pipe2                
      0.0            18368          2      9184.0      9184.0      7054       11314       3012.3  socket               
      0.0            11873          1     11873.0     11873.0     11873       11873          0.0  connect              
      0.0             2864          1      2864.0      2864.0      2864        2864          0.0  bind                 
      0.0             1886          1      1886.0      1886.0      1886        1886          0.0  listen               

[5/8] Executing 'cuda_api_sum' stats report

 Time (%)  Total Time (ns)  Num Calls   Avg (ns)     Med (ns)   Min (ns)  Max (ns)   StdDev (ns)           Name         
 --------  ---------------  ---------  -----------  ----------  --------  ---------  -----------  ----------------------
     66.9        326556539          3  108852179.7     69982.0     61601  326424956  188423551.5  cudaMallocManaged     
     17.4         84859556          1   84859556.0  84859556.0  84859556   84859556          0.0  cudaDeviceSynchronize 
     12.9         62931338          1   62931338.0  62931338.0  62931338   62931338          0.0  cudaLaunchKernel      
      2.8         13646243          3    4548747.7   4418958.0   4016391    5210894     607736.3  cudaFree              
      0.0             7054          1       7054.0      7054.0      7054       7054          0.0  cuModuleGetLoadingMode

[6/8] Executing 'cuda_gpu_kern_sum' stats report

 Time (%)  Total Time (ns)  Instances   Avg (ns)    Med (ns)   Min (ns)  Max (ns)  StdDev (ns)                       Name                     
 --------  ---------------  ---------  ----------  ----------  --------  --------  -----------  ----------------------------------------------
    100.0         84862110          1  84862110.0  84862110.0  84862110  84862110          0.0  addVectorsInto(float *, float *, float *, int)

[7/8] Executing 'cuda_gpu_mem_time_sum' stats report

 Time (%)  Total Time (ns)  Count  Avg (ns)  Med (ns)  Min (ns)  Max (ns)  StdDev (ns)              Operation            
 --------  ---------------  -----  --------  --------  --------  --------  -----------  ---------------------------------
     81.9         48978013  10109    4845.0    3455.0      2656     51328       5656.4  [CUDA Unified Memory memcpy HtoD]
     18.1         10801550    768   14064.5    4095.0      2463     79840      21695.6  [CUDA Unified Memory memcpy DtoH]

[8/8] Executing 'cuda_gpu_mem_size_sum' stats report

 Total (MB)  Count  Avg (MB)  Med (MB)  Min (MB)  Max (MB)  StdDev (MB)              Operation            
 ----------  -----  --------  --------  --------  --------  -----------  ---------------------------------
    402.653  10109     0.040     0.008     0.004     1.044        0.135  [CUDA Unified Memory memcpy HtoD]
    134.218    768     0.175     0.033     0.004     1.044        0.301  [CUDA Unified Memory memcpy DtoH]

Generated:
    /mnt/lustre/tursafs1/home/t01/t01/dc-user1/report1.nsys-rep
    /mnt/lustre/tursafs1/home/t01/t01/dc-user1/report1.sqlite
```

You can download the `report1.nsys-rep` file to your local system to load into the Nsight GUI for 
visualisation if you wish.

## Use Nsight Compute to investiage hardware counters

Create a job submission script to get a profile of the hardware counters for the example application:

```slurm
#!/bin/bash

#SBATCH --job-name=vector-add
#SBATCH --time=0:5:0
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=32
#SBATCH --cpus-per-task=1
#SBATCH --gres=gpu:4
#SBATCH --partition=gpu-a100-40
#SBATCH --qos=dev

#SBATCH --account=[add your budget code]

# Load the correct modules
module load nvhpc/23.5-nompi
module load gcc/12.2.0

ncu ./vector-add.exe
```

This should produce output something like:

```
[165308] vector-add.exe@127.0.0.1
  addVectorsInto(float *, float *, float *, int) (3456, 1, 1)x(256, 1, 1), Context 1, Stream 7, Device 0, CC 8.0
    Section: GPU Speed Of Light Throughput
    ----------------------- ------------- ------------
    Metric Name               Metric Unit Metric Value
    ----------------------- ------------- ------------
    DRAM Frequency          cycle/nsecond         1.20
    SM Frequency            cycle/nsecond         1.08
    Elapsed Cycles                  cycle       322427
    Memory Throughput                   %        85.69
    DRAM Throughput                     %        85.69
    Duration                      usecond       299.23
    L1/TEX Cache Throughput             %        17.69
    L2 Cache Throughput                 %        68.64
    SM Active Cycles                cycle    314986.82
    Compute (SM) Throughput             %         9.12
    ----------------------- ------------- ------------

    INF   The kernel is utilizing greater than 80.0% of the available compute or memory performance of the device. To   
          further improve performance, work will likely need to be shifted from the most utilized to another unit.      
          Start by analyzing DRAM in the Memory Workload Analysis section.                                              

    Section: Launch Statistics
    -------------------------------- --------------- ---------------
    Metric Name                          Metric Unit    Metric Value
    -------------------------------- --------------- ---------------
    Block Size                                                   256
    Function Cache Configuration                     CachePreferNone
    Grid Size                                                   3456
    Registers Per Thread             register/thread              26
    Shared Memory Configuration Size           Kbyte           32.77
    Driver Shared Memory Per Block       Kbyte/block            1.02
    Dynamic Shared Memory Per Block       byte/block               0
    Static Shared Memory Per Block        byte/block               0
    Threads                                   thread          884736
    Waves Per SM                                                   4
    -------------------------------- --------------- ---------------

    Section: Occupancy
    ------------------------------- ----------- ------------
    Metric Name                     Metric Unit Metric Value
    ------------------------------- ----------- ------------
    Block Limit SM                        block           32
    Block Limit Registers                 block            8
    Block Limit Shared Mem                block           32
    Block Limit Warps                     block            8
    Theoretical Active Warps per SM        warp           64
    Theoretical Occupancy                     %          100
    Achieved Occupancy                        %        96.24
    Achieved Active Warps Per SM           warp        61.59
    ------------------------------- ----------- ------------

    INF   This kernel's theoretical occupancy is not impacted by any block limit. 
```





