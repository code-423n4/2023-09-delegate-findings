
### Gas Optimization Issues
|Title|Issue|Instances|Total Gas Saved|
|-|:-|:-:|:-:|
|[G-1] Avoid unnecessary storage updates | [Avoid unnecessary storage updates](#avoid-unnecessary-storage-updates) | 1 | 800 |
|[G-2] Check Arguments Early | [Check Arguments Early](#check-arguments-early) | 5 | - |
|[G-3] Inefficient Parameter Storage | [Inefficient Parameter Storage](#inefficient-parameter-storage) | 3 | 150 |
|[G-4] Using Storage Instead of Memory for structs/arrays Saves Gas | [Using Storage Instead of Memory for structs/arrays Saves Gas](#using-storage-instead-of-memory-for-structsarrays-saves-gas) | 4 | 16800 |
|[G-5] Short-circuit rules can be used to optimize some gas usage | [Short-circuit rules can be used to optimize some gas usage](#short-circuit-rules-can-be-used-to-optimize-some-gas-usage) | 1 | 2100 |
|[G-6] Unnecessary Casting of Variables | [Unnecessary Casting of Variables](#unnecessary-casting-of-variables) | 9 | - |
|[G-7] Redundant Contract Existence Check in Consecutive External Calls | [Redundant Contract Existence Check in Consecutive External Calls](#redundant-contract-existence-check-in-consecutive-external-calls) | 10 | 1000 |


Total: 7 issues

#

## Short-circuit rules can be used to optimize some gas usage
- Severity: Gas Optimization
- Confidence: Medium
- Total Gas Saved: 2100

### Description
Some conditions may be reordered to save an SLOAD (2100 gas), as we avoid reading state variables when the first part of the condition fails (with &&), or succeeds (with ||). For instance, consider a scenario where you have a `stateVariable` (a variable stored in contract storage) and a `localVariable` (a variable in memory). 

If you have a condition like `stateVariable > 0 && localVariable > 0`, if `localVariable > 0` is false, the Solidity runtime will still execute `stateVariable > 0`, which costs an SLOAD operation (2100 gas). However, if you reorder the condition to `localVariable > 0 && stateVariable > 0`, the `stateVariable > 0` check won't happen if `localVariable > 0` is false, saving you the SLOAD gas cost.

Similarly, for the `||` operator, if you have a condition like `stateVariable > 0 || localVariable > 0`, and `stateVariable > 0` is true, the Solidity runtime will still execute `localVariable > 0`. But if you reorder the condition to `localVariable > 0 || stateVariable > 0`, and `localVariable > 0` is true, the `stateVariable > 0` check won't happen, again saving you the SLOAD gas cost.

This detector checks for such conditions in the contract and reports if any condition could be optimized by taking advantage of the short-circuiting behavior of `&&` and `||`.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
#

```
File: src/libraries/DelegateTokenTransferHelpers.sol

78    erc1155Pulled.flag == ERC1155_PULLED && address(this) == operator
```


```
 // @audit: Switch address(this) == operator && erc1155Pulled.flag == ERC1155_PULLED 
```
[https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenTransferHelpers.sol#L78](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenTransferHelpers.sol#L78)


</details>

# 


## Avoid unnecessary storage updates
- Severity: Gas Optimization
- Confidence: High
- Total Gas Saved: 800

### Description
Avoid updating storage when the value hasn't changed. If the old value is equal to the new value, not re-storing the value will avoid a `SSTORE` operation (costing 2900 gas), potentially at the expense of a `SLOAD` operation (2100 gas) or a `WARMACCESS` operation (100 gas).

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
#
The function `setApprovalForAll()` changes the state variable without first verifying if the values are different.


```
File: src/DelegateToken.sol

145    accountOperator[msg.sender][operator] = approved
```

[https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L145](https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L145)


</details>

# 



## Unnecessary Casting of Variables
- Severity: Gas Optimization
- Confidence: High

### Description
This detector scans for instances where a variable is casted to its own type. This is unnecessary and can be safely removed to improve code readability.

<details>

<summary>
There are 9 instances of this issue:

</summary>

###
#

```
File: lib/delegate-registry/src/DelegateRegistry.sol

145    _writeDelegation(location, Storage.POSITIONS_AMOUNT, uint256(0))
```
Unnecessary cast: `uint256(0)` it cast to the same type.<br>
[https://github.com/code-423n4/2023-09-delegate/blob/main/lib/delegate-registry/src/DelegateRegistry.sol#L145](https://github.com/code-423n4/2023-09-delegate/blob/main/lib/delegate-registry/src/DelegateRegistry.sol#L145)


#

```
File: src/libraries/CreateOffererLib.sol

361    consideration[0] = ReceivedItem({
362                itemType: maximumSpent[0].itemType,
363                token: maximumSpent[0].token,
364                identifier: maximumSpent[0].identifier,
365                amount: maximumSpent[0].amount,
366                recipient: payable(address(this))
367            })
```
Unnecessary cast: `payable(address(this))` it cast to the same type.<br>
[https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L361-L367](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L361-L367)


#

```
File: src/CreateOfferer.sol

121    !IERC20(erc20Order.info.tokenContract).approve(address(delegateToken), erc20Order.amount)
```
Unnecessary cast: `address(delegateToken)` it cast to the same type.<br>
[https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L121](https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L121)


#

```
File: src/CreateOfferer.sol

147    IERC1155(erc1155Order.info.tokenContract).setApprovalForAll(address(delegateToken), true)
```
Unnecessary cast: `address(delegateToken)` it cast to the same type.<br>
[https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L147](https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L147)


#

```
File: src/CreateOfferer.sol

99    IERC721(erc721Order.info.tokenContract).setApprovalForAll(address(delegateToken), true)
```
Unnecessary cast: `address(delegateToken)` it cast to the same type.<br>
[https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L99](https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L99)


#

```
File: src/CreateOfferer.sol

162    IERC1155(erc1155Order.info.tokenContract).setApprovalForAll(address(delegateToken), false)
```
Unnecessary cast: `address(delegateToken)` it cast to the same type.<br>
[https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L162](https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L162)


#

```
File: src/CreateOfferer.sol

138    IERC20(erc20Order.info.tokenContract).allowance(address(this), address(delegateToken)) != 0
```
Unnecessary cast: `address(delegateToken)` it cast to the same type.<br>
[https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L138](https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L138)


#

```
File: src/CreateOfferer.sol

114    IERC721(erc721Order.info.tokenContract).setApprovalForAll(address(delegateToken), false)
```
Unnecessary cast: `address(delegateToken)` it cast to the same type.<br>
[https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L114](https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L114)


#

```
File: src/libraries/DelegateTokenRegistryHelpers.sol

205    IDelegateRegistry(delegateRegistry).delegateERC20(
206                    from, underlyingContract, underlyingRights, calculateDecreasedAmount(delegateRegistry, registryHash, underlyingAmount)
207                ) == bytes32(registryHash)
208                    && IDelegateRegistry(delegateRegistry).delegateERC20(
209                        to, underlyingContract, underlyingRights, calculateIncreasedAmount(delegateRegistry, newRegistryHash, underlyingAmount)
210                    ) == newRegistryHash
```
Unnecessary cast: `bytes32(registryHash)` it cast to the same type.<br>
[https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenRegistryHelpers.sol#L205-L210](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenRegistryHelpers.sol#L205-L210)


</details>

# 


## Redundant Contract Existence Check in Consecutive External Calls
- Severity: Gas Optimization
- Confidence: Medium
- Total Gas Saved: 1000

### Description
This detector identifies instances where there are consecutive external calls to the same contract, where the subsequent calls could use a low-level call to save gas.

Note: This detector only triggers if the function call does not return any value. 

Prior to 0.8.10, the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent Solidity versions the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low-level calls never check for contract existence. 

<details>

<summary>
There are 10 instances of this issue:

</summary>

###
#

```
File: src/CreateOfferer.sol

78    Helpers.verifyCreate(delegateToken, offer[0].identifier, transientState.receivers, consideration[0], context)
```
This call could be replaced with a low-level call because the contract `CreateOffererHelpers` has already been checked in <br>`Line: 77    Helpers.processNonce(nonce, contractNonce)`<br>
[https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L78](https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L78)


#

```
File: src/CreateOfferer.sol

100    Helpers.createAndValidateDelegateTokenId(
101                    delegateToken,
102                    createOrderHashAsTokenId,
103                    IDelegateTokenStructs.DelegateInfo({
104                        principalHolder: erc721Order.info.targetToken == Enums.TargetToken.principal ? targetTokenReceiver : transientState.receivers.fulfiller,
105                        tokenType: tokenType,
106                        delegateHolder: erc721Order.info.targetToken == Enums.TargetToken.delegate ? targetTokenReceiver : transientState.receivers.fulfiller,
107                        amount: 0,
108                        tokenContract: erc721Order.info.tokenContract,
109                        tokenId: erc721Order.tokenId,
110                        rights: erc721Order.info.rights,
111                        expiry: Helpers.calculateExpiry(erc721Order.info.expiryType, erc721Order.info.expiryLength)
112                    })
113                )
```
This call could be replaced with a low-level call because the contract `CreateOffererHelpers` has already been checked in <br>`Line: 98    Helpers.validateCreateOrderHash(targetTokenReceiver, createOrderHashAsTokenId, abi.encode(erc721Order), tokenType)`<br>
[https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L100-L113](https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L100-L113)


#

```
File: src/CreateOfferer.sol

114    IERC721(erc721Order.info.tokenContract).setApprovalForAll(address(delegateToken), false)
```
This call could be replaced with a low-level call because the contract `IERC721` has already been checked in <br>`Line: 99    IERC721(erc721Order.info.tokenContract).setApprovalForAll(address(delegateToken), true)`<br>
[https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L114](https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L114)


#

```
File: src/CreateOfferer.sol

162    IERC1155(erc1155Order.info.tokenContract).setApprovalForAll(address(delegateToken), false)
```
This call could be replaced with a low-level call because the contract `IERC1155` has already been checked in <br>`Line: 147    IERC1155(erc1155Order.info.tokenContract).setApprovalForAll(address(delegateToken), true)`<br>
[https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L162](https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L162)


#

```
File: src/DelegateToken.sol

138    StorageHelpers.revertNotOperator(accountOperator, delegateTokenHolder)
```
This call could be replaced with a low-level call because the contract `DelegateTokenStorageHelpers` has already been checked in <br>`Line: 136    StorageHelpers.revertNotMinted(registryHash, delegateTokenId)`<br>
[https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L138](https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L138)


#

```
File: src/DelegateToken.sol

168    StorageHelpers.revertNotApprovedOrOperator(accountOperator, delegateTokenInfo, from, delegateTokenId)
```
This call could be replaced with a low-level call because the contract `DelegateTokenStorageHelpers` has already been checked in <br>`Line: 164    StorageHelpers.revertNotMinted(registryHash, delegateTokenId)`<br>
[https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L168](https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L168)


#

```
File: src/DelegateToken.sol

302    StorageHelpers.incrementBalance(balances, delegateInfo.delegateHolder)
```
This call could be replaced with a low-level call because the contract `DelegateTokenStorageHelpers` has already been checked in <br>`Line: 301    StorageHelpers.revertAlreadyExisted(delegateTokenInfo, delegateTokenId)`<br>
[https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L302](https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L302)


#

```
File: src/DelegateToken.sol

331    StorageHelpers.writeExpiry(delegateTokenInfo, delegateTokenId, newExpiry)
```
This call could be replaced with a low-level call because the contract `DelegateTokenStorageHelpers` has already been checked in <br>`Line: 326    StorageHelpers.revertNotMinted(delegateTokenInfo, delegateTokenId)`<br>
[https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L331](https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L331)


#

```
File: src/DelegateToken.sol

357    StorageHelpers.revertNotMinted(registryHash, delegateTokenId)
```
This call could be replaced with a low-level call because the contract `DelegateTokenStorageHelpers` has already been checked in <br>`Line: 355    StorageHelpers.writeRegistryHash(delegateTokenInfo, delegateTokenId, bytes32(StorageHelpers.ID_USED))`<br>
[https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L357](https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L357)


#

```
File: src/DelegateToken.sol

396    TransferHelpers.pullERC721AfterCheck(info.tokenContract, info.tokenId)
```
This call could be replaced with a low-level call because the contract `DelegateTokenTransferHelpers` has already been checked in <br>`Line: 395    TransferHelpers.checkERC721BeforePull(info.amount, info.tokenContract, info.tokenId)`<br>
[https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L396](https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L396)


</details>

# 

## Check Arguments Early
- Severity: Gas Optimization
- Confidence: High

### Description
Checks that require() or revert() statements that check input arguments are at the top of the function. Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a gas load in a function that may ultimately revert in the unhappy case.

<details>

<summary>
There are 5 instances of this issue:

</summary>

###
#

```
File: src/CreateOfferer.sol

32    parameters.principalToken == address(0)
```

[https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L32](https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L32)


#

```
File: src/DelegateToken.sol

299    delegateInfo.delegateHolder == address(0)
```

[https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L299](https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L299)


#

```
File: src/PrincipalToken.sol

23    setMarketMetadata == address(0)
```

[https://github.com/code-423n4/2023-09-delegate/blob/main/src/PrincipalToken.sol#L23](https://github.com/code-423n4/2023-09-delegate/blob/main/src/PrincipalToken.sol#L23)


#

```
File: src/libraries/CreateOffererLib.sol

297    context.length != 160
```

[https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L297](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L297)


#

```
File: src/libraries/DelegateTokenTransferHelpers.sol

72    erc1155Pulled.flag == ERC1155_PULLED
```

[https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenTransferHelpers.sol#L72](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenTransferHelpers.sol#L72)


</details>

# 
nefficient Parameter Storage
- Severity: Gas Optimization
- Confidence: Medium
- Total Gas Saved: 150

### Description
When passing function parameters, using the `calldata` area instead of `memory` can improve gas efficiency. Calldata is a read-only area where function arguments and external function calls' parameters are stored.

By using `calldata` for function parameters, you avoid unnecessary gas costs associated with copying data from `calldata` to memory. This is particularly beneficial when the parameter is read-only and doesn't require modification within the contract.

Using `calldata` for function parameters can help optimize gas usage, especially when making external function calls or when the parameter values are provided externally and don't need to be stored persistently within the contract.

<details>

<summary>
There are 3 instances of this issue:

</summary>

###
#

```
File: src/CreateOfferer.sol

29    Structs.Parameters memory parameters
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L29](https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L29)


#

```
File: src/DelegateToken.sol

52    Structs.DelegateTokenParameters memory parameters
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L52](https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L52)


#

```
File: src/libraries/CreateOffererLib.sol

204    CreateOffererStructs.Context memory decodedContext
```
 should be declared as `calldata` instead 

[https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L204](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L204)


</details>

# 


## Using Storage Instead of Memory for structs/arrays Saves Gas
- Severity: Gas Optimization
- Confidence: Medium
- Total Gas Saved: 16800

### Description
When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct / array to be read from storage. This incurs a Gcoldsload (2100 gas) for each field of the struct / array. If the fields are read from the new memory variable, they incur an additional MLOAD, which is more expensive than a simple stack read.

Instead of declaring the variable with the memory keyword, a more cost-effective approach is to declare the variable with the storage keyword. In this case, you can cache any fields that need to be re-read in stack variables. This approach significantly reduces gas costs, as you only incur the Gcoldsload for the fields actually read.

The only scenario where it makes sense to read the whole struct / array into a memory variable is if the full struct / array is being returned by the function or passed to a function that requires memory. Additionally, if the array / struct is being read from another memory array / struct, using a memory variable may be appropriate.

By carefully considering the storage and memory usage in your contract, you can optimize gas costs and improve the efficiency of your smart contract operations.

<details>

<summary>
There are 4 instances of this issue:

</summary>

###
#

```
File: src/CreateOfferer.sol

94    Structs.ERC721Order memory erc721Order = transientState.erc721Order
```
 should be declared as `storage` 

[https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L94](https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L94)


#

```
File: src/CreateOfferer.sol

116    Structs.ERC20Order memory erc20Order = transientState.erc20Order
```
 should be declared as `storage` 

[https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L116](https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L116)


#

```
File: src/CreateOfferer.sol

142    Structs.ERC1155Order memory erc1155Order = transientState.erc1155Order
```
 should be declared as `storage` 

[https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L142](https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L142)


#

```
File: src/libraries/CreateOffererLib.sol

157    CreateOffererStructs.Stage memory cacheStage = stage
```
 should be declared as `storage` 

[https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L157](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L157)


</details>

# 

