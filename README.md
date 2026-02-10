# Infiniband-rdma_rxe

How it all started:
Roie told me about the ucx project, RDMA, and Infiniband.  Network throughput and performance is pretty cool stuff and has always fascinated me.  

I pulled and built the ucx master branch on a Raspberry Pi 4 board and a Red Hat 8.10 system.  

Researching RDMA and Infiniband more led me to RDMA over coverged ethernet (ROCE), and I priced out some cool NICs on ebay.  Unfortunately those are outside of my play budget.  I did learn about the Infiniband rdma_rxe module though.

So I want to install the Infiniband rdma_rxe module on Raspberry Pi 4 to trick the Pi into thinking it has RDMA support to play with ucx and measure performance.

The Ubuntu man page nicely summarizes what this is: https://manpages.ubuntu.com/manpages/jammy/man7/rxe.7.html

"The rdma_rxe kernel module provides a software implementation of the RoCEv2 protocol.
The RoCEv2 (RDMA over Converged Ethernet) protocol is  an  RDMA  transport  protocol
that  exists  on top of UDP/IPv4 or UDP/IPv6.
The InfiniBand (IB) Base Transport Header (BTH) is encapsulated in the UDP packet."

Issue #1:
The rdma_rxe module was not available from any repos I could find for the Pi 4.

After pulling and building the UCX project on a Red Hat 8.10 system and a Raspberry Pi 4 board, I rand the hello world examples, and I really wanted to use perf and the ucx/ucx/src/tools/perf/ucx_perftest to generate some flamegraphs for different perftest options.  

Issue #2:
Finding perf and associated tools available for the Raspberry Pi was a lost effort after running several upgrades and updates for the past couple years.

Easiest way to get both the perf tools and the rdma_rxe module is to just build a kernel natively on the Pi, just takes some time.

Building Infiniband rdma_rxe module, and the perf tools, and a lot of other new and fun sounding modules:
mkdir Kernel
cd Kernel
git clone --depth=1 https://github.com/raspberrypi/linux
cd linux
make bcm2711_defconfig 
make prepare
make modules_prepare

then:
make menuconfig 
locate the Infiniband rdma support and select all the neat things

then:
make -j4 Image.gz modules dtbs

sudo su
make modules_install
cp arch/arm64/boot/dts/broadcom/*.dtb /boot/
cp arch/arm64/boot/dts/overlays/*.dtb* /boot/overlays/
cp arch/arm64/boot/dts/overlays/README /boot/overlays/
cp arch/arm64/boot/Image.gz /boot/kernel8.img

after a reboot, check the new kernel:
fred@galileo:~ $ uname -r
6.12.68-v8+
fred@galileo:~ $ 

Kernel looks good, try the rdma_rxe module:
fred@galileo:~ $ sudo modprobe rdma_rxe
fred@galileo:~ $ lsmod | grep rxe
rdma_rxe              122880  0
ib_uverbs             163840  1 rdma_rxe
ib_core               413696  2 rdma_rxe,ib_uverbs
ip6_udp_tunnel         12288  1 rdma_rxe
udp_tunnel             24576  1 rdma_rxe
fred@galileo:~ $ 

check for and then set up the (software) RDMA link to eth0 on the Pi:
fred@galileo:~ $ rdma link show
fred@galileo:~ $ sudo rdma link add rxe0 type rxe netdev eth0
fred@galileo:~ $ rdma link show
link rxe0/1 state ACTIVE physical_state LINK_UP netdev eth0 
fred@galileo:~ $ 


