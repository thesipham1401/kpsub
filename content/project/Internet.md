---
title: "Internet [Backend Eps 1]"
date: 2020-08-16T21:41:48+07:00
draft: false
authors: [Kien Pham]
tags: [Backend]
categories: [Backend]
---
**To understand the Internet concepts, I should answer 6 questions to dip into the backend works.**
1. How does the **Internet** work?
2. What is the **HTTP**?
3. **Browsers** and how they work?
4. **DNS** and how it works?
5. What is **Domain Name**?
6. What is **hosting**?
___________________________
### How does the Internet work?
- The Internet works through a packet routing network in accordance with the Internet Protocol (IP), the Transport Control Protocol (TCP) and other protocols.

- A protocol is a set of rules specifying how computers should communicate with each other over a network.

- In telecommunications and computer networking, a network packet is a formatted unit data carried by a packet-switched network. The digital packet switching concept was first invented by Paul Baran in the early 1960's. This leads the Internet explosion afterward, several new communications technologies in part based upon the concept of packets.

- A packet routing network is a network that routes packets from source computer to a destination computer. The Internet is made up of a massive network of specialized computers called routers. There are two kinds routing: Static and Dynamic.

  - Two major classes of Dynamic Routing Protocols: Distance Vector and Link State Protocols

    + Distance Vector Routing Protocol: **RIP - Routing Information Protocol** and **IGRP - Interior Gateway Routing Protocol.** Distance Vector routing protocols base their decisions on the best path to a given destination based on the distance. Distance Vector can route on the Internet environment, that have the dynamic cost, thank to Bellman-Ford equation. Bellman-Ford algorithm depends on a recursive formula, so it not depend on the cost.   

    + Link State Routing Protocol: **OSPF - Open Shortest Path First** and **IS-IS - Intermediate System to Intermediate System**. Link sate routing protocols send information about directly connected links to all the routers in the networks to complete picture of the network topology. To find the shortest path, it uses Dijkstra algorithm.

    + There are also routing protocols that are considered to be hybrid in sense that they use aspects of both distance vector and link state protocol. **EIGRP - Enhanced Interior Gateway Routing Protocols** is one of those hybrid routing protocols.

...
