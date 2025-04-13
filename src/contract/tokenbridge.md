# Cairo Academy: Understanding a Token Bridge Contract

This chapter explores a Token Bridge contract implemented in Cairo. This contract demonstrates a mechanism for transferring tokens between Layer 1 (L1) and Layer 2 (L2) on StarkNet, providing a practical example for learning how to build cross-layer token bridging systems.

## Purpose and Functionality

The provided Cairo code defines a basic Token Bridge contract that facilitates the movement of tokens between StarkNet (L2) and an Ethereum-based L1 blockchain. It implements essential functionalities such as:

- **Bridging to L1:** Initiates the transfer of tokens from L2 to L1 by burning them on L2 and sending a message to mint them on L1.
- **Handling Deposits from L1:** Processes incoming token deposits from L1 by minting equivalent tokens on L2.
- **Configuration Management:** Allows a governor to set the L1 bridge address and the L2 token contract address.
- **State Tracking:** Provides getters to retrieve the governor, L1 bridge address, and L2 token contract address.

This contract serves as a foundational example for understanding token bridging mechanics within the StarkNet ecosystem, showcasing cross-layer communication and token management.

## Key Components

### Interfaces

The contract defines two key interfaces:

1. **IMintableToken**: A simplified interface for a token contract that supports minting and burning.
   - `mint`: Creates new tokens and assigns them to an account.
   - `burn`: Destroys tokens from an account.

2. **ITokenBridge**: The main interface for the token bridge contract.
   - `bridge_to_l1`: Initiates a token transfer from L2 to L1.
   - `set_l1_bridge`: Sets the L1 bridge address (governor only).
   - `set_token`: Sets the L2 token contract address (governor only).
   - `governor`: Returns the governor’s address.
   - `l1_bridge`: Returns the L1 bridge address.
   - `l2_token`: Returns the L2 token contract address.

```rust
#[starknet::interface]
pub trait IMintableToken<TContractState> {
    fn mint(ref self: TContractState, account: ContractAddress, amount: u256);
    fn burn(ref self: TContractState, account: ContractAddress, amount: u256);
}

#[starknet::interface]
pub trait ITokenBridge<TContractState> {
    fn bridge_to_l1(ref self: TContractState, l1_recipient: EthAddress, amount: u256);
    fn set_l1_bridge(ref self: TContractState, l1_bridge_address: EthAddress);
    fn set_token(ref self: TContractState, l2_token_address: ContractAddress);
    fn governor(self: @TContractState) -> ContractAddress;
    fn l1_bridge(self: @TContractState) -> felt252;
    fn l2_token(self: @TContractState) -> ContractAddress;
}
```

### State Management

The contract uses a `Storage` struct to maintain its persistent state on StarkNet:

```rust
#[storage]
pub struct Storage {
    // The address of the L2 governor of this contract. Only the governor can set the other storage variables.
    pub governor: ContractAddress,
    // The L1 bridge address. Zero when unset.
    pub l1_bridge: felt252,
    // The L2 token contract address. Zero when unset.
    pub l2_token: ContractAddress,
}
```

- **`governor`:** Stores the address of the L2 governor, who has exclusive rights to configure the contract (e.g., setting the L1 bridge and L2 token addresses).
- **`l1_bridge`:** Holds the L1 bridge contract address as a `felt252`. It defaults to zero until set by the governor.
- **`l2_token`:** Stores the address of the L2 token contract that the bridge interacts with, also defaulting to zero until configured.

These variables are critical for the contract’s operation, ensuring that bridging actions reference the correct L1 and L2 contracts and that only the governor can modify the configuration.

### Core Functionalities

1. **Bridging Tokens to L1:**
   - Burns tokens on L2 from the caller’s balance.
   - Sends a message to the L1 bridge contract to mint the equivalent amount for the specified L1 recipient.
   - Emits a `WithdrawInitiated` event.

2. **Handling Deposits from L1:**
   - Receives a message from the L1 bridge (via an L1 handler).
   - Mints tokens on L2 for the specified recipient.
   - Emits a `DepositHandled` event.

3. **Configuration:**
   - `set_l1_bridge`: Allows the governor to specify the L1 bridge contract address.
   - `set_token`: Allows the governor to specify the L2 token contract address.
   - Both functions emit events (`L1BridgeSet` and `L2TokenSet`) to log changes.

4. **Security Checks:**
   - Ensures only the governor can configure the contract.
   - Validates that the L1 bridge and L2 token addresses are set before bridging operations.
   - Restricts deposit handling to messages from the configured L1 bridge.

### Event System

The contract includes an event system to log key actions:

```rust
#[event]
#[derive(Drop, starknet::Event)]
pub enum Event {
    WithdrawInitiated: WithdrawInitiated,
    DepositHandled: DepositHandled,
    L1BridgeSet: L1BridgeSet,
    L2TokenSet: L2TokenSet,
}

#[derive(Drop, starknet::Event)]
pub struct WithdrawInitiated {
    pub l1_recipient: EthAddress,
    pub amount: u256,
    pub caller_address: ContractAddress,
}

#[derive(Drop, starknet::Event)]
pub struct DepositHandled {
    pub account: ContractAddress,
    pub amount: u256,
}

#[derive(Drop, starknet::Event)]
struct L1BridgeSet {
    l1_bridge_address: EthAddress,
}

#[derive(Drop, starknet::Event)]
struct L2TokenSet {
    l2_token_address: ContractAddress,
}
```

### Error Handling

The contract defines a set of error messages in the `Errors` module to handle invalid states or inputs:

```rust
pub mod Errors {
    pub const EXPECTED_FROM_BRIDGE_ONLY: felt252 = 'Expected from bridge only';
    pub const INVALID_ADDRESS: felt252 = 'Invalid address';
    pub const INVALID_AMOUNT: felt252 = 'Invalid amount';
    pub const UNAUTHORIZED: felt252 = 'Unauthorized';
    pub const TOKEN_NOT_SET: felt252 = 'Token not set';
    pub const L1_BRIDGE_NOT_SET: felt252 = 'L1 bridge address not set';
}
```

- **`EXPECTED_FROM_BRIDGE_ONLY`:** Thrown when the `handle_deposit` function is called by an address other than the configured L1 bridge.
- **`INVALID_ADDRESS`:** Raised when an address (e.g., recipient, governor, or contract address) is zero or otherwise invalid.
- **`INVALID_AMOUNT`:** Triggered when a token amount is zero during bridging or deposit operations.
- **`UNAUTHORIZED`:** Emitted when a non-governor attempts to call configuration functions.
- **`TOKEN_NOT_SET`:** Occurs if the L2 token address hasn’t been set before a bridging operation.
- **`L1_BRIDGE_NOT_SET`:** Thrown if the L1 bridge address hasn’t been configured prior to bridging.

These error messages enhance the contract’s robustness by providing clear feedback on why an operation failed.

### Security Features

- **Governor-Only Access:** Only the governor can set the L1 bridge and L2 token addresses, ensuring controlled configuration.
- **Non-Zero Checks:** Validates addresses and amounts to prevent invalid operations.
- **L1 Handler Restriction:** The `handle_deposit` function only accepts messages from the configured L1 bridge.
- **Cross-Layer Messaging:** Uses StarkNet’s `send_message_to_l1_syscall` for secure L1 communication.

## Usage Example

1. **Setting Up the Bridge:**
   - The governor deploys the contract and sets the L1 bridge and L2 token addresses:
  
     ```rust
     token_bridge.set_l1_bridge(l1_bridge_address);
     token_bridge.set_token(l2_token_address);
     ```

2. **Bridging Tokens to L1:**
   - A user initiates a transfer of 100 tokens to an L1 address:
  
     ```rust
     token_bridge.bridge_to_l1(l1_recipient_address, 100);
     ```

   - The tokens are burned on L2, and a message is sent to L1 to mint them.

3. **Handling a Deposit from L1:**
   - The L1 bridge sends a message to deposit 50 tokens for an L2 user. The contract automatically mints them:

     ```rust
     // Handled via L1 message, not directly called
     token_bridge.handle_deposit(l1_bridge_address, l2_recipient, 50);
     ```

## Full Implementation

```rust
use starknet::{ContractAddress, EthAddress};
 
#[starknet::interface]
pub trait IMintableToken<TContractState> {
    fn mint(ref self: TContractState, account: ContractAddress, amount: u256);
    fn burn(ref self: TContractState, account: ContractAddress, amount: u256);
}
 
#[starknet::interface]
pub trait ITokenBridge<TContractState> {
    fn bridge_to_l1(ref self: TContractState, l1_recipient: EthAddress, amount: u256);
    fn set_l1_bridge(ref self: TContractState, l1_bridge_address: EthAddress);
    fn set_token(ref self: TContractState, l2_token_address: ContractAddress);
    fn governor(self: @TContractState) -> ContractAddress;
    fn l1_bridge(self: @TContractState) -> felt252;
    fn l2_token(self: @TContractState) -> ContractAddress;
}
 
#[starknet::contract]
pub mod TokenBridge {
    use core::num::traits::Zero;
    use starknet::{ContractAddress, EthAddress, get_caller_address, syscalls, SyscallResultTrait};
    use starknet::storage::{StoragePointerReadAccess, StoragePointerWriteAccess};
    use super::{IMintableTokenDispatcher, IMintableTokenDispatcherTrait};
 
    #[storage]
    pub struct Storage {
        pub governor: ContractAddress,
        pub l1_bridge: felt252,
        pub l2_token: ContractAddress,
    }
 
    #[event]
    #[derive(Drop, starknet::Event)]
    pub enum Event {
        WithdrawInitiated: WithdrawInitiated,
        DepositHandled: DepositHandled,
        L1BridgeSet: L1BridgeSet,
        L2TokenSet: L2TokenSet,
    }
 
    #[derive(Drop, starknet::Event)]
    pub struct WithdrawInitiated {
        pub l1_recipient: EthAddress,
        pub amount: u256,
        pub caller_address: ContractAddress,
    }
 
    #[derive(Drop, starknet::Event)]
    pub struct DepositHandled {
        pub account: ContractAddress,
        pub amount: u256,
    }
 
    #[derive(Drop, starknet::Event)]
    struct L1BridgeSet {
        l1_bridge_address: EthAddress,
    }
 
    #[derive(Drop, starknet::Event)]
    struct L2TokenSet {
        l2_token_address: ContractAddress,
    }
 
    pub mod Errors {
        pub const EXPECTED_FROM_BRIDGE_ONLY: felt252 = 'Expected from bridge only';
        pub const INVALID_ADDRESS: felt252 = 'Invalid address';
        pub const INVALID_AMOUNT: felt252 = 'Invalid amount';
        pub const UNAUTHORIZED: felt252 = 'Unauthorized';
        pub const TOKEN_NOT_SET: felt252 = 'Token not set';
        pub const L1_BRIDGE_NOT_SET: felt252 = 'L1 bridge address not set';
    }
 
    #[constructor]
    fn constructor(ref self: ContractState, governor: ContractAddress) {
        assert(governor.is_non_zero(), Errors::INVALID_ADDRESS);
        self.governor.write(governor);
    }
 
    #[abi(embed_v0)]
    impl TokenBridge of super::ITokenBridge<ContractState> {
        fn bridge_to_l1(ref self: ContractState, l1_recipient: EthAddress, amount: u256) {
            assert(l1_recipient.is_non_zero(), Errors::INVALID_ADDRESS);
            assert(amount.is_non_zero(), Errors::INVALID_AMOUNT);
            self._assert_l1_bridge_set();
            self._assert_token_set();
 
            let caller_address = get_caller_address();
            IMintableTokenDispatcher { contract_address: self.l2_token.read() }
                .burn(caller_address, amount);
 
            let mut payload: Array<felt252> = array![
                l1_recipient.into(), amount.low.into(), amount.high.into(),
            ];
            syscalls::send_message_to_l1_syscall(self.l1_bridge.read(), payload.span())
                .unwrap_syscall();
 
            self.emit(WithdrawInitiated { l1_recipient, amount, caller_address });
        }
 
        fn set_l1_bridge(ref self: ContractState, l1_bridge_address: EthAddress) {
            self._assert_only_governor();
            assert(l1_bridge_address.is_non_zero(), Errors::INVALID_ADDRESS);
 
            self.l1_bridge.write(l1_bridge_address.into());
            self.emit(L1BridgeSet { l1_bridge_address });
        }
 
        fn set_token(ref self: ContractState, l2_token_address: ContractAddress) {
            self._assert_only_governor();
            assert(l2_token_address.is_non_zero(), Errors::INVALID_ADDRESS);
 
            self.l2_token.write(l2_token_address);
            self.emit(L2TokenSet { l2_token_address });
        }
 
        fn governor(self: @ContractState) -> ContractAddress {
            self.governor.read()
        }
 
        fn l1_bridge(self: @ContractState) -> felt252 {
            self.l1_bridge.read()
        }
 
        fn l2_token(self: @ContractState) -> ContractAddress {
            self.l2_token.read()
        }
    }
 
    #[generate_trait]
    impl Internal of InternalTrait {
        fn _assert_only_governor(self: @ContractState) {
            assert(get_caller_address() == self.governor.read(), Errors::UNAUTHORIZED);
        }
 
        fn _assert_token_set(self: @ContractState) {
            assert(self.l2_token.read().is_non_zero(), Errors::TOKEN_NOT_SET);
        }
 
        fn _assert_l1_bridge_set(self: @ContractState) {
            assert(self.l1_bridge.read().is_non_zero(), Errors::L1_BRIDGE_NOT_SET);
        }
    }
 
    #[l1_handler]
    pub fn handle_deposit(
        ref self: ContractState, from_address: felt252, account: ContractAddress, amount: u256,
    ) {
        assert(from_address == self.l1_bridge.read(), Errors::EXPECTED_FROM_BRIDGE_ONLY);
        assert(account.is_non_zero(), Errors::INVALID_ADDRESS);
        assert(amount.is_non_zero(), Errors::INVALID_AMOUNT);
        self._assert_token_set();
 
        IMintableTokenDispatcher { contract_address: self.l2_token.read() }.mint(account, amount);
 
        self.emit(Event::DepositHandled(DepositHandled { account, amount }));
    }
}
```
