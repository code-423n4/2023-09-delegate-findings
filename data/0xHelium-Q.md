## Inapropriate comments

> There are lot of instances in this code where there are not appropriate comments before functions.
Some example are as follows:

<https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/libraries/DelegateTokenLib.sol#L91-L115>

<https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenRegistryHelpers.sol#L140-L324>

<https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/libraries/DelegateTokenStorageHelpers.sol#L117-L182>

<https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/libraries/DelegateTokenTransferHelpers.sol#L15-L87>

Comments before functions in solidity should be as follows:
// @notice: what the function does
// @dev: what is expected to arrive 
// @params: parameter name and what it is
// @return if there are any: what the function return.

Please consider adding appropriate comments before functions to facilitate code readability and maintainance.
Some example of well written comments include:

https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/libraries/DelegateTokenRegistryHelpers.sol#L9-L138


