---
title: "Benchmarking Terminology for AI Network Fabrics"
abbrev: "Bechmarking Terminology"
category: info

docname: draft-calabria-bmwg-ai-fabric-terminology
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
# area: OPS AREA
# workgroup: BMWG Working Group
keyword:
 - benchmarking
 - AI Network Fabrics
 - terminology

author:
 -
    fullname: "Fernando Calabria"
    organization: Cisco
    email: "fcalabri@gmail.com"

 -
    fullname: "Carlos Pignataro"
    organization: Blue Fern Consulting
    email: "carlos@bluefern.consulting"
 -
   fullname: "Qin Wu"
   organization: Huawei
   email: "bill.wu@huawei.com"
 -
    fullname: "Giuseppe Fioccola"
    organization: Huawei
    email: "giuseppe.fioccola@huawei.com"

normative:

informative:

  charter-ietf-bmwg:
    title: Benchmarking Methodology Working Group Charter
    target: https://datatracker.ietf.org/group/bmwg/about/
    date: 2024

  IBTA-ROCE:
    title: InfiniBand Architecture Specification, Annex 16: RoCE
    target: https://www.infinibandta.org
    date: 2010

  UEC-SPEC-1.0:
    title: Ultra Ethernet Specification 1.0
    target: https://ultraethernet.org
    date: 2024

--- abstract

This document defines benchmarking terminology for evaluating Ethernet-based network fabrics used in distributed
Artificial Intelligence (AI) training and inference workloads. It provides a unified vocabulary consolidating and
extending terms from RFC 1242, RFC 8238, and the companion AI fabric methodology documents, establishing precise,
vendor-neutral definitions for collective communication primitives, RDMA transport mechanisms (RoCEv2 and Ultra
Ethernet Transport), congestion control behaviors, AI-specific Key Performance Indicators (KPIs), and fabric
topology concepts.

This document is a companion to draft-bmwg-ai-fabric-training-bench and draft-bmwg-ai-fabric-inference-bench-00.
Those documents **SHOULD NOT** be applied without first consulting the terminology defined herein. Where definitions
herein overlap with RFC 1242 or RFC 8238, the AI fabric context definition in this document takes precedence.


--- middle

# Introduction

## Scope and Purpose

This document defines terminology specifically for benchmarking Ethernet-based AI network
fabrics in controlled laboratory environments as specified by the BMWG charter. The defined
terms cover: distributed AI training collective communication patterns, LLM inference serving
architectures, RDMA transport semantics (RoCEv2 and UET), congestion control mechanisms,
fabric topology characteristics, and performance metric definitions.

This document does **not** define acceptance criteria, performance requirements, or
configuration recommendations. It does not address benchmarking of live operational
networks, intra-node (NVLink/PCIe) interconnects, or storage networking.

## Relationship to Existing BMWG Work

This document extends the foundational BMWG terminology established in [RFC1242] (network
interconnect benchmarking terminology) and [RFC8238] (data center benchmarking terminology).
Where terms are defined in those RFCs, this document provides AI fabric context extensions;
the core definitions remain as established. This document also extends the test methodology
framework of [RFC2544] and [RFC8239] as applied in the companion AI fabric methodology
documents.

## Relationship to Companion Documents

This document is one of three companion Internet-Drafts addressing AI fabric benchmarking:

- `draft-bmwg-ai-fabric-terminology` **(this document):** Terminology definitions.
- `draft-bmwg-ai-fabric-training-bench` [TRAINING-BENCH]: Benchmarking methodology for AI training workloads.
- `draft-bmwg-ai-fabric-inference-bench` [INFERENCE-BENCH]: Benchmarking methodology for AI inference serving workloads.

Implementers and evaluators **SHOULD** read this terminology document before applying the
companion methodology documents. Terms defined here are used normatively in those documents
and are not redefined there unless the specific workload context introduces a substantive
difference, which is noted explicitly.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**,
**SHOULD NOT**, **RECOMMENDED**, **NOT RECOMMENDED**, **MAY**, and **OPTIONAL** in this
document are to be interpreted as described in BCP 14 [RFC2119] [RFC8174] when, and only
when, they appear in all capitals, as shown here.

# General Benchmarking Terms

   The following terms establish the general measurement framework
   applicable to all AI fabric benchmarking activities.

   AI Fabric
      The dedicated Ethernet backend network interconnecting
      accelerators (GPUs/XPUs) for distributed AI training and inference
      workloads.  Typically implemented as a non-blocking Clos (fat-
      tree) topology running RoCEv2 or UET transport.  Distinct from the
      front-end (management/storage) network.

   DUT
      Device Under Test.  The network element(s) whose performance
      characteristics are being measured.  In AI fabric benchmarking the
      DUT is one or more fabric elements: leaf switches, spine switches,
      NICs, or the complete fabric assembly.

   SUT
      System Under Test.  The complete AI compute system including
      accelerators, NICs, the fabric DUT, and serving/training software,
      when end-to-end metrics are the measurement objective.

   RT
      Router Tester / Traffic Generator.  Test equipment capable of
      generating and receiving network traffic at specified rates with
      nanosecond-resolution timestamping sufficient for the measurements
      defined in the companion methodology documents.

   JFI
      Jain's Fairness Index.  A scalar measure of flow-level throughput
      fairness across a set of n flows, defined as:

                          JFI = (Σxᵢ)² / (n · Σxᵢ²)

      where xᵢ is the throughput of flow i.  A value of 1.0 indicates
      perfect fairness; lower values indicate disparity.  SHOULD be
      computed per [RFC1242] reporting conventions.

   Offered Load
      The total traffic rate presented to the DUT from test equipment,
      expressed as a fraction of line rate (0–100%) or as absolute bit/
      s.  Offered load is controlled independently of DUT absorption,
      enabling characterization of saturation behavior.

   Trial Duration
      The time interval over which a single measurement is conducted.
      For AI fabric tests, the RECOMMENDED minimum trial duration is 60
      seconds for throughput tests and 300 seconds for soak/stability
      tests, per the methodology in [RFC2544] as extended herein.

   Warmup Period
      A mandatory pre-measurement interval during which traffic is sent
      but results are not recorded.  Ensures adaptive routing tables,
      PFC watermarks, and DCQCN/UET congestion controllers reach steady
      state before measurement begins.  RECOMMENDED minimum: 10 seconds.

   Binary Search
      An iterative test procedure for determining the maximum offered
      load at which a DUT meets a specified acceptance criterion (e.g.,
      zero packet loss).  The search halves the candidate load range at
      each iteration, converging to a resolution of 0.1% offered load
      within 10 iterations.

   Percentile Latency
      A latency statistic expressing that the specified fraction of all
      measured latency samples fall at or below the reported value.
      Denoted Pxx (e.g., P50, P95, P99, P99.9).  Tail latency (P99 and
      above) is especially relevant for AI fabric benchmarking because
      SLO violations are determined by worst-case, not median,
      performance.

#  Collective Communication Terms

   The following terms define the collective communication operations
   that are the primary traffic sources in distributed AI workloads.

   Collective Operation
      A coordinated communication pattern executed simultaneously across
      all accelerators in a training or inference group.  Core
      collectives: AllReduce (gradient aggregation), AllGather
      (parameter distribution), ReduceScatter (partial reduction
      followed by scatter), and AllToAll (expert dispatch in MoE
      models).


   AllReduce
      A collective operation in which each participant contributes a
      tensor and all participants receive the element-wise sum (or other
      reduction) of all contributions.  The dominant communication
      primitive in data-parallel and tensor-parallel training.  Bus
      bandwidth (BusBW) is the primary KPI.

   AllGather
      A collective operation in which each participant contributes a
      shard of a tensor and all participants receive the concatenation
      of all shards.  Used in tensor-parallel (Megatron-style) layers to
      reconstruct distributed activations or parameters.

   ReduceScatter
      A collective operation combining an element-wise reduction with a
      scatter, so each participant receives a distinct slice of the
      reduced result.  Used in ZeRO-stage optimizer strategies and as
      the first half of a ring-AllReduce.

   AllToAll
      A collective operation in which each participant sends a distinct
      payload to every other participant and receives a distinct payload
      from every other participant.  The critical collective for
      Mixture-of-Experts token dispatch.  Generates N²−1 independent
      point-to-point flows for N participants.

   Ring Algorithm
      An AllReduce (or AllGather/ReduceScatter) algorithm structured as
      a logical ring of participants.  Each participant sends data to
      its right neighbor and receives from its left neighbor in 2(N−1)
      steps.  Bus bandwidth efficiency approaches 2(N−1)/N, approaching
      100% for large N.  Standard baseline for BusBW calculation.

   BusBW
      Bus Bandwidth.  The effective data throughput achieved per
      accelerator during a collective operation, normalizing for
      algorithm overhead:

               BusBW = (data_size × algo_factor) / elapsed_time

      For ring AllReduce, algo_factor = 2(N−1)/N.  BusBW enables
      comparison across cluster sizes and collective algorithms.

   CCL
      Collective Communication Library.  A software library providing
      optimized implementations of collective operations (AllReduce,
      AllGather, etc.) over a specific transport (RDMA, shared memory).
      In benchmarking, the CCL implementation MUST be documented in the
      test report.

   SPMD
      Single Program Multiple Data.  The execution model underlying
      bulk-synchronous distributed training, in which all accelerators
      execute identical computation on distinct data partitions,
      synchronizing at collective barriers between steps.

   Bulk Synchronous Parallel (BSP)
      A distributed computation model structured as alternating compute
      and communicate phases, with a global synchronization barrier
      between phases.  Standard training workloads follow BSP: forward
      pass → backward pass → AllReduce gradient sync → optimizer step.

#  Distributed Parallelism Strategy Terms

   The following terms define the parallelism strategies used in
   distributed AI model training and inference, which determine traffic
   patterns and fabric requirements.

   Data Parallelism (DP)
      A distributed training strategy replicating the full model on each
      accelerator, partitioning the training dataset across replicas.
      Gradient synchronization after each backward pass requires an
      AllReduce across all DP ranks.  Memory-efficient for small models;
      communication overhead scales with parameter count.

   Tensor Parallelism (TP)
      A distributed training and inference strategy partitioning
      individual weight matrices across multiple accelerators (ranks).
      Each rank computes a partial result; AllGather or ReduceScatter
      collectives are required within each layer to aggregate results.
      Dominant parallelism within a node (intra-node).

   Pipeline Parallelism (PP)
      A distributed strategy assigning contiguous groups of transformer
      layers to distinct stages (accelerators or nodes).  Each stage
      processes one microbatch and forwards activations to the next
      stage.  Generates point-to-point inter-stage traffic across the
      fabric (activations and gradients).

   Expert Parallelism (EP)
      A parallelism strategy for Mixture-of-Experts models distributing
      expert sub-networks across accelerators.  Each token is routed to
      its designated experts (typically top-K of E total experts),
      requiring AllToAll communication for dispatch.  Wide EP (e.g.,
      96-way) generates dense inter-node AllToAll at every MoE layer.

   MoE
      Mixture of Experts.  A transformer architecture replacing dense
      feed-forward layers with a set of E expert sub-networks, of which
      only top-K experts (typically K=2 or K=4) are activated per token
      via a learned router.  MoE enables large model capacity with sub-
      linear compute, but introduces AllToAll communication requirements
      proportional to E and sequence length.

   DP Attention
      Data Parallelism applied to the attention computation, where the
      KV cache is partitioned across data-parallel ranks.  Each rank
      holds 1/DP_SIZE of the KV cache; AllToAll communication exchanges
      attention outputs.  Used in inference to reduce per-accelerator
      memory footprint for long contexts.

   ZeRO
      Zero Redundancy Optimizer.  A memory optimization strategy for
      data-parallel training that shards model states (parameters,
      gradients, optimizer states) across DP ranks instead of
      replicating them.  ZeRO Stage 1 shards optimizer states; Stage 2
      adds gradient sharding; Stage 3 adds parameter sharding.  Each
      stage increases AllGather/ReduceScatter communication.

#  Network Transport Terms

## RoCEv2 and RDMA Terms

   The following terms define RDMA and RoCEv2 transport semantics as
   used in AI fabric benchmarking.

   RDMA
      Remote Direct Memory Access.  A transport mechanism enabling
      direct memory-to-memory data transfer between hosts without
      involving the destination CPU, providing zero-copy semantics and
      kernel bypass.  Implementations include InfiniBand Verbs (native
      IB), iWARP (RDMA over TCP), and RoCEv2 (RDMA over Converged
      Ethernet v2).

   RoCEv2
      RDMA over Converged Ethernet version 2.  An RDMA transport
      encapsulating InfiniBand transport layer (BTH) over UDP/IP,

      enabling RDMA semantics on standard Ethernet infrastructure.
      Requires lossless fabric operation (PFC or equivalent) for
      correctness.  Standardized in IBTA Annex 16 [IBTA-ROCE] and
      transported over UDP destination port 4791.

   QP
      Queue Pair.  The fundamental RDMA communication endpoint
      comprising a Send Queue (SQ) and Receive Queue (RQ).  QPs are
      connection-oriented in Reliable Connected (RC) mode.  Multiple QPs
      per source-destination pair are used to increase ECMP entropy in
      fabric load balancing.

   Reliable Connected (RC)
      An RDMA QP transport service type providing reliable, in-order
      delivery between exactly two endpoints.  The primary QP type for
      AI collective operations via RoCEv2.  Requires connection setup
      before data transfer and maintains per-QP state for
      retransmission.

   RDMA Verb
      An operation primitive of the RDMA programming model.  Key verbs:
      SEND/RECV (two-sided, receiver must post a buffer), WRITE (one-
      sided, target memory written directly), READ (one-sided, remote
      memory read), and Atomic (compare-and-swap, fetch-and-add).  AI
      collectives predominantly use WRITE and SEND.

   UET
      Ultra Ethernet Transport.  A transport protocol defined by the
      Ultra Ethernet Consortium (UEC) Specification 1.0 [UEC-SPEC-1.0]
      as a next-generation AI/HPC fabric transport.  UET is
      connectionless, supports native packet spraying (Reliable
      Unordered Delivery), and integrates multipath load balancing and
      congestion control.  Transported over UDP destination port 4793
      (pending IANA verification).

   PDC
      Packet Delivery Context.  The ephemeral, lightweight transport
      endpoint in UET, analogous to but distinct from an RDMA Queue
      Pair.  PDCs are connectionless (no setup handshake), enabling low-
      latency initiation and reduced per-flow state in the NIC and
      switch.

   ROD
      Reliable Ordered Delivery.  A UET transport service providing
      reliable, in-order packet delivery, semantically equivalent to
      RoCEv2 RC mode.  Suitable for legacy RDMA applications requiring
      strict ordering guarantees.

##  Ultra Ethernet Transport (UET) Terms

   The following terms define UET-specific concepts introduced by the
   Ultra Ethernet Consortium (UEC) Specification 1.0 [UEC-SPEC-1.0].

   RUD
      Reliable Unordered Delivery.  A UET transport service providing
      reliable delivery without maintaining packet order across paths.
      Enables native packet spraying across ECMP paths without reorder-
      buffer overhead at the receiver NIC.  The preferred UET service
      class for AI training collectives.

   RUDI
      Reliable Unordered Delivery for Idempotent operations.  A UET
      transport service optimized for operations safe to execute more
      than once (e.g., RDMA Writes to non-accumulating targets),
      allowing simplified retransmission logic with reduced state
      overhead.

   UUD
      Unreliable Unordered Delivery.  A UET transport service providing
      best-effort, unordered packet delivery with minimal overhead.
      Suitable for telemetry, speculative operations, or workloads with
      application-layer loss tolerance.

   UEC Profile
      A defined subset of UET features targeting a specific use case: AI
      Base (core AI training/inference, mandatory feature set), AI Full
      (AI Base plus deferred send, exact-match tagging, extended
      atomics), or HPC (latency-optimized for traditional HPC workloads
      with fine-grained synchronization).

   LLR
      Link Layer Retry.  An optional UEC link-layer enhancement
      providing fast per-hop error recovery at the Ethernet link layer.
      LLR detects symbol errors at the FEC level and retransmits the
      affected frame before it is dropped, reducing the frequency of
      transport-layer retransmission and improving tail latency.

   Packet Trimming
      An optional UEC link-layer behavior in which a congested switch,
      rather than dropping the full packet, transmits only the packet
      header (trimmed packet) to the receiver.  Trimming enables the
      receiver to detect loss and initiate selective retransmission more
      rapidly, reducing bandwidth waste versus silent drop.

   CBFC
      Credit-Based Flow Control.  An optional UEC link-layer buffer
      management mechanism using explicit credit grants from downstream
      to upstream devices.  CBFC provides backpressure without
      transmitting PFC PAUSE frames, eliminating the head-of-line
      blocking and storm propagation risks associated with PFC.

   Entropy Value
      A per-packet field in the UET header used to distribute packets of
      a single message across available ECMP paths, providing explicit
      spray entropy independent of the IP 5-tuple.  Enables hardware-
      assisted packet spraying without requiring transport-layer state
      in the switch.

   GIN
      GPU-Initiated Networking.  A communication paradigm in which GPU
      threads directly initiate network RDMA operations (sends, one-
      sided writes/reads) to the NIC hardware without CPU involvement,
      eliminating the CPU-GPU synchronization round-trip.  Reduces
      effective latency by several microseconds for fine-grained
      operations.

   KVCXL
      KV Cache Transfer Library.  A software library providing
      standardized point-to-point data transfer primitives (register,
      transfer, notify) for inference engines, abstracting underlying
      transport mechanisms (intra-node interconnect, RDMA, PCIe, storage
      interfaces).  Enables transport-agnostic KV cache migration in
      disaggregated serving architectures.

#  Congestion Control and Fabric Behavior Terms

   The following terms define congestion management mechanisms and
   associated fabric behaviors critical to AI workload performance.

   PFC
      Priority Flow Control (IEEE 802.1Qbb).  A lossless Ethernet
      mechanism in which a receiver transmits a PAUSE frame to its
      upstream neighbor on a specific priority class when its ingress
      buffer approaches a configured threshold, temporarily halting
      transmission of that priority.  Required for lossless RoCEv2
      operation.  PFC operates hop-by-hop and can propagate congestion
      upstream (PFC storm risk).


   PFC Storm
      A pathological condition in which PFC PAUSE frames propagate
      across multiple hops, causing widespread throughput degradation or
      deadlock unrelated to the original congestion source.  Detection
      and mitigation SHOULD be part of soak test evaluation per the
      companion methodology documents.

   PFC Deadlock
      A circular PFC dependency in which sets of flows mutually pause
      each other indefinitely, resulting in zero progress for affected
      traffic classes.  Deadlock risk is elevated in non-tree topologies
      (e.g., those with fabric loops under failure convergence) and MUST
      be evaluated in fabric-level soak tests.

   ECN
      Explicit Congestion Notification ([RFC3168]).  An IP-layer
      mechanism in which a congested router marks packets with the
      Congestion Experienced (CE) codepoint in the IP ECN field instead
      of dropping them.  The receiver echoes congestion feedback to the
      sender via the transport protocol, triggering rate reduction.
      Used with RoCEv2 as part of DCQCN.

   DCQCN
      Data Center Quantized Congestion Notification.  An end-to-end
      congestion control algorithm for RoCEv2 flows, combining ECN
      marking at congested switches with rate-based sender reduction
      using an additive-increase/multiplicative-decrease (AIMD) scheme.
      Note: PFC serves as a separate, orthogonal backstop to prevent
      packet loss during DCQCN convergence; PFC is not a component of
      the DCQCN algorithm itself.

   ECN Marking Ratio
      The fraction of packets (expressed as a percentage) that are
      marked with the CE codepoint in the IP ECN field over a
      measurement interval.  A high ECN Marking Ratio indicates
      persistent congestion and is a primary Fabric Health Indicator in
      AI fabric benchmarking.

   Incast
      A traffic pattern in which multiple sources simultaneously send to
      a single destination, potentially overwhelming the destination's
      NIC receive buffer and the switch's egress port buffer.  Incast is
      a dominant congestion mechanism in AllReduce and collective
      operations, and a primary test scenario in the companion
      methodology documents.

   Incast Ratio
      The ratio of concurrent senders to receivers in an incast
      communication pattern (N:1).  The incast ratio determines the
      oversubscription factor at the destination port and is a primary
      test parameter for congestion characterization.

   Packet Spray
      A load balancing strategy distributing individual packets of a
      single RDMA message across all available ECMP paths, maximizing
      link utilization at the cost of potential out-of-order delivery at
      the receiver.  Native in UET (RUD mode); requires NIC reorder
      buffering for RoCEv2 RC mode.

   DLB / Flowlet
      Dynamic Load Balancing using flowlet detection.  A per-flow
      rerouting mechanism that reassigns a flow to a new ECMP path when
      the flow has been idle longer than the flowlet gap threshold
      (typically 500 ns–2 µs), reducing out-of-order packet risk
      compared to packet spray while improving utilization over static
      per-flow ECMP.

   ECMP
      Equal-Cost Multi-Path routing.  A forwarding mechanism
      distributing traffic across multiple equal-cost paths, typically
      via hash of the IP 5-tuple (or entropy field in UET).  ECMP
      imbalance (MMR > 1.0) is a primary fabric efficiency metric for AI
      traffic.

   MMR
      Max-Mean Ratio.  The ratio of the flow count (or traffic load) on
      the most heavily utilized link to the average flow count per link
      across all fabric links in the measurement path.  MMR = 1.0
      indicates perfect ECMP balance; MMR > 1.0 quantifies imbalance
      that degrades effective fabric bandwidth.

#  Fabric Topology and Infrastructure Terms

   The following terms define fabric topology architectures and
   infrastructure components referenced in the companion methodology
   documents.

   Clos / Fat-Tree Topology
      A multi-stage switch topology providing non-blocking or
      oversubscribed connectivity between all leaf-to-leaf pairs.  In AI
      fabric deployments, a two-tier (leaf-spine) or three-tier (leaf-
      spine-superspine) Clos is standard.  Full bisection bandwidth
      (1:1) is the target for training fabrics; 2:1 or 4:1
      oversubscription may be acceptable for inference fabrics.

   Rail-Optimized Topology
      A topology in which the NIC ports of each server are distributed
      across multiple top-of-rack (ToR) switches (one NIC port per
      switch), such that intra-node collective traffic between adjacent
      servers traverses different physical paths.  Rail-optimized
      designs minimize switch-to-switch traffic during ring AllReduce,
      maximizing effective BusBW.  Requires ECMP-aware collective
      placement.

   Bisection Bandwidth
      The aggregate bandwidth across the minimum cut that divides the
      fabric into two equal halves.  Non-blocking fabrics provide
      bisection bandwidth equal to half the total edge (server-facing)
      bandwidth.  Bisection bandwidth limits worst-case all-to-all
      communication throughput.

   Oversubscription Ratio
      The ratio of total edge (server-facing) bandwidth to total
      bisection bandwidth in a Clos fabric.  A 1:1 ratio is non-
      blocking; higher ratios (e.g., 2:1, 4:1) reduce fabric cost but
      may bottleneck all-to-all and AllReduce patterns when all server
      ports are active simultaneously.

   ToR Switch
      Top-of-Rack switch.  The first-hop aggregation switch connecting
      accelerator servers in a rack to the spine layer of the fabric.
      In rail-optimized topologies, multiple ToR switches serve a single
      rack, with each server's NICs distributed across ToRs.

   Spine / Superspine
      Intermediate and top-layer switches in a multi-tier Clos fabric,
      providing inter-rack and inter-pod connectivity respectively.
      Spine switches aggregate multiple ToR switches; superspine
      switches aggregate multiple spine pods.

   NIC
      Network Interface Controller.  The hardware device providing
      network connectivity for an accelerator host.  AI fabric NICs
      support RDMA (RoCEv2 or UET), hardware offload for collective
      operations, and, optionally, GPU-Initiated Networking (GIN).  NIC
      model and firmware version MUST be documented in all benchmark
      reports.

   Buffer Occupancy
      The instantaneous or time-averaged fill level of a switch port's
      packet buffer, expressed in bytes or as a fraction of total buffer
      capacity.  Elevated sustained buffer occupancy indicates
      congestion.  P99 buffer occupancy is a Fabric Health Indicator in
      the companion methodology documents.

   Zero-Impact Failover
      Sub-microsecond automatic path convergence upon a link or switch
      failure resulting in no measurable increase to JCT or TTFT.  Zero-
      impact failover requires pre-programmed alternate paths and
      hardware-level fast reroute (FRR) with sub-microsecond detection,
      not relying on routing protocol convergence.

   Link Utilization
      The fraction of the nominal link capacity actually used for data
      transmission over a measurement interval, expressed as a
      percentage.  Reported as mean, P95, and P99 per link.  High
      asymmetric link utilization (low average but high peak) is
      characteristic of bursty AI inference traffic.

#  Training-Specific Terms

   The following terms are specific to AI training workload benchmarking
   and are used normatively in [TRAINING-BENCH-00].

   JCT
      Job Completion Time.  The wall-clock elapsed time from the start
      of a training job (or benchmark iteration) until all participating
      accelerators complete their work, inclusive of all forward pass,
      backward pass, and collective communication phases.  JCT is the
      primary end-to-end training efficiency KPI.

   Roofline JCT
      The theoretical minimum JCT assuming perfect (zero-contention,
      zero-queuing) network behavior, calculated as:

            Roofline JCT = computation_time + serialization_delay
            serialization_delay = message_size / link_rate

      Provides a reference for evaluating fabric overhead.

   JCT Ratio
      The ratio of measured JCT to Roofline JCT.  A value of 1.0
      indicates no network-induced overhead.  Values > 1.0 quantify
      fabric inefficiency:

                   JCT Ratio = JCT_measured / JCT_roofline

      The JCT Ratio is the primary comparative metric for AI training
      fabric benchmarking.

   Gradient Synchronization
      The AllReduce collective operation performed after the backward
      pass of each training step to sum the locally computed gradients
      across all data-parallel replicas.  Gradient synchronization is
      the dominant communication event in data-parallel training,
      occurring once per training step per layer.

   Step Time
      The wall-clock duration of a single training iteration (one
      forward pass + one backward pass + gradient synchronization +
      optimizer step).  Step time = computation time + communication
      time, where the communication time is dominated by the AllReduce
      collective.

   Soak Test
      A sustained-load test run for an extended period (minimum 24 hours
      for stability evaluation) at a defined offered load fraction
      (e.g., 70% or 90% of maximum throughput).  Soak tests detect
      buffer leaks, ECMP imbalance drift, PFC storm initiation, and
      long-tail error accumulation not visible in short-duration tests.

#  Inference-Specific Terms

   The following terms are specific to AI inference serving workload
   benchmarking and are used normatively in [INFERENCE-BENCH-00].

   TTFT
      Time to First Token.  The elapsed time from receipt of an
      inference request by the serving system to emission of the first
      output token.  TTFT encompasses prompt processing (prefill), KV
      cache generation, optional KV cache transfer (in disaggregated
      architectures), and the initial decode step.  Interactive serving
      target: TTFT < 500 ms at P99.

   ITL
      Inter-Token Latency.  The elapsed time between successive output
      tokens during the autoregressive decode phase.  ITL is measured at
      P50, P95, P99, and P99.9 to characterize tail latency behavior.
      Interactive serving target: ITL < 50 ms at P99.

   TPS
      Tokens Per Second.  Aggregate throughput of the inference serving
      system, measured as the total number of output tokens generated
      per second across all concurrent requests.  Reported separately
      for input-side (prefill) TPS and output-side (decode) TPS.

   KV Cache
      Key-Value Cache.  The intermediate attention state (key and value
      projection matrices from the multi-head attention layers) computed
      during the prefill phase and reused during each decode step to
      avoid redundant recomputation.  KV cache size scales with: number
      of layers × number of attention heads × head dimension × sequence
      length × numerical precision.  The attention head configuration
      MUST be reported in all benchmark results.

   Prefill Phase
      The compute-bound phase of LLM inference in which the entire input
      prompt is processed in parallel to generate the KV cache and the
      first output token.  Characterized by high arithmetic intensity
      (200–400 ops/byte), high accelerator utilization (90–95%), and
      large activation tensors.  Prefill latency dominates TTFT for long
      prompts.

   Decode Phase
      The memory-bandwidth-bound phase of LLM inference in which output
      tokens are generated autoregressively, one token per forward pass,
      by reading the KV cache.  Characterized by low arithmetic
      intensity (60–80 ops/byte), lower accelerator utilization
      (20–40%), and memory-bandwidth-limited KV cache reads.  Decode
      throughput limits TPS.

   Disaggregated Serving
      An inference serving architecture in which the prefill phase and
      decode phase are executed on physically separate groups of
      accelerators (workers), connected by a network fabric.
      Disaggregated serving allows independent scaling of prefill and
      decode resources (xPyD) but introduces KV cache transfer as a
      fabric-critical data movement.

   xPyD Ratio
      The allocation ratio of x prefill workers to y decode workers in a
      disaggregated serving cluster.  Notation example: 3P9D denotes 3
      prefill nodes and 9 decode nodes.  The optimal xPyD ratio depends
      on model size, prompt length distribution, output length
      distribution, and TTFT/ITL SLO targets.

   Continuous Batching
      A dynamic inference scheduling technique that inserts new requests
      into an active decode batch as slots become available (without
      waiting for the current batch to complete), improving accelerator
      utilization compared to static batching.  Continuous batching
      generates variable batch sizes that affect fabric traffic
      burstiness.

   PagedAttention
      A KV cache memory management technique storing attention keys and
      values in fixed-size, non-contiguous virtual pages (typically
      16–64 KB), inspired by OS virtual memory management.
      PagedAttention reduces memory fragmentation and enables efficient
      KV cache sharing across requests with common prefixes.

   Prefix Caching
      Reuse of previously computed KV cache segments for inference
      requests sharing a common prompt prefix (e.g., a fixed system
      prompt), eliminating redundant prefill computation.  Prefix cache
      hit rate is a secondary KPI for inference serving efficiency.

   Normal Dispatch
      An AllToAll MoE dispatch communication mode optimized for the
      prefill phase.  Payload sizes are variable (depending on token-to-
      expert routing), generating dynamic tensor shapes incompatible
      with static graph capture.  Normal Dispatch maximizes throughput
      for large batches at the cost of higher per-dispatch latency.

   Low-Latency Dispatch
      An AllToAll MoE dispatch communication mode optimized for the
      decode phase.  Payload sizes are padded to fixed maximum
      dimensions (compatible with static graph capture), enabling lower
      kernel-launch overhead at the cost of slight bandwidth
      inefficiency.  Target latency: < 200 µs per dispatch round trip.

   SLO
      Service Level Objective.  A quantitative target for an inference
      serving KPI, established between the service provider and
      consumers.  AI inference SLOs typically specify maximum TTFT
      (e.g., < 500 ms P99) and maximum ITL (e.g., < 50 ms P99) under a
      specified request arrival rate.

   Speculative Decoding
      An inference acceleration technique using a small draft model to
      generate candidate token sequences that are verified in parallel
      by the target model.  Speculative decoding reduces effective ITL
      but generates bursty, variable-length KV cache traffic; this
      workload pattern is noted as a future benchmarking area not fully
      specified in the current companion documents.

#  KPI Classification Terms

   The following terms define the three-tier KPI taxonomy used across
   both companion methodology documents.

   Primary KPI
      A top-level performance indicator directly representing end-user
      experience or training efficiency.  In training: JCT Ratio and
      BusBW.  In inference: TTFT and ITL.  Primary KPIs are the
      principal reporting metric and the basis for comparative
      benchmarking across DUT implementations.

   Secondary KPI
      A fabric-level performance indicator providing mechanistic
      explanation for primary KPI values.  Examples: collective
      operation throughput (BusBW), KV cache transfer goodput, AllToAll
      dispatch latency, ECMP imbalance (MMR), and link utilization.
      Secondary KPIs enable root-cause analysis of Primary KPI
      deviations.

   Fabric Health Indicator (FHI)
      An operational metric characterizing fabric stability and anomaly
      conditions rather than peak performance.  FHIs include: PFC event
      rate, PFC storm occurrence, ECN marking ratio, packet loss rate,
      buffer occupancy (P99), and retransmission rate.  FHIs SHOULD be
      continuously monitored and reported throughout all test
      categories.

   Goodput
      The application-useful data delivered per unit time, excluding
      retransmitted packets, protocol overhead, and padding.  Goodput
      may differ significantly from raw throughput during congestion
      events; both SHOULD be reported in benchmarking results.

   Zero Packet Loss
      A test acceptance criterion requiring that no packets are dropped
      by the DUT during the measurement interval.  For RoCEv2 and UET
      transports, zero packet loss is the target operating condition.
      The binary search procedure in the companion methodology documents
      determines the maximum offered load satisfying this criterion.

#  Referenced Standards Abbreviations

   The following abbreviations refer to normative and informative IETF
   documents referenced throughout this document and the companion
   methodology documents.

   RFC 1242
      "Benchmarking Terminology for Network Interconnect Devices"
      (Bradner, 1991).  Defines foundational benchmarking terms
      (throughput, latency, frame loss rate, back-to-back frames).  The
      baseline terminology reference for BMWG work.  Where terms in this
      document overlap with RFC 1242 definitions, the AI fabric context
      definitions herein take precedence.

   RFC 2544
      "Benchmarking Methodology for Network Interconnect Devices"
      (Bradner & McQuaid, 1999).  Defines test methodologies for
      throughput, latency, frame loss rate, and back-to-back
      measurements.  The AI fabric methodology documents extend RFC 2544
      procedures for AI-specific traffic patterns and test durations.

   RFC 8238
      "Data Center Benchmarking Terminology" (Bitar et al., 2017).
      Extends RFC 1242 with data center-relevant terms including
      forwarding table scaling, congestion, and VM/SDN.  Incast, ECN,
      and buffer occupancy concepts in this document align with RFC 8238
      definitions.

   RFC 8239
      "Data Center Benchmarking Methodology" (Bitar et al., 2017).
      Defines test methodologies for data center network functions
      including incast, ECN marking, and lossless behavior.  The AI
      fabric companion methodology documents extend RFC 8239 for
      distributed AI collective traffic patterns.

   RFC 2119 / RFC 8174
      "Key words for use in RFCs to Indicate Requirement Levels"
      (Bradner, 1997; Leiba, 2017).  Define the normative requirement
      language: MUST, MUST NOT, REQUIRED, SHALL, SHALL NOT, SHOULD,
      SHOULD NOT, RECOMMENDED, MAY, and OPTIONAL.  RFC 8174 clarifies
      that these terms are normative only when in uppercase; lowercase
      uses are not normative.

#  Security Considerations

   This document defines terminology for use in controlled laboratory
   benchmarking environments.  As such it does not define protocols,
   algorithms, or operational procedures that introduce security
   vulnerabilities.

   Benchmarking activities SHOULD be conducted in isolated test networks
   that are not connected to production infrastructure.  AI fabric
   benchmarking generates substantial traffic volumes (multi-terabit
   aggregate); inadvertent exposure to operational networks could cause
   service disruption.

   Published benchmarking results MUST accurately represent the tested
   configuration including DUT hardware and firmware versions, transport
   configuration, and topology.  Selective reporting of favorable
   configurations while omitting constraints may mislead procurement
   decisions.

   Refer to [RFC2544] Section 6 and the companion methodology documents
   for additional security considerations related to test traffic
   isolation and result integrity.

#  IANA Considerations

   This document has no IANA actions.

   The companion document [INFERENCE-BENCH-00] references UDP port 4793
   as the destination port for UET traffic.  The authors note that IANA
   assignment of this port number for UET SHOULD be verified against the
   IANA Service Name and Transport Protocol Port Number Registry prior
   to publication of that document.  The UEC Specification 1.0
   assignment status SHOULD be confirmed by the document authors.

--- back

# Term Cross-Reference to Companion Documents

   The following table identifies which terms from this document are
   used in each companion methodology document.

~~~~
   +====================+====================+=======================+
   | Term Category      | Used in Training   | Used in Inference     |
   |                    | Bench              | Bench                 |
   +====================+====================+=======================+
   | General            | All terms          | All terms             |
   | Benchmarking Terms |                    |                       |
   | (Section 2)        |                    |                       |
   +--------------------+--------------------+-----------------------+
   | Collective         | AllReduce,         | AllToAll, BusBW       |
   | Communication      | AllGather,         |                       |
   | (Section 3)        | ReduceScatter,     |                       |
   |                    | AllToAll, BusBW,   |                       |
   |                    | CCL, Ring          |                       |
   |                    | Algorithm, BSP,    |                       |
   |                    | SPMD               |                       |
   +--------------------+--------------------+-----------------------+
   | Parallelism        | DP, TP, PP, EP,    | EP, MoE, DP Attention |
   | Strategies         | MoE, ZeRO          |                       |
   | (Section 4)        |                    |                       |
   +--------------------+--------------------+-----------------------+
   | RDMA / RoCEv2      | RDMA, RoCEv2, QP,  | RDMA, RoCEv2, QP, RC  |
   | (Section 5.1)      | RC mode, RDMA Verb | mode, GIN, KVCXL      |
   +--------------------+--------------------+-----------------------+
   | UET Terms          | UET, PDC, ROD,     | UET, RUD, GIN         |
   | (Section 5.2)      | RUD, RUDI, UUD,    |                       |
   |                    | LLR, Packet        |                       |
   |                    | Trimming, CBFC,    |                       |
   |                    | UEC Profile,       |                       |
   |                    | Entropy Value      |                       |
   +--------------------+--------------------+-----------------------+
   | Congestion Control | PFC, PFC Storm,    | PFC, ECN, DCQCN,      |
   | (Section 6)        | PFC Deadlock, ECN, | Incast, Packet Spray, |
   |                    | DCQCN, ECN Marking | ECMP                  |
   |                    | Ratio, Incast,     |                       |
   |                    | Incast Ratio,      |                       |
   |                    | Packet Spray, DLB/ |                       |
   |                    | Flowlet, ECMP, MMR |                       |
   +--------------------+--------------------+-----------------------+
   | Fabric Topology    | Clos, Rail-        | Clos, Bisection BW,   |
   | (Section 7)        | Optimized,         | ToR, NIC, Buffer      |
   |                    | Bisection BW,      | Occupancy, Link       |
   |                    | Oversubscription,  | Utilization           |
   |                    | ToR, Spine, NIC,   |                       |
   |                    | Buffer Occupancy,  |                       |
   |                    | Zero-Impact        |                       |
   |                    | Failover, Link     |                       |
   |                    | Utilization        |                       |
   +--------------------+--------------------+-----------------------+
   | Training-Specific  | JCT, Roofline JCT, | Soak Test             |
   | (Section 8)        | JCT Ratio,         |                       |
   |                    | Gradient Sync,     |                       |
   |                    | Step Time, Soak    |                       |
   |                    | Test               |                       |
   +--------------------+--------------------+-----------------------+
   | Inference-Specific | —                  | TTFT, ITL, TPS, KV    |
   | (Section 9)        |                    | Cache, Prefill,       |
   |                    |                    | Decode, Disaggregated |
   |                    |                    | Serving, xPyD,        |
   |                    |                    | Continuous Batching,  |
   |                    |                    | PagedAttention,       |
   |                    |                    | Prefix Caching,       |
   |                    |                    | Normal/Low-Latency    |
   |                    |                    | Dispatch, SLO         |
   +--------------------+--------------------+-----------------------+
   | KPI Classification | Primary KPI (JCT   | Primary KPI (TTFT,    |
   | (Section 10)       | Ratio, BusBW),     | ITL), Secondary KPI,  |
   |                    | Secondary KPI,     | FHI, Goodput, Zero    |
   |                    | FHI, Goodput, Zero | Packet Loss           |
   |                    | Packet Loss        |                       |
   +--------------------+--------------------+-----------------------+
~~~~
{: #embed title="Table 1: Term Cross-Reference to Companion Documents" artwork-align="center"}

# Term Taxonomy Summary

   The following table provides a concise summary of all defined terms
   organized by category, with the section reference for the full
   definition.

~~~~
   +=========+====================================+====================+
   | Section | Term(s)                            | Category           |
   +=========+====================================+====================+
   | 2       | DUT, SUT, RT, JFI, Offered         | General            |
   |         | Load, Trial Duration, Warmup       | Benchmarking       |
   |         | Period, Binary Search,             |                    |
   |         | Percentile Latency, AI             |                    |
   |         | Fabric                             |                    |
   +---------+------------------------------------+--------------------+
   | 3       | Collective Operation,              | Collective         |
   |         | AllReduce, AllGather,              | Communication      |
   |         | ReduceScatter, AllToAll,           |                    |
   |         | Ring Algorithm, BusBW, CCL,        |                    |
   |         | SPMD, BSP                          |                    |
   +---------+------------------------------------+--------------------+
   | 4       | Data Parallelism, Tensor           | Parallelism        |
   |         | Parallelism, Pipeline              | Strategies         |
   |         | Parallelism, Expert                |                    |
   |         | Parallelism, MoE, DP               |                    |
   |         | Attention, ZeRO                    |                    |
   +---------+------------------------------------+--------------------+
   | 5.1     | RDMA, RoCEv2, QP, Reliable         | Transport — RDMA / |
   |         | Connected (RC), RDMA Verb,         | RoCEv2             |
   |         | UET, PDC, ROD                      |                    |
   +---------+------------------------------------+--------------------+
   | 5.2     | RUD, RUDI, UUD, UEC Profile,       | Transport — UET    |
   |         | LLR, Packet Trimming, CBFC,        |                    |
   |         | Entropy Value, GIN, KVCXL          |                    |
   +---------+------------------------------------+--------------------+
   | 6       | PFC, PFC Storm, PFC                | Congestion Control |
   |         | Deadlock, ECN, DCQCN, ECN          |                    |
   |         | Marking Ratio, Incast,             |                    |
   |         | Incast Ratio, Packet Spray,        |                    |
   |         | DLB/Flowlet, ECMP, MMR             |                    |
   +---------+------------------------------------+--------------------+
   | 7       | Clos/Fat-Tree, Rail-               | Fabric Topology    |
   |         | Optimized, Bisection               |                    |
   |         | Bandwidth, Oversubscription        |                    |
   |         | Ratio, ToR Switch, Spine/          |                    |
   |         | Superspine, NIC, Buffer            |                    |
   |         | Occupancy, Zero-Impact             |                    |
   |         | Failover, Link Utilization         |                    |
   +---------+------------------------------------+--------------------+
   | 8       | JCT, Roofline JCT, JCT             | Training-Specific  |
   |         | Ratio, Gradient                    |                    |
   |         | Synchronization, Step Time,        |                    |
   |         | Soak Test                          |                    |
   +---------+------------------------------------+--------------------+
   | 9       | TTFT, ITL, TPS, KV Cache,          | Inference-Specific |
   |         | Prefill Phase, Decode Phase,       |                    |
   |         | Disaggregated Serving, xPyD        |                    |
   |         | Ratio, Continuous Batching,        |                    |
   |         | PagedAttention, Prefix             |                    |
   |         | Caching, Normal Dispatch,          |                    |
   |         | Low-Latency Dispatch, SLO,         |                    |
   |         | Speculative Decoding               |                    |
   +---------+------------------------------------+--------------------+
   | 10      | Primary KPI, Secondary KPI,        | KPI Classification |
   |         | Fabric Health Indicator,           |                    |
   |         | Goodput, Zero Packet Loss          |                    |
   +---------+------------------------------------+--------------------+
   | 11      | RFC 1242, RFC 2544, RFC            | Referenced         |
   |         | 8238, RFC 8239, RFC                | Standards          |
   |         | 2119/8174                          |                    |
   +---------+------------------------------------+--------------------+
~~~~
{: #embed title="Table 2: Complete Term Taxonomy" artwork-align="center"}
