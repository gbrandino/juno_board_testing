======================
Juno board benchmark 
======================

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

     make -f make_mpich

In both cases you will get some undefined references to MPI non-blocking collective operations
( such as ``MPI_Iallreduce``), and advanced MPI one-sided communications ( such has ``MPI_Win_flush``). These functions are part of the MPI 3.0 standard, which is supported 
from OpenMPI 1.8. 
In detail: 
 
-  MPI non-blocking collective: OpenMPI >= 1.7
-  MPI advanced one-sided:  OpenMPI >= 1.8 (Fully MPI 3.0 compliant)

However the juno boards currently use OpenMPI 1.6.5. 

The only make target that uses non-blocking collective functions is ``NBC``, while the one
using advanced one-sided is ``RMA``. ( This means, no explicit calls over RDMA fabric? )
Issuing 
::

  make -f make_mpich IMB-IO IMB-EXT IMB-MPI1

or, if you modified the othe make file, 
::

  make -f make_ict IMB-IO IMB-EXT IMB-MPI1


RDAM over sockets
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
    - single trip latency: ~4 us
    - bandwith: 1300MB/s

- internode ( both with and without sockets ove RDMA)
    - single trip latency: ~150 us
    - bandwith: ~100 MB/s


intranode
=========
::

  mpirun -np 2  /home/exactlab/imb/src/IMB-MPI1 PingPong
  

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




internode
==========
::

  mpirun -host junoC,junoD  /home/exactlab/imb/src/IMB-MPI1 PingPong

 
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
.. _`mpi_latency.c`: ./mpi_latency.c
.. _`latency_intranode.txt`: ./data/latency_intranode.txt
.. _`latency_internode.txt`: ./data/latency_internode.txt
.. _`latency_internode_Sockets.txt`: ./data/latency_internode_Sockets.txt



