### Medium Issues
|Title|Issue|Instances|
|-|:-|:-:|
|[M-1] Calls _mint | [Calls _mint](#calls-_mint) | 1 |
|[M-2] Zero-value transfers may revert | [Zero-value transfers may revert](#zero-value-transfers-may-revert) | 2 |

Total: 2 issues


### Low Issues
|Title|Issue|Instances|
|-|:-|:-:|
|[L-1] Array out of bounds accesses | [Array out of bounds accesses](#array-out-of-bounds-accesses) | 2 |
|[L-2] Missing contract-existence checks before low-level calls | [Missing contract-existence checks before low-level calls](#missing-contract-existence-checks-before-low-level-calls) | 1 |
|[L-3] NFT does not handle hard forks | [NFT does not handle hard forks](#nft-does-not-handle-hard-forks) | 1 |
|[L-4] Use of transferFrom() rather than safeTransferFrom() for NFTs | [Use of transferFrom() rather than safeTransferFrom() for NFTs](#use-of-transferfrom-rather-than-safetransferfrom-for-nfts) | 1 |

Total: 4 issues

### Non-Critical Issues
|Title|Issue|Instances|
|-|:-|:-:|
|[N-1] Library Should Be Defined in Separate Files From Their Usage | [Library Should Be Defined in Separate Files From Their Usage](#library-should-be-defined-in-separate-files-from-their-usage) | 5 |
|[N-2] Do not calculate constants | [Do not calculate constants](#do-not-calculate-constants) | 2 |
|[N-3] Constants in Comparisons on Left Side | [Constants in Comparisons on Left Side](#constants-in-comparisons-on-left-side) | 23 |
|[N-4] Cyclomatic complexity | [Cyclomatic complexity](#cyclomatic-complexity) | 1 |
|[N-5] Avoid double casting | [Avoid double casting](#avoid-double-casting) | 3 |
|[N-6] Else block not required | [Else block not required](#else-block-not-required) | 2 |
|[N-7] Functions that alter state should emit events | [Functions that alter state should emit events](#functions-that-alter-state-should-emit-events) | 2 |
|[N-8] Top level declarations should be separated by two blank lines | [Top level declarations should be separated by two blank lines](#top-level-declarations-should-be-separated-by-two-blank-lines) | 2 |


Total: 8 issues



### Note 
I reported those medium in the qa report becuase most of judges consider as them low issues 

## Calls _mint
- Severity: Medium
- Confidence: High

### Description
The _mint function is often used to create new tokens in a Solidity smart contract.However, if the function is not implemented correctly, it can introduce vulnerabilities such as integer overflow and underflow, reentrancy, and other types of attacks.To address these issues, the safeMint function was introduced as part of the OpenZeppelin library.The safeMint function includes additional checks to prevent potential attacks, making it a safer alternative to _mint.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
#

```
File: src/PrincipalToken.sol

35    _mint(to, id)
```
 use safeMint instead.

[https://github.com/code-423n4/2023-09-delegate/blob/main/src/PrincipalToken.sol#L35](https://github.com/code-423n4/2023-09-delegate/blob/main/src/PrincipalToken.sol#L35)


</details>

# 


## Zero-value transfers may revert
- Severity: Medium
- Confidence: High

### Note 
I reported only on issues that was missing in the winning bot.

### Description
This detector identifies instances where zero-valued ERC20 token transfers are made. Even though EIP-20 states that zero-valued transfers must be treated as normal transfers and events must be triggered, some tokens may revert if this is attempted, which can cause transactions that involve other tokens (such as batch operations) to fully revert. Therefore, it may be safer and more gas-efficient to skip the transfer if the amount is zero.

<details>

<summary>
There are 2 instances of this issue:

</summary>

###
#

```
File: src/DelegateToken.sol

375    SafeERC20.safeTransfer(IERC20(underlyingContract), msg.sender, erc20UnderlyingAmount)
```
This variable might be zero: `erc20UnderlyingAmount` 

[https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L375](https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L375)


#

```
File: src/DelegateToken.sol

399    SafeERC20.safeTransfer(IERC20(info.tokenContract), info.receiver, info.amount)
```
This variable might be zero: `info.amount` 

[https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L399](https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L399)


#

</details>

# 


## Array out of bounds accesses
- Severity: Low
- Confidence: Medium

### Description
If the lengths of arrays are not checked before they're accessed, user operations may not be executed properly due to the potential issue of accessing an array out of its bounds. In Solidity, accessing an array beyond its boundaries can cause a runtime error known as an out-of-bounds exception. This error arises when attempting to read or write to an element of an array that does not exist. Therefore, it is critical to avoid out-of-bounds exceptions while accessing arrays in Solidity code to prevent unexpected halts and possible loss of funds.

<details>

<summary>
There are 2 instances of this issue:

</summary>

###
#

```
File: src/CreateOfferer.sol

52    SpentItem[] calldata minimumReceived
```
 is accessed on 
```
File: src/CreateOfferer.sol

61    Helpers.updateTransientState(transientState, fulfiller, minimumReceived[0], maximumSpent[0], decodedContext)
```
 index that might be out-of-bounds

[https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L52](https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L52)


#

```
File: src/CreateOfferer.sol

71    SpentItem[] calldata offer
```
 is accessed on 
```
File: src/CreateOfferer.sol

78    Helpers.verifyCreate(delegateToken, offer[0].identifier, transientState.receivers, consideration[0], context)
```
 index that might be out-of-bounds

[https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L71](https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L71)


</details>

# 


## Missing contract-existence checks before low-level calls
- Severity: Low
- Confidence: High

### Description
Low-level calls return success if there is no code present at the specified address. In addition to the zero-address checks, add a check to verify that `<address>.code.length > 0` or use the `extcodesize` assembly operation to verify the presence of contract code at the specified address. Both these methods ensure the existence of a contract before making a low-level call.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
#

```
File: lib/delegate-registry/src/DelegateRegistry.sol

37    (success, results[i]) = address(this).delegatecall(data[i])
```

[https://github.com/code-423n4/2023-09-delegate/blob/main/lib/delegate-registry/src/DelegateRegistry.sol#L37](https://github.com/code-423n4/2023-09-delegate/blob/main/lib/delegate-registry/src/DelegateRegistry.sol#L37)


</details>

# 


## NFT does not handle hard forks
- Severity: Low
- Confidence: High

### Description
When there are hard forks, users often have to go through many hoops to ensure that they control ownership on every fork. Consider adding require(1 == chain.chainId), or the chain ID of whichever chain you prefer, to the functions below, or at least include the chain ID in the URI, so that there is no confusion about which chain is the owner of the NFT.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
#

```
File: src/DelegateToken.sol

227    function tokenURI(uint256 delegateTokenId) external view returns (string memory) 
```

[https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L227-L236](https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L227-L236)



</details>

# 


## Use of transferFrom() rather than safeTransferFrom() for NFTs
- Severity: Low
- Confidence: High

### Note 
I repot only on one issues that was missing in the wining bot.

### Description

Use of `transferFrom()` rather than `safeTransferFrom()` for NFTs in will lead to the loss of NFTs
The EIP-721 standard says the following about `transferFrom()`:
```solidity
    /// @notice Transfer ownership of an NFT -- THE CALLER IS RESPONSIBLE
    ///  TO CONFIRM THAT `_to` IS CAPABLE OF RECEIVING NFTS OR ELSE
    ///  THEY MAY BE PERMANENTLY LOST
    /// @dev Throws unless `msg.sender` is the current owner, an authorized
    ///  operator, or the approved address for this NFT. Throws if `_from` is
    ///  not the current owner. Throws if `_to` is the zero address. Throws if
    ///  `_tokenId` is not a valid NFT.
    /// @param _from The current owner of the NFT
    /// @param _to The new owner
    /// @param _tokenId The NFT to transfer
    function transferFrom(address _from, address _to, uint256 _tokenId) external payable;
```
https://github.com/ethereum/EIPs/blob/78e2c297611f5e92b6a5112819ab71f74041ff25/EIPS/eip-721.md?plain=1#L103-L113
Code must use the `safeTransferFrom()` flavor if it hasn't otherwise verified that the receiving address can handle it

    

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
#

```
File: src/DelegateToken.sol

345    transferFrom(
346                RegistryHelpers.loadTokenHolder(delegateRegistry, StorageHelpers.readRegistryHash(delegateTokenInfo, delegateTokenId)),
347                IERC721(principalToken).ownerOf(delegateTokenId),
348                delegateTokenId
349            )
```

[https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L345-L349](https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L345-L349)



</details>

# 


## Do not calculate constants
- Severity: Non-Critical
- Confidence: High

### Description
Due to how constant variables are implemented in Solidity (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas. While in most cases, the compiler will optimize these computations away, it is considered a best practice to write code that does not rely on the compiler optimization.

<details>

<summary>
There are 2 instances of this issue:

</summary>

###
#

```
File: lib/delegate-registry/src/libraries/RegistryStorage.sol

20    uint256 internal constant CLEAN_FIRST8_BYTES_ADDRESS = 0xffffffffffffffff << 96
```

[https://github.com/code-423n4/2023-09-delegate/blob/main/lib/delegate-registry/src/libraries/RegistryStorage.sol#L20](https://github.com/code-423n4/2023-09-delegate/blob/main/lib/delegate-registry/src/libraries/RegistryStorage.sol#L20)


#

```
File: lib/delegate-registry/src/libraries/RegistryStorage.sol

23    uint256 internal constant CLEAN_PACKED8_BYTES_ADDRESS = 0xffffffffffffffff << 160
```

[https://github.com/code-423n4/2023-09-delegate/blob/main/lib/delegate-registry/src/libraries/RegistryStorage.sol#L23](https://github.com/code-423n4/2023-09-delegate/blob/main/lib/delegate-registry/src/libraries/RegistryStorage.sol#L23)


</details>

# 


## Library Should Be Defined in Separate Files From Their Usage
- Severity: Non-Critical
- Confidence: High

### Description
Library should be defined in separate files, so that it's easier for future projects to import them, and to avoid duplication later on if they need to be used elsewhere in the project.

<details>

<summary>
There are 5 instances of this issue:

</summary>

###
#

```
File: src/libraries/CreateOffererLib.sol

10    library CreateOffererErrors 
```

[https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L10-L31](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L10-L31)


#

```
File: src/libraries/CreateOffererLib.sol

33    library CreateOffererEnums 
```

[https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L33-L62](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L33-L62)


#

```
File: src/libraries/CreateOffererLib.sol

64    library CreateOffererStructs 
```

[https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L64-L133](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L64-L133)


#

```
File: src/libraries/CreateOffererLib.sol

176    library CreateOffererHelpers 
```

[https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L176-L401](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L176-L401)


#

```
File: src/libraries/DelegateTokenLib.sol

42    library DelegateTokenErrors 
```

[https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenLib.sol#L42-L88](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenLib.sol#L42-L88)


</details>

# 


## Avoid double casting
- Severity: Non-Critical
- Confidence: High

### Note 
report only on issues that were mising by the wining bot.

### Description
This detector identifies instances where values are double casted in Solidity. Double casting can introduce unexpected truncations and/or rounding issues. Additionally, it reduces the readability of the code and can make it harder to maintain. It's recommended to avoid double casting to ensure clearer, safer, and more maintainable smart contracts.

<details>

<summary>
There are 3 instances of this issue:

</summary>

###



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

[https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L361-L367](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L361-L367)





#

```
File: src/libraries/DelegateTokenStorageHelpers.sol

46    address approved = address(uint160(delegateTokenInfo[delegateTokenId][PACKED_INFO_POSITION] >> 96))
```

[https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenStorageHelpers.sol#L46](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenStorageHelpers.sol#L46)





#

```
File: src/libraries/DelegateTokenStorageHelpers.sol

128    return address(uint160(delegateTokenInfo[delegateTokenId][PACKED_INFO_POSITION] >> 96))
```

[https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenStorageHelpers.sol#L128](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenStorageHelpers.sol#L128)


</details>

# 



## Constants in Comparisons on Left Side
- Severity: Non-Critical
- Confidence: High

### Note 
I only reported on instances that were missing from the winning bot. It seems the winning bot missed all the constants that aren't literal numerics, such as ERC1155_PULLED.

### Description
Where constants are not placed on the left side in comparison operations. In many programming languages, it is considered a best practice to place constants on the left side of comparisons to improve code readability and reduce the likelihood of accidental assignment bugs.

<details>

<summary>
There are 23 instances of this issue:

</summary>

###
#

```
File: lib/delegate-registry/src/DelegateRegistry.sol

52    rights != ""
```
 placed the const on the left

[https://github.com/code-423n4/2023-09-delegate/blob/main/lib/delegate-registry/src/DelegateRegistry.sol#L52](https://github.com/code-423n4/2023-09-delegate/blob/main/lib/delegate-registry/src/DelegateRegistry.sol#L52)


#

```
File: lib/delegate-registry/src/DelegateRegistry.sol

71    rights != ""
```
 placed the const on the left

[https://github.com/code-423n4/2023-09-delegate/blob/main/lib/delegate-registry/src/DelegateRegistry.sol#L71](https://github.com/code-423n4/2023-09-delegate/blob/main/lib/delegate-registry/src/DelegateRegistry.sol#L71)


#

```
File: lib/delegate-registry/src/DelegateRegistry.sol

91    rights != ""
```
 placed the const on the left

[https://github.com/code-423n4/2023-09-delegate/blob/main/lib/delegate-registry/src/DelegateRegistry.sol#L91](https://github.com/code-423n4/2023-09-delegate/blob/main/lib/delegate-registry/src/DelegateRegistry.sol#L91)




#

```
File: lib/delegate-registry/src/DelegateRegistry.sol

111    rights != ""
```
 placed the const on the left

[https://github.com/code-423n4/2023-09-delegate/blob/main/lib/delegate-registry/src/DelegateRegistry.sol#L111](https://github.com/code-423n4/2023-09-delegate/blob/main/lib/delegate-registry/src/DelegateRegistry.sol#L111)




#

```
File: lib/delegate-registry/src/DelegateRegistry.sol

136    rights != ""
```
 placed the const on the left

[https://github.com/code-423n4/2023-09-delegate/blob/main/lib/delegate-registry/src/DelegateRegistry.sol#L136](https://github.com/code-423n4/2023-09-delegate/blob/main/lib/delegate-registry/src/DelegateRegistry.sol#L136)


#

```
File: lib/delegate-registry/src/DelegateRegistry.sol

168    rights == ""
```
 placed the const on the left

[https://github.com/code-423n4/2023-09-delegate/blob/main/lib/delegate-registry/src/DelegateRegistry.sol#L168](https://github.com/code-423n4/2023-09-delegate/blob/main/lib/delegate-registry/src/DelegateRegistry.sol#L168)


#

```
File: lib/delegate-registry/src/DelegateRegistry.sol

181    rights == ""
```
 placed the const on the left

[https://github.com/code-423n4/2023-09-delegate/blob/main/lib/delegate-registry/src/DelegateRegistry.sol#L181](https://github.com/code-423n4/2023-09-delegate/blob/main/lib/delegate-registry/src/DelegateRegistry.sol#L181)


#

```
File: lib/delegate-registry/src/DelegateRegistry.sol

197    rights == ""
```
 placed the const on the left

[https://github.com/code-423n4/2023-09-delegate/blob/main/lib/delegate-registry/src/DelegateRegistry.sol#L197](https://github.com/code-423n4/2023-09-delegate/blob/main/lib/delegate-registry/src/DelegateRegistry.sol#L197)


#

```
File: lib/delegate-registry/src/DelegateRegistry.sol

215    rights == ""
```
 placed the const on the left

[https://github.com/code-423n4/2023-09-delegate/blob/main/lib/delegate-registry/src/DelegateRegistry.sol#L215](https://github.com/code-423n4/2023-09-delegate/blob/main/lib/delegate-registry/src/DelegateRegistry.sol#L215)


#

```
File: lib/delegate-registry/src/DelegateRegistry.sol

234    rights == ""
```
 placed the const on the left

[https://github.com/code-423n4/2023-09-delegate/blob/main/lib/delegate-registry/src/DelegateRegistry.sol#L234](https://github.com/code-423n4/2023-09-delegate/blob/main/lib/delegate-registry/src/DelegateRegistry.sol#L234)






#

```
File: src/libraries/DelegateTokenTransferHelpers.sol

78    erc1155Pulled.flag == ERC1155_PULLED
```
 placed the const on the left

[https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenTransferHelpers.sol#L78](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenTransferHelpers.sol#L78)


#

```
File: src/libraries/DelegateTokenStorageHelpers.sol

172    registryHash == ID_USED
```
 placed the const on the left

[https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenStorageHelpers.sol#L172](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenStorageHelpers.sol#L172)


#

```
File: src/libraries/DelegateTokenStorageHelpers.sol

172    registryHash == ID_AVAILABLE
```
 placed the const on the left

[https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenStorageHelpers.sol#L172](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenStorageHelpers.sol#L172)


#

```
File: src/libraries/DelegateTokenStorageHelpers.sol

179    uint256(registryHash) == ID_AVAILABLE
```
 placed the const on the left

[https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenStorageHelpers.sol#L179](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenStorageHelpers.sol#L179)



#

```
File: src/libraries/DelegateTokenStorageHelpers.sol

179    uint256(registryHash) == ID_USED
```
 placed the const on the left

[https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenStorageHelpers.sol#L179](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenStorageHelpers.sol#L179)


#


```
File: src/libraries/DelegateTokenStorageHelpers.sol

118    delegateTokenInfo[delegateTokenId][REGISTRY_HASH_POSITION] == ID_AVAILABLE
```
 placed the const on the left

[https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenStorageHelpers.sol#L118](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenStorageHelpers.sol#L118)


#

```
File: src/libraries/DelegateTokenStorageHelpers.sol

98    principalBurnAuthorization.flag == BURN_AUTHORIZED
```
 placed the const on the left

[https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenStorageHelpers.sol#L98](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenStorageHelpers.sol#L98)


#

```
File: src/libraries/DelegateTokenStorageHelpers.sol

106    principalMintAuthorization.flag == MINT_AUTHORIZED
```
 placed the const on the left

[https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenStorageHelpers.sol#L106](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenStorageHelpers.sol#L106)


#

```
File: src/libraries/DelegateTokenStorageHelpers.sol

85    principalMintAuthorization.flag == MINT_NOT_AUTHORIZED
```
 placed the const on the left

[https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenStorageHelpers.sol#L85](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenStorageHelpers.sol#L85)


#

```
File: src/libraries/DelegateTokenTransferHelpers.sol

62    pullAmount == 0
```
 placed the const on the left

[https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenTransferHelpers.sol#L62](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenTransferHelpers.sol#L62)





#

```
File: src/libraries/DelegateTokenTransferHelpers.sol

63    erc1155Pulled.flag == ERC1155_NOT_PULLED
```
 placed the const on the left

[https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenTransferHelpers.sol#L63](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenTransferHelpers.sol#L63)



#

```
File: src/libraries/DelegateTokenTransferHelpers.sol

72    erc1155Pulled.flag == ERC1155_PULLED
```
 placed the const on the left

[https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenTransferHelpers.sol#L72](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenTransferHelpers.sol#L72)


#


```
File: src/libraries/DelegateTokenStorageHelpers.sol

73    principalBurnAuthorization.flag == BURN_NOT_AUTHORIZED
```
 placed the const on the left

[https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenStorageHelpers.sol#L73](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenStorageHelpers.sol#L73)


</details>

# 



## Top level declarations should be separated by two blank lines
- Severity: Non-Critical
- Confidence: High

### Description
Detects when top-level declarations are not separated by two blank lines.

<details>

<summary>
There are 2 instances of this issue:

</summary>

###
#

```
File: src/PrincipalToken.sol

6    import {ERC721} from "openzeppelin-contracts/contracts/token/ERC721/ERC721.sol";
```

```
File: src/libraries/DelegateTokenStorageHelpers.sol

7    library DelegateTokenStorageHelpers 
```

[https://github.com/code-423n4/2023-09-delegate/blob/main/src/PrincipalToken.sol#L6](https://github.com/code-423n4/2023-09-delegate/blob/main/src/PrincipalToken.sol#L6)


#

```
File: src/libraries/DelegateTokenLib.sol

6    import {IERC721Receiver} from "openzeppelin/token/ERC721/IERC721Receiver.sol";
```

```
File: src/libraries/DelegateTokenStorageHelpers.sol

7    library DelegateTokenStorageHelpers 
```

[https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenLib.sol#L6](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenLib.sol#L6)


</details>

# 



## Cyclomatic complexity
- Severity: Non-Critical
- Confidence: High

### Description
Detects functions with high (> 11) cyclomatic complexity.

<details>

<summary>
There are 1 instances of this issue:

</summary>

###
#

```
File: src/CreateOfferer.sol

89    function transferFrom(address from, address targetTokenReceiver, uint256 createOrderHashAsTokenId) external checkStage(Enums.Stage.transfer, Enums.Stage.ratify) 
```
 has a high cyclomatic complexity (13).

[https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L89-L166](https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L89-L166)


</details>

# 


## Else block not required
- Severity: Non-Critical
- Confidence: High

### Description
This detector identifies unnecessary else blocks in the code. When an if-statement block ends with a return statement, any subsequent else block becomes superfluous and can be eliminated to reduce code complexity. 

Note that this check also applies to single-line else conditions. For instance, in 'return a > b ? a : b', the else condition is not needed.

<details>

<summary>
There are 2 instances of this issue:

</summary>

###
#

```
File: src/libraries/CreateOffererLib.sol

329    if (expiryType == CreateOffererEnums.ExpiryType.absolute) {
330                return expiryLength;
331            } else {
332                revert CreateOffererErrors.InvalidExpiryType(expiryType);
333            }
```
No nees to declare else block.
[https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L329-L333](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L329-L333)


#

```
File: src/libraries/CreateOffererLib.sol

327    if (expiryType == CreateOffererEnums.ExpiryType.relative) {
328                return block.timestamp + expiryLength;
329            } else if (expiryType == CreateOffererEnums.ExpiryType.absolute) {
330                return expiryLength;
331            } else {
332                revert CreateOffererErrors.InvalidExpiryType(expiryType);
333            }
```
No nees to declare else block.
[https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L327-L333](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L327-L333)


</details>

# 


## Functions that alter state should emit events
- Severity: Non-Critical
- Confidence: Medium

### Description
Functions that alter the state of the contract should emit an event to inform external observers of the change.

<details>

<summary>
There are 2 instances of this issue:

</summary>

###
#

```
File: lib/delegate-registry/src/DelegateRegistry.sol

338    function _pushDelegationHashes(address from, address to, bytes32 delegationHash) internal 
```
 The function `_pushDelegationHashes` changes state but does not emit an event.

[https://github.com/code-423n4/2023-09-delegate/blob/main/lib/delegate-registry/src/DelegateRegistry.sol#L338-L341](https://github.com/code-423n4/2023-09-delegate/blob/main/lib/delegate-registry/src/DelegateRegistry.sol#L338-L341)


#

```
File: src/CreateOfferer.sol

89    function transferFrom(address from, address targetTokenReceiver, uint256 createOrderHashAsTokenId) external checkStage(Enums.Stage.transfer, Enums.Stage.ratify) 
```
 The function `transferFrom` changes state but does not emit an event.

[https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L89-L166](https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L89-L166)


</details>

# 