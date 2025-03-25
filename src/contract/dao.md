# Cairo Academy: Understanding a Basic DAO Contract

This chapter explores a fundamental DAO (Decentralized Autonomous Organization) contract implemented in Cairo. This contract demonstrates core DAO functionalities, providing a practical example for learning how to build decentralized governance systems on StarkNet.

## Purpose and Functionality

The provided Cairo code defines a basic DAO contract with the following capabilities:

- **Proposal Management:** Create, vote on, and execute governance proposals
- **Membership Control:** Add and remove members with voting power
- **Voting System:** Track votes for and against proposals
- **Governance Rules:** Enforce voting periods and execution conditions

## Key Components

### Data Structures

```cairo
#[derive(Copy, Drop, Hash)]
struct ProposalId {
    id: u64
}

#[derive(Copy, Drop, Serde, starknet::Store)]
struct Proposal {
    proposer: ContractAddress,
    description: felt252,
    for_votes: u256,
    against_votes: u256,
    start_block: u64,
    end_block: u64,
    executed: bool
}

#[derive(Copy, Drop, Serde, starknet::Store)]
struct Member {
    address: ContractAddress,
    voting_power: u256
}
```

### Core Functionalities

1. **Proposal Creation:**
    - Only members can create proposals
    - Each proposal has a 100-block voting period
    - Proposals store vote counts and execution status

2. **Voting Mechanism:**
    - Members vote with their voting power
    - Votes can be for or against proposals
    - Voting is only allowed during the active period

3. **Proposal Execution:**
    - Can only execute after voting period ends
    - Requires more for-votes than against-votes
    - Prevents double execution

4. **Membership Management:**
    - Admin-controlled member additions/removals
    - Members have associated voting power

### Interface Definition

```cairo 
#[starknet::interface]
pub trait IDAO<TContractState> {
    fn create_proposal(ref self: TContractState, description: felt252);
    fn vote(ref self: TContractState, proposal_id: ProposalId, support: bool);
    fn execute_proposal(ref self: TContractState, proposal_id: ProposalId);
    fn add_member(ref self: TContractState, member: ContractAddress, voting_power: u256);
    fn remove_member(ref self: TContractState, member: ContractAddress);
    fn get_proposal(self: @TContractState, proposal_id: ProposalId) -> Proposal;
    fn get_member(self: @TContractState, member: ContractAddress) -> Member;
}
```

### Event System

```cairo
#[event]
enum Event {
    ProposalCreated: ProposalCreatedEvent,
    Voted: VotedEvent,
    ProposalExecuted: ProposalExecutedEvent,
    MemberAdded: MemberAddedEvent,
    MemberRemoved: MemberRemovedEvent
}
```

### Security Features

- Member verification for proposal creation
- Voting period enforcement
- Execution condition checks
- Admin-only member management

### Usage Example
- Admin adds members with `add_member`
- Members create proposals with `create_proposal`
- Other members vote with vote
- After voting period, anyone can execute_proposal if it passed

This contract serves as a foundation for building more complex DAO systems on StarkNet, demonstrating key concepts like decentralized governance and member voting.