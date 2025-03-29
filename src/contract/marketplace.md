# Cairo Academy: Understanding a Marketplace Contract

This chapter delves into the basics of a Marketplace contract implemented in Cairo. This contract demonstrates the core functionalities of a basic Marketplace contract, providing a practical example for learning how to build a basic Marketplace contract on StarkNet.

## Purpose and Functionality

The provided Cairo code defines a basic lottery contract. It implements essential Marketplace, ERC20 and IERC721, IERC1155 functionalities

```
use starknet::ContractAddress;
```

**Marketplace functionalities**

- **list_asset:** This creates new listing with nft ownership verification. This has parameters:
asset_contract, token_id, start_time, duration, quantity, payment_token, price_per_token, asset_type
- **remove_listing:** This removes listing by replacing with empty listing. This has parameters:
token_id
- **purchase:** Direct purchase at listed price. This has parameters:
listing_id, recipient, quantity, payment_token, total_price
- **accept_bid:**  Seller accepts a specific offer. This has parameters:
listing_id, bidder, payment_token, price_per_token
- **place_bid:**  Places a bid/offer on an existing listing. This has parameters:
listing_id, quantity, payment_token, price_per_token, expiration
- **modify_listing:** Modify existing listing parameters. This has parameters:
listing_id, quantity, reserve_price, buy_now_price, payment_token, start_time, duration
- **get_listing_count:** Returns total number of listings created.

```
#[starknet::interface]
trait IMarketplace<TContractState> {
    fn list_asset(
        ref self: TContractState,
        asset_contract: ContractAddress,
        token_id: u256,
        start_time: u256,
        duration: u256,
        quantity: u256,
        payment_token: ContractAddress,
        price_per_token: u256,
        asset_type: u256,
    );  
    fn remove_listing(ref self: TContractState, listing_id: u256);
    fn purchase(
        ref self: TContractState,
        listing_id: u256,
        recipient: ContractAddress,
        quantity: u256,
        payment_token: ContractAddress,
        total_price: u256,
    );
    fn accept_bid(
        ref self: TContractState,
        listing_id: u256,
        bidder: ContractAddress,
        payment_token: ContractAddress,
        price_per_token: u256
    );
    fn place_bid(
        ref self: TContractState,
        listing_id: u256,
        quantity: u256,
        payment_token: ContractAddress,
        price_per_token: u256,
        expiration: u256
    );
    fn modify_listing(
        ref self: TContractState,
        listing_id: u256,
        quantity: u256,
        reserve_price: u256,
        buy_now_price: u256,
        payment_token: ContractAddress,
        start_time: u256,
        duration: u256,
    );
    fn get_listing_count(self: @TContractState) -> u256;
}
```


**ERC20 functionalities**

The IERC20 interface defines the core functions for an ERC20 token contract 

- **balance_of:** This function returns the token balance of a specific account. This has parameters:
account: The account parameter is the address of the account whose balance is being queried, and the function returns a u256 representing the amount of tokens owned by that account
- **allowance:** This function returns the amount of tokens that one account (spender) is authorized to spend on behalf of another account (owner). This has parameters: owner and spender
- **transfer:** This function transfers a specified amount of tokens from the caller's account to the recipient account. This has parameters: spender, recipient and amount

```
#[starknet::interface]
trait IERC20<TContractState> {
    fn balance_of(self: @TContractState, account: ContractAddress) -> u256;
    fn allowance(self: @TContractState, owner: ContractAddress, spender: ContractAddress) -> u256;
    fn transfer(ref self: TContractState, recipient: ContractAddress, amount: u256);
    fn transfer_from(
        ref self: TContractState, sender: ContractAddress, recipient: ContractAddress, amount: u256
    );
}
```


**IERC721 functionalities**

The IERC721 interface defines the basic functions that an ERC721-compliant contract should implement.ERC721 is a standard for representing non-fungible tokens (NFTs) 

- **owner_of:** This function returns the address of the owner of the NFT represented by token_id. This has parameters: token_id, parameter is a unique identifier for the NFT, and the function returns the ContractAddress of the current owner
- **get_approved:** This function returns the address that has been approved to manage or transfer the specified NFT (token_id). This takes parameter: token_id 
- **is_approved_for_all:** This function checks whether an operator address has been authorized to manage all NFTs owned by an owner address. This takes parameter: owner and operator.
- **transfer_from:** This function transfers the ownership of the NFT represented by token_id from the 'from' address to the 'to' address. The function allows a transfer to occur. 
This has parameters : token_id, two addresses, 'from' and 'to'

```
#[starknet::interface]
trait IERC721<TContractState> {
    fn owner_of(self: @TContractState, token_id: u256) -> ContractAddress;
    fn get_approved(self: @TContractState, token_id: u256) -> ContractAddress;
    fn is_approved_for_all(
        self: @TContractState, owner: ContractAddress, operator: ContractAddress
    ) -> bool;
    fn transfer_from(
        ref self: TContractState, from: ContractAddress, to: ContractAddress, token_id: u256
    );
}
```


**IERC1155 functionalities**

The IERC1155 interface defines the core functions for an ERC1155 token contract. ERC1155 is a token standard that allows a single contract to manage multiple token types, which can be a mix of fungible and non-fungible tokens.

- **balance_of:** This function returns the balance of a specific token id for a given account. This takes parameter: 'account', the address of the account whose balance is being queried, and the 'id' parameter is the identifier of the token,
- **is_approved_for_all:** This function checks if an operator is approved to manage all tokens for a given account. This takes parameter: The 'account', the address of the token owner, and the 'operator', the address being checked for approval. The function returns a boolean value: true if the operator is approved for all tokens, and false otherwise.
- **safe_transfer_from:** This function transfers a specified amount of tokens of type id from the 'from' address to the 'to' address. This takes parameter: Two address 'from', the address from which the tokens are being released and 'to', the address from which the tokens are being sent to. The id is the token identifier, amount is the number of tokens to transfer, and data is additional data to be passed to the recipient.

```
#[starknet::interface]
trait IERC1155<TContractState> {
    fn balance_of(self: @TContractState, account: ContractAddress, id: u256) -> u256;
    fn is_approved_for_all(
        self: @TContractState, account: ContractAddress, operator: ContractAddress
    ) -> bool;
    fn safe_transfer_from(
        ref self: TContractState,
        from: ContractAddress,
        to: ContractAddress,
        id: u256,
        amount: u256,
        data: Span<felt252>
    );
}
```
