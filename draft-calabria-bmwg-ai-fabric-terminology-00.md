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
    organization: Your Organization Here
    email: "fcalabri@gmail.com"

normative:

informative:


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

## Requirements Language

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**,
**SHOULD NOT**, **RECOMMENDED**, **NOT RECOMMENDED**, **MAY**, and **OPTIONAL** in this
document are to be interpreted as described in BCP 14 [RFC2119] [RFC8174] when, and only
when, they appear in all capitals, as shown here.

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


# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back
