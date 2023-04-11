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
discussions-to: _TODO - Tarzan, Vecturne_
updated: 2023-04-07
---

## Abstract

For thousands of years, voting has played a key role in democratic society. Citizens vote to express preferences, hold their 
representatives accountable, and shape the nature of their government. The health of a democracy depends on the inclusiveness 
and integrity of its vote.

Voting is just as critical to governance of a decentralized application (dApp). An ideal vote distributes power among token holders to 
make decisions, resolve conflicts, and evolve the protocol. But this ideal does require a ledger that can support fast, fair, and secure 
on-chain voting, at a scale of millions or billions of token holders. An ideal ledger would also allow a dApp to use a vote's outcome as 
an input to its native programmability and embedded economics. We propose to make Hedera such a ledger. 

There are five main aspects of our proposal.
  1. The protocol adds a Hedera Voting Service (HVS) that creates and manages entities of type `Vote`.
  2. Every native Hedera token gains the ability to be used as voting power for any number of `Vote`s.
  3. Token holders who vote have their vote weight "checkpointed" at the `Vote` end time.
  4. An `Outcome` of a `Vote` can trigger any authorized transaction, e.g. `TokenMint`.
  5. The `Key` entity adds a `Vote`-derived authorization mode.

Below we specify all this in detail, giving special attention to the performance impact on the existing Hedera protocol. For example, 
we will show that even if every holder of a fungible Hedera token `0.0.T` is voting in multiple HVS elections for which `0.0.T` has 
voting power, it will remain possible to transfer units of `0.0.T` at 10,000 transactions per second (TPS).

## Motivation

### Prior art and ad-hoc Hedera voting schemes

_TODO - Michael_

### Token utility

_TODO - Tarzan, Vecturne_

Decentralized protocols commonly issue governance tokens to users as a reward for using the platform and providing value to the protocol's community (e.g. UNI, SUSHI, CRV, APE). Governance tokens are used to quantify voting power within their respective communities to create proposals and vote on their passage. Token utility is derived from this voting power by allowing token holders to influence the direction of the project and community, and is critical in the formation of a decentralized autonomous organization (DAO). There is already some real-world legal precedents for DAO's. In the United States, the State of Wyoming legislature approved a bill that grants legal company status to DAOs that operate on a blockchain [https://www.wyoleg.gov/2021/Introduced/SF0038.pdf]. 

Decentralized Finance (DeFi) protocols may choose an array of fungible and/or nonfungible tokens to compute voting power in order to minimize liquidity removal and keep operations functioning normally [https://docs.sushi.com/docs/Governance/Proposals%20&%20Voting]. For DeFi protocols, assigning governance power to an arbitrary set of tokens could be beneficial to incentivizing liquidity provision and avoid adversely affecting token reserves.

## Rationale

### Voting algorithms

_TODO - ???_

### Data structures

_TODO - Cody_

### HAPI changes

_TODO - Neeha, Michael_

## User stories

_TODO - ALL_

As a user, I want to submit proposals for community-wide voting, and participate in elections on proposals that reach quorum.
As a user, I want the results of an election to trigger an on-chain result autonomously, if the election pertains to triggering an on-chain transaction, e.g. a token transfer from a project's DAO treasury.

As a developer, I would like to create an election with arbitrary start and end times.
As a developer, I would like to see election results of past elections, and elections that are open.
  
## Specification

### HAPI protocol extensions

_TODO - Neeha, Michael_

#### The `Vote` entity

_TODO - ALL_

#### Nature of an `Outcome` 

_TODO - Neeha, Michael_

#### A `Key` for `Vote`-controlled assets

_TODO - Neeha, Michael_

### Algorithms and data structures

_TODO - Cody_

## Backwards Compatibility

Since HVS is an entirely new dimension to the protocol, strictly speaking there are no issues of backwards compatibility; but we may still
ask about its possible impact on the existing protocol. For a detailed analysis on performance and scalability impacts, please see the 
section on algorithms and data structures. For information on how existing entities might opt-in to `Vote`-controlled authorization 
patterns, please visit the section on the new `Key{control_archetype=Vote}` type.

## Security Implications

### Denial of service attacks

_TODO - Cody_

### Vote-derived authorization

_TODO - Michael_

## How to Teach This

_TODO - ALL_

## Reference Implementation

_TODO - ALL_

## Rejected Ideas

_TODO - ALL_

## Open Issues

_TODO - ALL_

## References

_TODO - ALL_

## Copyright/license

This document is licensed under the Apache License, Version 2.0 -- see [LICENSE](../LICENSE) or (https://www.apache.org/licenses/LICENSE-2.0)
