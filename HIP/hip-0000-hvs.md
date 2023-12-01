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

We propose a new gRPC service, the `VotingService.`

```
/**
 * The Voting Service lets users create elections where voting power derives from ownership
 * of HBAR, units of fungible tokens, or NFTs. One election may recognize multiple forms of
 * voting power. Elections end when either all voting power has been used or a deadline has 
 * passed. 
 * 
 * When creating an election, the user will directly specify the list of choices in the 
 * election; as well as the ContractID of a smart contract that implements a Hedera-specific
 * interface IElection. 
 *  
 * Upon the election's end, this contract will be called with the full results of the voting.
 * It must return the id of the elected choice. If it returns anything other than a string 
 * matching the id of a choice, the election will fail to resolve. 
 * 
 * Anywhere a key can be used in the Hedera ledger to enforce an authorization requirement,
 * that authorization requirement can now be made to depend on the outcome of an election 
 * by setting a new key type that specifies an election outcome. The key will be active iff
 * the referenced election has resolved to the given choice.
 */
service VotingService {
    /**
     * Create a new election with a set of choices that accepts voting not before a start time  
     * and not after an end time. The election's voting power derives from ownership of the 
     * given (weighted) set of token denominations (and/or HBAR); and its outcome is decided 
     * decided by the given contract. This may require authorization, such as a specific admin 
     * key to sign the transaction.
     * The response includes the unique identifier for the newly created election.
     * Request is [ElectionCreateTransactionBody](#proto.ElectionCreateTransactionBody)
     */
    rpc createElection (Transaction) returns (TransactionResponse);

    /**
     * Update an existing election before voting has started. This operation could include 
     * changing the election details, extending its duration, etc.
     * If there is an adminKey, it must sign the transaction for updates.
     * Changes to the adminKey require signatures from both the old and new keys.
     * Request is [ElectionUpdateTransactionBody](#proto.ElectionUpdateTransactionBody)
     */
    rpc updateElection (Transaction) returns (TransactionResponse);

    /**
     * Delete an election before voting has started.
     * This operation will make the election unavailable for further actions.
     * If an adminKey is set, it must sign the transaction to delete the election.
     * Without an adminKey, this operation will be unauthorized.
     * Request is [ElectionDeleteTransactionBody](#proto.ElectionDeleteTransactionBody)
     */
    rpc deleteElection (Transaction) returns (TransactionResponse);

    /**
     * Submit or alter a vote in an election by specifying its id, along with the amounts of
     * one or more tokens or HBAR to use as voting power. The assets used as voting power
     * are automatically locked until the election resolves; to transfer some or all of the
     * voted assets, the user must submit a reduced or repudiated vote.
     * The operation ensures that the vote is valid, authorized, and counted towards the election.
     * The response includes confirmation of the vote being counted.
     * Request is [ElectionSubmitVoteTransactionBody](#proto.ElectionSubmitVoteTransactionBody)
     */
    rpc submitVote (Transaction) returns (TransactionResponse);
}
```

Its four transactions will require a few extensions to the basic types in the system, as below.
```
/*
 * Represents a single choice in an election.
 */
message Choice {
    /*
     * Unique identifier for the choice. The ledger is agnostic about its interpretation.
     */
    string id = 1;
}

/**
 * Unique identifier for an election
 */
message ElectionID {
    /**
     * A nonnegative shard number
     */
    int64 shard_num = 1;

    /**
     * A nonnegative realm number
     */
    int64 realm_num = 2;

    /**
     * A positive election number
     */
    int64 election_num = 3;
}

/**
 * The outcome of an election.
 */
message ElectionOutcome {
    /**
     * The id of an election.
     */
    ElectionID election_id = 1;

    /**
     * A choice from that election.
     */
    Choice choice = 2;
}

/*
 * Represents voting power in an election.
 */
message VotingPower {
    /*
     * The denomination of a token to be used as voting power in an election; taken
     * as HBAR if left unset.
     */
    TokenID token_id = 1;

    /*
     * The relative weight of the voting power given to each whole unit of this TokenID 
     * used to vote.
     */
    int64 weight = 2;
}

/*
 * Represents a vote in an election for a specific form of voting power.
 */
message Vote {
    /*
     * The denomination of a token being used as voting power in an election; taken
     * as HBAR if left unset.
     */
    TokenID token_id = 1;

    /*
     * The amount of the token being voted in its lowest denominated units (e.g., tinybar 
     * if the denomination is HBAR).
     */
    int64 amount = 2;
}
```

With these types in hand, we can define the four transactions for the new `VotingService`.
First, the `ElectionCreateTransactionBody`.

```
import "basic_types.proto";
import "duration.proto";

/*
 * Transaction body for creating a new election.
 */
message ElectionCreateTransactionBody {
    /*
     * Title of the election.
     */
    string title = 1;

    /*
     * Description of the election.
     */
    string description = 2;

    /*
     * List of choices available in the election.
     */
    repeated Choice choices = 3;

    /*
     * The start time of the election.
     * This is when the election becomes active and can accept votes.
     */
    Timestamp start_time = 4;

    /*
     * The end time of the election.
     * This is when the election stops accepting votes and is considered complete.
     */
    Timestamp end_time = 5;

    /*
     * Details of the voting power mechanisms.
     * This can include units of fungible tokens, NFTs, or HBAR that are used for voting.
     * Each item in this repeated field represents a different form of voting power.
     */
    repeated VotingPower voting_powers = 6;

    /*
     * Identifier of the smart contract that determines the election outcome.
     * This contract implements the IElection interface and is called with the full results of the voting.
     */
    ContractID decision_contract_id = 7;
}
```

Second, the `ElectionUpdateTransactionBody`.

```
import "basic_types.proto";
import "duration.proto";
import "google/protobuf/wrappers.proto";

/*
 * Transaction body for updating an existing election before its voting window has opened.
 */
message ElectionUpdateTransactionBody {
    /*
     * Identifier of the election to be updated.
     */
    ElectionID election_id = 1;

    /*
     * Updated title of the election.
     * This field is optional and can be used to change the title of the election.
     */
    google.protobuf.StringValue title = 2;

    /*
     * Updated description of the election.
     * This field is optional and can be used to change the description of the election.
     */
    google.protobuf.StringValue description = 3;

    /*
     * Updated list of choices for the election.
     * This field is optional and can be used to modify the choices available in the election.
     */
    repeated Choice choices = 4;

    /*
     * New start time for the election.
     * This field is optional and can be used to change the start time of the election.
     */
    Timestamp start_time = 5;

    /*
     * New end time for the election.
     * This field is optional and can be used to change the end time of the election.
     */
    Timestamp end_time = 6;

    /*
     * Updated details of the voting power mechanisms.
     * This field is optional and can be used to change the voting tokens or HBAR used for voting.
     * Each item in this repeated field represents a different form of updated voting power.
     */
    repeated VotingPower voting_tokens = 7;

    /*
     * Identifier of the new smart contract for determining the election outcome.
     * This field is optional and can be used to change the contract that decides the election outcome.
     */
    ContractID decision_contract_id = 8;
}
```

Third, the `ElectionDeleteTransactionBody`.

```
import "basic_types.proto";

/*
 * Transaction body for deleting an election.
 */
message ElectionDeleteTransactionBody {
    /*
     * Identifier of the election to be deleted.
     * This field is required to specify which election is to be removed.
     */
    ElectionID election_id = 1;
}
```

Fourth, the `ElectionSubmitVoteTransactionBody`.

```
import "basic_types.proto";

/*
 * Transaction body for submitting or altering a vote in an election.
 */
message ElectionSubmitVoteTransactionBody {
    /*
     * Identifier of the election in which to vote.
     */
    ElectionID election_id = 1;

    /*
     * Identifier of the chosen option in the election.
     * This represents the specific choice the voter is casting their vote for.
     */
    string choice_id = 2;

    /*
     * Details of the voting power being utilized in the vote.
     * This includes the amounts of one or more tokens or HBAR that are being used as voting power.
     * The assets used as voting power are locked until the election resolves.
     */
    repeated VotingPower voting_powers = 3;
}
```

Externalizing the results of these transactions will require extensions to the `TransactionReceipt`
and `TransactionRecord` messages. For `TransactionReceipt` we need to add a field for the id of a newly
created election.
```
message TransactionReceipt {
    ...

    /**
     * In the receipt of an ElectionCreate, the id of the newly created election
     */
    ElectionID election_id = 16;
}
```

And for `TransactionRecord`, we need to add a repeated field for the cumulative votes for each form of 
voting power used in an successful `SubmitVote` transaction.
```
message TransactionRecord {
    ...

    /**
     * In the record of a successful SubmitVote transaction, the new cumulative votes for every form of
     * voting power used in the vote.
     */
    repeated new_cumulative_votes = 22;
}
```

Similarly, the externalized results will require new response codes such as the following.
```
  /*
   * The election is currently in progress and cannot be modified.
   */
  ELECTION_IN_PROGRESS = 334;

  /*
   * The election has already concluded.
   */
  ELECTION_CLOSED = 335;

  /*
   * Updating or deleting the election is not possible.
   */
  ELECTION_IS_IMMUTABLE = 336;

  /*
   * The transaction referenced an invalid election.
   */
  INVALID_ELECTION_ID = 337;

  /*
   * The user's voting power (e.g., tokens, HBAR) is insufficient for the attempted vote.
   */
  VOTING_POWER_INSUFFICIENT = 338;

  /*
   * Execution of the smart contract for the election failed.
   */
  DECISION_CONTRACT_FAILED = 339;

  /*
   * The smart contract did not pass validation checks.
   */
  INVALID_DECISION_CONTRACT_ID = 340;

  /*
   * The format of the choice data in the election is invalid.
   */
  INVALID_CHOICE_FORMAT = 341;

  /*
   * The start time specified for the election is invalid.
   */
  INVALID_START_TIME = 342;

  /*
   * The end time specified for the election is invalid.
   */
  INVALID_END_TIME = 343;

```

Lastly, to make elections first-class citizens in the protocol's authorization scheme, 
we will need extend the `Key` message as below.
```
message Key {
    oneof key {
        ...

        /**  
         * An election outcome which, upon having occurred, implies this key should be
         * treated as having an active signature, wherever it appears. For example, if
         * the payer account for a scheduled transaction has an election outcome key,
         * its scheduled transaction is executable only if the election resolves to the
         * given outcome.
         */
        ElectionOutcome election_outcome = 9; 
    }
}
```

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
