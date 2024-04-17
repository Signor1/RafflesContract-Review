# Raffle Contract

## Overview

### Introduction:
The Raffle Contract, owned by Aavegotchi, serves as a platform for conducting raffles, utilizing ERC1155 tokens, specifically tailored to the Aavegotchi ecosystem. Developed by [Nick Mudge] and [Daniel Mathieu]. The contract is designed to facilitate seamless interaction between users and the Aavegotchi platform, enhancing engagement and promoting community involvement.

### Business Logic:
The contract enables the initiation of raffles by the contract owner, Aavegotchi, with customizable parameters such as duration and associated prizes. Users participate by submitting ERC1155 tokens as entry tickets. The contract employs a secure random number generation process to determine winners at the end of each raffle period. Winners are then able to claim their prizes, promoting excitement and interaction within the Aavegotchi community.

So, GHST is Aavegotchi’s curvy native utility token. Users who stake GHST are awarded FRENS. These FRENS can be used to purchase Raffle Tickets which provide holders the chance to win valuable Aavegotchi NFTs. Hence, any user who wants to participate in the raffle must possess FRENS. 

## Table of Contents

- [Raffle Contract](#raffle-contract)
  - [Overview](#overview)
    - [Introduction:](#introduction)
    - [Business Logic:](#business-logic)
  - [Table of Contents](#table-of-contents)
  - [Pragma Version and License](#pragma-version-and-license)
  - [Imported Interfaces](#imported-interfaces)
    - [IERC1155.sol:](#ierc1155sol)
    - [LinkTokenInterface.sol:](#linktokeninterfacesol)
    - [IERC173.sol:](#ierc173sol)
    - [IERC165.sol:](#ierc165sol)
  - [Structs and State Variables](#structs-and-state-variables)
    - [AppStorage Struct](#appstorage-struct)
    - [Raffle Struct](#raffle-struct)
    - [Entry Struct](#entry-struct)
    - [RaffleItemPrize Struct](#raffleitemprize-struct)
    - [RaffleItem Struct](#raffleitem-struct)
    - [RaffleItemInput](#raffleiteminput)
    - [RaffleItemPrizeIO](#raffleitemprizeio)
    - [RaffleIO](#raffleio)
    - [RaffleItemOutput](#raffleitemoutput)
    - [EntryIO](#entryio)
    - [TicketStatsIO](#ticketstatsio)
    - [TicketItemIO](#ticketitemio)
    - [TicketWinIO](#ticketwinio)
    - [PrizesWinIO](#prizeswinio)

## Pragma Version and License
The contract uses Solidity version 0.8.0, which is a relatively recent version. It includes the SPDX license identifier MIT, indicating that the code is licensed under the MIT License. This is a permissive license that allows for reuse within proprietary software provided all copies of the licensed software include a copy of the MIT License terms and the copyright notice.

## Imported Interfaces

### IERC1155.sol: 
```
import "./interfaces/IERC1155.sol";
```
ERC1155 is an Ethereum token standard which is designed to be a versatile way of creating both fungible (interchangeable) and non-fungible tokens. With this multi-token standard, a single smart contract can handle multiple token types. IERC1155 interface is a standard interface that defines a set of functions that a contract must implement to be ERC1155 compliant. This standard allows for the creation, transfer, and management of unique tokens, each of which can represent ownership of a specific item or asset. In the context of the RafflesContract, this interface is used to interact with ERC1155 tokens that are entered into the raffles. The contract uses functions like safeTransferFrom to transfer tokens from one address to another, ensuring that the transfer is safe and that the receiving address is capable of handling ERC1155 tokens.

### LinkTokenInterface.sol: 
```
import "./chainlink/LinkTokenInterface.sol";
```
The LinkTokenInterface interface is specific to the Chainlink network and represents the LINK token, which is used for paying for Chainlink services, including VRF (Verifiable Random Function) requests. The VRF is a provably fair source of randomness, which is crucial for the fairness and security of the raffle draws. This contract uses this interface to interact with the LINK token, specifically to transfer LINK tokens from the contract owner to the contract for paying VRF fees and to check the balance of LINK tokens in the contract.

### IERC173.sol: 
```
import "./interfaces/IERC173.sol";
```
The IERC173 interface is a standard interface for ownership management on the Ethereum blockchain. It defines a set of functions that a contract must implement to be ERC173 compliant. This standard allows for the transfer of ownership of a contract, which is important for the governance and security of the contract. Here, this interface is used to implement functions like owner and transferOwnership, which allow the current owner of the contract to transfer ownership to a new address.

### IERC165.sol: 
```
import "./interfaces/IERC165.sol";
```
The IERC165 interface is a standard interface for interface detection. It defines a function `supportsInterface` that a contract must implement to be ERC165 compliant. This standard allows a contract to specify which interfaces it supports, which is useful for interacting with contracts that may implement multiple standards. In this RafflesContract, this interface is used to implement the supportsInterface function, which allows other contracts or addresses to check if the RafflesContract supports specific interfaces, such as ERC1155 or ERC165 itself.

## Structs and State Variables

### AppStorage Struct

```
struct AppStorage {
    // IERC165
    mapping(bytes4 => bool) supportedInterfaces;
    Raffle[] raffles;
    // Nonces for VRF keyHash from which randomness has been requested.
    // Must stay in sync with VRFCoordinator[_keyHash][this]
    // keyHash => nonce
    mapping(bytes32 => uint256) nonces;
    mapping(bytes32 => uint256) requestIdToRaffleId;
    bytes32 keyHash;
    uint96 fee;
    address contractOwner;
}

```
The `AppStorage` struct is a centralized storage mechanism for the contract's state variables. It encapsulates all state variables, making it easier to manage and understand the contract's state. This struct includes:

- supportedInterfaces: A mapping that tracks which interfaces the contract supports, keyed by the interface ID. This is used to implement the ERC165 standard.
- raffles: An array of Raffle structs, representing all the raffles that have been created.
- nonces: A mapping that tracks the nonces for VRF keyHashes, ensuring that each request for randomness is unique.
- requestIdToRaffleId: A mapping that associates each VRF request ID with a raffle ID, allowing the contract to identify which raffle a VRF response corresponds to.
- keyHash: The keyHash used for VRF requests, which is specific to the VRF coordinator.
- fee: The fee in LINK tokens required for VRF requests.
- contractOwner: The address of the current owner of the contract.

### Raffle Struct
```
struct Raffle {
    // associates ticket address and ticketId to raffleItems
    // if raffleItemIndexes == 0, then raffle item does not exist
    // This means all raffleItemIndexes have been incremented by 1
    // ticketAddress => (ticketId => index + 1)
    mapping(address => mapping(uint256 => uint256)) raffleItemIndexes;
    RaffleItem[] raffleItems;
    // maps what tickets entrants have entered into the raffle
    // entrant => tickets
    mapping(address => Entry[]) entries;
    // the addresses of people who have entered tickets into the raffle
    address[] entrants;
    // vrf randomness
    uint256 randomNumber;
    // requested vrf random number
    bool randomNumberPending;
    // date in timestamp seconds when a raffle ends
    uint256 raffleEnd;
}
```
The `Raffle` struct represents a raffle, including:

- raffleItemIndexes: A mapping that associates each ticket address and ticket ID with a raffle item index, allowing the contract to quickly look up raffle items by ticket.
- raffleItems: An array of `RaffleItem` structs, representing all the items that can be entered into the raffle.
- entries: A mapping that tracks all entries for each entrant, allowing the contract to keep track of which tickets each entrant has entered.
- entrants: An array of addresses representing all unique entrants in the raffle.
- randomNumber: The random number generated for the raffle, used to determine the winners.
- randomNumberPending: A boolean flag indicating whether a random number request is pending.
- raffleEnd: The timestamp when the raffle ends, after which no more tickets can be entered, and the random number can be drawn.

### Entry Struct
```
// The minimum rangeStart is 0
// The maximum rangeEnd is raffleItem.totalEntered
// rangeEnd - rangeStart == number of ticket entered for raffle item by a entrant entry
struct Entry {
    uint24 raffleItemIndex; // Which raffle item is entered into the raffleEnd
    // used to prevent users from claiming prizes more than once
    bool prizesClaimed;
    uint112 rangeStart; // Raffle number. Value is between 0 and raffleItem.totalEntered - 1
    uint112 rangeEnd; // Raffle number. Value is between 1 and raffleItem.totalEntered
}
```
The `Entry` struct represents an entry in a raffle, including:

- raffleItemIndex: The index of the raffle item that was entered, allowing the contract to quickly look up the raffle item.
- prizesClaimed: A boolean flag indicating whether the prizes for this entry have been claimed.
- rangeStart and rangeEnd: The range of ticket numbers entered for this raffle item. This allows the contract to verify that a winning ticket number falls within the range of numbers entered by an entrant.

### RaffleItemPrize Struct
```
struct RaffleItemPrize {
    address prizeAddress; // ERC1155 token contract
    uint96 prizeQuantity; // Number of ERC1155 tokens
    uint256 prizeId; // ERC1155 token type
}
```
The `RaffleItemPrize` struct represents a prize that can be won in a raffle, including:

- prizeAddress: The address of the ERC1155 contract that issued the prize tokens.
- prizeQuantity: The number of prize tokens that can be won.
- prizeId: The ID of the prize token type.

### RaffleItem Struct
```
// Ticket numbers are numbers between 0 and raffleItem.totalEntered - 1 inclusive.
struct RaffleItem {
    address ticketAddress; // ERC1155 token contract
    uint256 ticketId; // ERC1155 token type
    uint256 totalEntered; // Total number of ERC1155 tokens entered into raffle for this raffle item
    RaffleItemPrize[] raffleItemPrizes; // Prizes that can be won for this raffle item
}
```
The `RaffleItem` struct represents an item that can be entered into a raffle, including:

- ticketAddress: The address of the ERC1155 contract that issued the ticket tokens.
- ticketId: The ID of the ticket token type.
- totalEntered: The total number of this type of ticket that has been entered into the raffle.
- raffleItemPrizes: An array of `RaffleItemPrize` structs, representing all the prizes that can be won for this raffle item.

### RaffleItemInput
```
struct RaffleItemInput {
        address ticketAddress;
        uint256 ticketId;
        RaffleItemPrizeIO[] raffleItemPrizes;
    }
```
The `RaffleItemInput` struct represents the input for a raffle item when starting a new raffle, including:

- ticketAddress: The address of the ERC1155 contract that issued the ticket tokens.
- ticketId: The ID of the ticket token type.
- raffleItemPrizes: An array of RaffleItemPrizeIO structs, representing all the prizes that can be won for this raffle item.

### RaffleItemPrizeIO
```
struct RaffleItemPrizeIO {
        address prizeAddress;
        uint256 prizeId;
        uint256 prizeQuantity;
    }
```
The `RaffleItemPrizeIO` struct represents the input for a raffle item prize, including:

- prizeAddress: The address of the ERC1155 contract that issued the prize tokens.
- prizeId: The ID of the prize token type.
- prizeQuantity: The number of prize tokens that can be won.

### RaffleIO
```
struct RaffleIO {
        uint256 raffleId;
        uint256 raffleEnd;
        bool isOpen;
    }
```
The `RaffleIO` struct represents the output for a raffle when querying raffle information, including:

- raffleId: The unique identifier of the raffle.
- raffleEnd: The timestamp when the raffle ends.
- isOpen: A boolean indicating whether the raffle is currently open.

### RaffleItemOutput
```
struct RaffleItemOutput {
        address ticketAddress;
        uint256 ticketId;
        uint256 totalEntered;
        RaffleItemPrizeIO[] raffleItemPrizes;
    }
```
The `RaffleItemOutput` struct represents the output for a raffle item when querying raffle item information, including:

- ticketAddress: The address of the ERC1155 contract that issued the ticket tokens.
- ticketId: The ID of the ticket token type.
- totalEntered: The total number of this type of ticket that has been entered into the raffle.
- raffleItemPrizes: An array of RaffleItemPrizeIO structs, representing all the prizes that can be won for this raffle item.

### EntryIO
```
struct EntryIO {
        address ticketAddress; // ERC1155 contract address
        uint256 ticketId; // ERC1155 type id
        uint256 ticketQuantity; // Number of ERC1155 tokens
        uint256 rangeStart;
        uint256 rangeEnd;
        uint256 raffleItemIndex;
        bool prizesClaimed;
    }
```
The `EntryIO` struct represents the output for an entry when querying entry information, including:

- ticketAddress: The address of the ERC1155 contract that issued the ticket tokens.
- ticketId: The ID of the ticket token type.
- ticketQuantity: The number of ticket tokens entered.
- rangeStart: The start of the range of ticket numbers entered.
- rangeEnd: The end of the range of ticket numbers entered.
- raffleItemIndex: The index of the raffle item that was entered.
- prizesClaimed: A boolean indicating whether the prizes for this entry have been claimed.

### TicketStatsIO
```
 struct TicketStatsIO {
        address ticketAddress; // ERC1155 contract address
        uint256 ticketId; // ERC1155 type id
        uint256 numberOfEntrants; // number of unique addresses that entered tickets
        uint256 totalEntered; // Number of ERC1155 tokens
    }
```
The `TicketStatsIO` struct represents the output for ticket statistics when querying ticket statistics, including:

- ticketAddress: The address of the ERC1155 contract that issued the ticket tokens.
- ticketId: The ID of the ticket token type.
numberOfEntrants: The number of unique entrants that entered tickets.
- totalEntered: The total number of this type of ticket that has been entered into the raffle.

### TicketItemIO
```
struct TicketItemIO {
        address ticketAddress; // ERC1155 contract address (entry ticket), not prize
        uint256 ticketId; // ERC1155 type id
        uint256 ticketQuantity; // Number of ERC1155 tokens
    }
```
The `TicketItemIO` struct represents the input for a ticket item when entering tickets into a raffle, including:

- ticketAddress: The address of the ERC1155 contract that issued the ticket tokens.
- ticketId: The ID of the ticket token type.
- ticketQuantity: The number of ticket tokens to be entered.

### TicketWinIO
```
/* This struct information can be gotten from the return results of the winners function */
    struct ticketWinIO {
        uint256 entryIndex; // index into a user's array of tickets (which staking attempt won)
        PrizesWinIO[] prizes;
    }
```
The `TicketWinIO` struct represents the output for a winning ticket when claiming prizes, including:

- entryIndex: The index of the entry that won the prize.
- prizes: An array of PrizesWinIO structs, representing the prizes won.

### PrizesWinIO
```
// Winning prize numbers are prize numbers used to calculate winning ticket numbers
    struct PrizesWinIO {
        uint256 raffleItemPrizeIndex; // index into the raffleItemPrizes array (which prize was won)
        uint256[] winningPrizeNumbers; // ticket numbers between 0 and raffleItem.totalEntered that won
    }
```
The `PrizesWinIO` struct represents the output for a winning prize when claiming prizes, including:

- raffleItemPrizeIndex: The index of the raffle item prize that was won.
- winningPrizeNumbers: An array of numbers representing the winning ticket numbers for the prize.
