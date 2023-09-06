## [G-01] Do not initialize variables with default value
Description
Uninitialized variables are assigned with the types default value.

Explicitly initializing a variable with it's default value costs unnecessary gas.
```txt
2023-09-delegate/src/libraries/DelegateTokenRegistryHelpers.sol::153 => uint256 availableAmount = 0;
2023-09-delegate/src/libraries/DelegateTokenRegistryHelpers.sol::163 => uint256 availableAmount = 0;
```
## [G-02] Cache array length outside of loop
Description
Caching the array length outside a loop saves reading it on each iteration, as long as the array's length is not changed during the loop.
```txt
2023-09-delegate/src/CreateOfferer.sol::58 => if (context.length != 160) revert Errors.InvalidContextLength();
2023-09-delegate/src/CreateOfferer.sol::182 => if (context.length != 160) revert Errors.InvalidContextLength();
2023-09-delegate/src/libraries/CreateOffererLib.sol::290 => * @dev Should revert if context is the wrong length
2023-09-delegate/src/libraries/CreateOffererLib.sol::297 => if (context.length != 160) revert CreateOffererErrors.InvalidContextLength();
2023-09-delegate/src/libraries/CreateOffererLib.sol::342 => * @dev Must revert if calldata arrays are not length 1
2023-09-delegate/src/libraries/CreateOffererLib.sol::351 => if (!(minimumReceived.length == 1 && maximumSpent.length == 1)) revert CreateOffererErrors.NoBatchWrapping();
2023-09-delegate/src/libraries/DelegateTokenLib.sol::97 => if (to.code.length == 0 || IERC721Receiver(to).onERC721Received(msg.sender, from, delegateTokenId, data) == IERC721Receiver.onERC721Received.selector) return;
2023-09-delegate/src/libraries/DelegateTokenLib.sol::102 => if (to.code.length == 0 || IERC721Receiver(to).onERC721Received(msg.sender, from, delegateTokenId, "") == IERC721Receiver.onERC721Received.selector) return;
```
## [G-03] Use immutable for openzeppelin access controls roles declarations
⚡️ Only valid for solidity versions <0.6.12 ⚡️

Access roles marked as constant results in computing the keccak256 operation each time the variable is used because assigned operations for constant variables are re-evaluated every time.

Changing the variables to immutable results in computing the hash only once on deployment, leading to gas savings.
```txt
2023-09-delegate/src/libraries/CreateOffererLib.sol::301 => keccak256(
2023-09-delegate/src/libraries/CreateOffererLib.sol::314 => ) != keccak256(abi.encode(IDelegateToken(delegateToken).getDelegateInfo(DelegateTokenHelpers.delegateIdNoRevert(address(this), identifier))))
2023-09-delegate/src/libraries/CreateOffererLib.sol::398 => uint256 hashWithoutType = uint256(keccak256(abi.encode(targetTokenReceiver, conduit, createOrderInfo)));
2023-09-delegate/src/libraries/DelegateTokenLib.sol::114 => return uint256(keccak256(abi.encode(caller, salt)));
```
## [G-04] Long revert strings
Description
Shortening revert strings to fit in 32 bytes will decrease gas costs for deployment and gas costs when the revert condition has been met.

If the contract(s) in scope allow using Solidity >=0.8.4, consider using Custom Errors as they are more gas efficient while allowing developers to describe the error in detail using NatSpec.
```txt
2023-09-delegate/src/CreateOfferer.sol::4 => import {IDelegateRegistry} from "delegate-registry/src/IDelegateRegistry.sol";
2023-09-delegate/src/CreateOfferer.sol::6 => import {RegistryHashes} from "delegate-registry/src/libraries/RegistryHashes.sol";
2023-09-delegate/src/CreateOfferer.sol::7 => import {IERC20} from "openzeppelin/token/ERC20/IERC20.sol";
2023-09-delegate/src/CreateOfferer.sol::8 => import {IERC721} from "openzeppelin/token/ERC721/IERC721.sol";
2023-09-delegate/src/CreateOfferer.sol::9 => import {IERC1155} from "openzeppelin/token/ERC1155/IERC1155.sol";
2023-09-delegate/src/CreateOfferer.sol::11 => import {ContractOffererInterface, SpentItem, ReceivedItem, Schema} from "seaport/contracts/interfaces/ContractOffererInterface.sol";
2023-09-delegate/src/CreateOfferer.sol::18 => } from "src/libraries/CreateOffererLib.sol";
2023-09-delegate/src/CreateOfferer.sol::20 => import {ERC1155Holder} from "openzeppelin/token/ERC1155/utils/ERC1155Holder.sol";
2023-09-delegate/src/DelegateToken.sol::8 => import {ReentrancyGuard} from "openzeppelin/security/ReentrancyGuard.sol";
2023-09-delegate/src/DelegateToken.sol::10 => import {IDelegateRegistry, DelegateTokenErrors as Errors, DelegateTokenStructs as Structs, DelegateTokenHelpers as Helpers} from "src/libraries/DelegateTokenLib.sol";
2023-09-delegate/src/DelegateToken.sol::11 => import {DelegateTokenStorageHelpers as StorageHelpers} from "src/libraries/DelegateTokenStorageHelpers.sol";
2023-09-delegate/src/DelegateToken.sol::12 => import {DelegateTokenRegistryHelpers as RegistryHelpers, RegistryHashes} from "src/libraries/DelegateTokenRegistryHelpers.sol";
2023-09-delegate/src/DelegateToken.sol::13 => import {DelegateTokenTransferHelpers as TransferHelpers, SafeERC20, IERC721, IERC20, IERC1155} from "src/libraries/DelegateTokenTransferHelpers.sol";
2023-09-delegate/src/libraries/CreateOffererLib.sol::4 => import {SpentItem, ReceivedItem} from "seaport/contracts/interfaces/ContractOffererInterface.sol";
2023-09-delegate/src/libraries/CreateOffererLib.sol::5 => import {ItemType} from "seaport/contracts/lib/ConsiderationEnums.sol";
2023-09-delegate/src/libraries/CreateOffererLib.sol::6 => import {IDelegateToken, Structs as IDelegateTokenStructs} from "src/interfaces/IDelegateToken.sol";
2023-09-delegate/src/libraries/CreateOffererLib.sol::7 => import {RegistryHashes} from "delegate-registry/src/libraries/RegistryHashes.sol";
2023-09-delegate/src/libraries/CreateOffererLib.sol::8 => import {IDelegateRegistry, DelegateTokenHelpers} from "src/libraries/DelegateTokenLib.sol";
2023-09-delegate/src/libraries/DelegateTokenLib.sol::4 => import {IDelegateRegistry} from "delegate-registry/src/IDelegateRegistry.sol";
2023-09-delegate/src/libraries/DelegateTokenLib.sol::5 => import {IDelegateFlashloan} from "src/interfaces/IDelegateFlashloan.sol";
2023-09-delegate/src/libraries/DelegateTokenLib.sol::6 => import {IERC721Receiver} from "openzeppelin/token/ERC721/IERC721Receiver.sol";
2023-09-delegate/src/libraries/DelegateTokenRegistryHelpers.sol::4 => import {RegistryStorage} from "delegate-registry/src/libraries/RegistryStorage.sol";
2023-09-delegate/src/libraries/DelegateTokenRegistryHelpers.sol::5 => import {RegistryHashes} from "delegate-registry/src/libraries/RegistryHashes.sol";
2023-09-delegate/src/libraries/DelegateTokenRegistryHelpers.sol::6 => import {IDelegateRegistry, DelegateTokenErrors as Errors, DelegateTokenStructs as Structs} from "src/libraries/DelegateTokenLib.sol";
2023-09-delegate/src/libraries/DelegateTokenStorageHelpers.sol::4 => import {DelegateTokenErrors as Errors, DelegateTokenStructs as Structs} from "src/libraries/DelegateTokenLib.sol";
2023-09-delegate/src/libraries/DelegateTokenTransferHelpers.sol::4 => import {IDelegateRegistry, DelegateTokenErrors as Errors, DelegateTokenStructs as Structs} from "src/libraries/DelegateTokenLib.sol";
2023-09-delegate/src/libraries/DelegateTokenTransferHelpers.sol::5 => import {IERC1155} from "openzeppelin/token/ERC1155/IERC1155.sol";
2023-09-delegate/src/libraries/DelegateTokenTransferHelpers.sol::6 => import {IERC721} from "openzeppelin/token/ERC721/IERC721.sol";
2023-09-delegate/src/libraries/DelegateTokenTransferHelpers.sol::7 => import {IERC20} from "openzeppelin/token/ERC20/IERC20.sol";
2023-09-delegate/src/libraries/DelegateTokenTransferHelpers.sol::8 => import {SafeERC20} from "openzeppelin/token/ERC20/utils/SafeERC20.sol";
```
