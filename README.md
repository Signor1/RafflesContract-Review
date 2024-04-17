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
  - [Contract: RafflesContract](#contract-rafflescontract)
    - [Immutable Variables](#immutable-variables)
    - [Constants](#constants)
    - [State Variables](#state-variables)
    - [Constructor](#constructor)
    - [Events](#events)
    - [Function: supportsInterface](#function-supportsinterface)
    - [VRF Functionality](#vrf-functionality)
      - [Function: nonces](#function-nonces)
      - [Function: requestRandomness](#function-requestrandomness)
      - [Function: makeVRFInputSeed](#function-makevrfinputseed)
      - [Function: makeRequestId](#function-makerequestid)
      - [Function: drawRandomNumber](#function-drawrandomnumber)
      - [Function: rawFulfillRandomness](#function-rawfulfillrandomness)
      - [Function: changeVRFFee](#function-changevrffee)
      - [Function: removeLinkTokens](#function-removelinktokens)
      - [Function: linkBalance](#function-linkbalance)
    - [Owner Management](#owner-management)
      - [Function: owner](#function-owner)
      - [Function: transferOwnership](#function-transferownership)
    - [Raffle Management](#raffle-management)
      - [Function: startRaffle](#function-startraffle)
      - [Function: onERC1155Received](#function-onerc1155received)
      - [Function: getRaffles](#function-getraffles)
      - [Function: raffleSupply](#function-rafflesupply)
      - [Function: raffleInfo](#function-raffleinfo)
      - [Function: getEntries](#function-getentries)
      - [Function: ticketStats](#function-ticketstats)
      - [Function: enterTickets](#function-entertickets)
      - [Function: getEntrants](#function-getentrants)
      - [Function: claimPrize](#function-claimprize)
  - [Summary And Proposal](#summary-and-proposal)

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
