# Redundant cast to bytes32

https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/libraries/DelegateTokenRegistryHelpers.sol#L207

In `DelegateTokenRegistryHelpers` the function `transferERC20` takes an argument `registryHash` which is of type `bytes32`.

In the first part of the if statement in that function, the result of calling `IDelegateRegistry(delegateRegistry).delegateERC20(...)` is compared with the `registryHash` argument. As the `registryHash` argument is already of the correct type, there's no need to explicitly cast it to bytes32.

```solidity
function transferERC20(
        address delegateRegistry,
        bytes32 registryHash,
        address from,
        bytes32 newRegistryHash,
        address to,
        uint256 underlyingAmount,
        bytes32 underlyingRights,
        address underlyingContract
    ) internal {
        if (
            IDelegateRegistry(delegateRegistry).delegateERC20(
                from, underlyingContract, underlyingRights, calculateDecreasedAmount(delegateRegistry, registryHash, underlyingAmount)
            ) == bytes32(registryHash)
                && ...
        ) return;

        revert Errors.HashMismatch();
    }
```

# Perform all checks before writing to storage in CreateOfferer constructor

https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L32

```solidity
    constructor(Structs.Parameters memory parameters) Modifiers(parameters.seaport, Enums.Stage.generate) {
        if (parameters.delegateToken == address(0)) revert Errors.DelegateTokenIsZero();
        delegateToken = parameters.delegateToken;
        if (parameters.principalToken == address(0)) revert Errors.PrincipalTokenIsZero();
        principalToken = parameters.principalToken;
        ...
    }
```

The second if statement can be moved right after the first if, so that writing `delegateToken` to storage is spared in the uncanny scenario of providing `address(0)` for `parameters.principalToken`.