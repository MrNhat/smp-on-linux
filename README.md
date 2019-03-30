# Multiqueues Network Interfaces on Linux
## Introduces
In this article, we will research for optimizing about packet-forwarding performance by exploiting **Symmetric Multiprocessing** (SMP) with multiple queues. we will take advantage of some features of Linux kernel as:
- **[1. Receive Packet Steering - RPS](#rps)**
- **[2. Transmit Packet Steering - XPS](#xps)**
- **[3. IRQ affinity](#irq)**
- **[4. Tuning](#tuning)**
- **[5. Result](#result)**
- **[6. Reference](#ref)**

![smp](/images/smp.png)

First, we chose Intel I350 interface that is supported by the igb driver in Linux, which is produced by Intel itself and  has a good performer. Each I350 interface presents up to 8 send/receive queues with system.

## Mechanisms
The I350 implements its own hasing logic, packets which are part of the same flow should be consistently delivered via the same queue. By binding queues to different CPUs allows us improve the performance of system, create an affinity between network flows and CPUs.
So, we will learning about **RPS**, **XPS** and **IRQ affinity** and how to control affinity and bind queues to CPUs.

<a name="rps"></a>
## 1. Receive Packet Steering - RPS
This is one of Linux features which is controlled by reading/writing bitmaps to representing CPU assignments.
_/sys/class/net/eth0/queues/rx-0/rps_cpus_ is the file config of receive queue 0 on eth0.

<a name="xps"></a>
## 2. Transmit Packet Steering - XPS
Like as RPS, _/sys/class/net/eth0/queues/tx-0/xps_cpus_ is the file config of sending queue 0 on eth0. All receive/sending queues of this interface
in path _/sys/class/net/eth0/queues/_.

<a name="irq"></a>
## 3. IRQ affinity
_/proc/irq/*/smp_affinity_ is the **IRQ** number and _/proc/interrupts_ will describle mapping of IRQ to CPUs.

<a name="tuning"></a>
## 4. Tuning
We can caculate and edit file config with binary 000000 to 111111 in oder to delivere queues to different CPUs like as:
```
cpu0	cpu1	cpu2	cpu3	cpu4	cpu5
TxRx-0	×	·	·	·	·	·
TxRx-1	·	×	·	·	·	·
TxRx-2	·	·	×	·	·	·
TxRx-3	·	·	·	×	·	·
TxRx-4	·	·	·	·	×	·
TxRx-5	·	·	·	·	·	×
```
![affinity](/images/affinity.png)

Each pair of send/receive queues is associated with a single IRQ, each CPU will handle **1 send queue** + **1 receive queue** + **1 IRQ**.

<a name="result"></a>
## 5. Result
Bellow images is the graph before and after tuning **SMP**
![graph](/images/graph.png)

<a name="ref"></a>
## 6. Reference
- _[multi-queue-network-interfaces-with-smp-on-linux](https://greenhost.nl/2013/04/10/multi-queue-network-interfaces-with-smp-on-linux/)_
