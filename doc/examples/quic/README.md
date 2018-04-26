Ivy specification of QUIC
=========================

This directory contains work on specifying the QUIC protocol in Ivy.
The currently targeted version is 9, as described in [this
document](https://tools.ietf.org/html/draft-ietf-quic-transport-09).

The specification is written in a way that allows monitoring of
packets on the wire, and will eventually allow for modular testing of
implementations and possibly a fully verified implementation.

The Ivy spec is being developed from the informal IETF draft cited
above. Ambuiguities are resovled by observing the behavior of existing
implementations. In particular, we use the evolving specification to
monitor packet traces captured from implementations.  This allows us
to check consistency and possibly discover incompatibilities between
implementations.

Setup for virtual networking and packet capture
-----------------------------------------------

To monitor implementations of the protocol, it is useful to run then
in a virtual network environent. For this we use the [CORE virtual
networking
environment](https://www.nrl.navy.mil/itd/ncs/products/core), the
`tcpdump` command and the `pcap` library. To install these on an Ubuntu
system with version 14.04 or higher, do the following:

    sudo apt-get install core-network tcpdump libpcap-dev

Use the following command in this directory to set up a suitable
virtual network on your system:

    sudo ./vnet_setup.sh

Implementations of QUIC
-----------------------

### Google QUIC

The Google implementation of QUIC used in the Chromium browser is not
compatible with the IETF draft.

### MinQUIC

This implementation of verion 9 in the go language is available [on
github](https://github.com/ekr/minq). However, for Ivy you should use
the fork [here](https://github.com/kenmcmil/minq) which has been
modified to disable crypto.

#### Steps to get started with MinQUIC and Ivy

- Install the [go language](https://golang.org/doc/install) on your platform.
- Follow the instructions [here](https://github.com/kenmcmil/minq) to install MinQUIC.

#### Running MinQUIC and capturing packets

Change to the directory containin MinQUIC:

    cd $GOPATH/src/githuib.com/kenmcmil/minq

Create three terminals, A, B and C.

Terminal A: run a server in node `n0`:
    
    sudo vcmd -c /tmp/n0.ctl -- `which go` run `pwd`/bin/server/main.go --addr=10.0.0.1:4433 --server-name=10.0.0.1

Terminal B: trace packets with `tcpdump`:

    sudo tcpdump -s 0 -i veth0 -w mycap.pcap

Terminal C: run a client in node `n1`:

    sudo vcmd -c /tmp/n1.ctl -- `which go` run `pwd`/bin/client/main.go --addr=10.0.0.1:4433

Text typed into terminal C should appear on terminal A. When finishes,
kill the `tcpdump` process in terminal B. You should now have a file
`mycap.pcap` containing captured packets.

Build and run the Ivy monitor
-----------------------------

To build the Ivy monitor, change to this directory and compile `quic_monitor.ivy` like this:

    ivyc quic_monitor.ivy

This should create a binary file `quic_monitor`. Copy `mycap.pcap` into this directory and then do:

    ./quic_monitor mycap.pcap > log.iev

The file `log.iev` should have lines like this:

    < show_packet({protocol:udp,addr:0xa000002,port:0x869b},{protocol:udp,addr:0xa000001,port:0x1151},{hdr_long:0x1,hdr_type:0x7f,hdr_cid:0x7c74846907e4ce90,hdr_version:0xff000009,hdr_pkt_num:0x3dee3059,payload:[{frame.stream:{off:0x1,len:0x1,fin:0,id:0,offset:0,length:0x282,data:[0x16,0x3,0x3,...]}}]})

These are decoded packets. Each line consists of a source endpoint, a
destination endpoint and a packet. The structure of packets is
described in [quic_packet.ivy](quic_packet.md).

If the specification is violated by the packet trace, the file will
end with an error message indicating the requirement that was
violated. 

View the log
------------

View the log with the following command:

    ivy_ev_viewer log.iev

Useful links
------------

Capturing network traffic:

    https://askubuntu.com/questions/11709/how-can-i-capture-network-traffic-of-a-single-process

Network namespaces:

    http://code.google.com/p/coreemu/wiki/Namespaces

Using CORE to create virtual networks

    https://stackoverflow.com/questions/12671587/isolated-test-network-on-a-linux-server-running-a-web-server-lightttpd-and-cu

Running Google QUIC toy client/server

    http://www.chromium.org/quic/playing-with-quic

LibQUIC: just the QUIC API extracted from Chromium

    https://github.com/devsisters/libquic

Linux build instructions for Chromium:

    https://chromium.googlesource.com/chromium/src/+/master/docs/linux_build_instructions.md
