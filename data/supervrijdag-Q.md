## Immutable variables should be uppercase

**Severity**: Non-critical

In [`CreateOfferer.sol#L24-L25`](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/CreateOfferer.sol#L24):
```solidity
    address public immutable delegateToken;
    address public immutable principalToken;
```

In [`DelegateToken.sol#L21-L26`](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/DelegateToken.sol#L21)
```solidity
    address public immutable override delegateRegistry;

    /// @inheritdoc IDelegateToken
    address public immutable override principalToken;

    address public immutable marketMetadata;
```

In [`PrincipalToken.sol#L12-13`](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/PrincipalToken.sol#L12)
```solidity
    address public immutable delegateToken;
    address public immutable marketMetadata;
```


In [`CreateOffererLib.sol#L137`](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/libraries/CreateOffererLib.sol#L137)
```solidity
    address public immutable seaport;
```



