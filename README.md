
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


## Contract: RafflesContract
```
contract RafflesContract is IERC173, IERC165 {

}
```
This `RafflesContract` incorporates various functionalities, including VRF (Verifiable Random Function) for generating random numbers, ERC1155 token interactions, and raffle management. It is inheriting ERC173, ERC165 and structured around a central AppStorage struct that encapsulates all state variables, and it uses several imported interfaces to interact with other contracts and standards.

### Immutable Variables
Immutable variables in Solidity are variables that cannot be changed once they have been assigned a value and can only be set during contract deployment.
```
contract RafflesContract is IERC173, IERC165 {

  // Immutable values are prefixed with im_ to easily identify them in code
    LinkTokenInterface internal immutable im_link;
    address internal immutable im_vrfCoordinator;

}
```
- im_link: 
An instance of the LinkTokenInterface, which is set to the address of the LINK token contract upon deployment. This instance is used to interact with LINK tokens, specifically for transferring LINK tokens from the contract owner to the contract for paying VRF fees and to check the balance of LINK tokens in the contract.

- im_vrfCoordinator: 
The address of the VRF Coordinator, which is set upon deployment. This address is used to interact with the Chainlink VRF Coordinator, which is responsible for processing VRF requests and providing the random numbers.

### Constants
Constant variables in Solidity are variables that are defined with the constant keyword, indicating that their value cannot be changed once it has been set. A constant variable must have a value that is known at compile time.

```
contract RafflesContract is IERC173, IERC165 {

 bytes4 internal constant ERC1155_ACCEPTED = 0xf23a6e61; // Return value from `onERC1155Received` call if a contract accepts receipt (i.e `bytes4(keccak256("onERC1155Received(address,address,uint256,uint256,bytes)"))`).

}
```
- ERC1155_ACCEPTED: 
A constant bytes4 value representing the return value expected from the onERC1155Received function call if a contract accepts the receipt of an ERC1155 token. This constant is used in the onERC1155Received function to ensure that the contract correctly handles ERC1155 token transfers.

### State Variables
```
contract RafflesContract is IERC173, IERC165 {

  // State variables are prefixed with s.
  AppStorage internal s;

}
```
- s: An instance of the AppStorage struct, which is used to manage the contract's state. This includes mappings for supported interfaces, an array of raffles, nonces for VRF requests, a mapping from request IDs to raffle IDs, the keyHash for VRF requests, the fee for VRF requests, and the contract owner.

### Constructor
A `constructor` is an optional function that is executed upon contract creation.

```
contract RafflesContract is IERC173, IERC165 {

  constructor(address _contractOwner, address _vrfCoordinator, address _link, bytes32 _keyHash, uint256 _fee) {
        s.contractOwner = _contractOwner;
        im_vrfCoordinator = _vrfCoordinator;
        im_link = LinkTokenInterface(_link);
        s.keyHash = _keyHash; //0x6c3699283bda56ad74f6b855546325b68d482e983852a7a82979cc4807b641f4;
        s.fee = uint96(_fee);

        // adding ERC165 data
        s.supportedInterfaces[type(IERC165).interfaceId] = true;
        s.supportedInterfaces[type(IERC173).interfaceId] = true;
        // skip raffle 0
        s.raffles.push();
    }

}
```
Here, the constructor initializes the `contract owner`, the `VRF coordinator address`, the `LINK token interface`, the `keyHash for VRF requests`, and the `fee for VRF requests`. It also sets up the `ERC165 interface` support by marking the interfaces it supports as true in the `supportedInterfaces` mapping.

### Events
In this `RafflesContract`, there are several events that emit information about the contract's operations to external consumers, such as front-end applications, which can listen for these events to update their state accordingly. Here's a breakdown of the events:

```
contract RafflesContract is IERC173, IERC165 {

  event RaffleStarted(uint256 indexed raffleId, uint256 raffleEnd, RaffleItemInput[] raffleItems);
    event RaffleTicketsEntered(uint256 indexed raffleId, address entrant, TicketItemIO[] ticketItems);
    event RaffleRandomNumber(uint256 indexed raffleId, uint256 randomNumber);
    event RaffleClaimPrize(uint256 indexed raffleId, address entrant, address prizeAddress, uint256 prizeId, uint256 prizeQuantity);

}
```

- RaffleStarted:
  - Emitted when a new raffle is started.
  - Parameters:
    - `raffleId`: The unique identifier of the raffle.
    - `raffleEnd`: The timestamp when the raffle ends.
    - `raffleItems`: An array of `RaffleItemInput` structs, representing the items that can be entered into the raffle.

- RaffleTicketsEntered:
  - Emitted when tickets are entered into a raffle.
  - Parameters:
    - `raffleId`: The unique identifier of the raffle.
    - `entrant`: The address of the entrant who entered the tickets.
    - `ticketItems`: An array of `TicketItemIO` structs, representing the tickets that were entered.

- RaffleRandomNumber:
  - Emitted when a random number is generated for a raffle.
  - Parameters:
    - `raffleId`: The unique identifier of the raffle.
    - `randomNumber`: The generated random number.

- RaffleClaimPrize:
  - Emitted when an entrant claims a prize.
  - Parameters:
    - `raffleId`: The unique identifier of the raffle.
    - `entrant`: The address of the entrant who claimed the prize.
    - `prizeAddress`: The address of the ERC1155 contract that issued the prize tokens.
    - `prizeId`: The ID of the prize token type.
    - `prizeQuantity`: The number of prize tokens that were claimed.

These events provide a way for external entities to react to changes in the contract's state. For example, a front-end application could listen for the RaffleStarted event to update its UI with information about the new raffle, or it could listen for the RaffleClaimPrize event to update the display of claimed prizes.    

### Function: supportsInterface
```
contract RafflesContract is IERC173, IERC165 {

  function supportsInterface(bytes4 _interfaceId) external view override returns (bool) {
        return s.supportedInterfaces[_interfaceId];
    }

}
```
- Purpose: Implements the ERC165 standard for interface detection.
- Description: Allows external contracts or addresses to query if the RafflesContract supports a specific interface.
- Parameters:
  - _interfaceId: The identifier of the interface being queried.
  - Returns: A boolean indicating whether the contract supports the specified interface.
- Usage: External entities can call this function to check if the contract supports a specific interface, ensuring compatibility and interoperability.

### VRF Functionality
A verifiable random function (VRF) is a cryptographic function that takes a series of inputs, computes them, and produces a pseudorandom output and proof of authenticity that can be verified by anyone.

#### Function: nonces
```
contract RafflesContract is IERC173, IERC165 {

  function nonces(bytes32 _keyHash) external view returns (uint256 nonce_) {
        nonce_ = s.nonces[_keyHash];
    }

}
```
- Purpose: Returns the nonce for a given keyHash, which is used to ensure each VRF request is unique.
- Description: Provides the current nonce for a specific keyHash, which is crucial for the security and uniqueness of VRF requests.
- Parameters:
  - `_keyHash`: The keyHash for which to return the nonce.
- Returns: The current nonce for the given keyHash.
- Usage: Used internally by the contract to manage nonces for VRF requests.

#### Function: requestRandomness
```
contract RafflesContract is IERC173, IERC165 {

  function requestRandomness(bytes32 _keyHash, uint256 _fee, uint256 _seed) internal returns (bytes32 requestId) {
        im_link.transferAndCall(im_vrfCoordinator, _fee, abi.encode(_keyHash, _seed));
        uint256 vRFSeed = makeVRFInputSeed(_keyHash, _seed, address(this), s.nonces[_keyHash]);
        s.nonces[_keyHash]++;
        return makeRequestId(_keyHash, vRFSeed);
    }

}
```
- Purpose: Initiates a request for VRF output.
- Description: Sends a request to the Chainlink VRF Coordinator for a random number, using a specified keyHash and fee.
- Parameters:
  - `_keyHash`: The keyHash to use for the VRF request.
  - `_fee`: The fee to pay for the VRF request.
  - `_seed`: A seed value mixed into the input of the VRF.
- Returns: The requestId for the VRF request, which can be used to distinguish responses to concurrent requests.
- Usage: Called when a raffle needs a random number generated, for example, to determine the winners of a raffle.

#### Function: makeVRFInputSeed
```
contract RafflesContract is IERC173, IERC165 {

  function makeVRFInputSeed(bytes32 _keyHash, uint256 _userSeed, address _requester, uint256 _nonce) internal pure returns (uint256) {
        return uint256(keccak256(abi.encode(_keyHash, _userSeed, _requester, _nonce)));
    }

}
```
- Purpose: Generates the seed which is actually input to the VRF coordinator.
- Description: Combines the user-supplied seed with the user-specific nonce and the address of the requesting contract to create a unique seed for the VRF request.
- Parameters:
  - `_keyHash`: The keyHash to use for the VRF request.
  - `_userSeed`: The seed value provided by the user.
  - `_requester`: The address of the requesting contract.
  - `_nonce`: The user-specific nonce at the time of the request.
- Returns: The generated seed for the VRF request.
- Usage: Used internally by the contract to ensure the uniqueness and security of VRF requests.


#### Function: makeRequestId
```
contract RafflesContract is IERC173, IERC165 {

  function makeRequestId(bytes32 _keyHash, uint256 _vRFInputSeed) internal pure returns (bytes32) {
        return keccak256(abi.encodePacked(_keyHash, _vRFInputSeed));
    }

}
```
- Purpose: Generates a unique requestId for a VRF request.
- Description: Creates a unique identifier for a VRF request by hashing the keyHash and the VRF input seed.
- Parameters:
  - `_keyHash`: The keyHash to use for the VRF request.
  - `_vRFInputSeed`: The seed to be passed directly to the VRF.
- Returns: The generated requestId for the VRF request.
- Usage: Used internally by the contract to manage and track VRF requests.

#### Function: drawRandomNumber
```
contract RafflesContract is IERC173, IERC165 {

  function drawRandomNumber(uint256 _raffleId) external {
        require(_raffleId < s.raffles.length, "Raffle: Raffle does not exist");
        Raffle storage raffle = s.raffles[_raffleId];
        require(raffle.raffleEnd < block.timestamp, "Raffle: Raffle time has not expired");
        require(raffle.randomNumber == 0, "Raffle: Random number already generated");
        require(raffle.randomNumberPending == false || msg.sender == s.contractOwner, "Raffle: Random number is pending");
        raffle.randomNumberPending = true;
        // Use Chainlink VRF to generate random number
        require(im_link.balanceOf(address(this)) >= s.fee, "Not enough LINK");
        bytes32 requestId = requestRandomness(s.keyHash, s.fee, 0);
        s.requestIdToRaffleId[requestId] = _raffleId;
    }

}
```
- Purpose: Initiates the drawing of a random number for a raffle.
- Description: Checks that the raffle is in a valid state for drawing a random number (e.g., the raffle has ended and a random number has not yet been generated), then requests a random number from the VRF.
- Parameters:
  - `_raffleId`: The `ID` of the raffle for which to draw a random number.
- Usage: Called to initiate the drawing of a random number for a raffle, which is used to determine the winners.

#### Function: rawFulfillRandomness
```
contract RafflesContract is IERC173, IERC165 {

  function rawFulfillRandomness(bytes32 _requestId, uint256 _randomness) external {
        require(msg.sender == im_vrfCoordinator, "Only VRFCoordinator can fulfill");
        uint256 raffleId = s.requestIdToRaffleId[_requestId];
        require(raffleId < s.raffles.length, "Raffle: Raffle does not exist");
        Raffle storage raffle = s.raffles[raffleId];
        require(raffle.raffleEnd < block.timestamp, "Raffle: Raffle time has not expired");
        require(raffle.randomNumber == 0, "Raffle: Random number already generated");
        s.raffles[raffleId].randomNumber = _randomness;
        raffle.randomNumberPending = false;
        emit RaffleRandomNumber(raffleId, _randomness);
    }

}
```
- Purpose: Callback function for handling VRF responses.
- Description: Called by the VRF Coordinator with the random number generated for a request. Updates the state of the raffle with the generated random number.
- Parameters:
  - `_requestId`: The requestId for the VRF request.
  - `_randomness`: The random number generated by the VRF.
- Usage: Acts as the callback for VRF responses, updating the raffle's state with the generated random number.

#### Function: changeVRFFee
```
contract RafflesContract is IERC173, IERC165 {

  // Change the fee amount that is paid for VRF random numbers
    function changeVRFFee(uint256 _newFee, bytes32 _keyHash) external {
        require(msg.sender == s.contractOwner, "Raffle: Must be contract owner");
        s.fee = uint96(_newFee);
        s.keyHash = _keyHash;
    }

}
```
- Purpose: Allows changing the fee amount paid for VRF random numbers.
- Description: Updates the fee and keyHash used for VRF requests.
- Parameters:
  - `_newFee`: The new fee amount.
  - `_keyHash`: The new keyHash to use for VRF requests.
- Usage: Called by the contract owner to update the fee and keyHash for future VRF requests.

#### Function: removeLinkTokens
```
contract RafflesContract is IERC173, IERC165 {

  // Remove the LINK tokens from this contract that are used to pay for VRF random number fees
    function removeLinkTokens(address _to, uint256 _value) external {
        require(msg.sender == s.contractOwner, "Raffle: Must be contract owner");
        im_link.transfer(_to, _value);
    }

}
```
- Purpose: Allows removing LINK tokens from the contract.
- Description: Transfers LINK tokens from the contract to a specified address.
- Parameters:
  - `_to`: The address to transfer the LINK tokens to.
  - `_value`: The amount of LINK tokens to transfer.
- Usage: Called by the contract owner to manage the contract's LINK token balance, for example, to withdraw unused LINK tokens.

#### Function: linkBalance
```
contract RafflesContract is IERC173, IERC165 {

  function linkBalance() external view returns (uint256 linkBalance_) {
        linkBalance_ = im_link.balanceOf(address(this));
    }

}
```
- Purpose: Returns the balance of LINK tokens in the contract.
- Description: Provides the current balance of LINK tokens held by the contract.
- Returns: The balance of LINK tokens.
- Usage: Can be called by anyone to check the contract's LINK token balance.

### Owner Management
#### Function: owner
```
contract RafflesContract is IERC173, IERC165 {

  function owner() external view override returns (address) {
        return s.contractOwner;
    }

}
```
- Purpose: Returns the address of the current owner of the contract.
- Description: Provides the address of the entity that currently owns the contract.
- Returns: The address of the contract owner.
- Usage: Can be called by anyone to identify the current owner of the contract.

#### Function: transferOwnership
```
contract RafflesContract is IERC173, IERC165 {
  
  function transferOwnership(address _newContractOwner) external override {
        address previousOwner = s.contractOwner;
        require(msg.sender == previousOwner, "Raffle: Must be contract owner");
        s.contractOwner = _newContractOwner;
        emit OwnershipTransferred(previousOwner, _newContractOwner);
    }

}
```
- Purpose: Transfers ownership of the contract to a new address.
- Description: Allows the current owner of the contract to transfer ownership to a new address.
- Parameters:
  -`_newContractOwner`: The address of the new owner.
- Usage: Called by the current owner to transfer ownership of the contract to a new address.


### Raffle Management
#### Function: startRaffle
```
contract RafflesContract is IERC173, IERC165 {
  
   function startRaffle(uint256 _raffleDuration, RaffleItemInput[] calldata _raffleItems) external {
        require(msg.sender == s.contractOwner, "Raffle: Must be contract owner");
        require(_raffleDuration >= 3600, "Raffle: _raffleDuration must be greater than 1 hour");
        uint256 raffleEnd = block.timestamp + _raffleDuration;
        require(_raffleItems.length > 0, "Raffle: No raffle items");
        uint256 raffleId = s.raffles.length;
        emit RaffleStarted(raffleId, raffleEnd, _raffleItems);
        Raffle storage raffle = s.raffles.push();
        raffle.raffleEnd = raffleEnd;
        for (uint256 i; i < _raffleItems.length; i++) {
            RaffleItemInput calldata raffleItemInput = _raffleItems[i];
            require(raffleItemInput.raffleItemPrizes.length > 0, "Raffle: No prizes");
            // ticketAddress is ERC1155 contract address of tickets
            // ticketId is the ERC1155 type id, which type is it
            require(
                // The index is one greater than actual index.  If index is 0 it means the value does not exist yet.
                raffle.raffleItemIndexes[raffleItemInput.ticketAddress][raffleItemInput.ticketId] == 0,
                "Raffle: Raffle item already using ticketAddress and ticketId"
            );
            // A raffle item is a ticketAddress, ticketId and what prizes can be won.
            RaffleItem storage raffleItem = raffle.raffleItems.push();
            // The index is one greater than actual index.  If index is 0 it means the value does not exist yet.
            raffle.raffleItemIndexes[raffleItemInput.ticketAddress][raffleItemInput.ticketId] = raffle.raffleItems.length;
            raffleItem.ticketAddress = raffleItemInput.ticketAddress;
            raffleItem.ticketId = raffleItemInput.ticketId;
            for (uint256 j; j < raffleItemInput.raffleItemPrizes.length; j++) {
                RaffleItemPrizeIO calldata raffleItemPrizeIO = raffleItemInput.raffleItemPrizes[j];
                raffleItem.raffleItemPrizes.push(
                    RaffleItemPrize(raffleItemPrizeIO.prizeAddress, uint96(raffleItemPrizeIO.prizeQuantity), raffleItemPrizeIO.prizeId)
                );
                IERC1155(raffleItemPrizeIO.prizeAddress).safeTransferFrom(
                    msg.sender,
                    address(this),
                    raffleItemPrizeIO.prizeId,
                    raffleItemPrizeIO.prizeQuantity,
                    abi.encode(raffleId)
                );
            }
        }
    }

}
```

`startRaffle` function is a critical part of this RafflesContract, enabling the creation of new raffles. Here's a detailed explanation of its purpose, parameters, and functionality:

- Purpose:
The primary aim of the startRaffle function is to initialize a new raffle with the specified duration and items that can be entered into the raffle. This function is called by the contract owner to set up a new raffle, defining the conditions under which tickets can be entered and the prizes that can be won.

- Parameters:
  - `_raffleDuration`: A uint256 value representing the duration of the raffle in seconds. This parameter determines how long the raffle will be open for entries.
  - `_raffleItems`: An array of `RaffleItemInput` structs. Each RaffleItemInput struct represents an item that can be entered into the raffle, including the ticket address, ticket ID, and an array of RaffleItemPrizeIO structs representing the prizes that can be won for this item.

- Functionality:
  - Access Control: The function first checks that the caller is the contract owner, ensuring that only the authorized entity can start a new raffle.
  - Duration Validation: It validates that the `_raffleDuration` is at least 3600 seconds (1 hour), ensuring that raffles have a minimum duration to prevent spam or abuse.
  - Raffle End Calculation: The function calculates the end timestamp of the raffle by adding `_raffleDuration` to the current block timestamp. This end timestamp is when the raffle will close, and no more entries will be accepted.
  - Raffle Initialization: A new Raffle struct is created and added to the raffles array in the contract's state. The raffleEnd field of the Raffle struct is set to the calculated end timestamp.
  - Raffle Items Processing: The function iterates over the `_raffleItems` array. For each `RaffleItemInput`, it checks that the item does not already exist in the raffle (to prevent duplicate items) and then creates a new `RaffleItem` struct for each item, adding it to the raffleItems array of the Raffle struct. It also transfers the prizes from the contract owner to the contract, ensuring that the contract holds the prizes until they are claimed.
  - Event Emission: Finally, the function emits a `RaffleStarted` event, providing details about the new raffle, including its ID, end timestamp, and the items that can be entered.
- Usage:
This function is used by the contract owner to start a new raffle, specifying the duration of the raffle and the items (and their associated prizes) that can be entered. It's a key part of the raffle management process, allowing for the creation and configuration of new raffles.

#### Function: onERC1155Received
```
contract RafflesContract is IERC173, IERC165 {
  
  function onERC1155Received(address _operator, address _from, uint256 _id, uint256 _value, bytes calldata _data) external view returns (bytes4) {
        _operator; // silence not used warning
        _from; // silence not used warning
        _id; // silence not used warning
        _value; // silence not used warning
        require(_data.length == 32, "Raffle: Data of the wrong size sent on transfer");
        uint256 raffleId = abi.decode(_data, (uint256));
        require(raffleId < s.raffles.length, "Raffle: Raffle does not exist");
        Raffle storage raffle = s.raffles[raffleId];
        uint256 raffleEnd = raffle.raffleEnd;
        require(raffleEnd > block.timestamp, "Raffle: Can't accept transfer for expired raffle");
        return ERC1155_ACCEPTED;
    }

}
```
- Purpose: Handles the receipt of ERC1155 tokens.
- Description: Called by an ERC1155 token contract when tokens are transferred to this contract, ensuring that the contract accepts the transfer.
- Parameters:
  -`_operator`: The address which initiated the transfer.
  - `_from`: The address which previously owned the token.
  - `_id`: The ID of the token being transferred.
  - `_value`: The amount of tokens being transferred.
  - `_data`: Additional data with no specified format.
- Returns: A bytes4 value indicating that the contract accepts the transfer.
- Usage: Acts as the callback for ERC1155 token transfers, ensuring that the contract correctly handles incoming tokens.

#### Function: getRaffles
```
contract RafflesContract is IERC173, IERC165 {
  
  function getRaffles() external view returns (RaffleIO[] memory raffles_) {
        raffles_ = new RaffleIO[](s.raffles.length);
        for (uint256 i; i < s.raffles.length; i++) {
            uint256 raffleEnd = s.raffles[i].raffleEnd;
            raffles_[i].raffleId = i;
            raffles_[i].raffleEnd = raffleEnd;
            raffles_[i].isOpen = raffleEnd > block.timestamp;
        }
    }

}
```
- Purpose: Retrieves information about all raffles.
- Description: Returns an array of RaffleIO structs, each representing a raffle with its ID, end timestamp, and whether it is currently open.
- Returns: An array of RaffleIO structs.
- Usage: Can be called by anyone to get an overview of all raffles.

#### Function: raffleSupply
```
contract RafflesContract is IERC173, IERC165 {
  
  function raffleSupply() external view returns (uint256 raffleSupply_) {
        raffleSupply_ = s.raffles.length;
    }

}
```
- Purpose: Returns the total number of raffles that exist.
- Description: Provides the total count of raffles created by the contract.
- Returns: The total number of raffles.
- Usage: Can be called by anyone to check the total number of raffles.

#### Function: raffleInfo
```
contract RafflesContract is IERC173, IERC165 {
  
  function raffleInfo(uint256 _raffleId) external view returns (uint256 raffleEnd_, RaffleItemOutput[] memory raffleItems_, uint256 randomNumber_) {
        require(_raffleId < s.raffles.length, "Raffle: Raffle does not exist");
        Raffle storage raffle = s.raffles[_raffleId];
        raffleEnd_ = raffle.raffleEnd;
        if (raffle.randomNumberPending == true) {
            randomNumber_ = 1;
        } else {
            randomNumber_ = raffle.randomNumber;
        }
        // Loop over and get all the raffle itmes, which includes ERC1155 tickets and ERC1155 prizes
        raffleItems_ = new RaffleItemOutput[](raffle.raffleItems.length);
        for (uint256 i; i < raffle.raffleItems.length; i++) {
            RaffleItem storage raffleItem = raffle.raffleItems[i];
            raffleItems_[i].ticketAddress = raffleItem.ticketAddress;
            raffleItems_[i].ticketId = raffleItem.ticketId;
            raffleItems_[i].totalEntered = raffleItem.totalEntered;
            raffleItems_[i].raffleItemPrizes = new RaffleItemPrizeIO[](raffleItem.raffleItemPrizes.length);
            for (uint256 j; j < raffleItem.raffleItemPrizes.length; j++) {
                RaffleItemPrize storage raffleItemPrize = raffleItem.raffleItemPrizes[j];
                raffleItems_[i].raffleItemPrizes[j].prizeAddress = raffleItemPrize.prizeAddress;
                raffleItems_[i].raffleItemPrizes[j].prizeId = raffleItemPrize.prizeId;
                raffleItems_[i].raffleItemPrizes[j].prizeQuantity = raffleItemPrize.prizeQuantity;
            }
        }
    }

}
```
- Purpose: Retrieves detailed information about a specific raffle.
- Description: Returns detailed information about a raffle, including its end timestamp, the items that can be entered, and the random number generated for it.
- Parameters:
  - `_raffleId`: The ID of the raffle to retrieve information about.
- Returns: The end timestamp of the raffle, an array of RaffleItemOutput structs representing the items that can be entered, and the random number generated for the raffle.
- Usage: Can be called by anyone to get detailed information about a specific raffle.

#### Function: getEntries
```
contract RafflesContract is IERC173, IERC165 {
  
  function getEntries(uint256 _raffleId, address _entrant) external view returns (EntryIO[] memory entries_) {
        require(_raffleId < s.raffles.length, "Raffle: Raffle does not exist");
        Raffle storage raffle = s.raffles[_raffleId];
        entries_ = new EntryIO[](raffle.entries[_entrant].length);
        for (uint256 i; i < raffle.entries[_entrant].length; i++) {
            Entry memory entry = raffle.entries[_entrant][i];
            RaffleItem storage raffleItem = raffle.raffleItems[entry.raffleItemIndex];
            entries_[i].ticketAddress = raffleItem.ticketAddress;
            entries_[i].ticketId = raffleItem.ticketId;
            entries_[i].ticketQuantity = entry.rangeEnd - entry.rangeStart;
            entries_[i].rangeStart = entry.rangeStart;
            entries_[i].rangeEnd = entry.rangeEnd;
            entries_[i].raffleItemIndex = entry.raffleItemIndex;
            entries_[i].prizesClaimed = entry.prizesClaimed;
        }
    }

} 
```
- Purpose: Retrieves the entries for a specific entrant in a raffle.
- Description: Returns an array of EntryIO structs, each representing an entry made by an entrant in a raffle.
- Parameters:
  - `_raffleId`: The ID of the raffle.
  - `_entrant`: The address of the entrant.
- Returns: An array of `EntryIO` structs.
- Usage: Can be called by anyone to get the entries made by an entrant in a specific raffle.

#### Function: ticketStats
```
contract RafflesContract is IERC173, IERC165 {
  
  function ticketStats(uint256 _raffleId) external view returns (TicketStatsIO[] memory ticketStats_) {
        require(_raffleId < s.raffles.length, "Raffle: Raffle does not exist");
        Raffle storage raffle = s.raffles[_raffleId];
        ticketStats_ = new TicketStatsIO[](raffle.raffleItems.length);
        // loop through raffle items
        for (uint256 i; i < raffle.raffleItems.length; i++) {
            RaffleItem storage raffleItem = raffle.raffleItems[i];
            ticketStats_[i].ticketAddress = raffleItem.ticketAddress;
            ticketStats_[i].ticketId = raffleItem.ticketId;
            ticketStats_[i].totalEntered = raffleItem.totalEntered;
            // count the number of users that have ticketd for the raffle item
            for (uint256 j; j < raffle.entrants.length; j++) {
                address entrant = raffle.entrants[j];
                for (uint256 k; k < raffle.entries[entrant].length; k++) {
                    if (i == raffle.entries[entrant][k].raffleItemIndex) {
                        ticketStats_[i].numberOfEntrants++;
                        break;
                    }
                }
            }
        }
    }

}
```
- Purpose: Retrieves statistics about the tickets entered into a raffle.
- Description: Returns an array of TicketStatsIO structs, each representing statistics about a type of ticket entered into the raffle.
- Parameters:
  - `_raffleId`: The ID of the raffle.
- Returns: An array of TicketStatsIO structs.
- Usage: Can be called by anyone to get statistics about the tickets entered into a specific raffle.


#### Function: enterTickets
```
contract RafflesContract is IERC173, IERC165 {
  
  function enterTickets(uint256 _raffleId, TicketItemIO[] calldata _ticketItems) external {
        require(_raffleId < s.raffles.length, "Raffle: Raffle does not exist");
        require(_ticketItems.length > 0, "Raffle: No tickets");
        Raffle storage raffle = s.raffles[_raffleId];
        require(raffle.raffleEnd > block.timestamp, "Raffle: Raffle time has expired");
        emit RaffleTicketsEntered(_raffleId, msg.sender, _ticketItems);
        // Collect unique entrant addresses
        if (raffle.entries[msg.sender].length == 0) {
            raffle.entrants.push(msg.sender);
        }
        for (uint256 i; i < _ticketItems.length; i++) {
            TicketItemIO calldata ticketItem = _ticketItems[i];
            require(ticketItem.ticketQuantity > 0, "Raffle: Ticket quantity cannot be zero");
            // get the raffle item
            uint256 raffleItemIndex = raffle.raffleItemIndexes[ticketItem.ticketAddress][ticketItem.ticketId];
            require(raffleItemIndex > 0, "Raffle: Raffle item doesn't exist for this raffle");
            raffleItemIndex--;
            RaffleItem storage raffleItem = raffle.raffleItems[raffleItemIndex];
            uint256 totalEntered = raffleItem.totalEntered;
            // Create a range of unique numbers for ticket ids
            raffle.entries[msg.sender].push(
                Entry(uint24(raffleItemIndex), false, uint112(totalEntered), uint112(totalEntered + ticketItem.ticketQuantity))
            );
            // update the total quantity of tickets that have been entered for this raffle item
            raffleItem.totalEntered = totalEntered + ticketItem.ticketQuantity;
            // transfer the ERC1155 tokens to ticket to this contract
            IERC1155(ticketItem.ticketAddress).safeTransferFrom(
                msg.sender,
                address(this),
                ticketItem.ticketId,
                ticketItem.ticketQuantity,
                abi.encode(_raffleId)
            );
        }
    }

}
```
- Purpose: Allows an entrant to enter tickets into a raffle.
- Description: Transfers the specified ERC1155 tokens from the entrant to the contract and records the entries.
- Parameters:
  - `_raffleId`: The ID of the raffle.
  - `_ticketItems`: An array of TicketItemIO structs, each representing a type of ticket to be entered.
- Usage: Called by an entrant to enter tickets into a raffle.


#### Function: getEntrants
```
contract RafflesContract is IERC173, IERC165 {
  
  // Get the unique addresses of entrants in a raffle
    function getEntrants(uint256 _raffleId) external view returns (address[] memory entrants_) {
        require(_raffleId < s.raffles.length, "Raffle: Raffle does not exist");
        Raffle storage raffle = s.raffles[_raffleId];
        entrants_ = raffle.entrants;
    }

}
```
- Purpose: Retrieves the addresses of all entrants in a raffle.
- Description: Returns an array of addresses representing all unique entrants in a raffle.
- Parameters:
  - _raffleId: The ID of the raffle.
- Returns: An array of addresses.
- Usage: Can be called by anyone to get the list of all entrants in a specific raffle.

#### Function: claimPrize
```
function claimPrize(uint256 _raffleId, address _entrant, ticketWinIO[] calldata _wins) external {
        require(_raffleId < s.raffles.length, "Raffle: Raffle does not exist");
        Raffle storage raffle = s.raffles[_raffleId];
        uint256 randomNumber = raffle.randomNumber;
        require(randomNumber > 0, "Raffle: Random number not generated yet");
        // contractOwner can claim prizes for the entrant.  Prizes are only transferred to the entrant
        require(msg.sender == _entrant || msg.sender == s.contractOwner, "Raffle: Not claimed by owner or contractOwner");
        // Logic:
        // 1. Loop through wins
        // 2. Verify provided entryIndex exists and is not a duplicate
        // 3. Loop through prizes
        // 4. Verify provided prize exists and is not a duplicate
        // 5. Loop through winning prize numbers
        // 6. Verify winning prize number exists and is not a duplicate
        // 7. Verify that winning prize number actually won
        // 8. Transfer prizes to winner
        //--------------------------------------------
        // lastValue serves two purposes:
        // 1. Ensures that a value is less than the length of an array
        // 2. Prevents duplicates. Subsequent values must be lesser
        // lastValue gets reused by inner loops
        uint256 lastValue = raffle.entries[_entrant].length;
        for (uint256 i; i < _wins.length; i++) {
            ticketWinIO calldata win = _wins[i];
            // Serves two purposes: 1. Ensure is less than raffle.entries[_entrant].length. 2. prevents duplicates
            require(win.entryIndex < lastValue, "Raffle: User entry does not exist or is not lesser than last value");
            Entry memory entry = raffle.entries[_entrant][win.entryIndex];
            require(entry.prizesClaimed == false, "Raffles: Entry prizes have already been claimed");
            raffle.entries[_entrant][win.entryIndex].prizesClaimed = true;
            // total number of tickets that have been entered for a raffle item
            uint256 totalEntered = raffle.raffleItems[entry.raffleItemIndex].totalEntered;
            lastValue = raffle.raffleItems[entry.raffleItemIndex].raffleItemPrizes.length;
            for (uint256 j; j < win.prizes.length; j++) {
                PrizesWinIO calldata prize = win.prizes[j];
                // Serves two purposes: 1. Ensure is less than raffleItemPrizes.length. 2. prevents duplicates
                require(prize.raffleItemPrizeIndex < lastValue, "Raffle: Raffle prize type does not exist or is not lesser than last value");
                RaffleItemPrize memory raffleItemPrize = raffle.raffleItems[entry.raffleItemIndex].raffleItemPrizes[prize.raffleItemPrizeIndex];
                lastValue = raffleItemPrize.prizeQuantity;
                for (uint256 k; k < prize.winningPrizeNumbers.length; k++) {
                    uint256 prizeNumber = prize.winningPrizeNumbers[k];
                    // Serves two purposes: 1. Ensure is less than raffleItemPrize.prizeQuantity. 2. prevents duplicates
                    require(prizeNumber < lastValue, "Raffle: prizeNumber does not exist or is not lesser than last value");
                    uint256 winningTicketNumber = uint256(
                        keccak256(abi.encodePacked(randomNumber, entry.raffleItemIndex, prize.raffleItemPrizeIndex, prizeNumber))
                    ) % totalEntered;
                    require(winningTicketNumber >= entry.rangeStart && winningTicketNumber < entry.rangeEnd, "Raffle: Did not win prize");
                    lastValue = prizeNumber;
                }
                emit RaffleClaimPrize(_raffleId, _entrant, raffleItemPrize.prizeAddress, raffleItemPrize.prizeId, prize.winningPrizeNumbers.length);
                IERC1155(raffleItemPrize.prizeAddress).safeTransferFrom(
                    address(this),
                    _entrant,
                    raffleItemPrize.prizeId,
                    prize.winningPrizeNumbers.length,
                    ""
                );
                lastValue = prize.raffleItemPrizeIndex;
            }
            lastValue = win.entryIndex;
        }
    }
```

`claimPrize` function is another important component of the RafflesContract, enabling entrants to claim their won prizes. Here's a detailed explanation of its purpose, parameters, and functionality:

- Purpose:
The basic purpose of the claimPrize function is to allow an entrant or the contract owner to claim prizes won in a raffle. This function is crucial for the distribution of prizes, ensuring that winners can receive their prizes and that the contract owner can manage prize claims on behalf of entrants.

- Parameters: 
  - `_raffleId`: A uint256 value representing the ID of the raffle from which prizes are being claimed.
  - `_entrant`: The address of the entrant claiming the prizes.
  - `_wins`: An array of `ticketWinIO` structs. Each `ticketWinIO` struct represents a winning entry, including the entry index and an array of `PrizesWinIO` structs representing the prizes won.

- Functionality:
  - Access Control: The function first checks that the caller is either the entrant claiming the prizes or the contract owner. This ensures that only authorized entities can claim prizes.
  - Raffle Existence Validation: It validates that the specified raffle exists by checking that `_raffleId` is within the bounds of the raffles array.
  - Random Number Validation: The function checks that a random number has been generated for the raffle, as prizes can only be claimed after the raffle has ended and a random number has been determined.
  - Wins Validation and Processing: The function iterates over the _wins array. For each `ticketWinIO`, it validates the entry index and the prizes claimed, ensuring that each entry and prize is valid and has not been claimed before. It then marks the prizes as claimed in the contract's state.
  - Prize Transfer: For each validated win, the function transfers the corresponding prizes from the contract to the entrant. This is done by interacting with the ERC1155 token contracts specified in the `PrizesWinIO` structs, transferring the tokens to the entrant's address.
  - Event Emission: The function emits a `RaffleClaimPrize` event for each claimed prize, providing details about the raffle ID, the entrant address, the prize address, the prize ID, and the prize quantity.
- Usage:
This function is used by entrants to claim their won prizes after a raffle has concluded and a random number has been generated. It can also be used by the contract owner to manage prize claims on behalf of entrants, providing flexibility in how prizes are distributed. The detailed validation and processing steps ensure that only valid and unclaimed prizes can be claimed, maintaining the integrity and fairness of the raffle process.

## Summary And Proposal
The entire raffle contract.
```
//SPDX-License-Identifier: MIT
pragma solidity 0.8.0;

/*************/
/* Aavegotchi raffles contract
/* Authors: Nick Mudge (mudgen) and Daniel Mathieu (coderdan)
/*************/

import "./interfaces/IERC1155.sol";
import "./chainlink/LinkTokenInterface.sol";
import "./interfaces/IERC173.sol";
import "./interfaces/IERC165.sol";

// All state variables are accessed through this struct
// To avoid name clashes and make clear a variable is a state variable
// state variable access starts with "s." which accesses variables in this struct
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

struct RaffleItemPrize {
    address prizeAddress; // ERC1155 token contract
    uint96 prizeQuantity; // Number of ERC1155 tokens
    uint256 prizeId; // ERC1155 token type
}

// Ticket numbers are numbers between 0 and raffleItem.totalEntered - 1 inclusive.
struct RaffleItem {
    address ticketAddress; // ERC1155 token contract
    uint256 ticketId; // ERC1155 token type
    uint256 totalEntered; // Total number of ERC1155 tokens entered into raffle for this raffle item
    RaffleItemPrize[] raffleItemPrizes; // Prizes that can be won for this raffle item
}

contract RafflesContract is IERC173, IERC165 {
    // State variables are prefixed with s.
    AppStorage internal s;
    // Immutable values are prefixed with im_ to easily identify them in code
    LinkTokenInterface internal immutable im_link;
    address internal immutable im_vrfCoordinator;

    bytes4 internal constant ERC1155_ACCEPTED = 0xf23a6e61; // Return value from `onERC1155Received` call if a contract accepts receipt (i.e `bytes4(keccak256("onERC1155Received(address,address,uint256,uint256,bytes)"))`).
    event RaffleStarted(uint256 indexed raffleId, uint256 raffleEnd, RaffleItemInput[] raffleItems);
    event RaffleTicketsEntered(uint256 indexed raffleId, address entrant, TicketItemIO[] ticketItems);
    event RaffleRandomNumber(uint256 indexed raffleId, uint256 randomNumber);
    event RaffleClaimPrize(uint256 indexed raffleId, address entrant, address prizeAddress, uint256 prizeId, uint256 prizeQuantity);

    constructor(address _contractOwner, address _vrfCoordinator, address _link, bytes32 _keyHash, uint256 _fee) {
        s.contractOwner = _contractOwner;
        im_vrfCoordinator = _vrfCoordinator;
        im_link = LinkTokenInterface(_link);
        s.keyHash = _keyHash; //0x6c3699283bda56ad74f6b855546325b68d482e983852a7a82979cc4807b641f4;
        s.fee = uint96(_fee);

        // adding ERC165 data
        s.supportedInterfaces[type(IERC165).interfaceId] = true;
        s.supportedInterfaces[type(IERC173).interfaceId] = true;
        // skip raffle 0
        s.raffles.push();
    }

    function supportsInterface(bytes4 _interfaceId) external view override returns (bool) {
        return s.supportedInterfaces[_interfaceId];
    }

    // VRF Functionality ////////////////////////////////////////////////////////////////
    function nonces(bytes32 _keyHash) external view returns (uint256 nonce_) {
        nonce_ = s.nonces[_keyHash];
    }

    /**
     * @notice requestRandomness initiates a request for VRF output given _seed
     *
     * @dev See "SECURITY CONSIDERATIONS" above for more information on _seed.
     *
     * @dev The fulfillRandomness method receives the output, once it's provided
     * @dev by the Oracle, and verified by the vrfCoordinator.
     *
     * @dev The _keyHash must already be registered with the VRFCoordinator, and
     * @dev the _fee must exceed the fee specified during registration of the
     * @dev _keyHash.
     *
     * @param _keyHash ID of public key against which randomness is generated
     * @param _fee The amount of LINK to send with the request
     * @param _seed seed mixed into the input of the VRF
     *
     * @return requestId unique ID for this request
     *
     * @dev The returned requestId can be used to distinguish responses to *
     * @dev concurrent requests. It is passed as the first argument to
     * @dev fulfillRandomness.
     */
    function requestRandomness(bytes32 _keyHash, uint256 _fee, uint256 _seed) internal returns (bytes32 requestId) {
        im_link.transferAndCall(im_vrfCoordinator, _fee, abi.encode(_keyHash, _seed));
        // This is the seed passed to VRFCoordinator. The oracle will mix this with
        // the hash of the block containing this request to obtain the seed/input
        // which is finally passed to the VRF cryptographic machinery.
        // So the seed doesn't actually do anything and is left over from an old API.
        uint256 vRFSeed = makeVRFInputSeed(_keyHash, _seed, address(this), s.nonces[_keyHash]);
        // nonces[_keyHash] must stay in sync with
        // VRFCoordinator.nonces[_keyHash][this], which was incremented by the above
        // successful Link.transferAndCall (in VRFCoordinator.randomnessRequest).
        // This provides protection against the user repeating their input
        // seed, which would result in a predictable/duplicate output.
        s.nonces[_keyHash]++;
        return makeRequestId(_keyHash, vRFSeed);
    }

    /**
     * @notice returns the seed which is actually input to the VRF coordinator
     *
     * @dev To prevent repetition of VRF output due to repetition of the
     * @dev user-supplied seed, that seed is combined in a hash with the
     * @dev user-specific nonce, and the address of the consuming contract. The
     * @dev risk of repetition is mostly mitigated by inclusion of a blockhash in
     * @dev the final seed, but the nonce does protect against repetition in
     * @dev requests which are included in a single block.
     *
     * @param _userSeed VRF seed input provided by user
     * @param _requester Address of the requesting contract
     * @param _nonce User-specific nonce at the time of the request
     */
    function makeVRFInputSeed(bytes32 _keyHash, uint256 _userSeed, address _requester, uint256 _nonce) internal pure returns (uint256) {
        return uint256(keccak256(abi.encode(_keyHash, _userSeed, _requester, _nonce)));
    }

    /**
     * @notice Returns the id for this request
     * @param _keyHash The serviceAgreement ID to be used for this request
     * @param _vRFInputSeed The seed to be passed directly to the VRF
     * @return The id for this request
     *
     * @dev Note that _vRFInputSeed is not the seed passed by the consuming
     * @dev contract, but the one generated by makeVRFInputSeed
     */
    function makeRequestId(bytes32 _keyHash, uint256 _vRFInputSeed) internal pure returns (bytes32) {
        return keccak256(abi.encodePacked(_keyHash, _vRFInputSeed));
    }

    function drawRandomNumber(uint256 _raffleId) external {
        require(_raffleId < s.raffles.length, "Raffle: Raffle does not exist");
        Raffle storage raffle = s.raffles[_raffleId];
        require(raffle.raffleEnd < block.timestamp, "Raffle: Raffle time has not expired");
        require(raffle.randomNumber == 0, "Raffle: Random number already generated");
        require(raffle.randomNumberPending == false || msg.sender == s.contractOwner, "Raffle: Random number is pending");
        raffle.randomNumberPending = true;
        // Use Chainlink VRF to generate random number
        require(im_link.balanceOf(address(this)) >= s.fee, "Not enough LINK");
        bytes32 requestId = requestRandomness(s.keyHash, s.fee, 0);
        s.requestIdToRaffleId[requestId] = _raffleId;
    }

    // rawFulfillRandomness is called by VRFCoordinator when it receives a valid VRFproof.
    /**
     * @notice Callback function used by VRF Coordinator
     * @dev This is where you do something with randomness!
     * @dev The VRF Coordinator will only send this function verified responses.
     * @dev The VRF Coordinator will not pass randomness that could not be verified.
     */
    function rawFulfillRandomness(bytes32 _requestId, uint256 _randomness) external {
        require(msg.sender == im_vrfCoordinator, "Only VRFCoordinator can fulfill");
        uint256 raffleId = s.requestIdToRaffleId[_requestId];
        require(raffleId < s.raffles.length, "Raffle: Raffle does not exist");
        Raffle storage raffle = s.raffles[raffleId];
        require(raffle.raffleEnd < block.timestamp, "Raffle: Raffle time has not expired");
        require(raffle.randomNumber == 0, "Raffle: Random number already generated");
        s.raffles[raffleId].randomNumber = _randomness;
        raffle.randomNumberPending = false;
        emit RaffleRandomNumber(raffleId, _randomness);
    }

    // Change the fee amount that is paid for VRF random numbers
    function changeVRFFee(uint256 _newFee, bytes32 _keyHash) external {
        require(msg.sender == s.contractOwner, "Raffle: Must be contract owner");
        s.fee = uint96(_newFee);
        s.keyHash = _keyHash;
    }

    // Remove the LINK tokens from this contract that are used to pay for VRF random number fees
    function removeLinkTokens(address _to, uint256 _value) external {
        require(msg.sender == s.contractOwner, "Raffle: Must be contract owner");
        im_link.transfer(_to, _value);
    }

    function linkBalance() external view returns (uint256 linkBalance_) {
        linkBalance_ = im_link.balanceOf(address(this));
    }

    /////////////////////////////////////////////////////////////////////////////////////

    function owner() external view override returns (address) {
        return s.contractOwner;
    }

    function transferOwnership(address _newContractOwner) external override {
        address previousOwner = s.contractOwner;
        require(msg.sender == previousOwner, "Raffle: Must be contract owner");
        s.contractOwner = _newContractOwner;
        emit OwnershipTransferred(previousOwner, _newContractOwner);
    }

    // structs with IO at the end of their name mean they are only used for
    // arguments and/or return values of functions
    struct RaffleItemInput {
        address ticketAddress;
        uint256 ticketId;
        RaffleItemPrizeIO[] raffleItemPrizes;
    }
    struct RaffleItemPrizeIO {
        address prizeAddress;
        uint256 prizeId;
        uint256 prizeQuantity;
    }

    /**
     * @notice Starts a raffle
     * @dev The _raffleItems argument tells what ERC1155 tickets can be entered for what ERC1155 prizes.
     * The _raffleItems get stored in the raffleItems state variable
     * The raffle prizes that can be won are transferred into this contract.
     * @param _raffleDuration How long a raffle goes for, in seconds
     * @param _raffleItems What tickets to enter for what prizes
     */
    function startRaffle(uint256 _raffleDuration, RaffleItemInput[] calldata _raffleItems) external {
        require(msg.sender == s.contractOwner, "Raffle: Must be contract owner");
        require(_raffleDuration >= 3600, "Raffle: _raffleDuration must be greater than 1 hour");
        uint256 raffleEnd = block.timestamp + _raffleDuration;
        require(_raffleItems.length > 0, "Raffle: No raffle items");
        uint256 raffleId = s.raffles.length;
        emit RaffleStarted(raffleId, raffleEnd, _raffleItems);
        Raffle storage raffle = s.raffles.push();
        raffle.raffleEnd = raffleEnd;
        for (uint256 i; i < _raffleItems.length; i++) {
            RaffleItemInput calldata raffleItemInput = _raffleItems[i];
            require(raffleItemInput.raffleItemPrizes.length > 0, "Raffle: No prizes");
            // ticketAddress is ERC1155 contract address of tickets
            // ticketId is the ERC1155 type id, which type is it
            require(
                // The index is one greater than actual index.  If index is 0 it means the value does not exist yet.
                raffle.raffleItemIndexes[raffleItemInput.ticketAddress][raffleItemInput.ticketId] == 0,
                "Raffle: Raffle item already using ticketAddress and ticketId"
            );
            // A raffle item is a ticketAddress, ticketId and what prizes can be won.
            RaffleItem storage raffleItem = raffle.raffleItems.push();
            // The index is one greater than actual index.  If index is 0 it means the value does not exist yet.
            raffle.raffleItemIndexes[raffleItemInput.ticketAddress][raffleItemInput.ticketId] = raffle.raffleItems.length;
            raffleItem.ticketAddress = raffleItemInput.ticketAddress;
            raffleItem.ticketId = raffleItemInput.ticketId;
            for (uint256 j; j < raffleItemInput.raffleItemPrizes.length; j++) {
                RaffleItemPrizeIO calldata raffleItemPrizeIO = raffleItemInput.raffleItemPrizes[j];
                raffleItem.raffleItemPrizes.push(
                    RaffleItemPrize(raffleItemPrizeIO.prizeAddress, uint96(raffleItemPrizeIO.prizeQuantity), raffleItemPrizeIO.prizeId)
                );
                IERC1155(raffleItemPrizeIO.prizeAddress).safeTransferFrom(
                    msg.sender,
                    address(this),
                    raffleItemPrizeIO.prizeId,
                    raffleItemPrizeIO.prizeQuantity,
                    abi.encode(raffleId)
                );
            }
        }
    }

    /**
        @notice Handle the receipt of a single ERC1155 token type.
        @dev An ERC1155-compliant smart contract MUST call this function on the token recipient contract, at the end of a `safeTransferFrom` after the balance has been updated.        
        This function MUST return `bytes4(keccak256("onERC1155Received(address,address,uint256,uint256,bytes)"))` (i.e. 0xf23a6e61) if it accepts the transfer.
        This function MUST revert if it rejects the transfer.
        Return of any other value than the prescribed keccak256 generated value MUST result in the transaction being reverted by the caller.
        @param _operator  The address which initiated the transfer (i.e. msg.sender)
        @param _from      The address which previously owned the token
        @param _id        The ID of the token being transferred
        @param _value     The amount of tokens being transferred
        @param _data      Additional data with no specified format
        @return           `bytes4(keccak256("onERC1155Received(address,address,uint256,uint256,bytes)"))`
    */
    function onERC1155Received(address _operator, address _from, uint256 _id, uint256 _value, bytes calldata _data) external view returns (bytes4) {
        _operator; // silence not used warning
        _from; // silence not used warning
        _id; // silence not used warning
        _value; // silence not used warning
        require(_data.length == 32, "Raffle: Data of the wrong size sent on transfer");
        uint256 raffleId = abi.decode(_data, (uint256));
        require(raffleId < s.raffles.length, "Raffle: Raffle does not exist");
        Raffle storage raffle = s.raffles[raffleId];
        uint256 raffleEnd = raffle.raffleEnd;
        require(raffleEnd > block.timestamp, "Raffle: Can't accept transfer for expired raffle");
        return ERC1155_ACCEPTED;
    }

    struct RaffleIO {
        uint256 raffleId;
        uint256 raffleEnd;
        bool isOpen;
    }

    /**
     * @notice Get simple raffle information
     */
    function getRaffles() external view returns (RaffleIO[] memory raffles_) {
        raffles_ = new RaffleIO[](s.raffles.length);
        for (uint256 i; i < s.raffles.length; i++) {
            uint256 raffleEnd = s.raffles[i].raffleEnd;
            raffles_[i].raffleId = i;
            raffles_[i].raffleEnd = raffleEnd;
            raffles_[i].isOpen = raffleEnd > block.timestamp;
        }
    }

    /**
     * @notice Get total number of raffles that exist.
     */
    function raffleSupply() external view returns (uint256 raffleSupply_) {
        raffleSupply_ = s.raffles.length;
    }

    struct RaffleItemOutput {
        address ticketAddress;
        uint256 ticketId;
        uint256 totalEntered;
        RaffleItemPrizeIO[] raffleItemPrizes;
    }

    /**
     * @notice Get simple raffle info and all the raffle items in the raffle.
     * @param _raffleId Which raffle to get info about.
     */
    function raffleInfo(uint256 _raffleId) external view returns (uint256 raffleEnd_, RaffleItemOutput[] memory raffleItems_, uint256 randomNumber_) {
        require(_raffleId < s.raffles.length, "Raffle: Raffle does not exist");
        Raffle storage raffle = s.raffles[_raffleId];
        raffleEnd_ = raffle.raffleEnd;
        if (raffle.randomNumberPending == true) {
            randomNumber_ = 1;
        } else {
            randomNumber_ = raffle.randomNumber;
        }
        // Loop over and get all the raffle itmes, which includes ERC1155 tickets and ERC1155 prizes
        raffleItems_ = new RaffleItemOutput[](raffle.raffleItems.length);
        for (uint256 i; i < raffle.raffleItems.length; i++) {
            RaffleItem storage raffleItem = raffle.raffleItems[i];
            raffleItems_[i].ticketAddress = raffleItem.ticketAddress;
            raffleItems_[i].ticketId = raffleItem.ticketId;
            raffleItems_[i].totalEntered = raffleItem.totalEntered;
            raffleItems_[i].raffleItemPrizes = new RaffleItemPrizeIO[](raffleItem.raffleItemPrizes.length);
            for (uint256 j; j < raffleItem.raffleItemPrizes.length; j++) {
                RaffleItemPrize storage raffleItemPrize = raffleItem.raffleItemPrizes[j];
                raffleItems_[i].raffleItemPrizes[j].prizeAddress = raffleItemPrize.prizeAddress;
                raffleItems_[i].raffleItemPrizes[j].prizeId = raffleItemPrize.prizeId;
                raffleItems_[i].raffleItemPrizes[j].prizeQuantity = raffleItemPrize.prizeQuantity;
            }
        }
    }

    struct EntryIO {
        address ticketAddress; // ERC1155 contract address
        uint256 ticketId; // ERC1155 type id
        uint256 ticketQuantity; // Number of ERC1155 tokens
        uint256 rangeStart;
        uint256 rangeEnd;
        uint256 raffleItemIndex;
        bool prizesClaimed;
    }

    /**
     * @notice Get get ticket info for a single entrant (address)
     * @param _raffleId Which raffle to get ticket stats about
     * @param _entrant Who to get stats about
     */
    function getEntries(uint256 _raffleId, address _entrant) external view returns (EntryIO[] memory entries_) {
        require(_raffleId < s.raffles.length, "Raffle: Raffle does not exist");
        Raffle storage raffle = s.raffles[_raffleId];
        entries_ = new EntryIO[](raffle.entries[_entrant].length);
        for (uint256 i; i < raffle.entries[_entrant].length; i++) {
            Entry memory entry = raffle.entries[_entrant][i];
            RaffleItem storage raffleItem = raffle.raffleItems[entry.raffleItemIndex];
            entries_[i].ticketAddress = raffleItem.ticketAddress;
            entries_[i].ticketId = raffleItem.ticketId;
            entries_[i].ticketQuantity = entry.rangeEnd - entry.rangeStart;
            entries_[i].rangeStart = entry.rangeStart;
            entries_[i].rangeEnd = entry.rangeEnd;
            entries_[i].raffleItemIndex = entry.raffleItemIndex;
            entries_[i].prizesClaimed = entry.prizesClaimed;
        }
    }

    struct TicketStatsIO {
        address ticketAddress; // ERC1155 contract address
        uint256 ticketId; // ERC1155 type id
        uint256 numberOfEntrants; // number of unique addresses that entered tickets
        uint256 totalEntered; // Number of ERC1155 tokens
    }

    /**
     * @notice Returns what tickets have been entered, by how many addresses, and how many ERC1155 tickets entered
     * @dev It is possible for this function to run out of gas when called off-chain if there are very many users (Infura has gas limit for off-chain calls)
     * @param _raffleId Which raffle to get info about
     */
    function ticketStats(uint256 _raffleId) external view returns (TicketStatsIO[] memory ticketStats_) {
        require(_raffleId < s.raffles.length, "Raffle: Raffle does not exist");
        Raffle storage raffle = s.raffles[_raffleId];
        ticketStats_ = new TicketStatsIO[](raffle.raffleItems.length);
        // loop through raffle items
        for (uint256 i; i < raffle.raffleItems.length; i++) {
            RaffleItem storage raffleItem = raffle.raffleItems[i];
            ticketStats_[i].ticketAddress = raffleItem.ticketAddress;
            ticketStats_[i].ticketId = raffleItem.ticketId;
            ticketStats_[i].totalEntered = raffleItem.totalEntered;
            // count the number of users that have ticketd for the raffle item
            for (uint256 j; j < raffle.entrants.length; j++) {
                address entrant = raffle.entrants[j];
                for (uint256 k; k < raffle.entries[entrant].length; k++) {
                    if (i == raffle.entries[entrant][k].raffleItemIndex) {
                        ticketStats_[i].numberOfEntrants++;
                        break;
                    }
                }
            }
        }
    }

    struct TicketItemIO {
        address ticketAddress; // ERC1155 contract address (entry ticket), not prize
        uint256 ticketId; // ERC1155 type id
        uint256 ticketQuantity; // Number of ERC1155 tokens
    }

    /**
     * @notice Enter ERC1155 tokens for raffle prizes
     * @dev Creates a new entry in the userEntries array
     * @param _raffleId Which raffle to ticket in
     * @param _ticketItems The ERC1155 tokens to ticket
     */
    function enterTickets(uint256 _raffleId, TicketItemIO[] calldata _ticketItems) external {
        require(_raffleId < s.raffles.length, "Raffle: Raffle does not exist");
        require(_ticketItems.length > 0, "Raffle: No tickets");
        Raffle storage raffle = s.raffles[_raffleId];
        require(raffle.raffleEnd > block.timestamp, "Raffle: Raffle time has expired");
        emit RaffleTicketsEntered(_raffleId, msg.sender, _ticketItems);
        // Collect unique entrant addresses
        if (raffle.entries[msg.sender].length == 0) {
            raffle.entrants.push(msg.sender);
        }
        for (uint256 i; i < _ticketItems.length; i++) {
            TicketItemIO calldata ticketItem = _ticketItems[i];
            require(ticketItem.ticketQuantity > 0, "Raffle: Ticket quantity cannot be zero");
            // get the raffle item
            uint256 raffleItemIndex = raffle.raffleItemIndexes[ticketItem.ticketAddress][ticketItem.ticketId];
            require(raffleItemIndex > 0, "Raffle: Raffle item doesn't exist for this raffle");
            raffleItemIndex--;
            RaffleItem storage raffleItem = raffle.raffleItems[raffleItemIndex];
            uint256 totalEntered = raffleItem.totalEntered;
            // Create a range of unique numbers for ticket ids
            raffle.entries[msg.sender].push(
                Entry(uint24(raffleItemIndex), false, uint112(totalEntered), uint112(totalEntered + ticketItem.ticketQuantity))
            );
            // update the total quantity of tickets that have been entered for this raffle item
            raffleItem.totalEntered = totalEntered + ticketItem.ticketQuantity;
            // transfer the ERC1155 tokens to ticket to this contract
            IERC1155(ticketItem.ticketAddress).safeTransferFrom(
                msg.sender,
                address(this),
                ticketItem.ticketId,
                ticketItem.ticketQuantity,
                abi.encode(_raffleId)
            );
        }
    }

    // Get the unique addresses of entrants in a raffle
    function getEntrants(uint256 _raffleId) external view returns (address[] memory entrants_) {
        require(_raffleId < s.raffles.length, "Raffle: Raffle does not exist");
        Raffle storage raffle = s.raffles[_raffleId];
        entrants_ = raffle.entrants;
    }

    /* This struct information can be gotten from the return results of the winners function */
    struct ticketWinIO {
        uint256 entryIndex; // index into a user's array of tickets (which staking attempt won)
        PrizesWinIO[] prizes;
    }

    // Ticket numbers are numbers between 0 and raffleItem.totalEntered - 1 inclusive.
    // Winning ticket numbers are ticket numbers that won one or more prizes
    // Prize numbers are numbers between 0 and raffleItemPrize.prizeQuanity - 1 inclusive.
    // Prize numbers are used to calculate ticket numbers
    // Winning prize numbers are prize numbers used to calculate winning ticket numbers
    struct PrizesWinIO {
        uint256 raffleItemPrizeIndex; // index into the raffleItemPrizes array (which prize was won)
        uint256[] winningPrizeNumbers; // ticket numbers between 0 and raffleItem.totalEntered that won
    }

    /**
     * @notice Claim prizes won
     * @dev All items in _wins are verified as actually won by the address that calls this function and reverts otherwise.
     * @dev Each entrant address can only claim prizes once, so be sure to include all entries and prizes won.
     * @dev Prizes are transfered to the address that calls this function.
     * @dev Due to the possibility that an entrant does not claim all the prizes he/she won or the gas cost is too high,
     * the contractOwner can claim prizes for an entrant. This needs to be used with care so that contractOwner does not
     * accidentally claim prizes for an entrant that have already been claimed for or by the entrant.
     * @param _entrant The entrant that won the prizes
     * @param _raffleId The raffle that prizes were won in.
     * @param _wins Contains only winning entries and prizes that were won.
     */
    function claimPrize(uint256 _raffleId, address _entrant, ticketWinIO[] calldata _wins) external {
        require(_raffleId < s.raffles.length, "Raffle: Raffle does not exist");
        Raffle storage raffle = s.raffles[_raffleId];
        uint256 randomNumber = raffle.randomNumber;
        require(randomNumber > 0, "Raffle: Random number not generated yet");
        // contractOwner can claim prizes for the entrant.  Prizes are only transferred to the entrant
        require(msg.sender == _entrant || msg.sender == s.contractOwner, "Raffle: Not claimed by owner or contractOwner");
        // Logic:
        // 1. Loop through wins
        // 2. Verify provided entryIndex exists and is not a duplicate
        // 3. Loop through prizes
        // 4. Verify provided prize exists and is not a duplicate
        // 5. Loop through winning prize numbers
        // 6. Verify winning prize number exists and is not a duplicate
        // 7. Verify that winning prize number actually won
        // 8. Transfer prizes to winner
        //--------------------------------------------
        // lastValue serves two purposes:
        // 1. Ensures that a value is less than the length of an array
        // 2. Prevents duplicates. Subsequent values must be lesser
        // lastValue gets reused by inner loops
        uint256 lastValue = raffle.entries[_entrant].length;
        for (uint256 i; i < _wins.length; i++) {
            ticketWinIO calldata win = _wins[i];
            // Serves two purposes: 1. Ensure is less than raffle.entries[_entrant].length. 2. prevents duplicates
            require(win.entryIndex < lastValue, "Raffle: User entry does not exist or is not lesser than last value");
            Entry memory entry = raffle.entries[_entrant][win.entryIndex];
            require(entry.prizesClaimed == false, "Raffles: Entry prizes have already been claimed");
            raffle.entries[_entrant][win.entryIndex].prizesClaimed = true;
            // total number of tickets that have been entered for a raffle item
            uint256 totalEntered = raffle.raffleItems[entry.raffleItemIndex].totalEntered;
            lastValue = raffle.raffleItems[entry.raffleItemIndex].raffleItemPrizes.length;
            for (uint256 j; j < win.prizes.length; j++) {
                PrizesWinIO calldata prize = win.prizes[j];
                // Serves two purposes: 1. Ensure is less than raffleItemPrizes.length. 2. prevents duplicates
                require(prize.raffleItemPrizeIndex < lastValue, "Raffle: Raffle prize type does not exist or is not lesser than last value");
                RaffleItemPrize memory raffleItemPrize = raffle.raffleItems[entry.raffleItemIndex].raffleItemPrizes[prize.raffleItemPrizeIndex];
                lastValue = raffleItemPrize.prizeQuantity;
                for (uint256 k; k < prize.winningPrizeNumbers.length; k++) {
                    uint256 prizeNumber = prize.winningPrizeNumbers[k];
                    // Serves two purposes: 1. Ensure is less than raffleItemPrize.prizeQuantity. 2. prevents duplicates
                    require(prizeNumber < lastValue, "Raffle: prizeNumber does not exist or is not lesser than last value");
                    uint256 winningTicketNumber = uint256(
                        keccak256(abi.encodePacked(randomNumber, entry.raffleItemIndex, prize.raffleItemPrizeIndex, prizeNumber))
                    ) % totalEntered;
                    require(winningTicketNumber >= entry.rangeStart && winningTicketNumber < entry.rangeEnd, "Raffle: Did not win prize");
                    lastValue = prizeNumber;
                }
                emit RaffleClaimPrize(_raffleId, _entrant, raffleItemPrize.prizeAddress, raffleItemPrize.prizeId, prize.winningPrizeNumbers.length);
                IERC1155(raffleItemPrize.prizeAddress).safeTransferFrom(
                    address(this),
                    _entrant,
                    raffleItemPrize.prizeId,
                    prize.winningPrizeNumbers.length,
                    ""
                );
                lastValue = prize.raffleItemPrizeIndex;
            }
            lastValue = win.entryIndex;
        }
    }
}

```

The `Aavegotchi RafflesContract` is a smart contract designed to manage raffles involving ERC1155 tokens, incorporating functionalities for raffle creation, entry, prize claiming, and ownership management. It utilizes Chainlink's VRF for generating random numbers, ensuring fairness in raffle outcomes. The contract is structured around a central AppStorage struct for state management, with various structs defined for managing raffles, entries, and prizes.

- Key Points:
  - Raffle Management: Facilitates the creation and management of raffles, allowing entrants to enter tickets and claim prizes once the raffle has concluded.
  - VRF Integration: Leverages Chainlink's VRF to generate random numbers, ensuring the fairness and security of raffle outcomes.
  - ERC1155 Support: Fully supports ERC1155 tokens, enabling the use of unique tokens for raffle entries and prizes.
  - Ownership Management: Includes functions for transferring ownership of the contract. The current implementation allows for immediate transfer of ownership, which could possibily go wrong when a wrong address is entered.

- Proposed Improvement:
```
//state variables
address owner;
address newOwner;
uint256 timeLock;

function transferOwnership(address _newOwner) external {
    require(msg.sender == owner, "not the owner");
    newOwner = _newOwner;
    timeLock = block.timestamp + 7 days; // Set a 7-day time lock
}

function claimOwnership() external {
    require(msg.sender == newOwner, "not the next owner");
    require(block.timestamp >= timeLock, "ownership transfer not yet claimable");
    owner = msg.sender;
    newOwner = address(0);
    timeLock = 0;
}

function cancelOwnershipTransfer() external {
    require(msg.sender == owner, "not the owner");
    newOwner = address(0);
    timeLock = 0;
}
```
My proposal on the improvement of ownership transfer management is to introduce a two-step, time-locked ownership transfer process. This would involve a `transferOwnership` function that sets a `newOwner` address but does not immediately transfer ownership. Instead, the new owner would need to call a `claimOwnership` function to officially become the owner. This function would only be callable after a predetermined time lock has expired, ensuring that the new owner has sufficient time to claim ownership. If the new owner does not claim ownership within the time lock period, the previous owner retains control, and the `newOwner` address is reset. Additionally, the current owner has the option to cancel the ownership transfer before the time lock expires, further enhancing the security of the ownership transfer process. This approach significantly reduces the risk of unauthorized ownership claims and replay attacks, ensuring that ownership transfers are intentional and cannot be hijacked if the new owner's address is compromised.  