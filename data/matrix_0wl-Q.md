## Non Critical Issues

|       | Issue                                                                                             |
| ----- | :------------------------------------------------------------------------------------------------ |
| NC-1  | ADD A TIMELOCK TO CRITICAL FUNCTIONS                                                              |
| NC-2  | ADD TO BLACKLIST FUNCTION                                                                         |
| NC-3  | GENERATE PERFECT CODE HEADERS EVERY TIME                                                          |
| NC-4  | INCONSISTENT SOLIDITY VERSIONS                                                                    |
| NC-5  | CONSTANT VALUES SUCH AS A CALL TO KECCAK256(), SHOULD USE IMMUTABLE RATHER THAN CONSTANT          |
| NC-6  | DON’T WORRY ABOUT KECCAK256 COLLISIONS                                                            |
| NC-7  | FOR EXTENDED “USING-FOR” USAGE, USE THE LATEST PRAGMA VERSION                                     |
| NC-8  | USE A MORE RECENT VERSION OF SOLIDITY BUT PRAGMA VERSION ^0.8.21 VERSION TOO RECENT TO BE TRUSTED |
| NC-9  | Return values of `approve()` not checked                                                          |
| NC-10 | The tokenAddress state variable should be renamed to token                                        |

### [NC-1] ADD A TIMELOCK TO CRITICAL FUNCTIONS

#### Description:

It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate (less risk of a malicious owner making a sandwich attack on a user).

#### **Proof Of Concept**

```solidity
File: src/DelegateToken.sol

144:     function setApprovalForAll(address operator, bool approved) external {

```

### [NC-2] ADD TO BLACKLIST FUNCTION

#### Description:

NFT thefts have increased recently, so with the addition of hacked NFTs to the platform, NFTs can be converted into liquidity. To prevent this, I recommend adding the blacklist function.

Marketplaces such as Opensea have a blacklist feature that will not list NFTs that have been reported theft, NFT projects such as Manifold have blacklist functions in their smart contracts.

Here is the project example; Manifold

Manifold Contract
https://etherscan.io/address/0xe4e4003afe3765aca8149a82fc064c0b125b9e5a#code

#### Recommended Mitigation Steps:

Add to Blacklist function and modifier.

```solidity
     modifier nonBlacklistRequired(address extension) {
         require(!_blacklistedExtensions.contains(extension), "Extension blacklisted");
         _;
     }

```

### [NC-3] GENERATE PERFECT CODE HEADERS EVERY TIME

#### Description:

I recommend using header for Solidity code layout and readability:
https://github.com/transmissions11/headers

```solidity
/*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/
```

### [NC-4] INCONSISTENT SOLIDITY VERSIONS

#### Description:

The project is compiled with different versions of Solidity, which is not recommended because it can lead to undefined behaviors.

It is better to use one Solidity compiler version across all contracts instead of different versions with different bugs and security checks.

#### **Proof Of Concept**

```solidity
File: lib/delegate-registry/src/DelegateRegistry.sol

2: pragma solidity ^0.8.21;

```

```solidity
File: lib/delegate-registry/src/libraries/RegistryHashes.sol

2: pragma solidity ^0.8.21;

```

```solidity
File: lib/delegate-registry/src/libraries/RegistryOps.sol

2: pragma solidity ^0.8.21;

```

```solidity
File: lib/delegate-registry/src/libraries/RegistryStorage.sol

2: pragma solidity ^0.8.21;

```

```solidity
File: src/CreateOfferer.sol

2: pragma solidity ^0.8.21;

```

```solidity
File: src/DelegateToken.sol

2: pragma solidity ^0.8.21;

```

```solidity
File: src/PrincipalToken.sol

2: pragma solidity ^0.8.21;

```

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

### [NC-5] CONSTANT VALUES SUCH AS A CALL TO KECCAK256(), SHOULD USE IMMUTABLE RATHER THAN CONSTANT

#### Description:

There is a difference between constant variables and immutable variables, and they should each be used in their appropriate contexts.

While it doesn’t save any gas because the compiler knows that developers often make this mistake, it’s still best to use the right tool for the task at hand.

WConstants should be used for literal values written into the code, and immutable variables should be used for expressions, or values calculated in, or passed into the constructor.

#### **Proof Of Concept**

```solidity
File: src/libraries/CreateOffererLib.sol

398:         uint256 hashWithoutType = uint256(keccak256(abi.encode(targetTokenReceiver, conduit, createOrderInfo)));

```

### [NC-6] DON’T WORRY ABOUT KECCAK256 COLLISIONS

#### **Proof Of Concept**

```solidity
File: lib/delegate-registry/src/DelegateRegistry.sol

388:                 if (_invalidFrom(_loadFrom(Hashes.location(hash)))) continue;

419:                 if (_invalidFrom(_loadFrom(Hashes.location(hash)))) continue;

```

### [NC-7] FOR EXTENDED “USING-FOR” USAGE, USE THE LATEST PRAGMA VERSION

#### Description:

https://blog.soliditylang.org/2022/03/16/solidity-0.8.13-release-announcement/

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

#### Recommended Mitigation Steps:

Use solidity pragma version min. 0.8.13

### [NC-8] USE A MORE RECENT VERSION OF SOLIDITY BUT PRAGMA VERSION ^0.8.21 VERSION TOO RECENT TO BE TRUSTED

#### Description:

For security, it is best practice to use the latest Solidity version. For the security fix list in the versions; https://github.com/ethereum/solidity/blob/develop/Changelog.md

#### **Proof Of Concept**

```solidity
File: lib/delegate-registry/src/DelegateRegistry.sol

2: pragma solidity ^0.8.21;

```

```solidity
File: lib/delegate-registry/src/libraries/RegistryHashes.sol

2: pragma solidity ^0.8.21;

```

```solidity
File: lib/delegate-registry/src/libraries/RegistryOps.sol

2: pragma solidity ^0.8.21;

```

```solidity
File: lib/delegate-registry/src/libraries/RegistryStorage.sol

2: pragma solidity ^0.8.21;

```

```solidity
File: src/CreateOfferer.sol

2: pragma solidity ^0.8.21;

```

```solidity
File: src/DelegateToken.sol

2: pragma solidity ^0.8.21;

```

```solidity
File: src/PrincipalToken.sol

2: pragma solidity ^0.8.21;

```

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

### [NC-9] Return values of `approve()` not checked

#### Description:

Not all IERC20 implementations `revert()` when there's a failure in `approve()`. The function signature has a boolean return value and they indicate errors that way instead. By not checking the return value, operations that should have marked as failed, may potentially go through without actually approving anything

#### **Proof Of Concept**

```solidity
File: src/CreateOfferer.sol

121:             if (!IERC20(erc20Order.info.tokenContract).approve(address(delegateToken), erc20Order.amount)) {

```

### [NC-10] The tokenAddress state variable should be renamed to token

#### Description:

The tokenAddress state variable of VTVLVesting should be renamed to token as this variable represents an IERC20 interface rather that just an address. Renaming it to token aligns better with its usage.

#### **Proof Of Concept**

```solidity
File: src/libraries/CreateOffererLib.sol

25:     error ERC20ApproveFailed(address tokenAddress);

26:     error ERC20AllowanceInvariant(address tokenAddress);

```

## Low Issues

|     | Issue                                                                          |
| --- | :----------------------------------------------------------------------------- |
| L-1 | CRITICAL ADDRESS CHANGES SHOULD USE TWO-STEP PROCEDURE                         |
| L-2 | UNIFY RETURN CRITERIA                                                          |
| L-3 | Incorrect return values for ERC721 `ownerOf()`                                 |
| L-4 | THE SAFETRANSFER FUNCTION DOES NOT CHECK FOR POTENTIALLY SELF-DESTROYED TOKENS |
| L-5 | Timestamp Dependence                                                           |
| L-6 | Account existence check for low-level calls                                    |
| L-7 | Unsafe ERC20 operation(s)                                                      |
| L-8 | USE `_SAFEMINT` INSTEAD OF `_MINT`                                             |

### [L-1] CRITICAL ADDRESS CHANGES SHOULD USE TWO-STEP PROCEDURE

#### Description:

The critical procedures should be two step process.

Changing critical addresses in contracts should be a two-step process where the first transaction (from the old/current address) registers the new address (i.e. grants ownership) and the second transaction (from the new address) replaces the old address with the new one (i.e. claims ownership). This gives an opportunity to recover from incorrect addresses mistakenly used in the first step. If not, contract functionality might become inaccessible. (see [here](https://github.com/OpenZeppelin/openzeppelin-contracts/issues/1488) and [here](https://github.com/OpenZeppelin/openzeppelin-contracts/issues/2369))

#### **Proof Of Concept**

```solidity
File: src/DelegateToken.sol

144:     function setApprovalForAll(address operator, bool approved) external {

```

#### Recommended Mitigation Steps:

Lack of two-step procedure for critical operations leaves them error-prone. Consider adding two step procedure on the critical functions.

### [L-2] UNIFY RETURN CRITERIA

#### Description:

In contracts, sometimes the name of the return variable is not defined and sometimes is, unifying the way of writing the code makes the code more uniform and readable.

#### **Proof Of Concept**

```solidity
File: lib/delegate-registry/src/DelegateRegistry.sol

31:     function multicall(bytes[] calldata data) external payable override returns (bytes[] memory results) {

44:     function delegateAll(address to, bytes32 rights, bool enable) external payable override returns (bytes32 hash) {

63:     function delegateContract(address to, address contract_, bytes32 rights, bool enable) external payable override returns (bytes32 hash) {

82:     function delegateERC721(address to, address contract_, uint256 tokenId, bytes32 rights, bool enable) external payable override returns (bytes32 hash) {

102:     function delegateERC20(address to, address contract_, bytes32 rights, uint256 amount) external payable override returns (bytes32 hash) {

126:     function delegateERC1155(address to, address contract_, uint256 tokenId, bytes32 rights, uint256 amount) external payable override returns (bytes32 hash) {

165:     function checkDelegateForAll(address to, address from, bytes32 rights) external view override returns (bool valid) {

178:     function checkDelegateForContract(address to, address from, address contract_, bytes32 rights) external view override returns (bool valid) {

193:     function checkDelegateForERC721(address to, address from, address contract_, uint256 tokenId, bytes32 rights) external view override returns (bool valid) {

210:     function checkDelegateForERC20(address to, address from, address contract_, bytes32 rights) external view override returns (uint256 amount) {

229:     function checkDelegateForERC1155(address to, address from, address contract_, uint256 tokenId, bytes32 rights) external view override returns (uint256 amount) {

252:     function getIncomingDelegations(address to) external view override returns (Delegation[] memory delegations_) {

257:     function getOutgoingDelegations(address from) external view returns (Delegation[] memory delegations_) {

262:     function getIncomingDelegationHashes(address to) external view returns (bytes32[] memory delegationHashes) {

267:     function getOutgoingDelegationHashes(address from) external view returns (bytes32[] memory delegationHashes) {

272:     function getDelegationsFromHashes(bytes32[] calldata hashes) external view returns (Delegation[] memory delegations_) {

300:     function readSlot(bytes32 location) external view returns (bytes32 contents) {

306:     function readSlots(bytes32[] calldata locations) external view returns (bytes32[] memory contents) {

329:     function supportsInterface(bytes4 interfaceId) external pure returns (bool) {

380:     function _getValidDelegationsFromHashes(bytes32[] storage hashes) internal view returns (Delegation[] memory delegations_) {

411:     function _getValidDelegationHashesFromHashes(bytes32[] storage hashes) internal view returns (bytes32[] memory validHashes) {

430:     function _loadDelegationBytes32(bytes32 location, uint256 position) internal view returns (bytes32 data) {

437:     function _loadDelegationUint(bytes32 location, uint256 position) internal view returns (uint256 data) {

444:     function _loadFrom(bytes32 location) internal view returns (address) {

454:     function _validateFrom(bytes32 location, address from) internal view returns (bool) {

459:     function _loadDelegationAddresses(bytes32 location) internal view returns (address from, address to, address contract_) {

471:     function _invalidFrom(address from) internal pure returns (bool) {

```

```solidity
File: lib/delegate-registry/src/libraries/RegistryHashes.sol

41:     function decodeType(bytes32 inputHash) internal pure returns (IDelegateRegistry.DelegationType decodedType) {

54:     function location(bytes32 inputHash) internal pure returns (bytes32 computedLocation) {

73:     function allHash(address from, bytes32 rights, address to) internal pure returns (bytes32 hash) {

95:     function allLocation(address from, bytes32 rights, address to) internal pure returns (bytes32 computedLocation) {

120:     function contractHash(address from, bytes32 rights, address to, address contract_) internal pure returns (bytes32 hash) {

143:     function contractLocation(address from, bytes32 rights, address to, address contract_) internal pure returns (bytes32 computedLocation) {

170:     function erc721Hash(address from, bytes32 rights, address to, uint256 tokenId, address contract_) internal pure returns (bytes32 hash) {

195:     function erc721Location(address from, bytes32 rights, address to, uint256 tokenId, address contract_) internal pure returns (bytes32 computedLocation) {

222:     function erc20Hash(address from, bytes32 rights, address to, address contract_) internal pure returns (bytes32 hash) {

245:     function erc20Location(address from, bytes32 rights, address to, address contract_) internal pure returns (bytes32 computedLocation) {

272:     function erc1155Hash(address from, bytes32 rights, address to, uint256 tokenId, address contract_) internal pure returns (bytes32 hash) {

297:     function erc1155Location(address from, bytes32 rights, address to, uint256 tokenId, address contract_) internal pure returns (bytes32 computedLocation) {

```

```solidity
File: lib/delegate-registry/src/libraries/RegistryOps.sol

6:     function max(uint256 x, uint256 y) internal pure returns (uint256 z) {

19:     function and(bool x, bool y) internal pure returns (bool z) {

26:     function or(bool x, bool y) internal pure returns (bool z) {

```

```solidity
File: lib/delegate-registry/src/libraries/RegistryStorage.sol

34:     function packAddresses(address from, address to, address contract_) internal pure returns (bytes32 firstPacked, bytes32 secondPacked) {

50:     function unpackAddresses(bytes32 firstPacked, bytes32 secondPacked) internal pure returns (address from, address to, address contract_) {

64:     function unpackAddress(bytes32 packedSlot) internal pure returns (address unpacked) {

```

```solidity
File: src/CreateOfferer.sol

241:     function getSeaportMetadata() external pure returns (string memory, Schema[] memory) {

```

```solidity
File: src/DelegateToken.sol

65:     function supportsInterface(bytes4 interfaceId) external pure returns (bool) {

78:     function onERC1155BatchReceived(address, address, uint256[] calldata, uint256[] calldata, bytes calldata) external pure returns (bytes4) {

83:     function onERC721Received(address operator, address, uint256, bytes calldata) external view returns (bytes4) {

89:     function onERC1155Received(address operator, address, uint256, uint256, bytes calldata) external returns (bytes4) {

99:     function balanceOf(address delegateTokenHolder) external view returns (uint256) {

105:     function ownerOf(uint256 delegateTokenId) external view returns (address delegateTokenHolder) {

111:     function getApproved(uint256 delegateTokenId) external view returns (address) {

117:     function isApprovedForAll(address account, address operator) external view returns (bool) {

217:     function name() external pure returns (string memory) {

222:     function symbol() external pure returns (string memory) {

227:     function tokenURI(uint256 delegateTokenId) external view returns (string memory) {

239:     function isApprovedOrOwner(address spender, uint256 delegateTokenId) external view returns (bool) {

247:     function baseURI() external view returns (string memory) {

252:     function contractURI() external view returns (string memory) {

256:     function royaltyInfo(uint256 tokenId, uint256 salePrice) external view returns (address receiver, uint256 royaltyAmount) {

265:     function getDelegateInfo(uint256 delegateTokenId) external view returns (Structs.DelegateInfo memory delegateInfo) {

280:     function getDelegateId(address caller, uint256 salt) external view returns (uint256 delegateTokenId) {

296:     function create(Structs.DelegateInfo calldata delegateInfo, uint256 salt) external nonReentrant returns (uint256 delegateTokenId) {

```

```solidity
File: src/PrincipalToken.sol

50:     function isApprovedOrOwner(address account, uint256 id) external view returns (bool) {

54:     function tokenURI(uint256 id) public view override returns (string memory) {

```

```solidity
File: src/libraries/DelegateTokenLib.sol

113:     function delegateIdNoRevert(address caller, uint256 salt) internal pure returns (uint256) {

```

```solidity
File: src/libraries/DelegateTokenRegistryHelpers.sol

16:     function loadTokenHolder(address delegateRegistry, bytes32 registryHash) internal view returns (address delegateTokenHolder) {

32:     function loadContract(address delegateRegistry, bytes32 registryHash) internal view returns (address underlyingContract) {

52:     function loadTokenHolderAndContract(address delegateRegistry, bytes32 registryHash) internal view returns (address delegateTokenHolder, address underlyingContract) {

69:     function loadFrom(address delegateRegistry, bytes32 registryHash) internal view returns (address) {

82:     function loadAmount(address delegateRegistry, bytes32 registryHash) internal view returns (uint256) {

94:     function loadRights(address delegateRegistry, bytes32 registryHash) internal view returns (bytes32) {

106:     function loadTokenId(address delegateRegistry, bytes32 registryHash) internal view returns (uint256) {

119:     function calculateDecreasedAmount(address delegateRegistry, bytes32 registryHash, uint256 decreaseAmount) internal view returns (uint256) {

133:     function calculateIncreasedAmount(address delegateRegistry, bytes32 registryHash, uint256 increaseAmount) internal view returns (uint256) {

```

```solidity
File: src/libraries/DelegateTokenStorageHelpers.sol

127:     function readApproved(mapping(uint256 delegateTokenId => uint256[3] info) storage delegateTokenInfo, uint256 delegateTokenId) internal view returns (address) {

131:     function readExpiry(mapping(uint256 delegateTokenId => uint256[3] info) storage delegateTokenInfo, uint256 delegateTokenId) internal view returns (uint256) {

135:     function readRegistryHash(mapping(uint256 delegateTokenId => uint256[3] info) storage delegateTokenInfo, uint256 delegateTokenId) internal view returns (bytes32) {

139:     function readUnderlyingAmount(mapping(uint256 delegateTokenId => uint256[3] info) storage delegateTokenInfo, uint256 delegateTokenId) internal view returns (uint256) {

```

```solidity
File: src/libraries/DelegateTokenTransferHelpers.sol

77:     function checkERC1155Pulled(Structs.Uint256 storage erc1155Pulled, address operator) internal returns (bool) {

```

#### Recommended Mitigation Steps:

add `{return x}` if you want to return the updated value or else remove `returns(uint)` from the `function(){}` if no value you wanted to return

### [L-3] Incorrect return values for ERC721 `ownerOf()`

#### Description:

Incorrect return values for ERC721 ownerOf(): Contracts compiled with solc >= 0.4.22 interacting with ERC721 ownerOf() that returns a bool instead of address type will revert. Use OpenZeppelin’s ERC721 contracts.

https://github.com/crytic/slither/wiki/Detector-Documentation#incorrect-erc721-interface

#### **Proof Of Concept**

```solidity
File: src/DelegateToken.sol

105:     function ownerOf(uint256 delegateTokenId) external view returns (address delegateTokenHolder) {

234:             IERC721(principalToken).ownerOf(delegateTokenId)

271:         delegateInfo.principalHolder = IERC721(principalToken).ownerOf(delegateTokenId);

347:             IERC721(principalToken).ownerOf(delegateTokenId),

```

```solidity
File: src/libraries/DelegateTokenTransferHelpers.sol

35:         if (IERC721(underlyingContract).ownerOf(underlyingTokenId) != msg.sender) {

```

### [L-4] THE SAFETRANSFER FUNCTION DOES NOT CHECK FOR POTENTIALLY SELF-DESTROYED TOKENS

#### Description:

If a pair gets created and after a while one of the tokens gets self-destroyed (maybe because of a bug) then `safeTransfer` would still succeed. It’s probably a good idea to check if the contract still exists by checking the bytecode length.

#### **Proof Of Concept**

```solidity
File: src/DelegateToken.sol

375:             SafeERC20.safeTransfer(IERC20(underlyingContract), msg.sender, erc20UnderlyingAmount);

399:             SafeERC20.safeTransfer(IERC20(info.tokenContract), info.receiver, info.amount);

```

### [L-5] Timestamp Dependence

#### Description:

Contracts often need access to time values to perform certain types of functionality. Values such as block.timestamp, and block.number can give you a sense of the current time or a time delta, however, they are not safe to use for most purposes.

In the case of block.timestamp, developers often attempt to use it to trigger time-dependent events. As Ethereum is decentralized, nodes can synchronize time only to some degree. Moreover, malicious miners can alter the timestamp of their blocks, especially if they can gain advantages by doing so. However, miners cant set a timestamp smaller than the previous one (otherwise the block will be rejected), nor can they set the timestamp too far ahead in the future. Taking all of the above into consideration, developers cant rely on the preciseness of the provided timestamp.

As for block.number, considering the block time on Ethereum is generally about 14 seconds, it`s possible to predict the time delta between blocks. However, block times are not constant and are subject to change for a variety of reasons, e.g. fork reorganisations and the difficulty bomb. Due to variable block times, block.number should also not be relied on for precise calculations of time.
Reference: https://swcregistry.io/docs/SWC-116

#### **Proof Of Concept**

```solidity
File: src/DelegateToken.sol

341:         if (StorageHelpers.readExpiry(delegateTokenInfo, delegateTokenId) < block.timestamp) {

```

```solidity
File: src/libraries/CreateOffererLib.sol

328:             return block.timestamp + expiryLength;

```

```solidity
File: src/libraries/DelegateTokenLib.sol

109:         if (expiry > block.timestamp) return;

```

```solidity
File: src/libraries/DelegateTokenStorageHelpers.sol

162:         if (block.timestamp < readExpiry(delegateTokenInfo, delegateTokenId)) {

166:             revert Errors.WithdrawNotAvailable(delegateTokenId, readExpiry(delegateTokenInfo, delegateTokenId), block.timestamp);

```

### [L-6] Account existence check for low-level calls

#### Description:

Low-level calls `call`/`delegatecall`/`staticcall` return true even if the account called is non-existent (per EVM design). Account existence must be checked prior to calling if needed.

https://github.com/crytic/slither/wiki/Detector-Documentation#low-level-callsn

#### **Proof Of Concept**

```solidity
File: lib/delegate-registry/src/DelegateRegistry.sol

37:                 (success, results[i]) = address(this).delegatecall(data[i]);

156:             let result := call(gas(), sc, selfbalance(), 0, 0, 0, 0)

```

#### Recommended Mitigation Steps:

In addition to the zero-address checks, add a check to verify that <address>.code.length > 0

### [L-7] Unsafe ERC20 operation(s)

#### Description:

There are ERC20 tokens that charge fee for every transfer() / transferFrom().

#### **Proof Of Concept**

```solidity
File: src/CreateOfferer.sol

121:             if (!IERC20(erc20Order.info.tokenContract).approve(address(delegateToken), erc20Order.amount)) {

```

```solidity
File: src/DelegateToken.sol

369:             IERC721(underlyingContract).transferFrom(address(this), msg.sender, erc721UnderlyingTokenId);

393:             IERC721(info.tokenContract).transferFrom(address(this), info.receiver, info.tokenId);

```

```solidity
File: src/libraries/DelegateTokenTransferHelpers.sol

41:         IERC721(underlyingContract).transferFrom(msg.sender, address(this), underlyingTokenId);

```

### [L-8] USE `_SAFEMINT` INSTEAD OF `_MINT`

#### Description:

According to openzepplin’s ERC721, the use of `_mint` is discouraged, use `safeMint` whenever possible.

https://docs.openzeppelin.com/contracts/3.x/api/token/erc721#ERC721-mint-address-uint256-

#### **Proof Of Concept**

```solidity
File: src/PrincipalToken.sol

35:         _mint(to, id);

```

#### Recommended Mitigation Steps:

Use `_safeMint` whenever possible instead of `_mint`
