# Hybrid Reward Model Enabling Fractional Core vCPUs

## Overview

The Fluence protocol facilitates a decentralized compute marketplace where hardware owners can monetize their CPU resources by offering them to customers in a peer-to-peer manner. When resources are not actively rented, the protocol rewards providers with FLT tokens, helping to incentivize the retention of available capacity.

To manage this reward mechanism, Fluence uses a Proof of Capacity (PoC) protocol, which includes a Proof of Work (PoW) algorithm to verify server availability and benchmark and monitor performance. PoC defines compute units (CUs) as the primary metric for resources, where each CU consists of one CPU core and 4GB of RAM. For example, a server with 64 cores and at least 256GB of RAM would be partitioned into 64 CUs, each running a separate PoW instance. This design allows CUs to be efficiently and dynamically allocated between PoW mining and customer workloads.

While the CU-based approach simplifies resource and reward management, it introduces a limitation with respect to resource granularity. Customers may end up renting more computational power than they actually need, which tends to lead to inefficient resource utilization and higher costs. For instance, with an appropriately configured server, one CU could be mapped to four vCPUs, each with 1GB of RAM, which is rather typical configuration for entry-level virtual machines (VMs).

While fractionalizing CUs in some manner allows Fluence to near-optimally address and capture customer demand, the current rewards program does not properly incentivize providers to release reward-bound CUs when the demand for fractional CUs does not generate revenue at least equal to the reward generation. For example, if only one vCPU, out of four, is rented a CU removed from reward production is only utilized at 25% and it stands to reason that the revenue falls short of the CU's reward. Yet, for the provider to switch a CU from reward mining to serving customer demand, the revenue generated from customer demand for the fractional CU must be greater than the foregone mining reward.



## Model Extension





## Normalizing Compute Resources

In a decentralized network of heterogenous hardware, it is critical to normalize hardware to allow the accurate, trustless provisioning of compute resources. Since the proof system allows for the observation of hash rates on per compute unit, i.e., core, level. Once we normalize performance, via the hashrate proxies, we can use the weights to drive resource fractionalization.

To wit, we have a set of servers, each with a number of cores that perform Proof-of-Work (PoW) computations. Each core contributes a hash rate, and the sum of these contributions defines the server's hash rate. We want to rent vCPUs to customers such that the performance of a vCPU is the same regardless of which server or core it comes from. The performance of a vCPU is defined by a target hash rate $h$, which may not be directly advertised to the customer. Instead, a class system may be used. Regardless, our objective is to allocate fractions of cores (or whole cores) across servers to provide consistent vCPU performance matching $h$.

Our distributed network can be described as:

* $S= \{1,2,…,n\}$: the set of (heterogenous) servers
* $Ci,j$: the i-th core of server $S_j$
* $n_j$: the total number of cores in server $S_j$, for $j\in S$
* $h_i,j$: the hash rate produced by $C_i,j$ for some epoch $T$.
* $H_{j} = \sum_{i=1}^{n_{j}} h_{i,j}$: the total hashrate of server $S_j$

and the vCPU is defined as a fraction, $f_i$, of a server $S_J$'s total resources (cores), such that:

* $f_j \times H_j = h$ where $f_j \in [0,1]$

Our objective, then is to:

* Fractionalize the server for a given $h$ such that
$f_j = {h \over H_j}$
* Attain hash rate consistency for the vCPU such that
$f_j \times H_j = h$, where $0 \leq f_j \leq 1$

And that's it.

Example:

* target hash rate for a vCPU: $h = 10$
* $S_1$: $h_{1,1} = 8, h_{2,1} = 12, h_{3,1} = 15$
* $S_2$: $h_{1,2} = 10, h_{2,2} = 20$

$H_{1} = h_{1,1} + h_{2,1} + h_{3,1} = 8 + 12 + 15 = 35$
$H_{2} = h_{1,2} + h_{2,2} = 10 + 20 = 30$

$f_{1} = \frac{H_{1}}{h} = \frac{35}{10} \approx 0.286, \quad f_{2} = \frac{H_{2}}{h} = \frac{30}{10} \approx 0.333$

