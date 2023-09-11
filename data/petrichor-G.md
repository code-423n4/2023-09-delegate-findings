# gas 

# summary

|       | issue | instance |
|-------|-------|----------|
|[G-01]|Use constants instead of type(uintx).max|7|
|[G-02]|Amounts should be checked for 0 before calling a transfer|2|
|[G-03]|State variables only set in the constructor should be declared immutable|1|
|[G-04]|Use ERC721A instead ERC721|1|
|[G-05]|Can Make The Variable Outside The Loop To Save Gas|2|
|[G-06]| Sort Solidity operations using short-circuit mode|1|
|[G-07]|Structs can be packed into fewer storage slots|2|

## [G-01] Use constants instead of type(uintx).max

The reason for this is that the type(uintX).max expression involves a computation at runtime, whereas a constant is evaluated at compile-time. This means that using type(uintX).max can result in additional gas costs for each transaction that involves the expression.


By using a constant instead of type(uint120).max, we can make our code more efficient and reduce the gas cost of our contract.

It's important to note that using constants can make your code more readable and maintainable, since the value is defined in one place and can be easily updated if necessary. However, constants should be used with caution and only when their value is known at compile-time.

```solidity
213   ? type(uint256).max

215   if (!Ops.or(rights == "", amount == type(uint256).max)) {

217   ? type(uint256).max

234   if (!Ops.or(rights == "", amount == type(uint256).max)) {

236   ? type(uint256).max

372   uint256 cleanUpper12Bytes = type(uint256).max << 160;
```
https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L213
https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L215
https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L217
https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L234
https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L236
https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L372

```solidity
9  uint256 internal constant MAX_EXPIRY = type(uint96).max;
```
https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenStorageHelpers.sol#L9



## [G-02] Amounts should be checked for 0 before calling a transfer


Checking for zero values is especially important when transferring tokens or ether, as sending these assets to an address with a zero value will result in the loss of those assets.


```solidity
375   SafeERC20.safeTransfer(IERC20(underlyingContract), msg.sender, erc20UnderlyingAmount);

399   SafeERC20.safeTransfer(IERC20(info.tokenContract), info.receiver, info.amount);
```
https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L375
https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L399






## [G‑03]  State variables only set in the constructor should be declared immutable


While strings are not value types, and therefore cannot be immutable/constant if not hard-coded outside of the constructor, the same behavior can be achieved by making the current contract abstract with virtual functions for the string accessors, and having a child contract override the functions with the hard-coded implementation-specific values.

```solidity
26   Structs.TransientState internal transientState;
```
https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L26



## [G-04] Use ERC721A instead ERC721

ERC721A standard, ERC721A is an improvement standard for ERC721 tokens. It was proposed by the Azuki team and used for developing their NFT collection. Compared with ERC721, ERC721A is a more gas-efficient standard to mint a lot of of NFTs simultaneously. It allows developers to mint multiple NFTs at the same gas price. This has been a great improvement due to Ethereum's sky-rocketing gas fee.


```solidity
6   import {ERC721} from "openzeppelin-contracts/contracts/token/ERC721/ERC721.sol";
```
https://github.com/code-423n4/2023-09-delegate/blob/main/src/PrincipalToken.sol#L6




be declared immutable|1|

## [G-05] Can Make The Variable Outside The Loop To Save Gas 
When you declare a variable inside a loop, Solidity creates a new instance of the variable for each iteration of the loop. This can lead to unnecessary gas costs, especially if the loop is executed frequently or iterates over a large number of elements.

By declaring the variable outside the loop, you can avoid the creation of multiple instances of the variable and reduce the gas cost of your contract. 

```solidity
276      bytes32 location = Hashes.location(hashes[i]);

277      address from = _loadFrom(location);
```
https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L276
https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L277





## [G-06] Sort Solidity operations using short-circuit mode

Short-circuiting is a solidity contract development model that uses OR/AND logic to sequence different cost operations. It puts low gas cost operations in the front and high gas cost operations in the back, so that if the front is low If the cost operation is feasible, you can skip (short-circuit) the subsequent high-cost Ethereum virtual machine operation.



```solidity
78   if (erc1155Pulled.flag == ERC1155_PULLED && address(this) == operator) {
```
https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenTransferHelpers.sol#L78


## [G‑07] Structs can be packed into fewer storage slots

Each slot saved can avoid an extra Gsset (20000 gas) for the first setting of the struct. Subsequent reads as well as writes have smaller gas savings.

```solidity
98      struct Order {
        bytes32 rights;
        uint256 expiryLength;
        uint256 signerSalt;
        address tokenContract;
        CreateOffererEnums.ExpiryType expiryType;
        CreateOffererEnums.TargetToken targetToken;
    }
```
https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L98

```solidity
20     struct DelegateInfo {
        address principalHolder;
        IDelegateRegistry.DelegationType tokenType;
        address delegateHolder;
        uint256 amount;
        address tokenContract;
        uint256 tokenId;
        bytes32 rights;
        uint256 expiry;
    }

31      struct FlashInfo {
        address receiver; // The address to receive the loaned assets
        address delegateHolder; // The holder of the delegation
        IDelegateRegistry.DelegationType tokenType; // The type of contract, e.g. ERC20
        address tokenContract; // The contract of the underlying being loaned
        uint256 tokenId; // The tokenId of the underlying being loaned, if applicable
        uint256 amount; // The amount being lent, if applicable
        bytes data; // Arbitrary data structure, intended to contain user-defined parameters
    }    
```
https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenLib.sol#L31


