========================================
OpenMPI 2.0.0 compilation on juno board
========================================

Download OpenMPI 2.0.0.

After unpacking, use ``LDFLAGS`` to force the compilation to link the unimem modified ``libc`` and ``libpthread``. Set also a prefix to a local 
folder
::

  ./configure LDFLAGS=-Wl,--rpath=/unimem/lib/sockets/ --prefix=/home/exactlab/openmpi-2.0.0_local

``configure`` finds as available ``btl``
::

  self
  sm
  tcp 
  openib ( OpenFabrics support for infiniband)
  vader ( modern substitute of sm )

To compile
::

  nohup make all install

Apparently, compilation fails due to a known bug in gcc 4.9.2 --> 5.1.1 that should be solved in gcc 5.2. 
Linaro distribution seems however to support only ( for now ) gcc 4.9

========================================
OpenMPI 1.10.3 compilation on juno board
========================================

After unpacking, use ``LDFLAGS`` to force the compilation to link the unimem modified ``libc`` and ``libpthread``. Set also a prefix to a local 
folder
::

  ./configure LDFLAGS=-Wl,--rpath=/unimem/lib/sockets/ --prefix=/home/exactlab/openmpi-1.10.3_local

``configure`` finds as available ``btl``
::

  self
  sm
  tcp 
  openib ( OpenFabrics support for infiniband)
  vader ( modern substitute of sm )

To compile
::

  nohup make all install

``ldd`` now correctly gives
::

  ldd /home/exactlab/openmpi-1.10.3._local/bin/mpirun

	linux-vdso.so.1 (0x0000007f8b239000)
	libopen-rte.so.12 => /home/exactlab/openmpi-1.10.3._local/lib/libopen-rte.so.12 (0x0000007f8b1b6000)
	libopen-pal.so.13 => /home/exactlab/openmpi-1.10.3._local/lib/libopen-pal.so.13 (0x0000007f8b0f7000)
	libnuma.so.1 => /usr/lib/aarch64-linux-gnu/libnuma.so.1 (0x0000007f8b0cf000)
	libdl.so.2 => /lib/aarch64-linux-gnu/libdl.so.2 (0x0000007f8b0bc000)
	librt.so.1 => /lib/aarch64-linux-gnu/librt.so.1 (0x0000007f8b0a4000)
	libm.so.6 => /lib/aarch64-linux-gnu/libm.so.6 (0x0000007f8b004000)
	libutil.so.1 => /lib/aarch64-linux-gnu/libutil.so.1 (0x0000007f8aff1000)
	libpthread.so.0 => /unimem/lib/sockets/libpthread.so.0 (0x0000007f8afc2000)
	libc.so.6 => /unimem/lib/sockets/libc.so.6 (0x0000007f8ae74000)
	/lib/ld-linux-aarch64.so.1 (0x000000555d028000)

