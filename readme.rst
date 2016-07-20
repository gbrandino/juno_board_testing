======================
Juno board benchmark 
======================

.. contents:: Table of Contents


Intel MPI Benchmark 4.1
=======================

To complie you can either 

1. Edit the ``imb/src/make_mpich``
   ::

     MPI_HOME=/usr

   then 
   ::

     make -f make_mpich
 
2. Edit the ``imb/src/make_ict``
   ::

     MPICC=mpicc 

   then
   ::

     make -f make_ict

In both cases you will get some undefined references to MPI non-blocking collective operations
( such as ``MPI_Iallreduce``), and advanced MPI one-sided communications ( such has ``MPI_Win_flush``). These functions are part of the MPI 3.0 standard, which is supported 
from OpenMPI 1.8. 
In detail: 
 
-  MPI non-blocking collective: OpenMPI >= 1.7
-  MPI advanced one-sided:  OpenMPI >= 1.8 (Fully MPI 3.0 compliant)

However the juno boards currently use OpenMPI 1.6.5. 

The only make target that uses non-blocking collective functions is ``NBC``, while the one
using advanced one-sided is ``RMA``. 
Issuing 
::

  make -f make_mpich IMB-IO IMB-EXT IMB-MPI1

or, if you modified the othe make file, 
::

  make -f make_ict IMB-IO IMB-EXT IMB-MPI1


RDMA over sockets
=================

To run rdma over sockets, first run an ssh deamon on a port else than 22, using unimem ``libc`` and ``libpthread``
::

  sudo LD_LIBRARY_PATH=/unimem/lib/sockets /usr/sbin/sshd -f /etc/ssh/sshd_config -p 50000 -r

On the other node
::

  LD_LIBRARY_PATH=/unimem/lib/sockets mpirun --mca orte_rsh_agent "ssh -p 50000" -x LD_LIBRARY_PATH --host 192.168.1.12,192.168.1.13  imb/src/IMB-MPI1

In this way, system calls will be intercepeted by the unimem-modified ``libc`` and ``libpthread``

==========
Benchmarks
==========

The complete set of benchmarks for IMB-MPI1 (MPI 1.0 two-sided ) and MPI-EXT (MPI 2.0 one-side) are include in this repos.

The test are seraparete in 

- intranode --> communication between cores on the same board
- internode --> communication betwenn cores on different boards
- internode_SOckets --> communication betwenn cores on different boards, using the unimem libraries intercept

Here is the list of file ( locate in the ``data`` subfolder )

- `IMB-MPI1_intranode.txt`_
- `IMB-MPI1_internode.txt`_
- `IMB-MPI1_internode_Socket.txt`_

- `IMB-EXT_intranode.txt`_
- `IMB-EXT_internode.txt`_
- `IMB-EXT_internode_Socket.txt`_

Latency has been also measuread with another code, `mpi_latency.c`_

- `latency_intranode.txt`_
- `latency_internode.txt`_
- `latency_internode_Sockets.txt`_

What emerges from the benchmarks is 

- intranode
    - single trip latency: ~2-4 us
    - bandwith: 1300-3000MB/s

  depending on process binding


- internode ( both with and without sockets over RDMA)
    - single trip latency: ~150 us
    - bandwith: ~100 MB/s

Reference Intel CPU + Mellanox Infiniband

- intranode
    - single trip latency: ~0.25-0.5 us
    - bandwith: 5300-7300MB/s

  depending on process binding


- internode ( both with and without sockets over RDMA)
    - single trip latency: ~1 us
    - bandwith: ~100 MB/s
 


intranode cortex A53 socket
===========================
::

  mpirun -np 2 taskset -c 0,3 /home/exactlab/imb/src/IMB-MPI1 PingPong
  
(same result with taskset 0,4/0,5/3,5 etc)
::

  # PingPong

  #---------------------------------------------------
  # Benchmarking PingPong 
  # #processes = 2 
  #---------------------------------------------------
       #bytes #repetitions      t[usec]   Mbytes/sec
            0         1000         3.60         0.00
            1         1000         4.58         0.21
            2         1000         4.57         0.42
            4         1000         4.56         0.84
            8         1000         4.57         1.67
           16         1000         4.51         3.39
           32         1000         4.51         6.77
           64         1000         4.69        13.02
          128         1000         4.83        25.28
          256         1000         5.04        48.44
          512         1000         5.47        89.21
         1024         1000         6.14       158.94
         2048         1000         7.48       261.27
         4096         1000        14.04       278.30
         8192         1000        18.00       434.05
        16384         1000        26.87       581.55
        32768         1000        44.53       701.84
        65536          640        70.01       892.73
       131072          320       119.85      1042.99
       262144          160       224.11      1115.52
       524288           80       405.44      1233.23
      1048576           40       603.37      1657.34
      2097152           20      1209.72      1653.27
      4194304           10      2374.49      1684.57


intranode cortex A72 socket
===========================
::

  mpirun -np 2 taskset -c 1,2 /home/exactlab/imb/src/IMB-MPI1 PingPong
  

  # PingPong

  #---------------------------------------------------
  # Benchmarking PingPong 
  # #processes = 2 
  #---------------------------------------------------
       #bytes #repetitions      t[usec]   Mbytes/sec
            0         1000         1.96         0.00
            1         1000         2.30         0.41
            2         1000         2.30         0.83
            4         1000         2.29         1.66
            8         1000         2.30         3.32
           16         1000         2.31         6.61
           32         1000         2.44        12.53
           64         1000         2.38        25.61
          128         1000         2.45        49.84
          256         1000         2.60        93.77
          512         1000         2.80       174.30
         1024         1000         3.50       279.07
         2048         1000         4.07       480.00
         4096         1000         7.03       555.53
         8192         1000         9.69       805.90
        16384         1000        15.74       992.57
        32768         1000        25.86      1208.50
        65536          640        40.96      1525.90
       131072          320        68.89      1814.52
       262144          160       121.67      2054.76
       524288           80       230.58      2168.48
      1048576           40       578.71      1727.98
      2097152           20      1187.40      1684.35
      4194304           10      1320.51      3029.14


intranode - intersocket
===========================
::

  mpirun -np 2 taskset -c 0,1 /home/exactlab/imb/src/IMB-MPI1 PingPong

(same result with taskset 0,2/1,5/2,5 etc)
::

  # PingPong

  #---------------------------------------------------
  # Benchmarking PingPong 
  # #processes = 2 
  #---------------------------------------------------
       #bytes #repetitions      t[usec]   Mbytes/sec
            0         1000         3.13         0.00
            1         1000         3.72         0.26
            2         1000         3.71         0.51
            4         1000         3.68         1.04
            8         1000         3.73         2.05
           16         1000         3.66         4.17
           32         1000         3.99         7.65
           64         1000         4.03        15.15
          128         1000         4.31        28.30
          256         1000         4.53        53.88
          512         1000         4.80       101.66
         1024         1000         5.62       173.89
         2048         1000         7.01       278.56
         4096         1000        12.55       311.15
         8192         1000        15.64       499.54
        16384         1000        22.58       692.00
        32768         1000        37.72       828.36
        65536          640        44.67      1399.26
       131072          320        66.26      1886.53
       262144          160       122.46      2041.45
       524288           80       262.74      1902.99
      1048576           40       646.56      1546.64
      2097152           20      1445.45      1383.65
      4194304           10      2895.05      1381.67



internode
==========
::

  mpirun -host 192.168.1.12,192.168.1.13  /home/exactlab/imb/src/IMB-MPI1 PingPong

 
  # PingPong

  #---------------------------------------------------
  # Benchmarking PingPong 
  # #processes = 2 
  #---------------------------------------------------
       #bytes #repetitions      t[usec]   Mbytes/sec
            0         1000       168.87         0.00
            1         1000       157.73         0.01
            2         1000       152.24         0.01
            4         1000       152.38         0.03
            8         1000       152.46         0.05
           16         1000       152.64         0.10
           32         1000       153.07         0.20
           64         1000       154.63         0.39
          128         1000       156.16         0.78
          256         1000       159.14         1.53
          512         1000       166.53         2.93
         1024         1000       180.35         5.41
         2048         1000       156.71        12.46
         4096         1000       160.02        24.41
         8192         1000       233.09        33.52
        16384         1000       304.85        51.25
        32768         1000       469.14        66.61
        65536          640      1054.98        59.24
       131072          320      1595.81        78.33
       262144          160      2766.06        90.38
       524288           80      5071.94        98.58
      1048576           40      9646.45       103.67
      2097152           20     18744.83       106.70
      4194304           10     36954.80       108.24


internode, sockets over RDMA
=============================
::

  LD_LIBRARY_PATH=/unimem/lib/sockets mpirun --mca orte_rsh_agent "ssh -p 50000" -x LD_LIBRARY_PATH --host 192.168.1.12,192.168.1.13  imb/src/IMB-MPI1 PingPong

 
  # PingPong

  #---------------------------------------------------
  # Benchmarking PingPong 
  # #processes = 2 
  #---------------------------------------------------
       #bytes #repetitions      t[usec]   Mbytes/sec
            0         1000       179.73         0.00
            1         1000       164.07         0.01
            2         1000       160.36         0.01
            4         1000       156.96         0.02
            8         1000       157.08         0.05
           16         1000       157.18         0.10
           32         1000       157.66         0.19
           64         1000       159.05         0.38
          128         1000       160.63         0.76
          256         1000       163.78         1.49
          512         1000       171.48         2.85
         1024         1000       185.10         5.28
         2048         1000       163.93        11.91
         4096         1000       170.66        22.89
         8192         1000       238.15        32.81
        16384         1000       324.67        48.13
        32768         1000       470.85        66.37
        65536          640      1120.55        55.78
       131072          320      1607.64        77.75
       262144          160      2771.58        90.20
       524288           80      5072.18        98.58
      1048576           40      9629.32       103.85
      2097152           20     18746.67       106.69
      4194304           10     36914.00       108.36



=========================
Playing with OpenMPI BTL
=========================

The transport layer used effects a lot the latency and the bandwidth.

For example, in the intranode case, running using ``sm`` ( default in the intranode case) 
::

  mpirun -np 2 --mca btl self,sm /home/exactlab/imb/src/MPI-MPI1 PingPong 

equivalent to 
::

  mpirun -np 2 /home/exactlab/imb/src/MPI-MPI1 PingPong 


  # PingPong

  #---------------------------------------------------
  # Benchmarking PingPong 
  # #processes = 2 
  #---------------------------------------------------
       #bytes #repetitions      t[usec]   Mbytes/sec
            0         1000         3.29         0.00
            1         1000         4.03         0.24
            2         1000         4.10         0.47
            4         1000         4.04         0.94
            8         1000         4.05         1.88
           16         1000         4.03         3.79
           32         1000         4.13         7.39
           64         1000         4.23        14.42
          128         1000         4.41        27.66
          256         1000         4.56        53.57
          512         1000         4.93        99.11
         1024         1000         5.86       166.66
         2048         1000         7.40       263.81
         4096         1000        12.98       301.05
         8192         1000        16.06       486.53
        16384         1000        22.86       683.45
        32768         1000        38.56       810.37
        65536          640        60.99      1024.77
       131072          320       106.30      1175.95
       262144          160       193.67      1290.86
       524288           80       395.12      1265.44
      1048576           40       626.48      1596.23
      2097152           20      1414.55      1413.88
      4194304           10      2886.55      1385.74

while using ``tcp``
::

  mpirun -np 2 --mca btl self,tcp /home/exactlab/imb/src/MPI-MPI1 PingPon

gives
::

  # PingPong

  #---------------------------------------------------
  # Benchmarking PingPong 
  # #processes = 2 
  #---------------------------------------------------
       #bytes #repetitions      t[usec]   Mbytes/sec
            0         1000        44.26         0.00
            1         1000        46.48         0.02
            2         1000        46.53         0.04
            4         1000        46.52         0.08
            8         1000        46.55         0.16
           16         1000        46.46         0.33
           32         1000        23.56         1.30
           64         1000        23.63         2.58
          128         1000        23.89         5.11
          256         1000        24.08        10.14
          512         1000        24.55        19.89
         1024         1000        25.16        38.81
         2048         1000        27.79        70.27
         4096         1000        29.37       133.00
         8192         1000        32.82       238.06
        16384         1000        39.14       399.16
        32768         1000        61.99       504.12
        65536          640       145.80       428.66
       131072          320       226.63       551.56
       262144          160       314.83       794.07
       524288           80       570.02       877.16
      1048576           40      1114.86       896.97
      2097152           20      2174.32       919.83
      4194304           10      4266.39       937.56

``mip_latency.c`` gives similar results 
::

  mpirun  -np 2 --mca btl self,sm /home/exactlab/mpi_latency.x

gives
::

  *** Avg round trip time = 11 microseconds
  *** Avg one way latency = 5 microseconds

while
::

  mpirun  -np 2 --mca btl self,tcp /home/exactlab/mpi_latency.x

gives
::

  *** Avg round trip time = 99 microseconds
  *** Avg one way latency = 49 microseconds  

This means that the ``tcp`` stack is wasting a lot of time. A native transport layer, or maybe ``openib`` (OpenFabrics) compliant layer, will
probably greatly enhance performance.  


.. _`IMB-MPI1_intranode.txt` : ./data/IMB-MPI1_intranode.txt
.. _`IMB-MPI1_internode.txt` : ./data/IMB-MPI1_internode.txt
.. _`IMB-MPI1_internode_Socket.txt` : ./data/IMB-MPI1_internode_Socket.txt
.. _`IMB-EXT_intranode.txt` : ./data/IMB-EXT_intranode.txt
.. _`IMB-EXT_internode.txt`: ./data/IMB-EXT_internode.txt
.. _`IMB-EXT_internode_Socket.txt`: ./data/IMB-EXT_internode_Socket.txt
.. _`mpi_latency.c`: ./code/mpi_latency.c
.. _`latency_intranode.txt`: ./data/latency_intranode.txt
.. _`latency_internode.txt`: ./data/latency_internode.txt
.. _`latency_internode_Sockets.txt`: ./data/latency_internode_Sockets.txt

===========================
Reference INTEL + Mellanox
===========================

Thie tests are run on the cosilt infrastructure.
The nodes are dual socket Intel Xeon E5-2697, 12 cores per socket. 
Apparently this uses as byte trasport layer (``btl``)  ``vader`` for intranode communication and ``openib`` for internode. 

intranode - intrasocket
=======================
::

  mpirun -np 2 --map-by core IMB-MPI1 PingPong

  # PingPong

  #---------------------------------------------------
  # Benchmarking PingPong 
  # #processes = 2 
  #---------------------------------------------------
       #bytes #repetitions      t[usec]   Mbytes/sec
            0         1000         0.25         0.00
            1         1000         0.27         3.48
            2         1000         0.28         6.89
            4         1000         0.28        13.70
            8         1000         0.28        27.59
           16         1000         0.28        54.89
           32         1000         0.28       107.65
           64         1000         0.57       107.27
          128         1000         0.53       230.73
          256         1000         0.61       402.59
          512         1000         0.67       725.60
         1024         1000         0.83      1176.67
         2048         1000         1.13      1727.54
         4096         1000         1.97      1979.34
         8192         1000         2.68      2914.52
        16384         1000         4.54      3438.59
        32768         1000         7.63      4093.83
        65536          640        11.79      5301.19
       131072          320        21.58      5793.24
       262144          160        39.48      6332.58
       524288           80        73.71      6783.06
      1048576           40       143.19      6983.81
      2097152           20       279.10      7165.92
      4194304           10       550.15      7270.73


  # All processes entering MPI_Finalize

``mpi_latency.x`` gives
::

  *** Avg round trip time = 0 microseconds
  *** Avg one way latency = 0 microseconds


intranode - intersocket
=======================
::

    mpirun -np 2 --map-by socket IMB-MPI1 PingPong

  # PingPong

  #---------------------------------------------------
  # Benchmarking PingPong 
  # #processes = 2 
  #---------------------------------------------------
       #bytes #repetitions      t[usec]   Mbytes/sec
            0         1000         0.47         0.00
            1         1000         0.49         1.93
            2         1000         0.50         3.80
            4         1000         0.50         7.63
            8         1000         0.49        15.44
           16         1000         0.50        30.67
           32         1000         0.50        60.56
           64         1000         1.09        56.13
          128         1000         1.00       122.25
          256         1000         1.06       231.07
          512         1000         1.12       435.74
         1024         1000         1.22       803.06
         2048         1000         1.59      1230.31
         4096         1000         3.09      1263.13
         8192         1000         4.19      1866.54
        16384         1000         6.68      2339.94
        32768         1000        11.56      2703.99
        65536          640        16.56      3775.04
       131072          320        29.82      4191.53
       262144          160        54.01      4628.58
       524288           80       101.34      4933.68
      1048576           40       196.34      5093.27
      2097152           20       384.65      5199.58
      4194304           10       755.49      5294.58


  # All processes entering MPI_Finalize

``mpi_latency.x`` gives
::

  *** Avg round trip time = 1 microseconds
  *** Avg one way latency = 0 microseconds


internode
============ 
::

  mpirun -np 2 --map-by node IMB-MPI1 PingPong

  # PingPong

  #---------------------------------------------------
  # Benchmarking PingPong 
  # #processes = 2 
  #---------------------------------------------------
       #bytes #repetitions      t[usec]   Mbytes/sec
            0         1000         1.08         0.00
            1         1000         1.12         0.85
            2         1000         1.12         1.70
            4         1000         1.12         3.40
            8         1000         1.15         6.61
           16         1000         1.17        13.09
           32         1000         1.19        25.69
           64         1000         1.25        48.85
          128         1000         1.92        63.73
          256         1000         2.05       119.12
          512         1000         2.39       203.95
         1024         1000         2.92       334.15
         2048         1000         4.04       483.33
         4096         1000         4.98       785.11
         8192         1000         6.95      1124.58
        16384         1000        10.21      1530.43
        32768         1000        15.70      1989.93
        65536          640        26.76      2335.63
       131072          320        48.76      2563.54
       262144          160        90.56      2760.70
       524288           80       176.64      2830.64
      1048576           40       348.71      2867.70
      2097152           20       692.77      2886.95
      4194304           10      1383.76      2890.68


  # All processes entering MPI_Finalize

``mpi_latency.x`` gives
::

  *** Avg round trip time = 3 microseconds
  *** Avg one way latency = 1 microseconds
