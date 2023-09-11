
## Gas Optimizations

| |Issue|Instances|
|-|:-|:-:|
| [GAS-1](#GAS-1) | abi.encode() is less efficient than abi.encodepacked() | 10 |
| [GAS-2](#GAS-2) | array[index] += amount is cheaper than array[index] = array[index] + amount (or related variants) | 8 |
| [GAS-3](#GAS-3) | Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess | 4 |
| [GAS-4](#GAS-4) | State variables only set in the constructor should be declared immutable | 2 |
| [GAS-5](#GAS-5) | keccak256() hash of literals should only be computed once | 1 |
| [GAS-6](#GAS-6) | Using storage instead of memory in array saves gas | 6 |
| [GAS-7](#GAS-7) | `++i` costs less gas than `i++`, especially when it's used in `for`-loops (`--i`/`i--` too) | 2 |
| [GAS-8](#GAS-8) | Dont transfer with zero amount to save gas | 8 |

## [GAS-1] abi.encode() is less efficient than abi.encodepacked()
See for more information: https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison

*Instances (10)*:
```solidity
File: example/CreateOfferer.sol

98:             Helpers.validateCreateOrderHash(targetTokenReceiver, createOrderHashAsTokenId, abi.encode(erc721Order), tokenType);

120:             Helpers.validateCreateOrderHash(targetTokenReceiver, createOrderHashAsTokenId, abi.encode(erc20Order), tokenType);

146:             Helpers.validateCreateOrderHash(targetTokenReceiver, createOrderHashAsTokenId, abi.encode(erc1155Order), tokenType);

201:             Helpers.calculateOrderHashAndId(delegateToken, targetTokenReceiver, conduit, abi.encode(erc721Order), IDelegateRegistry.DelegationType.ERC721);

219:             Helpers.calculateOrderHashAndId(delegateToken, targetTokenReceiver, conduit, abi.encode(erc20Order), IDelegateRegistry.DelegationType.ERC20);

237:             Helpers.calculateOrderHashAndId(delegateToken, targetTokenReceiver, conduit, abi.encode(erc1155Order), IDelegateRegistry.DelegationType.ERC1155);

```

```solidity
File: example/CreateOffererLib.sol

302:                 abi.encode(

314:             ) != keccak256(abi.encode(IDelegateToken(delegateToken).getDelegateInfo(DelegateTokenHelpers.delegateIdNoRevert(address(this), identifier))))

398:         uint256 hashWithoutType = uint256(keccak256(abi.encode(targetTokenReceiver, conduit, createOrderInfo)));

```

```solidity
File: example/DelegateTokenLib.sol

114:         return uint256(keccak256(abi.encode(caller, salt)));

```

### [GAS-2] array[index] += amount is cheaper than array[index] = array[index] + amount (or related variants)
When updating a value in an array with arithmetic, using array[index] += amount is cheaper than array[index] = array[index] + amount. This is because you avoid an additional mload when the array is stored in memory, and an sload when the array is stored in storage. This can be applied for any arithmetic operation including +=, -=,/=,*=,^=,&=, %=, <<=,>>=, and >>>=. This optimization can be particularly significant if the pattern occurs during a loop.

*Instances (8)*:
```solidity
File: example/DelegateRegistry.sol

279:                     delegations_[i] = Delegation({type_: DelegationType.NONE, to: address(0), from: address(0), rights: "", amount: 0, contract_: address(0), tokenId: 0});

317:                 contents[i] = tempValue;

389:                 filteredHashes[count++] = hash;

397:                 delegations_[i] = Delegation({

420:                 filteredHashes[count++] = hash;

```

```solidity
File: example/DelegateToken.sol

145:         accountOperator[msg.sender][operator] = approved;

```

```solidity
File: example/DelegateTokenStorageHelpers.sol

39:         delegateTokenInfo[delegateTokenId][PACKED_INFO_POSITION] = (uint256(uint160(approved)) << 96) | expiry;

118:         if (delegateTokenInfo[delegateTokenId][REGISTRY_HASH_POSITION] == ID_AVAILABLE) return;

```


### [GAS-3] Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess
Use uint256(1) and uint256(2) for true/false to avoid a Gwarmaccess (100 gas), and to avoid Gsset (20000 gas) when changing from ‘false’ to ‘true’, after having been ‘true’ in the past. See [source](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/58f635312aa21f947cae5f8578638a85aa2519f5/contracts/security/ReentrancyGuard.sol#L23-L27).

*Instances (4)*:
```solidity
File: example/DelegateToken.sol

39:     mapping(address account => mapping(address operator => bool enabled)) internal accountOperator;

```

```solidity
File: example/DelegateTokenStorageHelpers.sol

122:     function revertNotOperator(mapping(address account => mapping(address operator => bool enabled)) storage accountOperator, address account) internal view {

144:         mapping(address account => mapping(address operator => bool enabled)) storage accountOperator,

157:         mapping(address account => mapping(address operator => bool enabled)) storage accountOperator,

```

### [GAS-4] State variables only set in the constructor should be declared immutable
Avoids a Gsset (20000 gas) in the constructor, and replaces the first access in each transaction (Gcoldsload - 2100 gas) and each access thereafter (Gwarmacces - 100 gas) with a PUSH32 (3 gas).While strings are not value types, and therefore cannot be immutable/constant if not hard-coded outside of the constructor, the same behavior can be achieved by making the current contract abstract with virtual functions for the string accessors, and having a child contract override the functions with the hard-coded implementation-specific values.

*Instances (2)*:
```solidity
File: example/DelegateToken.sol

52:     constructor(Structs.DelegateTokenParameters memory parameters) {

```

```solidity
File: example/PrincipalToken.sol

20:     constructor(address setDelegateToken, address setMarketMetadata) {

```
### <a name="GAS-5"></a>[GAS-5] keccak256() hash of literals should only be computed once
The result of the hash should be stored in an immutable variable, and the variable should be used instead. If the hash is being used as a part of a function selector, the cast to bytes4 should also only be done once.

*Instances (1)*:
```solidity
File: example/DelegateTokenLib.sol

114:         return uint256(keccak256(abi.encode(caller, salt)));

```

### [GAS-6] Using storage instead of memory in array saves gas
When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct

*Instances (6)*:
```solidity
File: example/CreateOfferer.sol

56:         returns (SpentItem[] memory offer, ReceivedItem[] memory consideration)

180:         returns (SpentItem[] memory offer, ReceivedItem[] memory consideration)

241:     function getSeaportMetadata() external pure returns (string memory, Schema[] memory) {

```

```solidity
File: example/CreateOffererLib.sol

349:         returns (SpentItem[] memory offer, ReceivedItem[] memory consideration)

```

```solidity
File: example/DelegateRegistry.sol

384:         bytes32[] memory filteredHashes = new bytes32[](hashesLength);

415:         bytes32[] memory filteredHashes = new bytes32[](hashesLength);

```

### <a name="GAS-7"></a>[GAS-7] `++i` costs less gas than `i++`, especially when it's used in `for`-loops (`--i`/`i--` too)
*Saves 5 gas per loop*

*Instances (2)*:
```solidity
File: example/DelegateRegistry.sol

389:                 filteredHashes[count++] = hash;

420:                 filteredHashes[count++] = hash;

```

### [GAS-8] Dont transfer with zero amount to save gas
In Solidity, unnecessary operations can waste gas. For example, a transfer function without a zero amount check uses gas even if called with a zero amount, since the contract state remains unchanged. Implementing a zero amount check avoids these unnecessary function calls, saving gas and improving efficiency.

*Instances (8)*:
```solidity
File: example/CreateOfferer.sol

54:         checkStage(Enums.Stage.generate, Enums.Stage.transfer)

89:     function transferFrom(address from, address targetTokenReceiver, uint256 createOrderHashAsTokenId) external checkStage(Enums.Stage.transfer, Enums.Stage.ratify) {

```

```solidity
File: example/DelegateToken.sol

180:             RegistryHelpers.transferERC721(delegateRegistry, registryHash, from, newRegistryHash, to, underlyingRights, underlyingContract, underlyingTokenId);

184:             RegistryHelpers.transferERC20(

198:             RegistryHelpers.transferERC1155(

369:             IERC721(underlyingContract).transferFrom(address(this), msg.sender, erc721UnderlyingTokenId);

393:             IERC721(info.tokenContract).transferFrom(address(this), info.receiver, info.tokenId);

```

```solidity
File: example/DelegateTokenTransferHelpers.sol

41:         IERC721(underlyingContract).transferFrom(msg.sender, address(this), underlyingTokenId);

```
