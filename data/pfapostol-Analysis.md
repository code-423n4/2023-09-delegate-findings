## Approach taken in evaluating the codebase

I first explored the scope of audit. I discovered that the project can be divided into 2 independent parts: `Delegate Registry` and `Delegate Marketplace`. I carried out all subsequent stages separately for each of this parts, and then analyzed the correctness of their interaction.

### Test coverage:

`Delegate Registry`:

Test coverage is 100% for most audit files. In this regard, I decided to concentrate on finding logical errors, since simple errors (errors due to typos, incorrect statements) should be excluded by tests.

| File                              | % Lines           | % Statements      | % Branches     | % Funcs         |
| --------------------------------- | ----------------- | ----------------- | -------------- | --------------- |
| src/DelegateRegistry.sol          | 100.00% (175/175) | 100.00% (219/219) | 98.78% (81/82) | 100.00% (33/33) |
| src/libraries/RegistryHashes.sol  | 100.00% (12/12)   | 100.00% (12/12)   | 100.00% (0/0)  | 100.00% (12/12) |
| src/libraries/RegistryOps.sol     | 66.67% (2/3)      | 66.67% (2/3)      | 100.00% (0/0)  | 66.67% (2/3)    |
| src/libraries/RegistryStorage.sol | 100.00% (6/6)     | 100.00% (6/6)     | 100.00% (0/0)  | 100.00% (3/3)   |

`Delegate Marketplace`:

| File                                             | % Lines          | % Statements      | % Branches       | % Funcs          |
|--------------------------------------------------|------------------|-------------------|------------------|------------------|
| src/CreateOfferer.sol                            | 90.70% (39/43)   | 92.00% (46/50)    | 77.27% (17/22)   | 100.00% (8/8)    |
| src/DelegateToken.sol                            | 88.55% (147/166) | 90.28% (195/216)  | 80.43% (37/46)   | 89.66% (26/29)   |
| src/PrincipalToken.sol                           | 100.00% (14/14)  | 100.00% (17/17)   | 100.00% (4/4)    | 100.00% (5/5)    |
| src/libraries/CreateOffererLib.sol               | 95.24% (40/42)   | 95.38% (62/65)    | 69.23% (18/26)   | 100.00% (9/9)    |
| src/libraries/DelegateTokenLib.sol               | 88.89% (8/9)     | 90.48% (19/21)    | 75.00% (6/8)     | 100.00% (5/5)    |
| src/libraries/DelegateTokenRegistryHelpers.sol   | 100.00% (57/57)  | 100.00% (87/87)   | 100.00% (26/26)  | 100.00% (21/21)  |
| src/libraries/DelegateTokenStorageHelpers.sol    | 91.67% (44/48)   | 92.11% (70/76)    | 80.77% (21/26)   | 100.00% (21/21)  |
| src/libraries/DelegateTokenTransferHelpers.sol   | 88.24% (30/34)   | 87.80% (36/41)    | 80.77% (21/26)   | 100.00% (9/9)    |


### Code review:

I studied the `Delegate Registry` code starting with the libraries, and also starting from the lowest level functions, moving to the top level functions. Having built a general understanding of what each of the functions does, I formed an idea of how the Registry works and built general diagrams.

#### Packed delegation data:

![2023-09-delegate-delegation-state.jpg](https://user-images.githubusercontent.com/50257230/267004960-285bf761-0dd8-47bf-b3c2-84971c8b0ab4.jpg)

##### Important external interfaces:

1. delegateAll - Delegates the entire wallet
2. delegateContract - Delegates the right to use the contract
3. delegateERC721 - Delegates the right to use a specific contract token
4. delegateERC20 - Delegates the right to use a certain amount of a token of a certain contract
5. delegateERC1155 - Delegates the right to use a certain amount of a certain token of a certain contract

There are also 5 functions to check the correctness of the delegation

<details>
  <summary>Details on each function and hashing schemes</summary>
  
`RegistryOps` library:

1. Contains 3 operation:
    1. `max`: use optimized assembly logic to calculate max of 2 numbers
    2. `and`: use 2x `iszero` to clean arguments before `and`
    3. `or`: use 2x `iszero` to clean arguments before `or`
    
`RegistryStorage` library:

1. Contains 10 constants:
    1. mostly offsets for packing/unpacking of addresses
2. Contains 3 functions:
    1. `packAddresses`: - store `from`, `to` and `contract` addresses in 2 storage slots
    2. `unpackAddresses`: - reverse to `packAddresses` operation
    3. `unpackAddress`: - helper to unpack `to` or `from`. Should not to be used for `contract` unwrapping

`RegistryHashes` library:

1. Contains 7 constants:
    1. mostly types of hashes
2. Contains 12 functions:
    1. `decodeType`: - decode hash type from last byte into `enum`, (potentially may overflow `enum`)
    2. `location`: - calculate storage key from hash
        
        ![2023-09-delegate-location.jpg](https://user-images.githubusercontent.com/50257230/267005098-69e33a94-3981-4fe4-9132-bff4b01fe6d6.jpg)
        
    3. `allHash`: - calculate hash for `all` type
        
        ![2023-09-delegate-all-hash.jpg](https://user-images.githubusercontent.com/50257230/267005120-2c0f64f1-9956-4831-9aa5-94b5db62eae3.jpg)
        
    4. `allLocation`: - calculate location for `all` type hash
        
        ![2023-09-delegate-all-hash-location.jpg](https://user-images.githubusercontent.com/50257230/267005108-f66df89c-bb88-4cfc-bd86-d2e207368f0a.jpg)
        
    5. `contractHash` : - similar to `allHash`
    6. `contractLocation`: - similar to `allLocation`
    7. `erc721Hash`: - similar to `allHash`
    8. `erc721Location`: - similar to `allLocation`
    9. `erc20Hash`: - similar to `allHash`
    10. `erc20Location`: - similar to `allLocation`
    11. `erc1155Hash`: - similar to `allHash`
    12. `erc1155Location`: - similar to `allLocation`

`DelegateRegistry` contract:

1. contains 3 state variables:
    1. `delegations`
    2. `outgoingDelegationHashes`
    3. `incomingDelegationHashes`
2. contains 33 functions:
    1. `sweep`: - transfer all contract balance to hardcoded address (Currently 0x0)
    2. `readSlot`: - perform `sload`
    3. `readSlots`: - perform `sload`s in loop
    4. `_pushDelegationHashes`: - *push delegation hash to the incoming and outgoing hashes mappings*
    5. `_writeDelegation` x2 : - perform `sstore` for `data` at `position` in `location`
    6. `_updateFrom`: - change `from` value in first slot, while keeping first 8 bytes of `contract` intact
    7. `_loadDelegationBytes32`: - perform `sload` at `position` in `location`
    8. `_loadDelegationUint`: - similar to `_loadDelegationBytes32`
    9. `multicall`: - payable multicall
    10. `supportsInterface`: -
    11. `_writeDelegationAddresses`: - `sstore` packed delegation at 0 and 1 slot in `location`
    12. `_loadFrom`: - `sload` `from` address from `location`
    13. `_loadDelegationAddresses`: - reverse to `_writeDelegationAddresses`
    14. `_invalidFrom`: - check if address is *`DELEGATION_EMPTY` or `DELEGATION_REVOKED` flags(addresses)*
    15. `_validateFrom`: - match passed `from` to value in `location`
    16. `checkDelegateForAll`: - validate that `from` delegated `to` the entire wallet
    17. `checkDelegateForContract`: - the same as `checkDelegateForContract` or delegated for specific `contract`
    18. `checkDelegateForERC721` : - the same as `checkDelegateForContract` or delegated for specific `tokenId` in specific `contract`
    19. `checkDelegateForERC20`: - return amount delegated `from` to `to` 
    20. `checkDelegateForERC1155` : - similar to `checkDelegateForERC20`
    21. `_getValidDelegationHashesFromHashes`: - remove invalid `from`s from hashes array
    22. `getIncomingDelegationHashes`: - return only valid hashes from `incomingDelegationHashes`
    23. `getOutgoingDelegationHashes`: - the same as `getIncomingDelegationHashes`, but  with `outgoingDelegationHashes`
    24. `_getValidDelegationsFromHashes`: - read storage for every valid hash in memory `Delegation` struct
    25. `getIncomingDelegations`: - return `Delegation` struct for only valid hashes from `incomingDelegationHashes`
    26. `getOutgoingDelegations`: - the same as `getIncomingDelegations`, but  with `outgoingDelegationHashes`
    27. `getDelegationsFromHashes`: - the same as `_getValidDelegationsFromHashes` but for invalid delegation return empty struct
    28. `delegateAll`: - `msg.sender` delegate the whole wallet to `from`
        
        ![2023-09-delegate-delegate-all-flow.jpg](https://user-images.githubusercontent.com/50257230/267005089-e7b3823d-d232-4180-91fd-2a94083b7d03.jpg)
        
    29. `delegateContract`: - similar to `delegateAll`, but for specific `contract`
    30. `delegateERC721`: - similar to `delegateContract`, but for specific `tokenId`
    31. `delegateERC20`: - similar to `delegateContract`, but for `ERC20` token `amount` + allow to change `amount` if already delegated
    32. `delegateERC1155`: - similar to `delegateERC20`, but for `ERC1155` (specific `tokenId`)
  
</details>

`Delegate Marketplace`:

`CreateOfferer` is a separate part of the marketplace that guarantees interaction with the seaport.


##### Important external interfaces:

1. create - Create `DelegateToken` and `PrincipalToken` tokens. Transfer one of token types to contract. Delegate to `delegateHolder``. mint principal token.
2. extend - Extend the expiration time for an existing `DelegateToken`. Called by `PrincipalToken` owner.
3. rescind - Return the `DelegateToken` to the `PrincipalToken` holder early. Called by `DelegateToken` holder or after the `DelegateToken` has expired, anyone can call this method. this does not release the spot asset from escrow, it merely cancels out the `DelegateToken`.
4. withdraw - burn the `PrincipalToken` and claim the spot asset from escrow. Called by the `PrincipalToken` owner. `PrincipalToken` owner can authorize others to call this on their behalf, and if `PrincipalToken` owner also owns the `DelegateToken` then they can skip calling rescind and go straight to withdraw

<details>
  <summary>Details for each function</summary>
  
# Delegate marketplace

`DelegateTokenStorageHelpers` library:

1. Contains 10 constants:
    1. mostly flags and storage positions
2. Contains 21 functions:
    1. `writeApproved`: - store `approved` to *`PACKED_INFO_POSITION*` while keeping `expiry` intact
    2. `writeExpiry`: - store `expiry` to *`PACKED_INFO_POSITION*` while keeping `approved` intact
    3. `writeRegistryHash`: - store `registryHash` to *`REGISTRY_HASH_POSITION`*
    4. `writeUnderlyingAmount`: - store `underlyingAmount` to *`UNDERLYING_AMOUNT_POSITION`*
    5. `incrementBalance`: - increment `balance` for `delegateTokenHolder`
    6. `decrementBalance`: - decrement `balance` for `delegateTokenHolder`
    7. `principalIsCaller`: - revert if `msg.sender` is not `principalToken`
    8. `revertAlreadyExisted`: - revert if `registryHash` is not zero
    9. `revertNotOperator`: - revert if not `operator` or “owner”
    10. `readApproved`: - shift *`PACKED_INFO_POSITION` to read `approved`*
    11. `readExpiry`: - read `expiry` from *`PACKED_INFO_POSITION`* 
    12. `readRegistryHash`: - read `registryHash` from *`REGISTRY_HASH_POSITION`*
    13. `readUnderlyingAmount`: - read `underlyingAmount` from *`UNDERLYING_AMOUNT_POSITION`*
    14. `revertNotMinted`: - revert if `registryHash` is not set or used (*`ID_AVAILABLE`, `ID_USED`*)
    15. `checkBurnAuthorized`: - revert if caller is not `principalToken` or delegate not authorized burn 
    16. `checkMintAuthorized`: - similar to `checkBurnAuthorized` but with mint
    17. `revertNotApprovedOrOperator`: - revert if caller is not “owner” or `operator` or `approved` in token
    18. `revertInvalidWithdrawalConditions`: - similar to `revertNotApprovedOrOperator` + check expiry
    19. `burnPrincipal` : - call `burn` on `PrincipalToken` with custom reentrancy guard
    20. `mintPrincipal`: - call `mint` on `PrincipalToken` with custom reentrancy guard

`DelegateTokenRegistryHelpers` library:

1. Contains 21 functions:
    1. `loadTokenHolder`: - read `to` from `delegateRegistry` at `location` from `registryHash`. Not revert on revoked!!!
    2. `loadContract`: - read `contract` from `delegateRegistry` at `location` from `registryHash`
    3. `loadTokenHolderAndContract`: - read `to` and `contract` from `delegateRegistry` at `location` from `registryHash`
    4. `loadFrom`: - similar with `from`
    5. `loadAmount`: - similar with `amount`
    6. `loadRights`: - similar with `rights`
    7. `loadTokenId`: - similar with `tokenId`
    8. `calculateDecreasedAmount`: - return `amount` - `decreaseAmount`. No underflow check!!!
    9. `calculateIncreasedAmount`: - similar to `calculateDecreasedAmount` ,but increased
    10. `transferERC721`: - revoke delegation to `from` and delegate `to` while validating both hashes
    11. `revokeERC721`: - revoke delegation and validate hash
    12. `delegateERC721`: - delegate and validate hash
    13. `revertERC721FlashUnavailable`: - revert if `contract` does not have `rights` for `flashloan` or `tokenId` itself
    14. `revertERC20FlashAmountUnavailable`: - revert if delegation does not have enough `amount` with `“”` and `flashloan` `rights`
    15. `revertERC1155FlashAmountUnavailable`: - similar to `revertERC20FlashAmountUnavailable`
    16. `transferERC20`: - decrease `amount` from old delegation and increase for new
    17. `transferERC1155`: - similar to `transferERC20`
    18. `incrementERC20`: - increase amount in delegation
    19. `incrementERC1155`: - the same with `ERC1155`
    20. `decrementERC20`: - similar to `incrementERC20`, but decrease
    21. `decrementERC1155`: - similar to `incrementERC1155`, but decrease

`DelegateTokenTransferHelpers` library:

1. Contains 2 constants:
    1. `ERC1155` callbacks
2. Contains 9 functions:
    1. `checkERC1155BeforePull` : - custom reentrancy guard + revert if amount == 0 
    2. `checkERC1155Pulled`:- bottom part of custom reentrancy guard + require contract to be operator
    3. `revertInvalidERC1155PullCheck`: - revert on `checkERC1155Pulled` condition
    4. `pullERC1155AfterCheck`: - transfer `ERC1155` from `msg.sender` to contract revert if *`ERC1155_PULLED`*
    5. `checkERC20BeforePull`: - check that it is `ERC20` check that `amount` ≠ 0, check that there is enough `allowance`
    6. `pullERC20AfterCheck`: - transfer `ERC20` from `msg.sender` to contract
    7. `checkERC721BeforePull`: - check that it is `ERC721`, check that owner is `msg.sender`
    8. `pullERC721AfterCheck`: - transfer `ERC721` from `msg.sender` to contract
    9. `checkAndPullByType`: - transfer one of token types from `msg.sender` to contract
    
`DelegateTokenHelpers` library:

1. Contains 5 functions:
    1. `revertOnCallingInvalidFlashloan`: - revert if selector does not match
    2. `revertOnInvalidERC721ReceiverCallback`: - the same
    3. `revertOnInvalidERC721ReceiverCallback`: - the same
    4. `revertOldExpiry`: - revert if `expiry` expired
    5. `delegateIdNoRevert`: - hash `caller` and `salt`
    

`DelegateToken` contract:

1. Contains 29 functions:
    1. `supportsInterface`: - supported interfaces
    2. `onERC1155BatchReceived`: - revert 
    3. `onERC721Received`: - revert if contract is not `operator`, else return selector
    4. `onERC1155Received`: - revert on custom reentrancy check fail else return selector
    5. `balanceOf`: - get balance of `delegateTokenHolder` if not `address(0)`
    6. `ownerOf`: - return `to` from registry for specific `delegateTokenId`
    7. `getApproved`: - return `approved` address revert if not minted
    8. `isApprovedForAll`: - return if  `accountOperator`
    9. `approve`: - store approved `spender`, revert if not minted or not operator
    10. `setApprovalForAll`: - set `accountOperator`
    11. `name`: - constant
    12. `symbol`: - constant
    13. `transferFrom`: - transfer `delegateTokenId` with underlying token
    14. `isApprovedOrOwner`: - check if it is `“owner"`or `operator` or `approved`
    15. `getDelegateId`: - get `delegateTokenId` revert if not available
    16. `burnAuthorizedCallback`: -  revert if caller is not `principalToken` or delegate not authorized burn 
    17. `mintAuthorizedCallback`: - similar
    18. `create`: - transfer one of token types to contract. delegate to `delegateHolder`. mint principal token
    19. `safeTransferFrom`: - call `transferFrom` and check selector callback
    20. `getDelegateInfo`: - build and get `DelegateInfo` from `delegateTokenId`
    21. `extend`: - allow principal or operator  to increase `expiry` if old not expired
    22. `rescind`: - allow delegate( or anyone after expiry) transfer `delegateTokenId` to principal
    23. `tokenURI`: - call `MarketMetadata` for `delegateTokenURI`
    24. `baseURI`: -  call `MarketMetadata` for `delegateTokenBaseURI`
    25. `contractURI`: - call `MarketMetadata` for `delegateTokenContractURI`
    26. `royaltyInfo`: - similar
    27. `withdraw`: - withdraw delegation, burn principal, transfer underlaying back to `msg.sender`
    28. `flashloan`: - flash-loan operation for all token types
    

`PrincipalToken` contract:

1. Contains 5 functions:
    1. `isApprovedOrOwner` : - Call `ERC721`  `_isApprovedOrOwner`
    2. `_checkDelegateTokenCaller`: - check caller is `delegateToken`
    3. `tokenURI`: - Call `MarketMetadata` for `principalTokenURI`
    4. `mint`: - mint. Called by `delegateToken` when authorized
    5. `burn`: - burn. Called by `delegateToken` when authorized

`CreateOffererModifiers` library:

1. store `seaport` address and Stage
2. Contains 2 modifiers:
    1. `onlySeaport`: - caller is `seaport`
    2. `checkStage`: - reentrancy check + stage change

`CreateOffererHelpers` library:

1. Contains 9 functions:
    1. `processNonce`: - check nonce and increment if correct
    2. `updateTransientState`: - fulfill `TransientState` struct
    3. `createAndValidateDelegateTokenId`: - Call `create` on `IDelegateToken`. And check correct `delegateId`
    4. `calculateExpiry`: - return absolute `expiry` for both types
    5. `processSpentItems`: - build `offer` and `consideration` from `minimumReceived` and `maximumSpent`
    6. `calculateOrderHash`: - hash `order` with with `tokenType`
    7. `calculateOrderHashAndId`: - get `delegateTokenId` from `calculateOrderHash`
    8. `verifyCreate`: - match hash to context
    9. `validateCreateOrderHash`: - match provided hash to actual

`CreateOfferer` contract:

1. Seaport iteraction
  
</details>

## Mechanism review

The contract consists of 2 parts, one part is a storage of delegation hashes, and the other part is ERC721 compatible tokens that reflect the ownership of the delegation.

`Delegate Registry`: uses hashing to compactly store the delegation. And also hash functions for calculating a unique location in storage. It also contains functions that check the hash based on the location and `from` address.

`Delegate Token`: Deposits all assets, in return issues an ERC721 token, which confirms the ownership of the delegation, for a certain period of time.

`PrincipalToken`: Depends on `Delegate Token`, cannot be called on its own. It is an ERC721 token that confirms the right to claim deposited assets after expiration.

`CreateOfferer`: Integration with seaport as specified in [documentation](https://github.com/ProjectOpenSea/seaport/blob/main/docs/SeaportDocumentation.md#contract-orders). When selling, the asset turns into a `Delegate Token` and is assigned to the buyer, the seller receives a `PrincipalToken`.

`Delegate Token` in its work relies entirely on `Delegate Registry`, which must reliably guarantee the authenticity and confirmation of the delegation.

## Codebase quality analysis

In general, the quality of the code base is quite high. The huge number of comments in NatSpec makes it very easy to determine what a particular function is intended for.

The downside is the use of assembler for gas optimization, which is not comparable to the damage it causes to code readability.

## Centralization risks

There is no risk of centralization since all rights are divided between the `Delegate Token` and the `PrincipalToken`. The only exception is `CreateOfferer`, which relies on the `seaport` address, which is immutable, but it is possible that the contract address will change in the future, it would be useful to add a function that allows you to change the address if necessary

## Systemic risks

The contract is used to delegate all types of tokens (`ERC20`, `ERC721`, `ERC1155`), but does not take into account that some tokens do not follow the standards.

Contracts are programmed for version ^0.8.21, by default the compiler will use version 0.8.21, which is very recent and may contain undetected vulnerabilities, as well as compatibility problems with different L2 chains.

## New insights and learning from this audit

I learned about `CreateOfferer` seaport integration, all other concepts were well known to me.

### Time spent:
33 hours