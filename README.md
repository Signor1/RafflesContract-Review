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