# Hybrid Reward Model Enabling Fractional Core vCPUs

## Overview

The Fluence protocol facilitates a decentralized compute marketplace where hardware owners can monetize their CPU resources by offering them to customers in a peer-to-peer manner. When resources are not actively rented, the protocol rewards providers with FLT tokens, helping to incentivize the retention of available capacity.

To manage this reward mechanism, Fluence uses a Proof of Capacity (PoC) protocol, which includes a Proof of Work (PoW) algorithm to verify server availability and benchmark and monitor performance. PoC defines compute units (CUs) as the primary metric for resources, where each CU consists of one CPU core and 4GB of RAM. For example, a server with 64 cores and at least 256GB of RAM would be partitioned into 64 CUs, each running a separate PoW instance. This design allows CUs to be efficiently and dynamically allocated between PoW mining and customer workloads.

While the CU-based approach simplifies resource and reward management, it introduces a limitation with respect to resource granularity. Customers may end up renting more computational power than they actually need, which tends to lead to inefficient resource utilization and higher costs. For instance, with an appropriately configured server, one CU could be mapped to four vCPUs, each with 1GB of RAM, which is rather typical configuration for entry-level virtual machines (VMs).

While fractionalizing CUs in some manner allows Fluence to near-optimally address and capture customer demand, the current rewards program does not properly incentivize providers to release reward-bound CUs when the demand for fractional CUs does not generate revenue at least equal to the reward generation. For example, if only one vCPU, out of four, is rented a CU removed from reward production is only utilized at 25% and it stands to reason that the revenue falls short of the CU's reward. Yet, for the provider to switch a CU from reward mining to serving customer demand, the revenue generated from customer demand for the fractional CU must be greater than the foregone mining reward.

From a customer perspective, each capacity unit rented should perform the same, cetris paribus, across the network. That is, a vCPU of some type, say "large", with 1 GB RAM should perform approximately the same regardless of the providing data center. Since the network is comprised of potentially heterogenous servers across heterogenous data centers, some normalization relative to the hashrate of a CU and, by extension, fractionalized CU, e.g., vCPU, needs to happen.

We propose a normalization and reward program adjustment to cover both provider and customer concerns. Specifically, we propose to:

* reward underutilized CU fractions so that the revenue + reward at most equal to total reward. so when the utilization, via fractionalized.. increase, the reward payout decreases and eventually hits zero. ... doing this without PoW.
* normalize over hashrates


## PoC Model Extension

In a decentralized network of heterogeneous hardware, it is critical to normalize performance across compute resources to enable accurate, trustless provisioning. A Proof-of-Work (PoW) system facilitates the observation of hash rates at the (physical) core level, providing a measurable proxy for performance. By using hash rate metrics, we can normalize performance both within individual servers and across the network, enabling reliable resource packaging and provisioning regardless of fractionalization.

### Provider Incentivization For Selling Fractional CUs

A compute unit (CU), $C$, generates a reward $R_{PoW}$ when running PoW xor revenue $R_{rent}$ when rented to a customer per epoch $T$. If $C$ is fractionalized into $n$ virtual CPUs (vCPUs), each vCPU has a capacity $c_i = \frac{C}{n}$, and the revenue from renting a single vCPU is $r_i$, where $r_i < R_{PoW}$.

To switch $C$ from PoW to fractional renting, the total revenue from renting out vCPUs is denoted as: $R_{\text{total\_rent}} = \sum_{i \in \mathcal{R}} r_i$.

where $\mathcal{R} \subseteq \{1, 2, \dots, n\}$ represents the set of rented vCPUs. This total revenue may be less than $R_{PoW}$, as it depends on the number of vCPUs rented out. Consequently, a provider may be hesitant to allocate $C$ from generating PoW rewards to renting CU fractions potentially leading to loss of (immediate) income.

In order to incentivize providers to allocate compute units (CUs) to serve fractional (CU) customer demand, we propose allocating proportional rewards, $R_{\text{frac}}$, to the CU when it is not running Proof-of-Work (PoW). The total reward for the CU is the sum of the proportional rewards, $R_{\text{frac}}$, and the revenue from renting virtual CPUs (vCPUs), $R_{\text{total\_rent}} = \sum_{i \in \mathcal{R}} r_i$, where $\mathcal{R}$ represents the set of rented vCPUs. To ensure the CU is properly compensated while not exceeding the PoW reward, the proportional reward is defined as: $R_{\text{frac}} = \max\left(0, R_{\text{PoW}} - R_{\text{total\_rent}}\right)$.

When the CU is fully utilized ($R_{total\_rent} = R_{PoW}$), the proportional reward becomes $R_{frac} = 0$ and the provider earns revenue solely from renting vCPUs. For underutilized CUs, where $R_{total\_rent} < R_{PoW}$, the proportional reward compensates for the revenue gap, ensuring that: $R_{frac} + R_{total\_rent} \leq R_{PoW}$. Of course, this calculation needs to be repeated every epoch $t_i$ for the subset of deals that rent fractions of a (specific) CU.

By way of an example, we may assume a CU has a PoW reward of $R_{PoW} = \$10$, is divided into $n = 4$ vCPUs and each rented vCPU generates $r_i = \$3.00$ per period. When fully utilized, the set of rented vCPUs is $\mathcal{R} = \{c_1, c_2, c_3, c_4\}$, and the total revenue from renting is $R_{\text{total\_rent}} = \sum_{i \in \mathcal{R}} r_i = 4 \cdot \$3.00 = \$12.00$. Since $R_{total\_rent} > R_{PoW}$, the proportional reward is $R_{frac} = \max(0, R_{PoW} - R_{total\_rent} = \max(0, \$10 - \$12.00) = \$0$.

If, on th eother hand, only two vCPUs are rented, the set of rented vCPUs is $\mathcal{R} = \{c_1, c_2\}$, and the total revenue from renting is $R_{total\_rent} = \sum_{i \in \mathcal{R}} r_i = 2 \cdot \$3.00 = \$6.00$. The proportional reward compensates for the shortfall, calculated as $R_{frac} = \max(0, R_{PoW} - R_{total\_rent}) = \max(0, \$10 - \$6.00) = \$4.00$. In this case, the total reward for the CU is $R_{total} = R_{total\_rent} + R_{frac} = \$6.00 + \$4.00 = \$10.00$.

### Performance Normalization By Hashrate Proxy

Our network consists of a set of servers, each with some number of cores and RAM mapped to CUs that perform Proof-of-Work (PoW) computations. Each CU contributes a hashrate, say, $h_{i,j}$, which should be fairly consistent for CUs within a server but may significantly differ across servers due to server type variations.

We want to rent capacity to customers such that the performance of a compute resource, e.g., a vCPU, is consistent regardless of which server or core it comes from. The performance of a vCPU is defined by a target hash rate $h$, which may not be directly advertised to the customer. Regardless, our objective is to allocate CUs, or fractions thereof, across servers to provide consistent performance that matches the (underlying) $h$ of the advertised capacity. To achieve this goal of provisioning consistent compute resources, e.g., vCPUs, against some target hash rate $h$, we need to consider performance normalization and, possibly, classification.

The compute resources in our decentralized network are described as follows. Let $S = \{1,2,\dots,n\}$ be the set of (heterogeneous) servers. For each server $S_j$, let $C_{i,j}$ represent the $i$-th CU, and let $n_j$ be the total number of CUs in $S_j$. Each CU $C_{i,j}$ produces a hash rate $h_{i,j}$ for some epoch $T$, and the total hash rate of server $S_j$ is $H_j = \sum_{i=1}^{n_j} h_{i,j}$.

Fractionalizing a CU into one or more vCPUs is defined as a fraction $c_i$ of a CU's total capacity. If the target hash rate $h$ is less than the capacity of a single CU, the fractionalization is represented as $c_i = \frac{h}{h_{i,j}}$.

If the target hash rate $h$ exceeds the capacity of a single CU on the same server, multiple CUs are combined. Let $\mathcal{C}_j \subseteq \{C_{1,j}, \dots, C_{n_j,j}\}$ denote the set of CUs used to satisfy $h$, where $\mathcal{C}_j$ contains the smallest number of CUs such that $\sum_{C_{i,j} \in \mathcal{C}_j} h_{i,j} \geq h$. For the final CU in $\mathcal{C}_j$, only a fraction of its capacity may be required. If $C_{k,j}$ is the last CU in $\mathcal{C}_j$, its fractional utilization is $c_k = \frac{h - \sum_{C_{i,j} \in \mathcal{C}_j \setminus \{C_{k,j}\}} h_{i,j}}{h_{k,j}}$.

Assume a target hash rate for a vCPU of $h = 25$. For server $S_1$, the CU hash rates are $h_{1,1} = 8$, $h_{2,1} = 12$, and $h_{3,1} = 15$. To satisfy $h = 25$, the smallest set of CUs from $S_1$ is $\mathcal{C}_1 = \{C_{1,1}, C_{2,1}, C_{3,1}\}$. The first two CUs provide $h_{1,1} + h_{2,1} = 8 + 12 = 20$. The remaining hash rate, $25 - 20 = 5$, is provided by a fraction of $C_{3,1}$, with fractional utilization $c_{3} = \frac{5}{15} = 0.333$.

A classification model over hashrates, such as Dynamic Equal Bin Widths, simplifies normalization by grouping CUs into bins based on their hashrate capabilities. For example, CUs can be classified into $\textbf{small} (h_{i,j} \leq 0.5 \cdot h_{max})$, $\textbf{medium} (0.5 \cdot h_{max} < h_{i,j} \leq 0.8 \cdot h_{max})$ and $\textbf{large} (h_{i,j} > 0.8 \cdot h_{max})$ performance bins. Each bin is assigned a target normalized hashrate, $h_t$, as a representative value, e.g., $0.4 \cdot h_{max}$ for small, $0.65 \cdot h_{max}$ for medium, and $0.9 \cdot h_{max}$ for large, enabling consistent and predictable resource allocation. This approach reduces complexity by treating performance ranges within a bin as equivalent.


## Summary

We provide solution candidates that handle both CU fractionalization and performance normalization with the goal to service both customer and provider expectation and requirements. The proposed models allow Fluence to offer a wide variety of "sized" vCPUs independently of the underlying CU metric, while incentivizing providers to release CUs from PoW even if the expected vCPU revenue is less than the foregone PoW reward.

With respect to normalization, multiple options exist to provide some sort of consistency with respect to vCPU, or CU< performance across the network. In the end, we expect customers will decide how much performance variance is acceptable, which in turn shapes the normalization model.