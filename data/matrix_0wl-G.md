## Gas Optimizations

|        | Issue                                                                                                                  |
| ------ | :--------------------------------------------------------------------------------------------------------------------- |
| GAS-1  | `ABI.ENCODE()` IS LESS EFFICIENT THAN `ABI.ENCODEPACKED()`                                                             |
| GAS-2  | <ARRAY>.LENGTH SHOULD NOT BE LOOKED UP IN EVERY LOOP OF A FOR-LOOP                                                     |
| GAS-3  | Setting the constructor to payable                                                                                     |
| GAS-4  | USE FUNCTION INSTEAD OF MODIFIERS                                                                                      |
| GAS-5  | Don't initialize variables with default value                                                                          |
| GAS-6  | MODIFIERS ARE REDUNDANT IF USED ONLY ONCE OR NOT USED AT ALL                                                           |
| GAS-7  | Functions guaranteed to revert when called by normal users can be marked payable                                       |
| GAS-8  | `++i` costs less gas than `i++`, especially when it's used in `for`-loops (`--i`/`i--` too) = for-loop and while-loops |
| GAS-9  | The increment in for loop postcondition can be made unchecked                                                          |
| GAS-10 | Use a more recent version of solidity                                                                                  |
| GAS-11 | Usage of `uint`/`int` smaller than 32 bytes (256 bits) incurs overhead                                                 |
| GAS-12 | USE BYTES32 INSTEAD OF STRING                                                                                          |

### [GAS-1] `ABI.ENCODE()` IS LESS EFFICIENT THAN `ABI.ENCODEPACKED()`

#### Description:

Use `abi.encodePacked()` where possible to save gas

#### **Proof Of Concept**

```solidity
File: src/CreateOfferer.sol

98:             Helpers.validateCreateOrderHash(targetTokenReceiver, createOrderHashAsTokenId, abi.encode(erc721Order), tokenType);

120:             Helpers.validateCreateOrderHash(targetTokenReceiver, createOrderHashAsTokenId, abi.encode(erc20Order), tokenType);

146:             Helpers.validateCreateOrderHash(targetTokenReceiver, createOrderHashAsTokenId, abi.encode(erc1155Order), tokenType);

201:             Helpers.calculateOrderHashAndId(delegateToken, targetTokenReceiver, conduit, abi.encode(erc721Order), IDelegateRegistry.DelegationType.ERC721);

219:             Helpers.calculateOrderHashAndId(delegateToken, targetTokenReceiver, conduit, abi.encode(erc20Order), IDelegateRegistry.DelegationType.ERC20);

237:             Helpers.calculateOrderHashAndId(delegateToken, targetTokenReceiver, conduit, abi.encode(erc1155Order), IDelegateRegistry.DelegationType.ERC1155);

```

```solidity
File: src/libraries/CreateOffererLib.sol

314:             ) != keccak256(abi.encode(IDelegateToken(delegateToken).getDelegateInfo(DelegateTokenHelpers.delegateIdNoRevert(address(this), identifier))))

398:         uint256 hashWithoutType = uint256(keccak256(abi.encode(targetTokenReceiver, conduit, createOrderInfo)));

```

```solidity
File: src/libraries/DelegateTokenLib.sol

114:         return uint256(keccak256(abi.encode(caller, salt)));

```

### [GAS-2] <ARRAY>.LENGTH SHOULD NOT BE LOOKED UP IN EVERY LOOP OF A FOR-LOOP

#### Description:

If not cached, the solidity compiler will always read the length of the array during each iteration. That is, if it is a storage array, this is an extra sload operation (100 additional extra gas for each iteration except for the first) and if it is a memory array, this is an extra mload operation (3 additional gas for each iteration except for the first).

[Source](https://forum.openzeppelin.com/t/a-collection-of-gas-optimisation-tricks/19966/6)

[Source](https://code4rena.com/reports/2022-12-backed#g14--arraylength-should-not-be-looked-up-in-every-loop-of-a-for-loop)

#### **Proof Of Concept**

```solidity
File: lib/delegate-registry/src/DelegateRegistry.sol

35:             for (uint256 i = 0; i < data.length; ++i) {

275:             for (uint256 i = 0; i < hashes.length; ++i) {

```

### [GAS-3] Setting the constructor to payable

#### Description:

Saves ~13 gas per instance

#### **Proof Of Concept**

```solidity
File: src/CreateOfferer.sol

29:     constructor(Structs.Parameters memory parameters) Modifiers(parameters.seaport, Enums.Stage.generate) {

```

```solidity
File: src/DelegateToken.sol

52:     constructor(Structs.DelegateTokenParameters memory parameters) {

```

```solidity
File: src/PrincipalToken.sol

20:     constructor(address setDelegateToken, address setMarketMetadata) {

```

```solidity
File: src/libraries/CreateOffererLib.sol

145:     constructor(address setSeaport, CreateOffererEnums.Stage firstStage) {

```

### [GAS-4] USE FUNCTION INSTEAD OF MODIFIERS

#### **Proof Of Concept**

```solidity
File: src/libraries/CreateOffererLib.sol

156:     modifier checkStage(CreateOffererEnums.Stage currentStage, CreateOffererEnums.Stage nextStage) {

169:     modifier onlySeaport(address caller) {

```

### [GAS-5] Don't initialize variables with default value

#### Description:

If a variable is not set/initialized, the default value is assumed (0, false, 0x0 … depending on the data type). You are simply wasting gas if you directly initialize it with its default value.

#### **Proof Of Concept**

```solidity
File: lib/delegate-registry/src/DelegateRegistry.sol

35:             for (uint256 i = 0; i < data.length; ++i) {

275:             for (uint256 i = 0; i < hashes.length; ++i) {

312:             for (uint256 i = 0; i < length; ++i) {

381:         uint256 count = 0;

386:             for (uint256 i = 0; i < hashesLength; ++i) {

393:             for (uint256 i = 0; i < count; ++i) {

412:         uint256 count = 0;

417:             for (uint256 i = 0; i < hashesLength; ++i) {

423:             for (uint256 i = 0; i < count; ++i) {

```

```solidity
File: lib/delegate-registry/src/libraries/RegistryHashes.sol

31:     uint256 internal constant DELEGATION_SLOT = 0;

```

```solidity
File: lib/delegate-registry/src/libraries/RegistryStorage.sol

10:     uint256 internal constant POSITIONS_FIRST_PACKED = 0; //  | 4 bytes empty | first 8 bytes of contract address | 20 bytes of from address |

```

```solidity
File: src/libraries/DelegateTokenRegistryHelpers.sol

153:         uint256 availableAmount = 0;

163:         uint256 availableAmount = 0;

```

```solidity
File: src/libraries/DelegateTokenStorageHelpers.sol

15:     uint256 internal constant ID_AVAILABLE = 0;

22:     uint256 internal constant REGISTRY_HASH_POSITION = 0;

```

### [GAS-6] MODIFIERS ARE REDUNDANT IF USED ONLY ONCE OR NOT USED AT ALL

#### **Proof Of Concept**

```solidity
File: src/libraries/CreateOffererLib.sol

156:     modifier checkStage(CreateOffererEnums.Stage currentStage, CreateOffererEnums.Stage nextStage) {

169:     modifier onlySeaport(address caller) {

```

### [GAS-7] Functions guaranteed to revert when called by normal users can be marked payable

#### Description:

If a function modifier such as onlyOwner/onlyX is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2),DUP1(3),ISZERO(3),PUSH2(3),JUMPI(10),PUSH1(3),DUP1(3),REVERT(0),JUMPDEST(1),POP(2), which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

#### **Proof Of Concept**

```solidity
File: src/CreateOfferer.sol

55:         onlySeaport(msg.sender)

74:         onlySeaport(msg.sender)

179:         onlySeaport(caller)

```

```solidity
File: src/libraries/CreateOffererLib.sol

169:     modifier onlySeaport(address caller) {

```

### [GAS-8] `++i` costs less gas than `i++`, especially when it's used in `for`-loops (`--i`/`i--` too) = for-loop and while-loops

#### **Proof Of Concept**

```solidity
File: lib/delegate-registry/src/DelegateRegistry.sol

389:                 filteredHashes[count++] = hash;

420:                 filteredHashes[count++] = hash;

```

### [GAS-9] The increment in for loop postcondition can be made unchecked

#### Description:

This is only relevant if you are using the default solidity checked arithmetic.
the for loop postcondition, i.e., `i++` involves checked arithmetic, which is not required. This is because the value of i is always strictly less than `length <= 2**256 - 1`. Therefore, the theoretical maximum value of i to enter the for-loop body is `2**256 - 2`. This means that the `i++` in the for loop can never overflow. Regardless, the overflow checks are performed by the compiler.

Unfortunately, the Solidity optimizer is not smart enough to detect this and remove the checks.One can manually do this.

[Source](https://forum.openzeppelin.com/t/a-collection-of-gas-optimisation-tricks/19966/6)

#### **Proof Of Concept**

```solidity
File: lib/delegate-registry/src/DelegateRegistry.sol

389:                 filteredHashes[count++] = hash;

420:                 filteredHashes[count++] = hash;

```

### [GAS-10] Use a more recent version of solidity

#### **Proof Of Concept**

```solidity
File: src/libraries/CreateOffererLib.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: src/libraries/DelegateTokenLib.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: src/libraries/DelegateTokenRegistryHelpers.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: src/libraries/DelegateTokenStorageHelpers.sol

2: pragma solidity ^0.8.4;

```

```solidity
File: src/libraries/DelegateTokenTransferHelpers.sol

2: pragma solidity ^0.8.4;

```

### [GAS-11] Usage of `uint`/`int` smaller than 32 bytes (256 bits) incurs overhead

#### Description:

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size.
Each operation involving a `uint8` costs an extra 22-28 gas (depending on whether the other operand is also a variable of type `uint8`) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so. Use a larger size then downcast where needed.
https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html
Use a larger size then downcast where needed.

#### **Proof Of Concept**

```solidity
File: src/libraries/DelegateTokenStorageHelpers.sol

9:     uint256 internal constant MAX_EXPIRY = type(uint96).max;

23:     uint256 internal constant PACKED_INFO_POSITION = 1; // PACKED (address approved, uint96 expiry)

38:         uint96 expiry = uint96(delegateTokenInfo[delegateTokenId][PACKED_INFO_POSITION]);

38:         uint96 expiry = uint96(delegateTokenInfo[delegateTokenId][PACKED_INFO_POSITION]);

132:         return uint96(delegateTokenInfo[delegateTokenId][PACKED_INFO_POSITION]);

```

### [GAS-12] USE BYTES32 INSTEAD OF STRING

#### Description:

Use bytes32 instead of string to save gas whenever possible. String is a dynamic data structure and therefore is more gas consuming then bytes32.

#### **Proof Of Concept**

```solidity
File: src/CreateOfferer.sol

241:     function getSeaportMetadata() external pure returns (string memory, Schema[] memory) {

```

```solidity
File: src/DelegateToken.sol

217:     function name() external pure returns (string memory) {

222:     function symbol() external pure returns (string memory) {

227:     function tokenURI(uint256 delegateTokenId) external view returns (string memory) {

247:     function baseURI() external view returns (string memory) {

252:     function contractURI() external view returns (string memory) {

```

```solidity
File: src/PrincipalToken.sol

54:     function tokenURI(uint256 id) public view override returns (string memory) {

```
