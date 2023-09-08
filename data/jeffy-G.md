## [G-01] ++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for- and while-loops
```solidity
File: lib/delegate-registry/src/DelegateRegistry.sol
35: for (uint256 i = 0; i < data.length; ++i) {
275: for (uint256 i = 0; i < hashes.length; ++i) {
312: for (uint256 i = 0; i < length; ++i) {
386: for (uint256 i = 0; i < hashesLength; ++i) {
393: for (uint256 i = 0; i < count; ++i) {
417: for (uint256 i = 0; i < hashesLength; ++i) {
423: for (uint256 i = 0; i < count; ++i) {

File: src/libraries/CreateOffererLib.sol
187: ++nonce.value;

File: src/libraries/DelegateTokenStorageHelpers.sol
60: ++balances[delegateTokenHolder];
66: --balances[delegateTokenHolder];

```
 - DelegateRegistry.sol : [[35](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/lib/delegate-registry/src/DelegateRegistry.sol#L35), [275](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/lib/delegate-registry/src/DelegateRegistry.sol#L275), [312](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/lib/delegate-registry/src/DelegateRegistry.sol#L312), [386](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/lib/delegate-registry/src/DelegateRegistry.sol#L386), [393](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/lib/delegate-registry/src/DelegateRegistry.sol#L393), [417](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/lib/delegate-registry/src/DelegateRegistry.sol#L417), [423](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/lib/delegate-registry/src/DelegateRegistry.sol#L423)]
 - CreateOffererLib.sol : [[187](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/libraries/CreateOffererLib.sol#L187)]
 - DelegateTokenStorageHelpers.sol : [[60](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/libraries/DelegateTokenStorageHelpers.sol#L60), [66](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/libraries/DelegateTokenStorageHelpers.sol#L66)]

## [G-02] Usage of `int`s/`uint`s smaller than 32 bytes incurs overhead
Using ints/uints smaller than 32 bytes may cost more gas. This is because the EVM operates on 32 bytes at a time, so if an element is smaller than 32 bytes, the EVM must perform more operations to reduce the size of the element from 32 bytes to the desired size.

Here are some examples of where this optimization could be used:

```solidity
File: src/libraries/DelegateTokenStorageHelpers.sol
38: uint96 expiry = uint96(delegateTokenInfo[delegateTokenId][PACKED_INFO_POSITION]);
46: address approved = address(uint160(delegateTokenInfo[delegateTokenId][PACKED_INFO_POSITION] >> 96));
128: return address(uint160(delegateTokenInfo[delegateTokenId][PACKED_INFO_POSITION] >> 96));
132: return uint96(delegateTokenInfo[delegateTokenId][PACKED_INFO_POSITION]);

```
- DelegateTokenStorageHelpers.sol : [[38](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/libraries/DelegateTokenStorageHelpers.sol#L38), [46](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/libraries/DelegateTokenStorageHelpers.sol#L46), [128](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/libraries/DelegateTokenStorageHelpers.sol#L128), [132](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/libraries/DelegateTokenStorageHelpers.sol#L132)]

## [G-03] Using 2 if statements instead of a logical AND (&&) can provide similar short-circuiting behavior whereas double if is [slightly more gas efficient](https://gist.github.com/DadeKuma/931ce6794a050201ec6544dbcc31316c).

Here are some examples of where this optimization could be used:

```solidity
File: src/libraries/CreateOffererLib.sol
351: if (!(minimumReceived.length == 1 && maximumSpent.length == 1)) revert CreateOffererErrors.NoBatchWrapping();
355: if (maximumSpent[0].itemType != ItemType.ERC721 && maximumSpent[0].itemType != ItemType.ERC20 && maximumSpent[0].itemType != ItemType.ERC1155) {

```
 - CreateOffererLib.sol : [[351](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/libraries/CreateOffererLib.sol#L351), [355](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/libraries/CreateOffererLib.sol#L355)]

## [G-04] Using private for constants saves gas
If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that [returns a tuple](https://github.com/code-423n4/2022-08-frax/blob/90f55a9ce4e25bceed3a74290b854341d8de6afa/src/contracts/FraxlendPair.sol#L156-L178) of the values of all currently-public constants. Saves 3406-3606 gas in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it's used, and not adding another entry to the method ID table

Here are some examples of where this optimization could be used:

```solidity
File: lib/delegate-registry/src/libraries/RegistryHashes.sol
23: uint256 internal constant EXTRACT_LAST_BYTE = 0xff;
25: uint256 internal constant ALL_TYPE = 1;
26: uint256 internal constant CONTRACT_TYPE = 2;
27: uint256 internal constant ERC721_TYPE = 3;
28: uint256 internal constant ERC20_TYPE = 4;
29: uint256 internal constant ERC1155_TYPE = 5;
31: uint256 internal constant DELEGATION_SLOT = 0;

File: lib/delegate-registry/src/libraries/RegistryStorage.sol
6: address internal constant DELEGATION_EMPTY = address(0);
7: address internal constant DELEGATION_REVOKED = address(1);
10: uint256 internal constant POSITIONS_FIRST_PACKED = 0; // | 4 bytes empty | first 8 bytes of contract address | 20 bytes of from address |
11: uint256 internal constant POSITIONS_SECOND_PACKED = 1; // | last 12 bytes of contract address | 20 bytes of to address |
12: uint256 internal constant POSITIONS_RIGHTS = 2;
13: uint256 internal constant POSITIONS_TOKEN_ID = 3;
14: uint256 internal constant POSITIONS_AMOUNT = 4;
17: uint256 internal constant CLEAN_ADDRESS = 0x00ffffffffffffffffffffffffffffffffffffffff;
20: uint256 internal constant CLEAN_FIRST8_BYTES_ADDRESS = 0xffffffffffffffff << 96;
23: uint256 internal constant CLEAN_PACKED8_BYTES_ADDRESS = 0xffffffffffffffff << 160;

File: src/libraries/DelegateTokenStorageHelpers.sol
9: uint256 internal constant MAX_EXPIRY = type(uint96).max;
15: uint256 internal constant ID_AVAILABLE = 0;
16: uint256 internal constant ID_USED = 1;
22: uint256 internal constant REGISTRY_HASH_POSITION = 0;
23: uint256 internal constant PACKED_INFO_POSITION = 1; // PACKED (address approved, uint96 expiry)
24: uint256 internal constant UNDERLYING_AMOUNT_POSITION = 2; // Not used by 721 delegations
31: uint256 internal constant MINT_NOT_AUTHORIZED = 1;
32: uint256 internal constant MINT_AUTHORIZED = 2;
33: uint256 internal constant BURN_NOT_AUTHORIZED = 3;
34: uint256 internal constant BURN_AUTHORIZED = 4;

```
 - RegistryHashes.sol : [[23](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/lib/delegate-registry/src/libraries/RegistryHashes.sol#L23), [25](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/lib/delegate-registry/src/libraries/RegistryHashes.sol#L25), [26](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/lib/delegate-registry/src/libraries/RegistryHashes.sol#L26), [27](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/lib/delegate-registry/src/libraries/RegistryHashes.sol#L27), [28](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/lib/delegate-registry/src/libraries/RegistryHashes.sol#L28), [29](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/lib/delegate-registry/src/libraries/RegistryHashes.sol#L29), [31](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/lib/delegate-registry/src/libraries/RegistryHashes.sol#L31)]
 - RegistryStorage.sol : [[6](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/lib/delegate-registry/src/libraries/RegistryStorage.sol#L6), [7](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/lib/delegate-registry/src/libraries/RegistryStorage.sol#L7), [10](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/lib/delegate-registry/src/libraries/RegistryStorage.sol#L10), [11](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/lib/delegate-registry/src/libraries/RegistryStorage.sol#L11), [12](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/lib/delegate-registry/src/libraries/RegistryStorage.sol#L12), [13](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/lib/delegate-registry/src/libraries/RegistryStorage.sol#L13), [14](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/lib/delegate-registry/src/libraries/RegistryStorage.sol#L14), [17](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/lib/delegate-registry/src/libraries/RegistryStorage.sol#L17), [20](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/lib/delegate-registry/src/libraries/RegistryStorage.sol#L20), [23](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/lib/delegate-registry/src/libraries/RegistryStorage.sol#L23)]
 - DelegateTokenStorageHelpers.sol : [[9](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/libraries/DelegateTokenStorageHelpers.sol#L9), [15](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/libraries/DelegateTokenStorageHelpers.sol#L15), [16](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/libraries/DelegateTokenStorageHelpers.sol#L16), [22](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/libraries/DelegateTokenStorageHelpers.sol#L22), [23](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/libraries/DelegateTokenStorageHelpers.sol#L23), [24](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/libraries/DelegateTokenStorageHelpers.sol#L24), [31](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/libraries/DelegateTokenStorageHelpers.sol#L31), [32](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/libraries/DelegateTokenStorageHelpers.sol#L32), [33](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/libraries/DelegateTokenStorageHelpers.sol#L33), [34](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/libraries/DelegateTokenStorageHelpers.sol#L34)]
