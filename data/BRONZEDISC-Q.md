## QA
---

### Function Visibility [1]

- Order of Functions: Ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. Functions should be grouped according to their visibility and ordered: constructor, receive function (if exists), fallback function (if exists), external, public, internal, private. Within a grouping, place the view and pure functions last.

https://github.com/code-423n4/2023-09-delegate/blob/main/src/PrincipalToken.sol

```solidity
// place this internal function for last.
27:    function _checkDelegateTokenCaller() internal view {
```

---

### natSpec missing [2]

Some functions are missing @params or @returns. Specification Format.” These are written with a triple slash (///) or a double asterisk block(/** ... */) directly above function declarations or statements to generate documentation in JSON format for developers and end-users. It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). These comments contain different types of tags:
- @title: A title that should describe the contract/interface @author: The name of the author (for contract, interface) 
- @notice: Explain to an end user what this does (for contract, interface, function, public state variable, event) 
- @dev: Explain to a developer any extra details (for contract, interface, function, state variable, event) 
- @param: Documents a parameter (just like in doxygen) and must be followed by parameter name (for function, event)
- @return: Documents the return variables of a contract’s function (function, public state variable)
- @inheritdoc: Copies all missing tags from the base function and must be followed by the contract name (for function, public state variable)
- @custom…: Custom tag, semantics is application-defined (for everywhere)

https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenTransferHelpers.sol

```solidity
10:  library DelegateTokenTransferHelpers {
15:    function checkAndPullByType(Structs.Uint256 storage erc1155Pulled, Structs.DelegateInfo calldata delegateInfo) internal {
31:    function checkERC721BeforePull(uint256 underlyingAmount, address underlyingContract, uint256 underlyingTokenId) internal view {
40:    function pullERC721AfterCheck(address underlyingContract, uint256 underlyingTokenId) internal {
45:    function checkERC20BeforePull(uint256 underlyingAmount, address underlyingContract, uint256 underlyingTokenId) internal view {
57:    function pullERC20AfterCheck(address underlyingContract, uint256 pullAmount) internal {
61:    function checkERC1155BeforePull(Structs.Uint256 storage erc1155Pulled, uint256 pullAmount) internal {
70:    function pullERC1155AfterCheck(Structs.Uint256 storage erc1155Pulled, uint256 pullAmount, address underlyingContract, uint256 underlyingTokenId) internal {
77:    function checkERC1155Pulled(Structs.Uint256 storage erc1155Pulled, address operator) internal returns (bool) {
85:    function revertInvalidERC1155PullCheck(Structs.Uint256 storage erc1155PullAuthorization, address operator) internal {
```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenStorageHelpers.sol

```solidity
37:    function writeApproved(mapping(uint256 delegateTokenId => uint256[3] info) storage delegateTokenInfo, uint256 delegateTokenId, address approved) internal {
44:    function writeExpiry(mapping(uint256 delegateTokenId => uint256[3] info) storage delegateTokenInfo, uint256 delegateTokenId, uint256 expiry) internal {
50:    function writeRegistryHash(mapping(uint256 delegateTokenId => uint256[3] info) storage delegateTokenInfo, uint256 delegateTokenId, bytes32 registryHash) internal {
54:    function writeUnderlyingAmount(mapping(uint256 delegateTokenId => uint256[3] info) storage delegateTokenInfo, uint256 delegateTokenId, uint256 underlyingAmount)
58:    function incrementBalance(mapping(address delegateTokenHolder => uint256 balance) storage balances, address delegateTokenHolder) internal {
64:    function decrementBalance(mapping(address delegateTokenHolder => uint256 balance) storage balances, address delegateTokenHolder) internal {
72:    function burnPrincipal(address principalToken, Structs.Uint256 storage principalBurnAuthorization, uint256 delegateTokenId) internal {
84:    function mintPrincipal(address principalToken, Structs.Uint256 storage principalMintAuthorization, address principalRecipient, uint256 delegateTokenId) internal {
96:    function checkBurnAuthorized(address principalToken, Structs.Uint256 storage principalBurnAuthorization) internal view {
104:    function checkMintAuthorized(address principalToken, Structs.Uint256 storage principalMintAuthorization) internal view {
112:    function principalIsCaller(address principalToken) internal view {
117:    function revertAlreadyExisted(mapping(uint256 delegateTokenId => uint256[3] info) storage delegateTokenInfo, uint256 delegateTokenId) internal view {
122:    function revertNotOperator(mapping(address account => mapping(address operator => bool enabled)) storage accountOperator, address account) internal view {
127:    function readApproved(mapping(uint256 delegateTokenId => uint256[3] info) storage delegateTokenInfo, uint256 delegateTokenId) internal view returns (address) {
131:    function readExpiry(mapping(uint256 delegateTokenId => uint256[3] info) storage delegateTokenInfo, uint256 delegateTokenId) internal view returns (uint256) {
135:    function readRegistryHash(mapping(uint256 delegateTokenId => uint256[3] info) storage delegateTokenInfo, uint256 delegateTokenId) internal view returns (bytes32) {
139:    function readUnderlyingAmount(mapping(uint256 delegateTokenId => uint256[3] info) storage delegateTokenInfo, uint256 delegateTokenId) internal view returns (uint256) {
143:    function revertNotApprovedOrOperator(
155:    function revertInvalidWithdrawalConditions(
170:    function revertNotMinted(mapping(uint256 delegateTokenId => uint256[3] info) storage delegateTokenInfo, uint256 delegateTokenId) internal view {
178:    function revertNotMinted(bytes32 registryHash, uint256 delegateTokenId) internal pure {
```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenRegistryHelpers.sol

```solidity
8:  library DelegateTokenRegistryHelpers {
140:    function revertERC721FlashUnavailable(address delegateRegistry, Structs.FlashInfo calldata info) internal view {
152:    function revertERC20FlashAmountUnavailable(address delegateRegistry, Structs.FlashInfo calldata info) internal view {
162:    function revertERC1155FlashAmountUnavailable(address delegateRegistry, Structs.FlashInfo calldata info) internal view {
174:    function transferERC721(
194:    function transferERC20(
218:    function transferERC1155(
240:    function delegateERC721(address delegateRegistry, bytes32 newRegistryHash, Structs.DelegateInfo calldata delegateInfo) internal {
250:    function incrementERC20(address delegateRegistry, bytes32 newRegistryHash, Structs.DelegateInfo calldata delegateInfo) internal {
261:    function incrementERC1155(address delegateRegistry, bytes32 newRegistryHash, Structs.DelegateInfo calldata delegateInfo) internal {
275:    function revokeERC721(
291:    function decrementERC20(
309:    function decrementERC1155(
```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenLib.sol

```solidity
8:  library DelegateTokenStructs {
9:    struct Uint256 {
13:    struct DelegateTokenParameters {
20:    struct DelegateInfo {
42:library DelegateTokenErrors {
43:    error DelegateRegistryZero();
44:    error PrincipalTokenZero();
45:    error DelegateTokenHolderZero();
46:    error MarketMetadataZero();
47:    error ToIsZero();
49:    error NotERC721Receiver();
50:    error InvalidERC721TransferOperator();
51:    error ERC1155PullNotRequested(address operator);
52:    error BatchERC1155TransferUnsupported();
54:    error InsufficientAllowanceOrInvalidToken();
55:    error CallerNotOwnerOrInvalidToken();
57:    error NotOperator(address caller, address account);
58:    error NotApproved(address caller, uint256 delegateTokenId);
60:    error FromNotDelegateTokenHolder();
62:    error HashMismatch();
64:    error NotMinted(uint256 delegateTokenId);
65:    error AlreadyExisted(uint256 delegateTokenId);
66:    error WithdrawNotAvailable(uint256 delegateTokenId, uint256 expiry, uint256 timestamp);
68:    error ExpiryInPast();
69:    error ExpiryTooLarge();
70:    error ExpiryTooSmall();
72:    error WrongAmountForType(IDelegateRegistry.DelegationType tokenType, uint256 wrongAmount);
73:    error WrongTokenIdForType(IDelegateRegistry.DelegationType tokenType, uint256 wrongTokenId);
74:    error InvalidTokenType(IDelegateRegistry.DelegationType tokenType);
76:    error ERC721FlashUnavailable();
77:    error ERC20FlashAmountUnavailable();
78:    error ERC1155FlashAmountUnavailable();
80:    error BurnNotAuthorized();
81:    error MintNotAuthorized();
82:    error CallerNotPrincipalToken();
83:    error BurnAuthorized();
84:    error MintAuthorized();
86:    error ERC1155Pulled();
87:    error ERC1155NotPulled();

90:library DelegateTokenHelpers {
91:    function revertOnCallingInvalidFlashloan(DelegateTokenStructs.FlashInfo calldata info) internal {
96:    function revertOnInvalidERC721ReceiverCallback(address from, address to, uint256 delegateTokenId, bytes calldata data) internal {
101:    function revertOnInvalidERC721ReceiverCallback(address from, address to, uint256 delegateTokenId) internal {
107:    function revertOldExpiry(uint256 expiry) internal view {
113:    function delegateIdNoRevert(address caller, uint256 salt) internal pure returns (uint256) {
```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol

```solidity
10:library CreateOffererErrors {
11:    error DelegateTokenIsZero();
12:    error PrincipalTokenIsZero();
13:    error InvalidTokenType(IDelegateRegistry.DelegationType invalidType);
14:    error NoBatchWrapping();
15:    error InvalidExpiryType(CreateOffererEnums.ExpiryType invalidType);
16:    error SeaportIsZero();
17:    error Locked();
18:    error WrongStage(CreateOffererEnums.Stage expected, CreateOffererEnums.Stage actual);
19:    error CallerNotSeaport(address caller);
20:    error DelegateTokenIdInvariant(uint256 requested, uint256 actual);
21:    error CreateOrderHashInvariant(uint256 requested, uint256 actual);
22:    error MinimumReceivedInvalid(SpentItem minimumReceived);
23:    error MaximumSpentInvalid(SpentItem maximumSpent);
24:    error FromNotCreateOfferer(address from);
25:    error ERC20ApproveFailed(address tokenAddress);
26:    error ERC20AllowanceInvariant(address tokenAddress);
27:    error InvalidContractNonce(uint256 actual, uint256 seaportExpected);
28:    error DelegateInfoInvariant();
29:    error InvalidContextLength();
30:    error TargetTokenInvalid(CreateOffererEnums.TargetToken invalidTargetToken);
33:  library CreateOffererEnums {
64:library CreateOffererStructs {
```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol

```solidity
29:    constructor(Structs.Parameters memory parameters) Modifiers(parameters.seaport, Enums.Stage.generate) {

// @return missing
241:    function getSeaportMetadata() external pure returns (string memory, Schema[] memory) {
```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/PrincipalToken.sol

```solidity
15:    error DelegateTokenZero();
16:    error MarketMetadataZero();
17:    error CallerNotDelegateToken();
18:    error NotApproved(address spender, uint256 id);
20:    constructor(address setDelegateToken, address setMarketMetadata) {
27:    function _checkDelegateTokenCaller() internal view {
33:    function mint(address to, uint256 id) external {
40:    function burn(address spender, uint256 id) external {
50:    function isApprovedOrOwner(address account, uint256 id) external view returns (bool) {
54:    function tokenURI(uint256 id) public view override returns (string memory) {
```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol

```solidity
52:    constructor(Structs.DelegateTokenParameters memory parameters) {
65:    function supportsInterface(bytes4 interfaceId) external pure returns (bool) {
256:    function royaltyInfo(uint256 tokenId, uint256 salePrice) external view returns (address receiver, uint256 royaltyAmount) {
```

https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/libraries/RegistryOps.sol

```solidity
4:  library RegistryOps {

// @params and @return missing
6:    function max(uint256 x, uint256 y) internal pure returns (uint256 z) {
19:    function and(bool x, bool y) internal pure returns (bool z) {
26:    function or(bool x, bool y) internal pure returns (bool z) {
```

https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol

```solidity
300:    function readSlot(bytes32 location) external view returns (bytes32 contents) {
306:    function readSlots(bytes32[] calldata locations) external view returns (bytes32[] memory contents) {


// @params/@return missing
338:    function _pushDelegationHashes(address from, address to, bytes32 delegationHash) internal {
344:    function _writeDelegation(bytes32 location, uint256 position, bytes32 data) internal {
351:    function _writeDelegation(bytes32 location, uint256 position, uint256 data) internal {
358:    function _writeDelegationAddresses(bytes32 location, address from, address to, address contract_) internal {
369:    function _updateFrom(bytes32 location, address from) internal {
380:    function _getValidDelegationsFromHashes(bytes32[] storage hashes) internal view returns (Delegation[] memory delegations_) {
411:    function _getValidDelegationHashesFromHashes(bytes32[] storage hashes) internal view returns (bytes32[] memory validHashes) {
430:    function _loadDelegationBytes32(bytes32 location, uint256 position) internal view returns (bytes32 data) {
437:    function _loadDelegationUint(bytes32 location, uint256 position) internal view returns (uint256 data) {
444:    function _loadFrom(bytes32 location) internal view returns (address) {
454:    function _validateFrom(bytes32 location, address from) internal view returns (bool) {
459:    function _loadDelegationAddresses(bytes32 location) internal view returns (address from, address to, address contract_) {
471:    function _invalidFrom(address from) internal pure returns (bool) {
```

---

### State variable and function names [3]

- Variables should be named according to their specifications
- private and internal `variables` should preppend with `underline`
- private and internal `functions` should preppend with `underline`
- constant state variables should be UPPER_CASE

https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenTransferHelpers.sol

```solidity
//  private and internal `functions` should preppend with `underline`
15:    function checkAndPullByType(Structs.Uint256 storage erc1155Pulled, Structs.DelegateInfo calldata delegateInfo) internal {
31:    function checkERC721BeforePull(uint256 underlyingAmount, address underlyingContract, uint256 underlyingTokenId) internal view {
40:    function pullERC721AfterCheck(address underlyingContract, uint256 underlyingTokenId) internal {
45:    function checkERC20BeforePull(uint256 underlyingAmount, address underlyingContract, uint256 underlyingTokenId) internal view {
57:    function pullERC20AfterCheck(address underlyingContract, uint256 pullAmount) internal {
61:    function checkERC1155BeforePull(Structs.Uint256 storage erc1155Pulled, uint256 pullAmount) internal {
70:    function pullERC1155AfterCheck(Structs.Uint256 storage erc1155Pulled, uint256 pullAmount, address underlyingContract, uint256 underlyingTokenId) internal {
77:    function checkERC1155Pulled(Structs.Uint256 storage erc1155Pulled, address operator) internal returns (bool) {
85:    function revertInvalidERC1155PullCheck(Structs.Uint256 storage erc1155PullAuthorization, address operator) internal {
```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenStorageHelpers.sol

```solidity
//  private and internal `functions` should preppend with `underline`
37:    function writeApproved(mapping(uint256 delegateTokenId => uint256[3] info) storage delegateTokenInfo, uint256 delegateTokenId, address approved) internal {
44:    function writeExpiry(mapping(uint256 delegateTokenId => uint256[3] info) storage delegateTokenInfo, uint256 delegateTokenId, uint256 expiry) internal {
50:    function writeRegistryHash(mapping(uint256 delegateTokenId => uint256[3] info) storage delegateTokenInfo, uint256 delegateTokenId, bytes32 registryHash) internal {
54:    function writeUnderlyingAmount(mapping(uint256 delegateTokenId => uint256[3] info) storage delegateTokenInfo, uint256 delegateTokenId, uint256 underlyingAmount) internal {
58:    function incrementBalance(mapping(address delegateTokenHolder => uint256 balance) storage balances, address delegateTokenHolder) internal {
64:    function decrementBalance(mapping(address delegateTokenHolder => uint256 balance) storage balances, address delegateTokenHolder) internal {
72:    function burnPrincipal(address principalToken, Structs.Uint256 storage principalBurnAuthorization, uint256 delegateTokenId) internal {
84:    function mintPrincipal(address principalToken, Structs.Uint256 storage principalMintAuthorization, address principalRecipient, uint256 delegateTokenId) internal {
96:    function checkBurnAuthorized(address principalToken, Structs.Uint256 storage principalBurnAuthorization) internal view {
104:    function checkMintAuthorized(address principalToken, Structs.Uint256 storage principalMintAuthorization) internal view {
112:    function principalIsCaller(address principalToken) internal view {
117:    function revertAlreadyExisted(mapping(uint256 delegateTokenId => uint256[3] info) storage delegateTokenInfo, uint256 delegateTokenId) internal view {
122:    function revertNotOperator(mapping(address account => mapping(address operator => bool enabled)) storage accountOperator, address account) internal view {
127:    function readApproved(mapping(uint256 delegateTokenId => uint256[3] info) storage delegateTokenInfo, uint256 delegateTokenId) internal view returns (address) {
131:    function readExpiry(mapping(uint256 delegateTokenId => uint256[3] info) storage delegateTokenInfo, uint256 delegateTokenId) internal view returns (uint256) {
135:    function readRegistryHash(mapping(uint256 delegateTokenId => uint256[3] info) storage delegateTokenInfo, uint256 delegateTokenId) internal view returns (bytes32) {
139:    function readUnderlyingAmount(mapping(uint256 delegateTokenId => uint256[3] info) storage delegateTokenInfo, uint256 delegateTokenId) internal view returns (uint256) {
143:    function revertNotApprovedOrOperator(
155:    function revertInvalidWithdrawalConditions(
170:    function revertNotMinted(mapping(uint256 delegateTokenId => uint256[3] info) storage delegateTokenInfo, uint256 delegateTokenId) internal view {
178:    function revertNotMinted(bytes32 registryHash, uint256 delegateTokenId) internal pure {
```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenRegistryHelpers.sol

```solidity
// private and internal `functions` should preppend with `underline`
16:    function loadTokenHolder(address delegateRegistry, bytes32 registryHash) internal view returns (address delegateTokenHolder) {
52:    function loadTokenHolderAndContract(address delegateRegistry, bytes32 registryHash) internal view returns (address delegateTokenHolder, address underlyingContract) 
69:    function loadFrom(address delegateRegistry, bytes32 registryHash) internal view returns (address) {
82:    function loadAmount(address delegateRegistry, bytes32 registryHash) internal view returns (uint256) {
94:    function loadRights(address delegateRegistry, bytes32 registryHash) internal view returns (bytes32) {
106:    function loadTokenId(address delegateRegistry, bytes32 registryHash) internal view returns (uint256) {
119:    function calculateDecreasedAmount(address delegateRegistry, bytes32 registryHash, uint256 decreaseAmount) internal view returns (uint256) {
133:    function calculateIncreasedAmount(address delegateRegistry, bytes32 registryHash, uint256 increaseAmount) internal view returns (uint256) {
140:    function revertERC721FlashUnavailable(address delegateRegistry, Structs.FlashInfo calldata info) internal view {
152:    function revertERC20FlashAmountUnavailable(address delegateRegistry, Structs.FlashInfo calldata info) internal view {
162:    function revertERC1155FlashAmountUnavailable(address delegateRegistry, Structs.FlashInfo calldata info) internal view {
174:    function transferERC721(
194:    function transferERC20(
218:    function transferERC1155(
240:    function delegateERC721(address delegateRegistry, bytes32 newRegistryHash, Structs.DelegateInfo calldata delegateInfo) internal {
250:    function incrementERC20(address delegateRegistry, bytes32 newRegistryHash, Structs.DelegateInfo calldata delegateInfo) internal {
261:    function incrementERC1155(address delegateRegistry, bytes32 newRegistryHash, Structs.DelegateInfo calldata delegateInfo) internal {
275:    function revokeERC721(
291:    function decrementERC20(
309:    function decrementERC1155(
```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenLib.sol

```solidity
//  private and internal `functions` should preppend with `underline`
91:    function revertOnCallingInvalidFlashloan(DelegateTokenStructs.FlashInfo calldata info) internal {
96:    function revertOnInvalidERC721ReceiverCallback(address from, address to, uint256 delegateTokenId, bytes calldata data) internal {
101:    function revertOnInvalidERC721ReceiverCallback(address from, address to, uint256 delegateTokenId) internal {
107:    function revertOldExpiry(uint256 expiry) internal view {
113:    function delegateIdNoRevert(address caller, uint256 salt) internal pure returns (uint256) {
```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol

```solidity
// private and internal `variables` should preppend with `underline`
139:    CreateOffererStructs.Stage private stage;

// private and internal `functions` should preppend with `underline` 
184:    function processNonce(CreateOffererStructs.Nonce storage nonce, uint256 contractNonce) internal {
199:    function updateTransientState(
257:    function createAndValidateDelegateTokenId(address delegateToken, uint256 createOrderHash, IDelegateTokenStructs.DelegateInfo memory delegateInfo) internal {
274:    function calculateOrderHashAndId(address delegateToken, address targetTokenReceiver, address conduit, bytes memory orderInfo, IDelegateRegistry.DelegationType delegationType)
292:    function verifyCreate(address delegateToken, uint256 identifier, CreateOffererStructs.Receivers storage receivers, ReceivedItem calldata consideration, bytes calldata context)
326:    function calculateExpiry(CreateOffererEnums.ExpiryType expiryType, uint256 expiryLength) internal view returns (uint256) {
346:    function processSpentItems(SpentItem[] calldata minimumReceived, SpentItem[] calldata maximumSpent)
378:    function validateCreateOrderHash(address targetTokenReceiver, uint256 createOrderHash, bytes memory encodedOrder, IDelegateRegistry.DelegationType tokenType) internal view {
393:    function calculateOrderHash(address targetTokenReceiver, address conduit, bytes memory createOrderInfo, IDelegateRegistry.DelegationType tokenType)
```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol

```solidity
// private and internal `variables` should preppend with `underline`
26:    Structs.TransientState internal transientState;
27:    Structs.Nonce internal nonce;
```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol

```solidity
// private and internal `variables` should preppend with `underline`
33:    mapping(uint256 delegateTokenId => uint256[3] info) internal delegateTokenInfo;
36:    mapping(address delegateTokenHolder => uint256 balance) internal balances;
39:    mapping(address account => mapping(address operator => bool enabled)) internal accountOperator;
42:    Structs.Uint256 internal principalMintAuthorization = Structs.Uint256(StorageHelpers.MINT_NOT_AUTHORIZED);
43:    Structs.Uint256 internal principalBurnAuthorization = Structs.Uint256(StorageHelpers.BURN_NOT_AUTHORIZED);
46:    Structs.Uint256 internal erc1155PullAuthorization = Structs.Uint256(TransferHelpers.ERC1155_NOT_PULLED);
```

https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/libraries/RegistryOps.sol

```solidity
// private and internal `functions` should preppend with `underline`
6:    function max(uint256 x, uint256 y) internal pure returns (uint256 z) {
19:    function and(bool x, bool y) internal pure returns (bool z) {
26:    function or(bool x, bool y) internal pure returns (bool z) {
```

https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/libraries/RegistryStorage.sol

```solidity
// private and internal `functions` should preppend with `underline`
34:    function packAddresses(address from, address to, address contract_) internal pure returns (bytes32 firstPacked, bytes32 secondPacked) {
50:    function unpackAddresses(bytes32 firstPacked, bytes32 secondPacked) internal pure returns (address from, address to, address contract_) {
64:    function unpackAddress(bytes32 packedSlot) internal pure returns (address unpacked) {
```

https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/libraries/RegistryHashes.sol

```solidity
// private and internal `functions` should preppend with `underline`
41:    function decodeType(bytes32 inputHash) internal pure returns (IDelegateRegistry.DelegationType decodedType) {
54:    function location(bytes32 inputHash) internal pure returns (bytes32 computedLocation) {
73:    function allHash(address from, bytes32 rights, address to) internal pure returns (bytes32 hash) {
95:    function allLocation(address from, bytes32 rights, address to) internal pure returns (bytes32 computedLocation) {
120:    function contractHash(address from, bytes32 rights, address to, address contract_) internal pure returns (bytes32 hash) {
143:    function contractLocation(address from, bytes32 rights, address to, address contract_) internal pure returns (bytes32 computedLocation) {
170:    function erc721Hash(address from, bytes32 rights, address to, uint256 tokenId, address contract_) internal pure returns (bytes32 hash) {
195:    function erc721Location(address from, bytes32 rights, address to, uint256 tokenId, address contract_) internal pure returns (bytes32 computedLocation) {
222:    function erc20Hash(address from, bytes32 rights, address to, address contract_) internal pure returns (bytes32 hash) {
245:    function erc20Location(address from, bytes32 rights, address to, address contract_) internal pure returns (bytes32 computedLocation) {
272:    function erc1155Hash(address from, bytes32 rights, address to, uint256 tokenId, address contract_) internal pure returns (bytes32 hash) {
297:    function erc1155Location(address from, bytes32 rights, address to, uint256 tokenId, address contract_) internal pure returns (bytes32 computedLocation) {
```

https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol

```solidity
// private and internal `variables` should preppend with `underline`
18:    mapping(bytes32 delegationHash => bytes32[5] delegationStorage) internal delegations;
21:    mapping(address from => bytes32[] delegationHashes) internal outgoingDelegationHashes;
24:    mapping(address to => bytes32[] delegationHashes) internal incomingDelegationHashes;
```


