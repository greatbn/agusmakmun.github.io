---
layout: post
title: P4 -  Language for programable switches
date: '2019-11-18 15:45:10'
---

Programming Protocol independent Packet Processors (P4) is a programming language for describing how packets are processed by the data plane. 
Its also considered as a protocol between the controller and the network devices. P4 design has three main goals:

- Protocol independence: P4 enables a programmable switch which can define new header formats with new field names and types.

- Reconfigurability: The protocol independence and the abstract language model enable reconfigurability by changing the way of packets processing after deploying the switches.

- Target independence: P4 programs can be compiled on many different types of network infrastructure such as CPUs, FPGAs, and ASICs. Instead, a compiler should take into account the switchs capabilities when turning a P4 program into a target-dependent program (i.e., used to configure the switch). 

<img src="https://i.imgur.com/jDHLQgK.jpg">

P4 program is based upon an abstract forwarding model consisting of a parser and a set of match+action table resources, divided between ingress and egress. The parser defines the headers present in each incoming packet. Each match+action table identifies the type of lookup (i.e., rules) to perform, the input fields to use and the actions that may be applied.

- Counters: P4 maintains information across packets using stateful memories: Counters, Meters and Registers.

- Tables: As P4 specification, tables are the fundamental units of the match action pipeline. Tables define rules to perform the input fields to use, and the actions that may be applied. Actions in P4 are declared as functions (i.e., compound actions) which are built from primitive actions (i.e., basic actions).

- Control Program: The control program organizes the layout of tables within ingress and egress pipeline, and the packet flow through the pipeline. It may-be expressed with an imperative program (i.e., main program in P4 language) which may apply tables, call other control flow functions, or test conditions


**References**:

- [SDN-based SYN Flooding Defense in Cloud](https://www.researchgate.net/profile/Safaa_Mahrach2/publication/327651724_SDN-based_SYN_Flooding_Defense_in_Cloud/links/5b9ba9da92851ca9ed07ea70/SDN-based-SYN-Flooding-Defense-in-Cloud.pdf)

- [P4 Specs](https://p4.org/p4-spec/docs/P4-16-v1.2.0.html)
 
