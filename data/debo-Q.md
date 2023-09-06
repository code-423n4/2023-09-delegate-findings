## [L-01]  Unsafe ERC20 Operation(s)
Description
ERC20 operations can be unsafe due to different implementations and vulnerabilities in the standard.

It is therefore recommended to always either use OpenZeppelin's SafeERC20 library or at least to wrap each operation in a require statement.

To circumvent ERC20's approve functions race-condition vulnerability use OpenZeppelin's SafeERC20 library's safe{Increase|Decrease}Allowance functions.

In case the vulnerability is of no danger for your implementation, provide enough documentation explaining the reasonings.
```txt
2023-09-delegate/src/CreateOfferer.sol::121 => if (!IERC20(erc20Order.info.tokenContract).approve(address(delegateToken), erc20Order.amount)) {
2023-09-delegate/src/DelegateToken.sol::369 => IERC721(underlyingContract).transferFrom(address(this), msg.sender, erc721UnderlyingTokenId);
2023-09-delegate/src/DelegateToken.sol::393 => IERC721(info.tokenContract).transferFrom(address(this), info.receiver, info.tokenId);
2023-09-delegate/src/libraries/DelegateTokenTransferHelpers.sol::41 => IERC721(underlyingContract).transferFrom(msg.sender, address(this), underlyingTokenId);
```
## [L-02] Unspecific Compiler Version Pragma
Description
Avoid floating pragmas for non-library contracts.

While floating pragmas make sense for libraries to allow them to be included with multiple different versions of applications, it may be a security risk for application implementations.

A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up checking a different EVM compilation that is ultimately deployed on the blockchain.

It is recommended to pin to a concrete compiler version.
```txt
2023-09-delegate/src/CreateOfferer.sol::2 => pragma solidity ^0.8.21;
2023-09-delegate/src/DelegateToken.sol::2 => pragma solidity ^0.8.21;
2023-09-delegate/src/PrincipalToken.sol::2 => pragma solidity ^0.8.21;
2023-09-delegate/src/libraries/CreateOffererLib.sol::2 => pragma solidity ^0.8.4;
2023-09-delegate/src/libraries/DelegateTokenLib.sol::2 => pragma solidity ^0.8.4;
2023-09-delegate/src/libraries/DelegateTokenRegistryHelpers.sol::2 => pragma solidity ^0.8.4;
2023-09-delegate/src/libraries/DelegateTokenStorageHelpers.sol::2 => pragma solidity ^0.8.4;
2023-09-delegate/src/libraries/DelegateTokenTransferHelpers.sol::2 => pragma solidity ^0.8.4;
```