# Cairo Academy: Understanding a Voting Contract

This chapter explores a comprehensive voting contract implemented in Cairo. This contract demonstrates how to build a decentralized voting system on StarkNet, providing a practical example for learning how to develop governance mechanisms for blockchain applications.

## Purpose and Functionality

The provided Cairo code defines a voting contract with ownership controls. It implements essential functionalities such as:

- **Vote Creation:** Creating new votes with custom titles, descriptions, and candidates.
- **Vote Management:** Setting voting periods with configurable start and end times.
- **Vote Casting:** Allowing users to cast votes for their preferred candidates.
- **Vote Tracking:** Maintaining records of votes cast and preventing duplicate voting.
- **Results Retrieval:** Providing mechanisms to access vote results and details.
- **Access Control:** Implementing ownership mechanisms to control administrative functions.

This contract serves as a foundational example for understanding the mechanics of decentralized voting systems within the StarkNet ecosystem.

## Contract Structure

```rust
#[starknet::interface]
trait IVoteContract<TContractState> {
    fn create_vote(
        ref self: TContractState,
        title: felt252,
        description: felt252,
        candidates: Span<felt252>,
        voting_delay: u64,
        voting_period: u64
    ) -> u32;
    fn get_candidate_count(self: @TContractState, vote_id: u32) -> u32;
    fn vote(ref self: TContractState, vote_id: u32, candidate_id: u32);
    fn get_vote_details(self: @TContractState, vote_id: u32) -> (felt252, felt252, u64, u64);
    fn get_vote_count(self: @TContractState) -> u32;
    fn get_vote_results(self: @TContractState, vote_id: u32) -> Span<(felt252, u32)>;
    fn is_voting_active(self: @TContractState, vote_id: u32) -> bool;
}

#[starknet::contract]
mod VoteContract {
    use core::starknet::storage::{StoragePointerReadAccess, StoragePointerWriteAccess};
    use OwnableComponent::InternalTrait;
    use openzeppelin::access::ownable::OwnableComponent;
    use starknet::{
        ContractAddress, get_caller_address, get_block_timestamp, event::EventEmitter, storage::Map
    };

    component!(path: OwnableComponent, storage: ownable, event: OwnableEvent);

    #[abi(embed_v0)]
    impl OwnableImpl = OwnableComponent::OwnableImpl<ContractState>;
    impl OwnableInternalImpl = OwnableComponent::InternalImpl<ContractState>;

    #[storage]
    struct Storage {
        votes: Map<u32, VoteNode>,
        vote_count: u32,
        candidates: Map<(u32, u32), felt252>,
        votes_cast: Map<(u32, u32), u32>,
        voters: Map<(u32, ContractAddress), bool>,
        #[substorage(v0)]
        ownable: OwnableComponent::Storage
    }

    #[derive(Drop, Serde, starknet::Store, Clone)]
    struct VoteNode {
        title: felt252,
        description: felt252,
        candidates_count: u32,
        voting_start_time: u64,
        voting_end_time: u64,
    }

    #[event]
    #[derive(Drop, starknet::Event)]
    enum Event {
        VoteCast: VoteCast,
        VoteCreated: VoteCreated,
        #[flat]
        OwnableEvent: OwnableComponent::Event
    }

    #[derive(Drop, starknet::Event)]
    struct VoteCreated {
        #[key]
        vote_id: u32,
        title: felt252,
        description: felt252,
        voting_start_time: u64,
        voting_end_time: u64,
    }

    #[derive(Drop, starknet::Event)]
    struct VoteCast {
        #[key]
        vote_id: u32,
        voter: ContractAddress,
        candidate_id: u32,
    }

    #[constructor]
    fn constructor(ref self: ContractState, initial_owner: ContractAddress) {
        self.vote_count.write(0);
        self.ownable.initializer(initial_owner);
    }

    #[abi(embed_v0)]
    impl IVoteContractImpl of super::IVoteContract<ContractState> {
        fn create_vote(
            ref self: ContractState,
            title: felt252,
            description: felt252,
            candidates: Span<felt252>,
            voting_delay: u64,
            voting_period: u64
        ) -> u32 {
            self.ownable.assert_only_owner();

            assert!(title != 0, "Title cannot be empty");
            assert!(description != 0, "Description cannot be empty");

            assert!(candidates.len() > 0, "At least one candidate is required");
            for i in 0
                ..candidates
                    .len() {
                        assert!(*candidates.at(i.into()) != 0, "Candidate name cannot be empty");
                    };

            assert!(voting_delay > 0, "Voting delay must be greater than 0");
            assert!(voting_period > 0, "Voting period must be greater than 0");

            let mut vote_count = self.vote_count.read() + 1;
            self.vote_count.write(vote_count);

            let current_time = get_block_timestamp();
            let voting_start_time = current_time + voting_delay;
            let voting_end_time = voting_start_time + voting_period;

            let vote = VoteNode {
                title,
                description,
                candidates_count: candidates.len(),
                voting_start_time,
                voting_end_time,
            };

            self.votes.write(vote_count, vote.clone());

            let mut i: u32 = 0;
            loop {
                if i >= candidates.len() {
                    break;
                }
                self.candidates.write((vote_count, i), *candidates.at(i.into()));
                i += 1;
            };

            self
                .emit(
                    Event::VoteCreated(
                        VoteCreated {
                            vote_id: vote_count,
                            title,
                            description,
                            voting_start_time,
                            voting_end_time,
                        }
                    )
                );
            vote_count
        }

        fn get_candidate_count(self: @ContractState, vote_id: u32) -> u32 {
            assert!(vote_id > 0 && vote_id <= self.vote_count.read(), "Invalid vote ID");
            let vote = self.votes.read(vote_id);
            vote.candidates_count
        }

        fn vote(ref self: ContractState, vote_id: u32, candidate_id: u32) {
            assert!(vote_id > 0 && vote_id <= self.vote_count.read(), "Invalid vote ID");
            let vote = self.votes.read(vote_id);
            assert!(candidate_id < vote.candidates_count, "Invalid candidate ID");

            let caller = get_caller_address();
            let current_time = get_block_timestamp();

            assert!(current_time >= vote.voting_start_time, "Voting has not started yet");
            assert!(current_time <= vote.voting_end_time, "Voting period has ended");
            assert!(!self.voters.read((vote_id, caller)), "Caller has already voted");

            let current_votes = self.votes_cast.read((vote_id, candidate_id));
            self.votes_cast.write((vote_id, candidate_id), current_votes + 1);
            self.voters.write((vote_id, caller), true);

            self.emit(Event::VoteCast(VoteCast { vote_id, voter: caller, candidate_id, }));
        }

        fn get_vote_details(self: @ContractState, vote_id: u32) -> (felt252, felt252, u64, u64) {
            assert!(vote_id > 0 && vote_id <= self.vote_count.read(), "Invalid vote ID");
            let vote = self.votes.read(vote_id);
            (vote.title, vote.description, vote.voting_start_time, vote.voting_end_time)
        }

        fn get_vote_count(self: @ContractState) -> u32 {
            self.vote_count.read()
        }

        fn get_vote_results(self: @ContractState, vote_id: u32) -> Span<(felt252, u32)> {
            assert!(vote_id > 0 && vote_id <= self.vote_count.read(), "Invalid vote ID");
            let vote = self.votes.read(vote_id);
            let mut results = ArrayTrait::new();
            let mut i: u32 = 0;
            loop {
                if i >= vote.candidates_count {
                    break;
                }
                let candidate = self.candidates.read((vote_id, i));
                let votes = self.votes_cast.read((vote_id, i));
                results.append((candidate, votes));
                i += 1;
            };
            results.span()
        }

        fn is_voting_active(self: @ContractState, vote_id: u32) -> bool {
            assert!(vote_id > 0 && vote_id <= self.vote_count.read(), "Invalid vote ID");
            let vote = self.votes.read(vote_id);
            let current_time = get_block_timestamp();
            current_time >= vote.voting_start_time && current_time <= vote.voting_end_time
        }
    }
}
```

## Key Components

### Storage Structure

The contract uses a sophisticated storage system to manage voting data:

1. **votes**: A mapping from vote IDs to `VoteNode` structures containing vote details
2. **vote_count**: A counter to keep track of the total number of votes created
3. **candidates**: A mapping to store candidate names for each vote
4. **votes_cast**: A mapping to track the number of votes each candidate has received
5. **voters**: A mapping to prevent users from voting multiple times
6. **ownable**: A sub-storage component that implements ownership controls

### Vote Creation

The `create_vote` function enables the owner to create new votes by specifying:
- A title and description
- A list of candidates
- A voting delay (time before voting starts)
- A voting period (duration of the voting window)

The function performs several validations:
- Only the contract owner can create votes
- Title and description cannot be empty
- At least one candidate must be provided
- Candidate names cannot be empty
- Voting delay and period must be greater than zero

Upon creation, a unique vote ID is assigned and returned, and a `VoteCreated` event is emitted.

### Vote Casting

The `vote` function allows users to cast votes for specific candidates in active voting sessions:
- Users can only vote once per voting session
- Voting is only allowed during the active voting period
- The function validates both vote and candidate IDs
- A `VoteCast` event is emitted when a vote is successfully cast

### Result Retrieval

The contract provides several functions to access vote information:
- `get_vote_details`: Returns basic information about a vote
- `get_vote_results`: Returns the current vote count for each candidate
- `get_candidate_count`: Returns the number of candidates in a specific vote
- `is_voting_active`: Checks if a vote is currently active

## Events

The contract emits events to track important actions:

1. **VoteCreated**: Emitted when a new vote is created, including vote details and timing information
2. **VoteCast**: Emitted when a user successfully casts a vote
3. **OwnableEvent**: Events related to ownership changes (inherited from the OwnableComponent)

## Access Control

The contract leverages OpenZeppelin's ownable component to implement access control:
- Only the designated owner can create new votes
- The owner is set during contract initialization
- Standard ownership functions are available through the OwnableComponent

## Advanced Features

### Time-Based Controls

The contract implements time-based controls for voting periods:
- Votes have configurable start and end times
- Users can only vote during the active voting period
- The contract uses StarkNet's block timestamp to enforce these constraints

### Double-Voting Prevention

To maintain integrity, the contract prevents users from voting multiple times in the same voting session by tracking voters in the storage.
