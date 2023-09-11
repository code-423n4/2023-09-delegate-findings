# Gas-Optimization

# Summary

| Number | Gas Optimization Details                                                 | Contexts |
| :----: | :----------------------------------------------------------------------- | :------: |
| [G-01] | Structs can be packed into fewer storage slots                           |    4     |
| [G-02] | State variables only set in the constructor should be declared immutable |    1     |
| [G-03] | Can Make The Variable Outside The Loop To Save Gas                       |    2     |
| [G-04] | Do not calculate constants                                               |    2     |
| [G-05] | Use constants instead of type(uintx).max                                 |    8     |
| [G-06] | Sort Solidity operations using short-circuit mode                        |    1     |
| [G-07] | USE BITMAP TO SAVE GAS                                                   |    1     |
| [G-08] | Use ERC721A instead ERC721                                               |    1     |
| [G-09] | Amounts should be checked for 0 before calling a transfer                |    3     |
| [G-10] | Use hardcode address instead address(this)                               |    24    |
| [G-11] | abi.encode() is less efficient than abi.encodePacked()                   |    10    |
| [G-12] | Use uint256(1)/uint256(2) instead for true and false boolean states      |          |

## [G‑01] Structs can be packed into fewer storage slots

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

https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol

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

https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenLib.sol

## [G‑02] State variables only set in the constructor should be declared immutable

Avoids a Gsset (20000 gas)

Avoids a Gsset (20000 gas) in the constructor, and replaces the first access in each transaction (Gcoldsload - 2100 gas) and each access thereafter (Gwarmacces - 100 gas) with a PUSH32 (3 gas).

While strings are not value types, and therefore cannot be immutable/constant if not hard-coded outside of the constructor, the same behavior can be achieved by making the current contract abstract with virtual functions for the string accessors, and having a child contract override the functions with the hard-coded implementation-specific values.

```solidity
26   Structs.TransientState internal transientState;
```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol

## [G-03] Can Make The Variable Outside The Loop To Save Gas

When you declare a variable inside a loop, Solidity creates a new instance of the variable for each iteration of the loop. This can lead to unnecessary gas costs, especially if the loop is executed frequently or iterates over a large number of elements.

By declaring the variable outside the loop, you can avoid the creation of multiple instances of the variable and reduce the gas cost of your contract. Here's an example:

```
contract MyContract {
    function sum(uint256[] memory values) public pure returns (uint256) {
        uint256 total = 0;

        for (uint256 i = 0; i < values.length; i++) {
            total += values[i];
        }

        return total;
    }
}
```

```solidity
276      bytes32 location = Hashes.location(hashes[i]);

277      address from = _loadFrom(location);
```

https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol

## [G-04] Do not calculate constants

```solidity
File: libraries/RegistryStorage.sol

20    uint256 internal constant CLEAN_FIRST8_BYTES_ADDRESS = 0xffffffffffffffff << 96;

23    uint256 internal constant CLEAN_PACKED8_BYTES_ADDRESS = 0xffffffffffffffff << 160;

```

https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/libraries/RegistryStorage.sol#L20

## [G-05] Use constants instead of type(uintx).max

it's generally more gas-efficient to use constants instead of type(uintX).max when you need to set the maximum value of an unsigned integer type.

The reason for this is that the type(uintX).max expression involves a computation at runtime, whereas a constant is evaluated at compile-time. This means that using type(uintX).max can result in additional gas costs for each transaction that involves the expression.

By using a constant instead of type(uintX).max, you can avoid these additional gas costs and make your code more efficient.

Here's an example of how you can use a constant instead of type(uintX).max:

```
contract MyContract {
    uint120 constant MAX_VALUE = 2**120 - 1;

    function doSomething(uint120 value) public {
        require(value <= MAX_VALUE, "Value exceeds maximum");

        // Do something
    }
}
```

In the above example, we have a contract with a constant MAX_VALUE that represents the maximum value of a uint120. When the doSomething function is called with a value parameter, it checks whether the value is less than or equal to MAX_VALUE using the <= operator.

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

https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol

```solidity
9  uint256 internal constant MAX_EXPIRY = type(uint96).max;
```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenStorageHelpers.sol

## [G-06] Sort Solidity operations using short-circuit mode

Short-circuiting is a solidity contract development model that uses OR/AND logic to sequence different cost operations. It puts low gas cost operations in the front and high gas cost operations in the back, so that if the front is low If the cost operation is feasible, you can skip (short-circuit) the subsequent high-cost Ethereum virtual machine operation.

```
//f(x) is a low gas cost operation
//g(y) is a high gas cost operation
//Sort operations with different gas costs as follows
f(x) || g(y)
f(x) && g(y)
```

```solidity
78   if (erc1155Pulled.flag == ERC1155_PULLED && address(this) == operator) {
```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenTransferHelpers.sol

## [G-07] USE BITMAP TO SAVE GAS

This is a good Example: [Resource](https://github.com/code-423n4/2023-09-ondo/blob/main/bot-report.md#g-38-using-bitmap-to-store-bool-states-can-save-gas)

```solidity
File: src/DelegateToken.sol

39    mapping(address account => mapping(address operator => bool enabled)) internal accountOperator;

```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L39

## [G-08] Use ERC721A instead ERC721

ERC721A standard, ERC721A is an improvement standard for ERC721 tokens. It was proposed by the Azuki team and used for developing their NFT collection. Compared with ERC721, ERC721A is a more gas-efficient standard to mint a lot of of NFTs simultaneously. It allows developers to mint multiple NFTs at the same gas price. This has been a great improvement due to Ethereum's sky-rocketing gas fee.

[Reffrence](https://nextrope.com/erc721-vs-erc721a-2/)

```solidity
6   import {ERC721} from "openzeppelin-contracts/contracts/token/ERC721/ERC721.sol";
```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/PrincipalToken.sol

## [G-09] Amounts should be checked for 0 before calling a transfer

It is generally a good practice to check for zero values before making any transfers in smart contract functions. This can help to avoid unnecessary external calls and can save gas costs.

Checking for zero values is especially important when transferring tokens or ether, as sending these assets to an address with a zero value will result in the loss of those assets.

In Solidity, you can check whether a value is zero by using the == operator. Here's an example of how you can check for a zero value before making a transfer:

```
function transfer(address payable recipient, uint256 amount) public {
    require(amount > 0, "Amount must be greater than zero");
    recipient.transfer(amount);
}
```

In the above example, we check to make sure that the amount parameter is greater than zero before making the transfer to the recipient address. If the amount is zero or negative, the function will revert and the transfer will not be made.

```solidity
375   SafeERC20.safeTransfer(IERC20(underlyingContract), msg.sender, erc20UnderlyingAmount);

399   SafeERC20.safeTransfer(IERC20(info.tokenContract), info.receiver, info.amount);
```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol

## [G-10] Use hardcode address instead address(this)

```solidity
File: src/DelegateRegistry.sol

37                (success, results[i]) = address(this).delegatecall(data[i]);
```

https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L37

```solidity
File: src/DelegateToken.sol


84        if (address(this) == operator) return IERC721Receiver.onERC721Received.selector;

178            newRegistryHash = RegistryHashes.erc721Hash(address(this), underlyingRights, to, underlyingTokenId, underlyingContract);

182            newRegistryHash = RegistryHashes.erc20Hash(address(this), underlyingRights, to, underlyingContract);

196            newRegistryHash = RegistryHashes.erc1155Hash(address(this), underlyingRights, to, underlyingTokenId, underlyingContract);

307            newRegistryHash = RegistryHashes.erc721Hash(address(this), delegateInfo.rights, delegateInfo.delegateHolder, delegateInfo.tokenId, delegateInfo.tokenContract);

312            newRegistryHash = RegistryHashes.erc20Hash(address(this), delegateInfo.rights, delegateInfo.delegateHolder, delegateInfo.tokenContract);

317            newRegistryHash = RegistryHashes.erc1155Hash(address(this), delegateInfo.rights, delegateInfo.delegateHolder, delegateInfo.tokenId, delegateInfo.tokenContract);

369            IERC721(underlyingContract).transferFrom(address(this), msg.sender, erc721UnderlyingTokenId);

384            IERC1155(underlyingContract).safeTransferFrom(address(this), msg.sender, erc11551UnderlyingTokenId, erc1155UnderlyingAmount, "");

393            IERC721(info.tokenContract).transferFrom(address(this), info.receiver, info.tokenId);

406            IERC1155(info.tokenContract).safeTransferFrom(address(this), info.receiver, info.tokenId, info.amount, "");

```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol

```solidity
File: src/CreateOfferer.sol

90        if (from != address(this)) revert Errors.FromNotCreateOfferer(from);


138            if (IERC20(erc20Order.info.tokenContract).allowance(address(this), address(delegateToken)) != 0) {

```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol

```solidity
File: src/libraries/CreateOffererLib.sol


259        uint256 requestedDelegateId = DelegateTokenHelpers.delegateIdNoRevert(address(this), createOrderHash);

280        delegateTokenId = IDelegateToken(delegateToken).getDelegateId(address(this), createOrderHash); // This should revert if already existed

314            ) != keccak256(abi.encode(IDelegateToken(delegateToken).getDelegateInfo(DelegateTokenHelpers.delegateIdNoRevert(address(this), identifier))))

352        if (minimumReceived[0].itemType != ItemType.ERC721 || minimumReceived[0].token != address(this) || minimumReceived[0].amount != 1) {

366            recipient: payable(address(this))

```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol

```solidity
File: libraries/DelegateTokenTransferHelpers.sol

41        IERC721(underlyingContract).transferFrom(msg.sender, address(this), underlyingTokenId);

52        if (IERC20(underlyingContract).allowance(msg.sender, address(this)) < underlyingAmount) {

58        SafeERC20.safeTransferFrom(IERC20(underlyingContract), msg.sender, address(this), pullAmount);

71        IERC1155(underlyingContract).safeTransferFrom(msg.sender, address(this), underlyingTokenId, pullAmount, "");

78        if (erc1155Pulled.flag == ERC1155_PULLED && address(this) == operator) {

```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenTransferHelpers.sol#L71

## [G-11] abi.encode() is less efficient than abi.encodePacked()

```solidity
File: src/CreateOfferer.sol

98            Helpers.validateCreateOrderHash(targetTokenReceiver, createOrderHashAsTokenId, abi.encode(erc721Order), tokenType);

120            Helpers.validateCreateOrderHash(targetTokenReceiver, createOrderHashAsTokenId, abi.encode(erc20Order), tokenType);

146            Helpers.validateCreateOrderHash(targetTokenReceiver, createOrderHashAsTokenId, abi.encode(erc1155Order), tokenType);

201            Helpers.calculateOrderHashAndId(delegateToken, targetTokenReceiver, conduit, abi.encode(erc721Order), IDelegateRegistry.DelegationType.ERC721);

219            Helpers.calculateOrderHashAndId(delegateToken, targetTokenReceiver, conduit, abi.encode(erc20Order), IDelegateRegistry.DelegationType.ERC20);

237            Helpers.calculateOrderHashAndId(delegateToken, targetTokenReceiver, conduit, abi.encode(erc1155Order), IDelegateRegistry.DelegationType.ERC1155);

```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol

```solidity
File: libraries/CreateOffererLib.sol

302                abi.encode(

398        uint256 hashWithoutType = uint256(keccak256(abi.encode(targetTokenReceiver, conduit, createOrderInfo)));

```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L302

```solidity
File: libraries/DelegateTokenLib.sol

114        return uint256(keccak256(abi.encode(caller, salt)));

```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenLib.sol#L114

## [G-12] Use uint256(1)/uint256(2) instead for true and false boolean states

```solidity
File: src/CreateOfferer.sol

99             IERC721(erc721Order.info.tokenContract).setApprovalForAll(address(delegateToken), true);

147            IERC1155(erc1155Order.info.tokenContract).setApprovalForAll(address(delegateToken), true);

```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol

```solidity
File: libraries/DelegateTokenTransferHelpers.sol

80            return true;

```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenTransferHelpers.sol#L80

```solidity
File: libraries/DelegateTokenTransferHelpers.sol

82        return false;
```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenTransferHelpers.sol#L82