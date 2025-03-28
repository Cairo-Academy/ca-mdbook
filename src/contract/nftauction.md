# Cairo Academy: Understanding a Basic NFT Auction Contract

This chapter delves into the basics of an NFT Auction Contract implemented in Cairo. This contract demonstrates the core functionalities of a NFT Auction Contract, providing a practical example for learning how to build a basic NFT Auction Contract on StarkNet. 

## Purpose and Functionality

The provided Cairo code defines a NFT Auction Contract. It implements essential functionalities of ERC20 such as:

- **Get Name:** This gets the name of the ERC20.
- **Get Symbol:** This gets the symbol of the ERC20
- **Get Decinals:** This gets the decimals of the ERC20
- **Get Balance:** This get balance of the account. This takes a parameter:
 account : This stores the token
- **Allowance:** This is the amount of token the spender can make on behalf of the owner. This takes parameters:
owner : This is the account owner
spender : Authorized person to spend on behalf of the owner
- **Tranfer:** This makes transfer to recipient. This takes parameter:
recipient : This is the reciever of the token
account : This stores the token
- **Transfer From:** This makes transfer from sender to reciever. This takes parameters:
sender, recipient and amount
- **Approve:** This gives authority to spender to spend on behalf of the owner
- **Increase Allowance:** This increases the spender's allowance
- **Decrease Allowance:** This decreases the spender's allowance

```
use starknet::ContractAddress;
```

```
#[starknet::interface]
pub trait IERC20<TContractState> {
    fn get_name(self: @TContractState) -> felt252;
    fn get_symbol(self: @TContractState) -> felt252;
    fn get_decimals(self: @TContractState) -> u8;
    fn get_total_supply(self: @TContractState) -> felt252;
    fn balance_of(self: @TContractState, account: ContractAddress) -> felt252;
    fn allowance(
        self: @TContractState, owner: ContractAddress, spender: ContractAddress,
    ) -> felt252;
    fn transfer(ref self: TContractState, recipient: ContractAddress, amount: felt252);
    fn transfer_from(
        ref self: TContractState,
        sender: ContractAddress,
        recipient: ContractAddress,
        amount: felt252,
    );
    fn approve(ref self: TContractState, spender: ContractAddress, amount: felt252);
    fn increase_allowance(ref self: TContractState, spender: ContractAddress, added_value: felt252);
    fn decrease_allowance(
        ref self: TContractState, spender: ContractAddress, subtracted_value: felt252,
    );
}
```
This NFT Auction Contract also implements essential functionalities of IERC721 such as:

- **Get Name:** This function returns the name of the NFT
- **Get Symbol:** This function returns the symbol of the NFT token (e.g., "NFT")
- **Get Token URI:** This function returns the Uniform Resource Identifier (URI) for a specific token ID. This takes parameter:
token_id : This is specific token identification
- **Get Balance:** This function returns the number of NFTs owned by a specific account. This takes parameter:
account : This holds the NFT value
- **Get Approval:** This function returns the address that has been approved to transfer a specific token ID. This takes parameter:
token_id
- **Is Approved for All:** This function checks if an operator has been approved to manage all of the tokens for a given owner. This takes parameter: owner and operator
- **Approve:** This function allows the owner of a token to approve another address (to) to transfer the token (token_id). This takes parameter: token_id and an address
- **Set Is Approved for All:** This function enables or disables approval for an operator to manage all of the caller’s assets. This takes parameter: operator and a bool (true/false)
- **Transfer from:** This function transfers the ownership of a specific token (token_id) from one address (from) to another address (to). The caller must be approved to make this transfer. This takes parameter: token_id, address (from) and address (to)
- **Mint:** This function creates a new token (token_id) and assigns it to a specific address (to). This takes parameter: token_id and an address

```
#[starknet::interface]
trait IERC721<TContractState> {
    fn get_name(self: @TContractState) -> felt252;
    fn get_symbol(self: @TContractState) -> felt252;
    fn get_token_uri(self: @TContractState, token_id: u256) -> felt252;
    fn balance_of(self: @TContractState, account: ContractAddress) -> u256;
    fn owner_of(self: @TContractState, token_id: u256) -> ContractAddress;
    fn get_approved(self: @TContractState, token_id: u256) -> ContractAddress;
    fn is_approved_for_all(
        self: @TContractState, owner: ContractAddress, operator: ContractAddress,
    ) -> bool;
    fn approve(ref self: TContractState, to: ContractAddress, token_id: u256);
    fn set_approval_for_all(ref self: TContractState, operator: ContractAddress, approved: bool);
    fn transfer_from(
        ref self: TContractState, from: ContractAddress, to: ContractAddress, token_id: u256,
    );
    fn mint(ref self: TContractState, to: ContractAddress, token_id: u256);
}
```
This NFT Auction Contract implements **Buy** and **Get Price** functions. This functions buys and gets price of the NFTs

```
pub trait INFTAuction<TContractState> {
    fn buy(ref self: TContractState, token_id: u256);
    fn get_price(self: @TContractState) -> u64;
}
```
## Implementation

```
#[starknet::contract]
pub mod NFTAuction {
    use super::{IERC20Dispatcher, IERC20DispatcherTrait, IERC721Dispatcher, IERC721DispatcherTrait};
    use starknet::{ContractAddress, get_caller_address, get_block_timestamp};
    use starknet::storage::{StoragePointerReadAccess, StoragePointerWriteAccess};
 
    #[storage]
    struct Storage {
        erc20_token: ContractAddress,
        erc721_token: ContractAddress,
        starting_price: u64,
        seller: ContractAddress,
        duration: u64,
        discount_rate: u64,
        start_at: u64,
        expires_at: u64,
        purchase_count: u128,
        total_supply: u128,
    }
 
    mod Errors {
        pub const AUCTION_ENDED: felt252 = 'auction has ended';
        pub const LOW_STARTING_PRICE: felt252 = 'low starting price';
        pub const INSUFFICIENT_BALANCE: felt252 = 'insufficient balance';
    }
 
    #[constructor]
    fn constructor(
        ref self: ContractState,
        erc20_token: ContractAddress,
        erc721_token: ContractAddress,
        starting_price: u64,
        seller: ContractAddress,
        duration: u64,
        discount_rate: u64,
        total_supply: u128,
    ) {
        assert(starting_price >= discount_rate * duration, Errors::LOW_STARTING_PRICE);
 
        self.erc20_token.write(erc20_token);
        self.erc721_token.write(erc721_token);
        self.starting_price.write(starting_price);
        self.seller.write(seller);
        self.duration.write(duration);
        self.discount_rate.write(discount_rate);
        self.start_at.write(get_block_timestamp());
        self.expires_at.write(get_block_timestamp() + duration * 1000);
        self.total_supply.write(total_supply);
    }
 
    #[abi(embed_v0)]
    impl NFTDutchAuction of super::INFTAuction<ContractState> {
        fn get_price(self: @ContractState) -> u64 {
            let time_elapsed = (get_block_timestamp() - self.start_at.read())
                / 1000; // Ignore milliseconds
            let discount = self.discount_rate.read() * time_elapsed;
            self.starting_price.read() - discount
        }
 
        fn buy(ref self: ContractState, token_id: u256) {
            // Check duration
            assert(get_block_timestamp() < self.expires_at.read(), Errors::AUCTION_ENDED);
            // Check total supply
            assert(self.purchase_count.read() < self.total_supply.read(), Errors::AUCTION_ENDED);
 
            let erc20_dispatcher = IERC20Dispatcher { contract_address: self.erc20_token.read() };
            let erc721_dispatcher = IERC721Dispatcher {
                contract_address: self.erc721_token.read(),
            };
 
            let caller = get_caller_address();
            // Get NFT price
            let price: u256 = self.get_price().into();
            let buyer_balance: u256 = erc20_dispatcher.balance_of(caller).into();
            // Ensure buyer has enough token for payment
            assert(buyer_balance >= price, Errors::INSUFFICIENT_BALANCE);
            // Transfer payment token from buyer to seller
            erc20_dispatcher.transfer_from(caller, self.seller.read(), price.try_into().unwrap());
            // Mint token to buyer's address
            erc721_dispatcher.mint(caller, token_id);
            // Increase purchase count
            self.purchase_count.write(self.purchase_count.read() + 1);
        }
    }
}
```