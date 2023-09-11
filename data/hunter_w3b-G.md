# Gas Optimization

While striving for enhanced code clarity in the provided snippets, certain functions have been abbreviated to emphasize affected sections.

Developers should remain vigilant during the incorporation of these proposed modifications to avert potential vulnerabilities. Despite prior testing of the optimizations, developers bear the responsibility of conducting comprehensive reevaluation.

Conducting code reviews and additional testing is highly recommended to mitigate any plausible hazards that may arise from the refactoring endeavor.

# Summary

| Number | Issue                                                                         | Instances |
| :----: | :---------------------------------------------------------------------------- | :-------: |
| [G-01] | Use do while  loops instead of for loops                                      |     7     |
| [G-02] | STACK VARIABLE USED AS A CHEAPER CACHE FOR A STATE VARIABLE IS ONLY USED ONCE |     1     |
| [G-03] | CAN MAKE THE VARIABLE OUTSIDE THE LOOP TO SAVE GAS                            |     2     |
| [G-04] | BEFORE SOME FUNCTIONS, WE SHOULD CHECK SOME VARIABLES FOR POSSIBLE GAS SAVE   |     2     |
| [G-05] | USE BITMAPS TO SAVE GAS                                                       |     1     |
| [G-06] | Structs can be packed into fewer storage slots                                |     4     |
| [G-07] | Using Storage instead of memory for structs/arrays saves gas                  |     9     |
| [G-08] | Use constants instead of type(uintx).max                                      |     7     |
| [G-09] | Use assembly to write address storage values                                  |     8     |
| [G-10] | Multiple accesses of a mapping/array should use a local variable cache        |    27     |
| [G-11] | abi.encode() is less efficient than abi.encodePacked()                        |     9     |
| [G-12] | Do not calculate constants                                                    |     2     |
| [G-13] | ++i Costs less gas than i++                                                   |     2     |
| [G-14] | Duplicated if() checks should be refactored to a modifier or function         |     1     |
| [G-15] | Sort Solidity operations using short-circuit mode                             |    16     |
| [G-16] | Use assembly for math (add, sub, mul, div)                                    |     1     |
| [G-17] | Use hardcode address instead address(this)                                    |    24     |
| [G-18] | Shorten the array rather than copying to a new one                            |     9     |
| [G-19] | Use uint256(1)/uint256(2) instead for true and false boolean states           |     4     |
| [G-20] | Use ERC721A instead ERC721                                                    |     1     |
| [G-21] | State variables only set in the constructor should be declared immutable      |     2     |
| [G-22] | Using PRIVATE rather than PUBLIC FOR Constants/Immutable, Saves Gas           |     8     |
| [G-23] | Don't initialize variables with default value                                 |     7     |

## [G-01] Use do while  loops instead of for loops

A do while loop will cost less gas since the condition is not being checked for the first iteration.

```solidity
File: src/DelegateRegistry.sol

35            for (uint256 i = 0; i < data.length; ++i) {

275            for (uint256 i = 0; i < hashes.length; ++i) {

312            for (uint256 i = 0; i < length; ++i) {

386            for (uint256 i = 0; i < hashesLength; ++i) {

393            for (uint256 i = 0; i < count; ++i) {

417            for (uint256 i = 0; i < hashesLength; ++i) {

423            for (uint256 i = 0; i < count; ++i) {

```

https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol

## [G-02] STACK VARIABLE USED AS A CHEAPER CACHE FOR A STATE VARIABLE IS ONLY USED ONCE

```solidity
File: libraries/DelegateTokenStorageHelpers.sol

9    uint256 internal constant MAX_EXPIRY = type(uint96).max;
```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenStorageHelpers.sol#L9

This variable is only used in this function it's cheaper to use stack variable

```solidity
File: libraries/DelegateTokenStorageHelpers.sol

44  function writeExpiry(mapping(uint256 delegateTokenId => uint256[3] info) storage delegateTokenInfo, uint256 delegateTokenId, uint256 expiry) internal {
45        if (expiry > MAX_EXPIRY) revert Errors.ExpiryTooLarge();
46        address approved = address(uint160(delegateTokenInfo[delegateTokenId][PACKED_INFO_POSITION] >> 96));
47        delegateTokenInfo[delegateTokenId][PACKED_INFO_POSITION] = (uint256(uint160(approved)) << 96) | expiry;
48    }
```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenStorageHelpers.sol#L44C1-L48C6

## [G-03] CAN MAKE THE VARIABLE OUTSIDE THE LOOP TO SAVE GAS

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

## [G-04] BEFORE SOME FUNCTIONS, WE SHOULD CHECK SOME VARIABLES FOR POSSIBLE GAS SAVE

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
File: src/DelegateToken.sol


375   SafeERC20.safeTransfer(IERC20(underlyingContract), msg.sender, erc20UnderlyingAmount);

399   SafeERC20.safeTransfer(IERC20(info.tokenContract), info.receiver, info.amount);
```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol

## [G-05] USE BITMAPS TO SAVE GAS

```solidity
File: src/DelegateToken.sol

39    mapping(address account => mapping(address operator => bool enabled)) internal accountOperator;
```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L39

## [G-06] Structs can be packed into fewer storage slots

Each slot saved can avoid an extra Gsset (20000 gas) for the first setting of the struct. Subsequent reads as well as writes have smaller gas savings.

```solidity
File: libraries/CreateOffererLib.sol

98      struct Order {
99        bytes32 rights;
100        uint256 expiryLength;
101        uint256 signerSalt;
102        address tokenContract;
103        CreateOffererEnums.ExpiryType expiryType;
104        CreateOffererEnums.TargetToken targetToken;
105    }
```

### This will save 1 storage slot

```diff
98      struct Order {
99         bytes32 rights;
100        uint256 expiryLength;
101        uint256 signerSalt;
-102       address tokenContract;
103        CreateOffererEnums.ExpiryType expiryType;
104        CreateOffererEnums.TargetToken targetToken;
+          address tokenContract;
105    }
```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol

```solidity
File: libraries/DelegateTokenLib.sol


20     struct DelegateInfo {
21        address principalHolder;
22        IDelegateRegistry.DelegationType tokenType;
23        address delegateHolder;
24        uint256 amount;
25        address tokenContract;
26        uint256 tokenId;
27        bytes32 rights;
28        uint256 expiry;
29    }



31      struct FlashInfo {
32        address receiver; // The address to receive the loaned assets
33        address delegateHolder; // The holder of the delegation
34        IDelegateRegistry.DelegationType tokenType; // The type of contract, e.g. ERC20
35        address tokenContract; // The contract of the underlying being loaned
36        uint256 tokenId; // The tokenId of the underlying being loaned, if applicable
37        uint256 amount; // The amount being lent, if applicable
38        bytes data; // Arbitrary data structure, intended to contain user-defined parameters
39    }
```

```diff
20     struct DelegateInfo {
-21        address principalHolder;
-22        IDelegateRegistry.DelegationType tokenType;
-23        address delegateHolder;
24        uint256 amount;
-25        address tokenContract;
26        uint256 tokenId;
27        bytes32 rights;
28        uint256 expiry;
29
+        address principalHolder;
+        IDelegateRegistry.DelegationType tokenType;
+        address delegateHolder;
+        address tokenContract;
    }



31      struct FlashInfo {
32        address receiver; // The address to receive the loaned assets
33        address delegateHolder; // The holder of the delegation
34        IDelegateRegistry.DelegationType tokenType; // The type of contract, e.g. ERC20
35        address tokenContract; // The contract of the underlying being loaned
36        uint256 tokenId; // The tokenId of the underlying being loaned, if applicable
37        uint256 amount; // The amount being lent, if applicable
38        bytes data; // Arbitrary data structure, intended to contain user-defined parameters
39    }
```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenLib.sol

## [G-07] Using Storage instead of memory for structs/arrays saves gas

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from
storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array.

If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read.

Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to
be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read.

The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

```solidity
File: src/DelegateRegistry.sol

384        bytes32[] memory filteredHashes = new bytes32[](hashesLength);


415        bytes32[] memory filteredHashes = new bytes32[](hashesLength);

```

https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L384

```solidity
File: src/CreateOfferer.sol

34        Structs.Order memory defaultInfo =

59        Structs.Context memory decodedContext = abi.decode(context, (Structs.Context));

94            Structs.ERC721Order memory erc721Order = transientState.erc721Order;

116            Structs.ERC20Order memory erc20Order = transientState.erc20Order;

142            Structs.ERC1155Order memory erc1155Order = transientState.erc1155Order;

```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L34

```solidity
File: libraries/CreateOffererLib.sol

157        CreateOffererStructs.Stage memory cacheStage = stage;

```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L157

```solidity
File: libraries/CreateOffererLib.sol

298        CreateOffererStructs.Context memory decodedContext = abi.decode(context, (CreateOffererStructs.Context));

```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L298

## [G-08] Use constants instead of type(uintx).max

Use constants instead of type(uintx).max
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
File: src/DelegateRegistry.sol


213   ? type(uint256).max

215   if (!Ops.or(rights == "", amount == type(uint256).max)) {

217   ? type(uint256).max

234   if (!Ops.or(rights == "", amount == type(uint256).max)) {

236   ? type(uint256).max

372   uint256 cleanUpper12Bytes = type(uint256).max << 160;
```

https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol

```solidity
File: libraries/DelegateTokenStorageHelpers.sol

9  uint256 internal constant MAX_EXPIRY = type(uint96).max;
```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenStorageHelpers.sol

## [G-09] Use assembly to write address storage values

In Solidity, you can use inline assembly to interact with Ethereum's Ethereum Virtual Machine (EVM) at a low level. If you want to store values in an address in storage using inline assembly, you can do so as follows:

For-Example:

```

contract StorageExample {
    uint256 public storedValue;

    function setValue(uint256 _newValue) public {
        assembly {
            // Load the storage slot for the variable storedValue
            sstore(0x00, _newValue)
        }
    }

    function getValue() public view returns (uint256) {
        assembly {
            // Load the storage slot for the variable storedValue
            storedValue := sload(0x00)
        }
        return storedValue;
    }
}

```

```solidity
File: src/DelegateToken.sol

56        delegateRegistry = parameters.delegateRegistry;
57        principalToken = parameters.principalToken;
58        marketMetadata = parameters.marketMetadata;
```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L56-L58

```solidity
File: src/PrincipalToken.sol

22        delegateToken = setDelegateToken;

24        marketMetadata = setMarketMetadata;

```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/PrincipalToken.sol#L22

```solidity
File: src/CreateOfferer.sol

31        delegateToken = parameters.delegateToken;

33        principalToken = parameters.principalToken;
```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L31

```solidity
File: libraries/CreateOffererLib.sol

147        seaport = setSeaport;

```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L147

## [G-10] Multiple accesses of a mapping/array should use a local variable cache

The instances below point to the second+ access of a value inside a mapping/array, within a function. Caching a mapping’s value in a local storage or calldata variable when the value is accessed multiple times, saves ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations. Caching an array’s struct avoids recalculating the array offsets into memory/calldata.

### `delegateTokenInfo` should use a local variable cache

```solidity
File: src/DelegateToken.sol

        StorageHelpers.revertNotMinted(delegateTokenInfo, delegateTokenId);
        return StorageHelpers.readApproved(delegateTokenInfo, delegateTokenId);



// @audit `delegateTokenInfo` should use a local variable cache in this transferFrom function
function transferFrom(address from, address to, uint256 delegateTokenId) public {
        if (to == address(0)) revert Errors.ToIsZero();
        bytes32 registryHash = StorageHelpers.readRegistryHash(delegateTokenInfo, delegateTokenId);
        StorageHelpers.revertNotMinted(registryHash, delegateTokenId);
        (address delegateTokenHolder, address underlyingContract) = RegistryHelpers.loadTokenHolderAndContract(delegateRegistry, registryHash);
        if (from != delegateTokenHolder) revert Errors.FromNotDelegateTokenHolder();
        // We can use `from` here instead of delegateTokenHolder since we've just verified that from == delegateTokenHolder
        StorageHelpers.revertNotApprovedOrOperator(accountOperator, delegateTokenInfo, from, delegateTokenId);
        StorageHelpers.incrementBalance(balances, to);
        StorageHelpers.decrementBalance(balances, from);
        StorageHelpers.writeApproved(delegateTokenInfo, delegateTokenId, address(0));
        emit Transfer(from, to, delegateTokenId);
        IDelegateRegistry.DelegationType underlyingType = RegistryHashes.decodeType(registryHash);
        bytes32 underlyingRights = RegistryHelpers.loadRights(delegateRegistry, registryHash);
        bytes32 newRegistryHash = 0;
        if (underlyingType == IDelegateRegistry.DelegationType.ERC721) {
            uint256 underlyingTokenId = RegistryHelpers.loadTokenId(delegateRegistry, registryHash);
            newRegistryHash = RegistryHashes.erc721Hash(address(this), underlyingRights, to, underlyingTokenId, underlyingContract);
            StorageHelpers.writeRegistryHash(delegateTokenInfo, delegateTokenId, newRegistryHash);
            RegistryHelpers.transferERC721(delegateRegistry, registryHash, from, newRegistryHash, to, underlyingRights, underlyingContract, underlyingTokenId);
        } else if (underlyingType == IDelegateRegistry.DelegationType.ERC20) {
            newRegistryHash = RegistryHashes.erc20Hash(address(this), underlyingRights, to, underlyingContract);
            StorageHelpers.writeRegistryHash(delegateTokenInfo, delegateTokenId, newRegistryHash);
            RegistryHelpers.transferERC20(
                delegateRegistry,
                registryHash,
                from,
                newRegistryHash,
                to,
                StorageHelpers.readUnderlyingAmount(delegateTokenInfo, delegateTokenId),
                underlyingRights,
                underlyingContract
            );
        } else if (underlyingType == IDelegateRegistry.DelegationType.ERC1155) {
            uint256 underlyingTokenId = RegistryHelpers.loadTokenId(delegateRegistry, registryHash);
            newRegistryHash = RegistryHashes.erc1155Hash(address(this), underlyingRights, to, underlyingTokenId, underlyingContract);
            StorageHelpers.writeRegistryHash(delegateTokenInfo, delegateTokenId, newRegistryHash);
            RegistryHelpers.transferERC1155(
                delegateRegistry,
                registryHash,
                from,
                newRegistryHash,
                to,
                StorageHelpers.readUnderlyingAmount(delegateTokenInfo, delegateTokenId),
                underlyingRights,
                underlyingContract,
                underlyingTokenId
            );
        }
    }




// @audit `delegateTokenInfo` should use a local variable cache in this create function

        function create(Structs.DelegateInfo calldata delegateInfo, uint256 salt) external nonReentrant returns (uint256 delegateTokenId) {
        TransferHelpers.checkAndPullByType(erc1155PullAuthorization, delegateInfo);
        Helpers.revertOldExpiry(delegateInfo.expiry);
        if (delegateInfo.delegateHolder == address(0)) revert Errors.ToIsZero();
        delegateTokenId = Helpers.delegateIdNoRevert(msg.sender, salt);
        StorageHelpers.revertAlreadyExisted(delegateTokenInfo, delegateTokenId);
        StorageHelpers.incrementBalance(balances, delegateInfo.delegateHolder);
        StorageHelpers.writeExpiry(delegateTokenInfo, delegateTokenId, delegateInfo.expiry);
        emit Transfer(address(0), delegateInfo.delegateHolder, delegateTokenId);
        bytes32 newRegistryHash = 0;
        if (delegateInfo.tokenType == IDelegateRegistry.DelegationType.ERC721) {
            newRegistryHash = RegistryHashes.erc721Hash(address(this), delegateInfo.rights, delegateInfo.delegateHolder, delegateInfo.tokenId, delegateInfo.tokenContract);
            StorageHelpers.writeRegistryHash(delegateTokenInfo, delegateTokenId, newRegistryHash);
            RegistryHelpers.delegateERC721(delegateRegistry, newRegistryHash, delegateInfo);
        } else if (delegateInfo.tokenType == IDelegateRegistry.DelegationType.ERC20) {
            StorageHelpers.writeUnderlyingAmount(delegateTokenInfo, delegateTokenId, delegateInfo.amount);
            newRegistryHash = RegistryHashes.erc20Hash(address(this), delegateInfo.rights, delegateInfo.delegateHolder, delegateInfo.tokenContract);
            StorageHelpers.writeRegistryHash(delegateTokenInfo, delegateTokenId, newRegistryHash);
            RegistryHelpers.incrementERC20(delegateRegistry, newRegistryHash, delegateInfo);
        } else if (delegateInfo.tokenType == IDelegateRegistry.DelegationType.ERC1155) {
            StorageHelpers.writeUnderlyingAmount(delegateTokenInfo, delegateTokenId, delegateInfo.amount);
            newRegistryHash = RegistryHashes.erc1155Hash(address(this), delegateInfo.rights, delegateInfo.delegateHolder, delegateInfo.tokenId, delegateInfo.tokenContract);
            StorageHelpers.writeRegistryHash(delegateTokenInfo, delegateTokenId, newRegistryHash);
            RegistryHelpers.incrementERC1155(delegateRegistry, newRegistryHash, delegateInfo);
        }
        StorageHelpers.mintPrincipal(principalToken, principalMintAuthorization, delegateInfo.principalHolder, delegateTokenId);
    }



// @audit `delegateTokenInfo` should use a local variable cache in this create function

   function withdraw(uint256 delegateTokenId) external nonReentrant {
        bytes32 registryHash = StorageHelpers.readRegistryHash(delegateTokenInfo, delegateTokenId);
        StorageHelpers.writeRegistryHash(delegateTokenInfo, delegateTokenId, bytes32(StorageHelpers.ID_USED));
        // Sets registry pointer to used flag
        StorageHelpers.revertNotMinted(registryHash, delegateTokenId);
        (address delegateTokenHolder, address underlyingContract) = RegistryHelpers.loadTokenHolderAndContract(delegateRegistry, registryHash);
        StorageHelpers.revertInvalidWithdrawalConditions(delegateTokenInfo, accountOperator, delegateTokenId, delegateTokenHolder);
        StorageHelpers.decrementBalance(balances, delegateTokenHolder);
        delete delegateTokenInfo[delegateTokenId][StorageHelpers.PACKED_INFO_POSITION]; // Deletes both expiry and approved
        emit Transfer(delegateTokenHolder, address(0), delegateTokenId);
        IDelegateRegistry.DelegationType delegationType = RegistryHashes.decodeType(registryHash);
        bytes32 underlyingRights = RegistryHelpers.loadRights(delegateRegistry, registryHash);
        if (delegationType == IDelegateRegistry.DelegationType.ERC721) {
            uint256 erc721UnderlyingTokenId = RegistryHelpers.loadTokenId(delegateRegistry, registryHash);
            RegistryHelpers.revokeERC721(delegateRegistry, registryHash, delegateTokenHolder, underlyingContract, erc721UnderlyingTokenId, underlyingRights);
            StorageHelpers.burnPrincipal(principalToken, principalBurnAuthorization, delegateTokenId);
            IERC721(underlyingContract).transferFrom(address(this), msg.sender, erc721UnderlyingTokenId);
        } else if (delegationType == IDelegateRegistry.DelegationType.ERC20) {
            uint256 erc20UnderlyingAmount = StorageHelpers.readUnderlyingAmount(delegateTokenInfo, delegateTokenId);
            StorageHelpers.writeUnderlyingAmount(delegateTokenInfo, delegateTokenId, 0); // Deletes amount
            RegistryHelpers.decrementERC20(delegateRegistry, registryHash, delegateTokenHolder, underlyingContract, erc20UnderlyingAmount, underlyingRights);
            StorageHelpers.burnPrincipal(principalToken, principalBurnAuthorization, delegateTokenId);
            SafeERC20.safeTransfer(IERC20(underlyingContract), msg.sender, erc20UnderlyingAmount);
        } else if (delegationType == IDelegateRegistry.DelegationType.ERC1155) {
            uint256 erc1155UnderlyingAmount = StorageHelpers.readUnderlyingAmount(delegateTokenInfo, delegateTokenId);
            StorageHelpers.writeUnderlyingAmount(delegateTokenInfo, delegateTokenId, 0); // Deletes amount
            uint256 erc11551UnderlyingTokenId = RegistryHelpers.loadTokenId(delegateRegistry, registryHash);
            RegistryHelpers.decrementERC1155(
                delegateRegistry, registryHash, delegateTokenHolder, underlyingContract, erc11551UnderlyingTokenId, erc1155UnderlyingAmount, underlyingRights
            );
            StorageHelpers.burnPrincipal(principalToken, principalBurnAuthorization, delegateTokenId);
            IERC1155(underlyingContract).safeTransferFrom(address(this), msg.sender, erc11551UnderlyingTokenId, erc1155UnderlyingAmount, "");
        }
    }
```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L112

## [G-11] abi.encode() is less efficient than abi.encodePacked()

In terms of efficiency, abi.encodePacked() is generally considered to be more gas-efficient than abi.encode(), because it skips the step of adding function signatures and other metadata to the encoded data. However, this comes at the cost of reduced safety, as abi.encodePacked() does not perform any type checking or padding of data.

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

## [G-12] Do not calculate constants

```solidity
File: libraries/RegistryStorage.sol

20    uint256 internal constant CLEAN_FIRST8_BYTES_ADDRESS = 0xffffffffffffffff << 96;

23    uint256 internal constant CLEAN_PACKED8_BYTES_ADDRESS = 0xffffffffffffffff << 160;

```

https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/libraries/RegistryStorage.sol#L20

## [G-13] ++i Costs less gas than i++

In general, ++i (pre-increment) is likely to consume less gas compared to i++ (post-increment) when used in loops or other repetitive constructs in Solidity. This is because ++i increments the value before using it in an operation, while i++ uses the current value before incrementing it. The gas cost in Solidity depends on the number of storage or memory operations, and ++i can potentially result in fewer operations, making it more gas-efficient.

```solidity
File: src/DelegateRegistry.sol

389                filteredHashes[count++] = hash;

420                filteredHashes[count++] = hash;

```

https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L389

## [G-14] Duplicated if() checks should be refactored to a modifier or function

to reduce code duplication and improve readability.
•  Modifiers can be used to perform additional checks on the function inputs or state before it is executed. By defining a modifier to perform a specific check, we can reuse it across multiple functions that require the same check.
• A function can also be used to perform a specific check and return a boolean value indicating whether the check has passed or failed. This can be useful when the check is more complex and cannot be performed easily in a modifier.

```solidity
file: /src/DelegateRegistry.sol

52                if (rights != "") _writeDelegation(location, Storage.POSITIONS_RIGHTS, rights);

```

https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L52

## [G-15] Sort Solidity operations using short-circuit mode  

Short-circuiting is a solidity contract development model that uses OR/AND logic to sequence different cost operations. It puts low gas cost operations in the front and high gas cost operations in the back, so that if the front is low If the cost operation is feasible, you can skip (short-circuit) the subsequent high-cost Ethereum virtual machine operation.

```
//f(x) is a low gas cost operation
//g(y) is a high gas cost operation
//Sort operations with different gas costs as follows
f(x) || g(y)
f(x) && g(y)
```

```solidity
File: src/DelegateRegistry.sol

180            valid = _validateFrom(Hashes.allLocation(from, "", to), from) || _validateFrom(Hashes.contractLocation(from, "", to, contract_), from);


182                valid = _validateFrom(Hashes.allLocation(from, rights, to), from) || _validateFrom(Hashes.contractLocation(from, rights, to, contract_), from);



195            valid = _validateFrom(Hashes.allLocation(from, "", to), from) || _validateFrom(Hashes.contractLocation(from, "", to, contract_), from)
196                || _validateFrom(Hashes.erc721Location(from, "", to, tokenId, contract_), from);
197            if (!Ops.or(rights == "", valid)) {
198                valid = _validateFrom(Hashes.allLocation(from, rights, to), from) || _validateFrom(Hashes.contractLocation(from, rights, to, contract_), from)
199                    || _validateFrom(Hashes.erc721Location(from, rights, to, tokenId, contract_), from);
200            }


212            amount = (_validateFrom(Hashes.allLocation(from, "", to), from) || _validateFrom(Hashes.contractLocation(from, "", to, contract_), from))


216                uint256 rightsBalance = (_validateFrom(Hashes.allLocation(from, rights, to), from) || _validateFrom(Hashes.contractLocation(from, rights, to, contract_), from))


231            amount = (_validateFrom(Hashes.allLocation(from, "", to), from) || _validateFrom(Hashes.contractLocation(from, "", to, contract_), from))


235                uint256 rightsBalance = (_validateFrom(Hashes.allLocation(from, rights, to), from) || _validateFrom(Hashes.contractLocation(from, rights, to, contract_), from))

```

https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L180

```solidity
File: src/CreateOfferer.sol

95            if (!(erc721Order.info.targetToken == Enums.TargetToken.delegate || erc721Order.info.targetToken == Enums.TargetToken.principal)) {


117            if (!(erc20Order.info.targetToken == Enums.TargetToken.delegate || erc20Order.info.targetToken == Enums.TargetToken.principal)) {


143            if (!(erc1155Order.info.targetToken == Enums.TargetToken.delegate || erc1155Order.info.targetToken == Enums.TargetToken.principal)) {

```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L95

```solidity
File: libraries/CreateOffererLib.sol

352        if (minimumReceived[0].itemType != ItemType.ERC721 || minimumReceived[0].token != address(this) || minimumReceived[0].amount != 1) {

```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L352

## [G-16] Use assembly for math (add, sub, mul, div)

```solidity
File: libraries/CreateOffererLib.sol

328            return block.timestamp + expiryLength;

```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L328

## [G-17] Use hardcode address instead address(this)

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.

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

## [G-18] Shorten the array rather than copying to a new one

Inline-assembly can be used to shorten the array by changing the length slot, so that the entries don't have to be copied to a new, shorter array

```solidity
File: src/DelegateRegistry.sol

32        results = new bytes[](data.length);

273        delegations_ = new Delegation[](hashes.length);

308        contents = new bytes32[](length);

384        bytes32[] memory filteredHashes = new bytes32[](hashesLength);

391            delegations_ = new Delegation[](count);

415        bytes32[] memory filteredHashes = new bytes32[](hashesLength);

422            validHashes = new bytes32[](count);

```

https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L32

```solidity
File: libraries/CreateOffererLib.sol

358        offer = new SpentItem[](1);

360        consideration = new ReceivedItem[](1);

```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L358

## [G-19] Use uint256(1)/uint256(2) instead for true and false boolean states

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

## [G-20] Use ERC721A instead ERC721

ERC721A standard, ERC721A is an improvement standard for ERC721 tokens. It was proposed by the Azuki team and used for developing their NFT collection. Compared with ERC721, ERC721A is a more gas-efficient standard to mint a lot of of NFTs simultaneously. It allows developers to mint multiple NFTs at the same gas price. This has been a great improvement due to Ethereum's sky-rocketing gas fee.

[Reffrence](https://nextrope.com/erc721-vs-erc721a-2/)

```solidity
File: src/PrincipalToken.sol

6   import {ERC721} from "openzeppelin-contracts/contracts/token/ERC721/ERC721.sol";

```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/PrincipalToken.sol

## [G-21] State variables only set in the constructor should be declared immutable

Avoids a Gsset (20000 gas) in the constructor, and replaces the first access in each transaction (Gcoldsload - 2100 gas) and each access thereafter (Gwarmacces - 100 gas) with a PUSH32 (3 gas).

While strings are not value types, and therefore cannot be immutable/constant if not hard-coded outside of the constructor, the same behavior can be achieved by making the current contract abstract with virtual functions for the string accessors, and having a child contract override the functions with the hard-coded implementation-specific values.

```solidity
File: src/CreateOfferer.sol

26   Structs.TransientState internal transientState;
```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol

```solidity
file: /src/libraries/CreateOffererLib.sol

139    CreateOffererStructs.Stage private stage;
```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L139

## [G-22] Using PRIVATE rather than PUBLIC FOR Constants/Immutable, Saves Gas

If needed, the value can be read from the verified contract source code. Savings are due to the compiler not having to create non-payable getter functions for deployment calldata, and not adding another entry to the method ID table

```solidity
file: /src/DelegateToken.sol

21    address public immutable override delegateRegistry;

24    address public immutable override principalToken;

26    address public immutable marketMetadata;

```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L21-L26

```solidity
file: /src/PrincipalToken.sol

12    address public immutable delegateToken;

13    address public immutable marketMetadata;

```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/PrincipalToken.sol#L12-L13

```solidity
file: /src/CreateOfferer.sol

24    address public immutable delegateToken;

25    address public immutable principalToken;

```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L24-L25

```solidity
file: /src/libraries/CreateOffererLib.sol

137    address public immutable seaport;

```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L137

## [G-23] Don't initialize variables with default value

Uninitialized variables are assigned with the types default value. Explicitly initializing a variable with it's default value costs unnecesary gas.

```solidity
file: /src/DelegateRegistry.sol

381        uint256 count = 0;

412        uint256 count = 0;

```

https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L381

```solidity
file: /src/libraries/RegistryHashes.sol

31    uint256 internal constant DELEGATION_SLOT = 0;

```

https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/libraries/RegistryHashes.sol#L31

```solidity
file: /src/DelegateToken.sol

175        bytes32 newRegistryHash = 0;

305        bytes32 newRegistryHash = 0;

```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L175

```solidity
file: /src/libraries/DelegateTokenRegistryHelpers.sol

153        uint256 availableAmount = 0;

163        uint256 availableAmount = 0;

```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenRegistryHelpers.sol#L153
