| | issue |
| ----------- | ----------- |
| 1 | [state variables can be packed into fewer storage slots](#1-state-variables-can-be-packed-into-fewer-storage-slots) |
| 2 | [state variables should be cached in stack variables rather than re-reading them from storage](#2-state-variables-should-be-cached-in-stack-variables-rather-than-re-reading-them-from-storage-ones-not-found-by-bot) |
| 3 | [it costs more gas to initialize non-constant/non-immutable variables to zero than to let the default of zero be applied](#3-it-costs-more-gas-to-initialize-non-constantnon-immutable-variables-to-zero-than-to-let-the-default-of-zero-be-applied) |
| 4 | [Use hardcoded address instead address(this)](#4-use-hardcoded-address-instead-addressthis) |
| 5 | [Avoid updating storage when the value hasn't changed](#5-avoid-updating-storage-when-the-value-hasnt-changed) |



## 1. state variables can be packed into fewer storage slots

If variables occupying the same slot are both written the same function or by the constructor, avoids a separate Gsset (20000 gas). Reads of the variables are also cheaper.

`principalMintAuthorization` and `principalBurnAuthorization` and `erc1155PullAuthorization` are state variables with type `Structs.Uint256` but they only have one value named `flag` which holds a small number like `1 or 3 or 5` so we can make the `Uint256` struct in `DelegateTokenLib` into smaller size and instead of using uint256 for `flag` use a smaller version like 1 byte instead so all three state variables `principalMintAuthorization` and `principalBurnAuthorization` and `erc1155PullAuthorization` can be put together in the same slot. (can also use a simple state var with size 1 byte instead of using the struct because the struct only has 1 entity `flag`, this will be a upgrade to readability too). 2 less slot used so 2 less Gsset (20000 gas each) and also one function uses 2 of the state variables so saves 100 gas there too.

- [DelegateToken.sol#L42-L46](https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L42-L46)
- [DelegateTokenLib.sol#L9-L11](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenLib.sol#L9-L11)


## 2. state variables should be cached in stack variables rather than re-reading them from storage (ones not found by bot)

The instances below point to the second+ access of a state variable within a function. Caching of a state variable replace each Gwarmaccess (100 gas) with a much cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses. 

its stated in the bot report that we should get `transientState` but it is very expensive and we dont need all parts for it. so it is better to just keep the parts that we need in stack variables. `transientState.receivers.fulfiller` is the only part being reread from storage so we should cache it before to reduce one complex SLOAD.
it is more efficient to cache it before the usages and inside the if statements and else if statements, so cache it before these lines #L103,127,151

- [CreateOfferer.sol#L104-L154](https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L104-L154)


## 3. it costs more gas to initialize non-constant/non-immutable variables to zero than to let the default of zero be applied

- [DelegateToken.sol#L175](https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L175)
- [DelegateToken.sol#L305](https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L305)

- [DelegateRegistry.sol#L35](https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L35)
- [DelegateRegistry.sol#L275](https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L275)
- [DelegateRegistry.sol#L312](https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L312)
- [DelegateRegistry.sol#L381](https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L381)
- [DelegateRegistry.sol#L386](https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L386)
- [DelegateRegistry.sol#L393](https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L393)
- [DelegateRegistry.sol#L412](https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L412)
- [DelegateRegistry.sol#L417](https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L417)
- [DelegateRegistry.sol#L423](https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L423)

- [DelegateTokenRegistryHelpers.sol#L153](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenRegistryHelpers.sol#L153)
- [DelegateTokenRegistryHelpers.sol#L163](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenRegistryHelpers.sol#L163)

- [DelegateTokenStorageHelpers.sol#L15](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenStorageHelpers.sol#L15)
- [DelegateTokenStorageHelpers.sol#L22](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenStorageHelpers.sol#L22)


## 4. Use hardcoded address instead address(this)

Instead of using `address(this)`, it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry's `script.sol` and solmate's `LibRlp.sol` contracts can help achieve this. - [Reference](https://twitter.com/transmissions11/status/1518507047943245824)

- [DelegateToken.sol#L84](https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L84)
- [DelegateToken.sol#L178](https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L178)
- [DelegateToken.sol#L182](https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L182)
- [DelegateToken.sol#L196](https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L196)
- [DelegateToken.sol#L307](https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L307)
- [DelegateToken.sol#L312](https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L312)
- [DelegateToken.sol#L317](https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L317)
- [DelegateToken.sol#L369](https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L369)
- [DelegateToken.sol#L384](https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L384)
- [DelegateToken.sol#L393](https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L393)
- [DelegateToken.sol#L406](https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L406)

- [CreateOfferer.sol#L90](https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L90)
- [CreateOfferer.sol#L138](https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L138)

- [DelegateRegistry.sol#L37](https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L37)

- [DelegateTokenTransferHelpers.sol#L41](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenTransferHelpers.sol#L41)
- [DelegateTokenTransferHelpers.sol#L52](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenTransferHelpers.sol#L52)
- [DelegateTokenTransferHelpers.sol#L58](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenTransferHelpers.sol#L58)
- [DelegateTokenTransferHelpers.sol#L71](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenTransferHelpers.sol#L71)
- [DelegateTokenTransferHelpers.sol#L78](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenTransferHelpers.sol#L78)

- [CreateOffererLib.sol#L259](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L259)
- [CreateOffererLib.sol#L280](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L280)
- [CreateOffererLib.sol#L314](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L314)
- [CreateOffererLib.sol#L352](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L352)
- [CreateOffererLib.sol#L366](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L366)

- [DelegateTokenRegistryHelpers.sol#L146](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenRegistryHelpers.sol#L146)
- [DelegateTokenRegistryHelpers.sol#L147](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenRegistryHelpers.sol#L147)
- [DelegateTokenRegistryHelpers.sol#L156](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenRegistryHelpers.sol#L156)
- [DelegateTokenRegistryHelpers.sol#L157](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenRegistryHelpers.sol#L157)
- [DelegateTokenRegistryHelpers.sol#L165](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenRegistryHelpers.sol#L165)
- [DelegateTokenRegistryHelpers.sol#L166](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenRegistryHelpers.sol#L166)


## 5. Avoid updating storage when the value hasn't changed

In Solidity, manipulating contract storage comes with significant gas costs. One can optimize gas usage by preventing unnecessary storage updates when the new value is the same as the existing one. If an existing value is the same as the new one, not reassigning it to the storage could potentially save substantial amounts of gas, notably 2900 gas for a 'Gsreset'. This saving may come at the expense of a cold storage load operation ('Gcoldsload'), which costs 2100 gas, or a warm storage access operation ('Gwarmaccess'), which costs 100 gas. Therefore, the gas efficiency of your contract can be significantly improved by adding a check that compares the new value with the current one before any storage update operation. If the values are the same, you can bypass the storage operation, thereby saving gas.

check `accountOperator[msg.sender][operator]` value compared to `approved`
- [DelegateToken.sol#L145](https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L145)

check `transientState.receivers.targetTokenReceiver` value compared to `targetTokenReceiver`
- [CreateOfferer.sol#L91](https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L91)
