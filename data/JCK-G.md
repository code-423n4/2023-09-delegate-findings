# Gas Optimizations

| Number | Issue | Instances | Total gas saved |
|--------|-------|-----------|-----------------|
|[G-01]| Don’t cache value if it is only used once  | 6 |  |
|[G-02]| Structs can be packed into fewer storage slots  | 2 | 4200 |
|[G-03]| Use hardcode address instead address(this)  | 24 |  |
|[G-04]| uint256 variable initialization to default value of 0 can be omitted  | 8 |  |
|[G-05]| Arithmatic operations can be unchecked, since there is no underflow or overflow  | 1 | 85 |
|[G-06]| Public Functions To External  | 1 |  |
|[G-07]| Use assembly in place of abi.decode to extract calldata values more efficiently  | 2 | 224 |
|[G-08]| Duplicate code should be remove to save deployment gas  | 1 |  |
|[G-09]| Combine events to save 2 Glogtopic (375 gas)  | 3 | 1125 |
|[G-10]| Using a positive conditional flow to save a NOT opcode  | 14 | 2100 |
|[G-11]| internal functions not called by the contract should be removed to save deployment gas  | 14 |  |
|[G-12]| State variables which are not modified within functions should be set as constant or immutable for values set at deployment  | 7 | 70000 |
|[G-13]| Don’t apply the same value to state variables  | 3 | 6300 |
|[G-14]| Inverting the condition of an if-else-statement wastes gas  | 10 | 30 |
|[G-15]| abi.encode() is less efficient than abi.encodePacked()  | 9 | 900 |
|[G-16]| Use assembly to validate msg.sender  | 5 | 15 |
|[G-17]| Don’t initialize variables with default value  | 9 |  |
|[G-18]| Use assembly for loops  | 5 |  |
|[G-19]| Use assembly to make more efficient back-to-back calls  | 2 |  |
|[G-20]| Use a do while loop instead of a for loop  | 5 |  |
|[G-21]| A modifier used only once and not being inherited should be inlined to save gas  | 2 |  |

## [G-01] Don’t cache value if it is only used once

If a value is only intended to be used once then it should not be cached. Caching the value will result in unnecessary stack manipulation.

### don't cache they value of  Storage.POSITIONS_FIRST_PACKED; and Storage.POSITIONS_SECOND_PACKED; because they used only once during they function

```solidity
file:   blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol

358    function _writeDelegationAddresses(bytes32 location, address from, address to, address contract_) internal {
        (bytes32 firstSlot, bytes32 secondSlot) = Storage.packAddresses(from, to, contract_);
        uint256 firstPacked = Storage.POSITIONS_FIRST_PACKED;
        uint256 secondPacked = Storage.POSITIONS_SECOND_PACKED;
        assembly {
            sstore(add(location, firstPacked), firstSlot)
            sstore(add(location, secondPacked), secondSlot)
        }
    }

```

<https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L358-L366>

### don't cache they value of  Storage.CLEAN_ADDRESS; and type(uint256).max << 160; because they used only once during they function

```solidity
file:    blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol

369       function _updateFrom(bytes32 location, address from) internal {
        uint256 firstPacked = Storage.POSITIONS_FIRST_PACKED;
        uint256 cleanAddress = Storage.CLEAN_ADDRESS;
        uint256 cleanUpper12Bytes = type(uint256).max << 160;
        assembly {
            let slot := and(sload(add(location, firstPacked)), cleanUpper12Bytes)
            sstore(add(location, firstPacked), or(slot, and(from, cleanAddress)))
        }
    }

```

<https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L369-L377>

### don't cache they value of Storage.POSITIONS_FIRST_PACKED; because they used only once during they function

```solidity
file:  blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol

444    function _loadFrom(bytes32 location) internal view returns (address) {
        bytes32 data;
        uint256 firstPacked = Storage.POSITIONS_FIRST_PACKED;
        assembly {
            data := sload(add(location, firstPacked))
        }
        return Storage.unpackAddress(data);
    }

```

<https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L444-L451>

### don't cache they value of Storage.POSITIONS_FIRST_PACKED; and  Storage.POSITIONS_SECOND_PACKED; because they used only once during they function

```solidity
file:   blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol

459      function _loadDelegationAddresses(bytes32 location) internal view returns (address from, address  to, address contract_) {
        bytes32 firstSlot;
        bytes32 secondSlot;
        uint256 firstPacked = Storage.POSITIONS_FIRST_PACKED;
        uint256 secondPacked = Storage.POSITIONS_SECOND_PACKED;
        assembly {
            firstSlot := sload(add(location, firstPacked))
            secondSlot := sload(add(location, secondPacked))
        }
        (from, to, contract_) = Storage.unpackAddresses(firstSlot, secondSlot);
    }

```

<https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L459-L469>

### don't cache they value of StorageHelpers.readRegistryHash(delegateTokenInfo, delegateTokenId); because they used only once during they function

```solidity
file:  blob/main/src/DelegateToken.sol

239    function isApprovedOrOwner(address spender, uint256 delegateTokenId) external view returns (bool) {
        bytes32 registryHash = StorageHelpers.readRegistryHash(delegateTokenInfo, delegateTokenId);
        StorageHelpers.revertNotMinted(registryHash, delegateTokenId);
        address delegateTokenHolder = RegistryHelpers.loadTokenHolder(delegateRegistry, registryHash);
        return spender == delegateTokenHolder || accountOperator[delegateTokenHolder][spender] || StorageHelpers.readApproved(delegateTokenInfo, delegateTokenId) == spender;
    }

```

<https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L239-L244>

### don't cache they value of uint96(delegateTokenInfo[delegateTokenId][PACKED_INFO_POSITION]); because they used only once during they function

```solidity
file:  blob/main/src/libraries/DelegateTokenStorageHelpers.sol

37   function writeApproved(mapping(uint256 delegateTokenId => uint256[3] info) storage delegateTokenInfo, uint256 delegateTokenId, address approved) internal {
        uint96 expiry = uint96(delegateTokenInfo[delegateTokenId][PACKED_INFO_POSITION]);
        delegateTokenInfo[delegateTokenId][PACKED_INFO_POSITION] = (uint256(uint160(approved)) << 96) | expiry;
    }

```

<https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenStorageHelpers.sol#L37-L40>

## [G-02] Structs can be packed into fewer storage slots

The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to each other in storage and this will pack the values together into a single 32 byte storage slot (if values combined are <= 32 bytes). If the variables packed together are retrieved together in functions (more likely with structs), we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a Gwarmaccess (100 gas) versus a Gcoldsload (2100 gas).

```solidity
file:  blob/main/src/libraries/DelegateTokenLib.sol

20    struct DelegateInfo {
        address principalHolder;
        IDelegateRegistry.DelegationType tokenType;
        address delegateHolder;
        uint256 amount;
        address tokenContract;
        uint256 tokenId;
        bytes32 rights;
        uint256 expiry;
    }

31   struct FlashInfo {
        address receiver; // The address to receive the loaned assets
        address delegateHolder; // The holder of the delegation
        IDelegateRegistry.DelegationType tokenType; // The type of contract, e.g. ERC20
        address tokenContract; // The contract of the underlying being loaned
        uint256 tokenId; // The tokenId of the underlying being loaned, if applicable
        uint256 amount; // The amount being lent, if applicable
        bytes data; // Arbitrary data structure, intended to contain user-defined parameters
    }

```

<https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenLib.sol#L20-L29>

## [G-03] Use hardcode address instead address(this)

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.

```solidity
file:  blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol

37    (success, results[i]) = address(this).delegatecall(data[i]);

```

<https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L37>

```solidity
file:  blob/main/src/CreateOfferer.sol

90    if (from != address(this)) revert Errors.FromNotCreateOfferer(from);

138   if (IERC20(erc20Order.info.tokenContract).allowance(address(this), address(delegateToken)) != 0) {
    
```

<https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L90>

```solidity
file:  blob/main/src/DelegateToken.sol

84   if (address(this) == operator) return IERC721Receiver.onERC721Received.selector;

178  newRegistryHash = RegistryHashes.erc721Hash(address(this), underlyingRights, to, underlyingTokenId, underlyingContract);

182  newRegistryHash = RegistryHashes.erc20Hash(address(this), underlyingRights, to, underlyingContract);

196  newRegistryHash = RegistryHashes.erc1155Hash(address(this), underlyingRights, to, underlyingTokenId, underlyingContract);

307  newRegistryHash = RegistryHashes.erc721Hash(address(this), delegateInfo.rights, delegateInfo.delegateHolder, delegateInfo.tokenId, delegateInfo.tokenContract);

312  newRegistryHash = RegistryHashes.erc20Hash(address(this), delegateInfo.rights, delegateInfo.delegateHolder, delegateInfo.tokenContract);

317  newRegistryHash = RegistryHashes.erc1155Hash(address(this), delegateInfo.rights, delegateInfo.delegateHolder, delegateInfo.tokenId, delegateInfo.tokenContract);

369  IERC721(underlyingContract).transferFrom(address(this), msg.sender, erc721UnderlyingTokenId);

384  IERC1155(underlyingContract).safeTransferFrom(address(this), msg.sender, erc11551UnderlyingTokenId, erc1155UnderlyingAmount, "");

393  IERC721(info.tokenContract).transferFrom(address(this), info.receiver, info.tokenId);

406  IERC1155(info.tokenContract).safeTransferFrom(address(this), info.receiver, info.tokenId, info.amount, "");

```

<https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L84>

```solidity
file:  blob/main/src/libraries/CreateOffererLib.sol

259    uint256 requestedDelegateId = DelegateTokenHelpers.delegateIdNoRevert(address(this), createOrderHash);

280    delegateTokenId = IDelegateToken(delegateToken).getDelegateId(address(this), createOrderHash);

314    ) != keccak256(abi.encode(IDelegateToken(delegateToken).getDelegateInfo(DelegateTokenHelpers.delegateIdNoRevert(address(this), identifier))))

352    if (minimumReceived[0].itemType != ItemType.ERC721 || minimumReceived[0].token != address(this) || minimumReceived[0].amount != 1) {

366    recipient: payable(address(this))

```

<https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L259>

```solidity
file:  blob/main/src/libraries/DelegateTokenTransferHelpers.sol

41   IERC721(underlyingContract).transferFrom(msg.sender, address(this), underlyingTokenId);
    }

52   if (IERC20(underlyingContract).allowance(msg.sender, address(this)) < underlyingAmount) {

58   SafeERC20.safeTransferFrom(IERC20(underlyingContract), msg.sender, address(this), pullAmount);

71   IERC1155(underlyingContract).safeTransferFrom(msg.sender, address(this), underlyingTokenId, pullAmount, "");

78   if (erc1155Pulled.flag == ERC1155_PULLED && address(this) == operator) {


```

<https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenTransferHelpers.sol#L41>

## [G-04] uint256 variable initialization to default value of 0 can be omitted

There is no need to initialize variables to their default values during declaration, since they are any way initialized to default value once declared.

```solidity
file: blob/main/src/libraries/DelegateTokenRegistryHelpers.sol

153   uint256 availableAmount = 0;

163   uint256 availableAmount = 0;

```

<https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenRegistryHelpers.sol#L153>

```solidity
file:  blob/main/src/libraries/DelegateTokenStorageHelpers.sol

15     uint256 internal constant ID_AVAILABLE = 0;

22     uint256 internal constant REGISTRY_HASH_POSITION = 0;

```

<https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenStorageHelpers.sol#L15>

```solidity
file:  blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/libraries/RegistryStorage.sol

10     uint256 internal constant POSITIONS_FIRST_PACKED = 0;

```

<https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/libraries/RegistryStorage.sol#L10>

```solidity
file:  blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/libraries/RegistryHashes.sol

31    uint256 internal constant DELEGATION_SLOT = 0;

```

<https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/libraries/RegistryHashes.sol#L31>

```solidity
file:  blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol

381  uint256 count = 0;

412  uint256 count = 0;

```

<https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L381>

## [G-05] Arithmatic operations can be unchecked, since there is no underflow or overflow

```solidity
file:  blob/main/src/libraries/CreateOffererLib.sol

328    return block.timestamp + expiryLength;

```

<https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L328>

## [G-06] Public Functions To External

The following functions could be set external to save gas and improve code quality.
External call cost is less expensive than of public functions.

```solidity  
file:  blob/main/src/PrincipalToken.sol

54     function tokenURI(uint256 id) public view override returns (string memory) {

```

<https://github.com/code-423n4/2023-09-delegate/blob/main/src/PrincipalToken.sol#L54>

## [G-07] Use assembly in place of abi.decode to extract calldata values more efficiently

Instead of using abi.decode, we can use assembly to decode our desired calldata values directly. This will allow us to avoid decoding calldata values that we will not use.

```solidity
file:  blob/main/src/CreateOfferer.sol

59     Structs.Context memory decodedContext = abi.decode(context, (Structs.Context));

```

<https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L59>

```solidity
file:  blob/main/src/libraries/CreateOffererLib.sol

298    CreateOffererStructs.Context memory decodedContext = abi.decode(context, (CreateOffererStructs.Context));

```

<https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L298>

## [G-08] Duplicate code should be remove to save deployment gas

```solidity
file:   blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol

344       function _writeDelegation(bytes32 location, uint256 position, bytes32 data) internal {
        assembly {
            sstore(add(location, position), data)
        }
    }

    /// @dev Helper function that writes uint256 data to delegation data location at array position
    function _writeDelegation(bytes32 location, uint256 position, uint256 data) internal {
        assembly {
            sstore(add(location, position), data)
        }
    }

```

<https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L344-L355>

## [G-09] Combine events to save 2 Glogtopic (375 gas)

The events below are only emitted once in the handleRewards function. We can combine the events into one singular event to save two Glogtopic (375 gas) that would otherwise be paid for the additional two events.

```solidity
file:  blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol

59   emit DelegateAll(msg.sender, to, rights, enable);

78   emit DelegateContract(msg.sender, to, contract_, rights, enable);

98   emit DelegateERC721(msg.sender, to, contract_, tokenId, rights, enable);

```

<https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L59>

```solidity
file:  blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/libraries/RegistryOps.sol

6     function max(uint256 x, uint256 y) internal pure returns (uint256 z) {

```

<https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/libraries/RegistryOps.sol#L6>

## [G-10] Using a positive conditional flow to save a NOT opcode

Estimated savings: 3 gas
Max savings according to yarn profile: 150 gas

```solidity
file:  blob/main/src/CreateOfferer.sol

95    if (!(erc721Order.info.targetToken == Enums.TargetToken.delegate || erc721Order.info.targetToken == Enums.TargetToken.principal)) {

117   if (!(erc20Order.info.targetToken == Enums.TargetToken.delegate || erc20Order.info.targetToken == Enums.TargetToken.principal)) {

121   if (!IERC20(erc20Order.info.tokenContract).approve(address(delegateToken), erc20Order.amount)) {

143   if (!(erc1155Order.info.targetToken == Enums.TargetToken.delegate || erc1155Order.info.targetToken == Enums.TargetToken.principal)) {

```

<https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L95>

```solidity
file:  blob/main/src/libraries/CreateOffererLib.sol

351    if (!(minimumReceived.length == 1 && maximumSpent.length == 1)) revert CreateOffererErrors.NoBatchWrapping();

```

<https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L351>

```solidity
file: blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol

38    if (!success) revert MulticallFailed();

166   if (!_invalidFrom(from)) {

168   if (!Ops.or(rights == "", valid)) valid = _validateFrom(Hashes.allLocation(from, rights, to), from);

179   if (!_invalidFrom(from)) {

181   if (!Ops.or(rights == "", valid)) {

194   if (!_invalidFrom(from)) {

197   if (!Ops.or(rights == "", valid)) {

211   if (!_invalidFrom(from)) {

215   if (!Ops.or(rights == "", amount == type(uint256).max)) {

```

<https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L38>

## [G‑11] internal functions not called by the contract should be removed to save deployment gas

If the functions are required by an interface, the contract should inherit from that interface and use the override keyword.

```solidity
file:  blob/main/src/libraries/CreateOffererLib.sol

184   function processNonce(CreateOffererStructs.Nonce storage nonce, uint256 contractNonce) internal {

```

<https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L184>

```solidity
file:  blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/libraries/RegistryStorage.sol

34    function packAddresses(address from, address to, address contract_) internal pure returns (bytes32 firstPacked, bytes32 secondPacked) {

50    function unpackAddresses(bytes32 firstPacked, bytes32 secondPacked) internal pure returns (address from, address to, address contract_) {

```

<https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/libraries/RegistryStorage.sol#L34>

```solidity
file:  blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/libraries/RegistryHashes.sol

41   function decodeType(bytes32 inputHash) internal pure returns (IDelegateRegistry.DelegationType decodedType) {

73   function allHash(address from, bytes32 rights, address to) internal pure returns (bytes32 hash) {

95   function allLocation(address from, bytes32 rights, address to) internal pure returns (bytes32 computedLocation) {

120  function contractHash(address from, bytes32 rights, address to, address contract_) internal pure returns (bytes32 hash) {

143  function contractLocation(address from, bytes32 rights, address to, address contract_) internal pure returns (bytes32 computedLocation) {

170  function erc721Hash(address from, bytes32 rights, address to, uint256 tokenId, address contract_) internal pure returns (bytes32 hash) {

195   function erc721Location(address from, bytes32 rights, address to, uint256 tokenId, address contract_) internal pure returns (bytes32 computedLocation) {

222   function erc20Hash(address from, bytes32 rights, address to, address contract_) internal pure returns (bytes32 hash) {

245   function erc20Location(address from, bytes32 rights, address to, address contract_) internal pure returns (bytes32 computedLocation) {

272   function erc1155Hash(address from, bytes32 rights, address to, uint256 tokenId, address contract_) internal pure returns (bytes32 hash) {

297   function erc1155Location(address from, bytes32 rights, address to, uint256 tokenId, address contract_) internal pure returns (bytes32 computedLocation) {

```

<https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/libraries/RegistryHashes.sol#L41>

## [G-12] State variables which are not modified within functions should be set as constant or immutable for values set at deployment

Cache such variables and perform operations on them, if operations include modifications to the state variable(s) then remember to equate the state variable to it's cached counterpart at the e

```solidity
file:  blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/libraries/RegistryStorage.sol

6       address internal constant DELEGATION_EMPTY = address(0);

7      address internal constant DELEGATION_REVOKED = address(1);

    /// @dev Standardizes storage positions of delegation data
    
10    uint256 internal constant POSITIONS_FIRST_PACKED = 0;

11    uint256 internal constant POSITIONS_SECOND_PACKED = 1

12    uint256 internal constant POSITIONS_RIGHTS = 2;

13    uint256 internal constant POSITIONS_TOKEN_ID = 3;

14    uint256 internal constant POSITIONS_AMOUNT = 4;

```

<https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/libraries/RegistryStorage.sol#L6>

## [G‑13] Don’t apply the same value to state variables

If _whenDefault is already NEVER, it’ll save 2100 gas to not set it to that value again

```solidity
file:  blob/main/src/libraries/DelegateTokenStorageHelpers.sol

15   uint256 internal constant ID_AVAILABLE = 0;

22   uint256 internal constant REGISTRY_HASH_POSITION = 0;

```

<https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenStorageHelpers.sol#L15>

```solidity
file:  blob/main/src/libraries/DelegateTokenStorageHelpers.sol

16    uint256 internal constant ID_USED = 1;

23    uint256 internal constant PACKED_INFO_POSITION = 1;

31    uint256 internal constant MINT_NOT_AUTHORIZED = 1;

```

<https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenStorageHelpers.sol#L16>

```solidity
file:  blob/main/src/libraries/DelegateTokenStorageHelpers.sol
 
24    uint256 internal constant UNDERLYING_AMOUNT_POSITION = 2

32    uint256 internal constant MINT_AUTHORIZED = 2;

```

<https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenStorageHelpers.sol#L24>

## [G-14] Inverting the condition of an if-else-statement wastes gas

Flipping the true and false blocks instead saves 3 gas.

```solidity
file:  blob/main/src/CreateOfferer.sol

104   principalHolder: erc721Order.info.targetToken == Enums.TargetToken.principal ? targetTokenReceiver : transientState.receivers.fulfiller,

106   delegateHolder: erc721Order.info.targetToken == Enums.TargetToken.delegate ? targetTokenReceiver : transientState.receivers.fulfiller,

128   principalHolder: erc20Order.info.targetToken == Enums.TargetToken.principal ? targetTokenReceiver : transientState.receivers.fulfiller,
                    tokenType: tokenType,

130   delegateHolder: erc20Order.info.targetToken == Enums.TargetToken.delegate ? targetTokenReceiver : transientState.receivers.fulfiller,
                    amount: erc20Order.amount,

152   principalHolder: erc1155Order.info.targetToken == Enums.TargetToken.principal ? targetTokenReceiver : transientState.receivers.fulfiller,
                    tokenType: tokenType,
                
154   delegateHolder: erc1155Order.info.targetToken == Enums.TargetToken.delegate ? targetTokenReceiver : transientState.receivers.fulfiller,
                    amount: erc1155Order.amount,

```

<https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L104>

```solidity
file:  blob/main/src/libraries/CreateOffererLib.sol

305    principalHolder: decodedContext.targetToken == CreateOffererEnums.TargetToken.principal ? receivers.targetTokenReceiver : receivers.fulfiller,

306    delegateHolder: decodedContext.targetToken == CreateOffererEnums.TargetToken.delegate ? receivers.targetTokenReceiver : receivers.fulfiller,

310    tokenId: (tokenType != IDelegateRegistry.DelegationType.ERC20) ? consideration.identifier : 0,

311    amount: (tokenType != IDelegateRegistry.DelegationType.ERC721) ? consideration.amount : 0

```

<https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L305>

## [G-15] abi.encode() is less efficient than abi.encodePacked()

Consider changing it if possible.

```solidity
file:  blob/main/src/CreateOfferer.sol

98    Helpers.validateCreateOrderHash(targetTokenReceiver, createOrderHashAsTokenId, abi.encode(erc721Order), tokenType);

120   Helpers.validateCreateOrderHash(targetTokenReceiver, createOrderHashAsTokenId, abi.encode(erc20Order), tokenType);

146   Helpers.validateCreateOrderHash(targetTokenReceiver, createOrderHashAsTokenId, abi.encode(erc1155Order), tokenType);

201   Helpers.calculateOrderHashAndId(delegateToken, targetTokenReceiver, conduit, abi.encode(erc721Order), IDelegateRegistry.DelegationType.ERC721);

237   Helpers.calculateOrderHashAndId(delegateToken, targetTokenReceiver, conduit, abi.encode(erc1155Order), IDelegateRegistry.DelegationType.ERC1155);

```

<https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L98>

```solidity
file: blob/main/src/libraries/CreateOffererLib.sol

302    abi.encode(

320    ) != keccak256(abi.encode(IDelegateToken(delegateToken).getDelegateInfo(DelegateTokenHelpers.delegateIdNoRevert(address(this), identifier))))

398    uint256 hashWithoutType = uint256(keccak256(abi.encode(targetTokenReceiver, conduit, createOrderInfo)));

```

<https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L302>

```solidity
file:  blob/main/src/libraries/DelegateTokenLib.sol

114    return uint256(keccak256(abi.encode(caller, salt)));

```

<https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenLib.sol#L114>

## [G-16] Use assembly to validate msg.sender

We can use assembly to efficiently validate msg.sender for the didPay and uniswapV3SwapCallback functions with the least amount of opcodes necessary. Additionally, we can use xor() instead of iszero(eq()), saving 3 gas. We can also potentially save gas on the unhappy path by using scratch space to store the error selector, potentially avoiding memory expansion costs.

```solidity
file:  blob/main/src/PrincipalToken.sol

28    if (msg.sender == delegateToken) return;

```

<https://github.com/code-423n4/2023-09-delegate/blob/main/src/PrincipalToken.sol#L28>

```solidity
file:  blob/main/src/libraries/DelegateTokenStorageHelpers.sol

113   if (msg.sender == principalToken) return;

123   if (msg.sender == account || accountOperator[account][msg.sender]) return;

149   if (msg.sender == account || accountOperator[account][msg.sender] || msg.sender == readApproved(delegateTokenInfo, delegateTokenId)) return;

```

<https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenStorageHelpers.sol#L113>

```solidity
file: blob/main/src/libraries/DelegateTokenTransferHelpers.sol

35    if (IERC721(underlyingContract).ownerOf(underlyingTokenId) != msg.sender) {

```

<https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenTransferHelpers.sol#L35>

## [G-17] Don’t initialize variables with default value

```solidity
file:  code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol

175    bytes32 newRegistryHash = 0;

305    bytes32 newRegistryHash = 0;

```

<https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L175>

```solidity
file:  blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol


35   for (uint256 i = 0; i < data.length; ++i) {

275  for (uint256 i = 0; i < hashes.length; ++i) {

312  for (uint256 i = 0; i < length; ++i) {

386  for (uint256 i = 0; i < hashesLength; ++i) {

393  for (uint256 i = 0; i < count; ++i) {

417  for (uint256 i = 0; i < hashesLength; ++i) {

423  for (uint256 i = 0; i < count; ++i) {

```

<https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L35>

## [G-18] Use assembly for loops

We can use assembly to write a more gas efficient loop. See the final diffs for comments regarding the assembly code.

```solidity
file:   blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol

35    for (uint256 i = 0; i < data.length; ++i) {
                //slither-disable-next-line calls-loop,delegatecall-loop
                (success, results[i]) = address(this).delegatecall(data[i]);
                if (!success) revert MulticallFailed();
            }

312   for (uint256 i = 0; i < length; ++i) {
                tempLocation = locations[i];
                assembly {
                    tempValue := sload(tempLocation)
                }
                contents[i] = tempValue;
            }
    
386   for (uint256 i = 0; i < hashesLength; ++i) {
                hash = hashes[i];
                if (_invalidFrom(_loadFrom(Hashes.location(hash)))) continue;
                filteredHashes[count++] = hash;
            }

417   for (uint256 i = 0; i < hashesLength; ++i) {
                hash = hashes[i];
                if (_invalidFrom(_loadFrom(Hashes.location(hash)))) continue;
                filteredHashes[count++] = hash;
            }


423     for (uint256 i = 0; i < count; ++i) {
                validHashes[i] = filteredHashes[i];
            }
            
```

<https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L35-L39>

## [G-19] Use assembly to make more efficient back-to-back calls

In the instance below, two external calls, both of which take two function parameters, are performed. We can potentially avoid memory expansion costs by using assembly to utilize scratch space + free memory pointer memory offsets for the function calls. We will use the same memory space for both function calls.

Note: In order to do this optimization safely we will cache the free memory pointer value and restore it once we are done with our function calls.

```solidity
file:  libraries/DelegateTokenRegistryHelpers.sol

37      IDelegateRegistry(delegateRegistry).readSlot(bytes32(registryLocation + RegistryStorage.POSITIONS_FIRST_PACKED)),
38      IDelegateRegistry(delegateRegistry).readSlot(bytes32(registryLocation + RegistryStorage.POSITIONS_SECOND_PACKED))

```

<https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenRegistryHelpers.sol#L37-L38>

```solidity
file:

57       IDelegateRegistry(delegateRegistry).readSlot(bytes32(registryLocation + RegistryStorage.POSITIONS_FIRST_PACKED)),
58       IDelegateRegistry(delegateRegistry).readSlot(bytes32(registryLocation + RegistryStorage.POSITIONS_SECOND_PACKED))

```

<https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenRegistryHelpers.sol#L57-L58>

## [G-20] Use a do while loop instead of a for loop

A do while loop will cost less gas since the condition is not being checked for the first iteration.

Note: Other optimizations, such as caching length, precrementing, and using unchecked blocks are not included in the Diff since they are included in the automated findings report.

```solidity
file:   blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol

35    for (uint256 i = 0; i < data.length; ++i) {
                //slither-disable-next-line calls-loop,delegatecall-loop
                (success, results[i]) = address(this).delegatecall(data[i]);
                if (!success) revert MulticallFailed();
            }

312   for (uint256 i = 0; i < length; ++i) {
                tempLocation = locations[i];
                assembly {
                    tempValue := sload(tempLocation)
                }
                contents[i] = tempValue;
            }
    
386   for (uint256 i = 0; i < hashesLength; ++i) {
                hash = hashes[i];
                if (_invalidFrom(_loadFrom(Hashes.location(hash)))) continue;
                filteredHashes[count++] = hash;
            }

417   for (uint256 i = 0; i < hashesLength; ++i) {
                hash = hashes[i];
                if (_invalidFrom(_loadFrom(Hashes.location(hash)))) continue;
                filteredHashes[count++] = hash;
            }


423     for (uint256 i = 0; i < count; ++i) {
                validHashes[i] = filteredHashes[i];
            }
            
```

<https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L35-L39>

## [G-21] A modifier used only once and not being inherited should be inlined to save gas

```solidity
file:  blob/main/src/libraries/CreateOffererLib.sol

156     modifier checkStage(CreateOffererEnums.Stage currentStage, CreateOffererEnums.Stage nextStage) {

169     modifier onlySeaport(address caller) {

```

<https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L156>
