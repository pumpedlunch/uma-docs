---
description: >-
  Please note that these tutorials are for an old version of the Optimistic
  Oracle and are only left for reference.
---

# Optimistic Oracle v2

This section showcases different design patterns for building contracts that integrate with the UMA Optimistic Oracle (OO). These include:

* A [simple deposit box](solidity-examples.md) to showcase the basic OO request lifecycle.
* An [event based prediction market](in-depth-tutorial-event-based-prediction-market.md). In this example, settlement requests are submitted at the time of contract deployment, and the OO proposer network is used as a decentralized keeper service that identifies when settlement events happen and propose the outcomes.
* An ["internal Optimistic Oracle"](internal-optimistic-oracle.md), where an integrating contract handles proposals and state management, and only uses the OO when proposal verification is needed or a dispute is filed.
* An [insurance contract](in-depth-tutorial-insurance-claims-arbitration.md), where insurance recipients can submit insurance claims for verification through the OO and the DVM can be used to arbitrate these claims.
* An [Optimistic Arbitrator contract](in-depth-tutorial-optimistic-arbitrator.md), where useful OO functions are wrapped together into single contract calls. Showcases how requests and proposals can be wrapped together into single calls to create "assertions", and requests, proposals and disputes can be wrapped together to create instant DVM arbitration situations.&#x20;
