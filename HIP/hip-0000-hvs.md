---
hip: <HIP number (this is determined by the HIP editor)>
title: Hedera Voting Service
author: Matthew DeLorenzo <mdelore.ufl@gmail.com>, Vae Vecturne <vea.vecturne@gmail.com>
working-group: Cody Littley <cody@swirldslabs.com>, Neeharika Sompalli <neeharika.sompalli@swirldslabs.com>, Michael Tinker <michael.tinker@swirldslabs.com>, GPT-4 <info@openai.com> 
type: Standards Track
category: Core
needs-council-approval: Yes
status: Draft
created: 2023-04-07
discussions-to: {{TODO - Tarzan, Vecturne}}
updated: 2023-04-07
---

## Abstract

For thousands of years, voting has played a central role in democratic society. Citizens vote to express preferences, hold their 
representatives accountable, and shape the nature of governance. The health of a democracy depends on the inclusiveness and integrity 
of its vote.

Voting is just as central to governance of a decentralized application (dApp). An ideal vote distributes power among token holders to 
make decisions, resolve conflicts, and evolve the protocol. But this ideal requires a ledger that can support fast, fair, and secure 
on-chain voting, at a scale of millions of token holders. An ideal ledger would also allow a dApp to use a vote's outcome as an input 
to its native programmability and embedded economics. We propose to make Hedera such a ledger. 

There are five main aspects of our proposal.
  1. The protocol adds a Hedera Voting Service (HVS) that creates and manages entities of type `Vote`.
  2. Every native Hedera token gains the ability to be used as voting power for any number of `Vote`s.
  3. Token holders who vote have their vote weight "checkpointed" at the `Vote` end time.
  4. An `Outcome` of a `Vote` can trigger any authorized transaction, e.g. `TokenMint`.
  5. The `Key` entity adds a `Vote`-derived authorization mode.

Below we specify all this in detail, giving special attention to the performance impact on the existing Hedera protocol. For example, 
we will show that even if every holder of a fungible Hedera token `0.0.T` was voting in multiple HVS elections for which `0.0.T` has 
voting power, it will remain possible to transfer units of `0.0.T` at 10,000 transactions per second (TPS).

## Motivation

### Prior art and ad-hoc Hedera voting schemes

{{TODO - Michael}}

### Token utility

{{TODO - Tarzan, Vecturne}}

## Rationale

### Voting algorithms

{{TODO - ???}}

### Data structures

{{TODO - Cody}}

### HAPI changes

{{TODO - Neeha, Michael}}

## User stories

{{TODO - ALL}}
  
## Specification

## Voting algorithms

{{TODO - ???}}

### HAPI changes

{{TODO - Neeha, Michael}}

### Data structures

{{TODO - Cody}}

### 

## Backwards Compatibility

As HVS is an entirely new protocol dimension, the only backward

## Security Implications

### Denial of service attacks

{{TODO - Cody}}

### Vote-derived authorization

{{TODO - Michael}}

## How to Teach This

{{TODO - ALL}}

## Reference Implementation

{{TODO - ALL}}

## Rejected Ideas

{{TODO - ALL}}

## Open Issues

{{TODO - ALL}}

## References

{{TODO - ALL}}

## Copyright/license

This document is licensed under the Apache License, Version 2.0 -- see [LICENSE](../LICENSE) or (https://www.apache.org/licenses/LICENSE-2.0)
