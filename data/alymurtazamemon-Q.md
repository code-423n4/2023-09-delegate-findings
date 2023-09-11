# Delegate - Quality Assurence Report

# Low Risk Findings

## Table Of Content

| Number   | Issue                                                                                                | Instances |
| :------- | :--------------------------------------------------------------------------------------------------- | --------: |
| [L-01]() | Use the fixed `pragma version`                                                                       |        12 |
| [L-02]() | Missing validation for `address(0)` in the constructors or in the functions changing state variables |         2 |
| [L-03]() | Contracts should use the `latest version` of Solidity to be safe from `bug fixes`                    |         5 |

### [L-01] - Use the fixed `pragma version`.

**Details**

Pragma versions are designed in a way that `0.x.0` introduces breaking changes and `0.0.x` introduces bug fixes in the previous versions.

Now using `^` with the pragma versions opens the code compilation till the next `< 0.x.0` version, there will be no breaking changes until version `0.x.0`, you can be sure that your code compiles the way you intended but due to the exact version of the compiler is not fixed, newly introduced bugfix can still affect the code.

It is recommended by [Solidity Docs](https://docs.soliditylang.org/en/v0.8.21/layout-of-source-files.html#version-pragma) to use fixed version for your projects.

Which version should we choose?

Well Solidity docs always suggest using the latest version of Solidity for the deployment of smart contracts.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

`Example`

```diff
-   pragma solidity ^0.8.21;
+   pragma solidity 0.8.21;
```

`Findings Locations`

[DelegateRegistry.sol - Line 02](https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L2)

[RegistryHashes.sol - Line 02](https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/libraries/RegistryHashes.sol#L2)

[RegistryOps.sol - Line 02](https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/libraries/RegistryOps.sol#L2)

[RegistryStorage.sol - Line 02](https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/libraries/RegistryStorage.sol#L2)

[CreateOfferer.sol - Line 02](https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L2)

[DelegateToken.sol - Line 02](https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L2)

[PrincipalToken.sol - Line 02](https://github.com/code-423n4/2023-09-delegate/blob/main/src/PrincipalToken.sol#L2)

[CreateOffererLib.sol - Line 02](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L2)

[DelegateTokenLib.sol - Line 02](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenLib.sol#L2)

[DelegateTokenRegistryHelpers.sol - Line 02](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenRegistryHelpers.sol#L2)

[DelegateTokenStorageHelpers.sol - Line 02](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenStorageHelpers.sol#L2)

[DelegateTokenTransferHelpers.sol - Line 02](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenTransferHelpers.sol#L2)

</details>

### [L-02] - Missing validation for `address(0)` in the constructors or in the functions changing state variables.

**Details**

It is recommended to check for `address(0)` when updating state variables through constructors or functions. This can save the protocol from accidental updates.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[CreateOfferer.sol - Line 61](https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L61)

[PrincipalToken.sol - Line 35](https://github.com/code-423n4/2023-09-delegate/blob/main/src/PrincipalToken.sol#L35)

</details>

### [L-03] - Contracts should use the `latest version` of Solidity to be safe from `bug fixes`.

**Details**

The contracts in scope use the older version of solidity (`0.8.4`) and the current version while writing this report is (`0.8.21`).

Solidity docs always suggest using the latest version of Solidity for the deployment of smart contracts.

There are no breaking changes between these versions but numerous minor bug fixes have been done between them.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

`Example`

```diff
-   pragma solidity ^0.8.4;
+   pragma solidity 0.8.21;
```

`Findings Locations`

[CreateOffererLib.sol - Line 02](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L2)

[DelegateTokenLib.sol - Line 02](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenLib.sol#L2)

[DelegateTokenRegistryHelpers.sol - Line 02](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenRegistryHelpers.sol#L2)

[DelegateTokenStorageHelpers.sol - Line 02](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenStorageHelpers.sol#L2)

[DelegateTokenTransferHelpers.sol - Line 02](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenTransferHelpers.sol#L2)

</details>

**Recommended Mitigation Steps**

Update the version of the contract to be safe from minor bug fixes between the used version and the current solidity version.

# Informational Findings

## Table Of Content

| Number   | Issue                                                                                                                              | Instances |
| :------- | :--------------------------------------------------------------------------------------------------------------------------------- | --------: |
| [I-01]() | Internal functions and variables should follow the `naming convention`                                                             |         8 |
| [I-02]() | Use `natspec` comments for smart contracts, interfaces and libraries                                                               |        11 |
| [I-03]() | Use `natspec` comments for events, state variables, constructors, functions and all other things which will be included in the ABI |         5 |
| [I-04]() | Set the Internal Layout of `Contracts`, `Interfaces`, and `Libraries`                                                              |         2 |

### [I-01] - Internal functions and variables should follow the `naming convention`.

**Details**

Underscore Prefix for Non-external Functions and Variables

-   `_singleLeadingUnderscore`

This convention is suggested for non-external functions and state variables (`private` or `internal`).

<details>
  <summary>Spotted Findings</summary>
  <p></p>

`Example`

```diff
-   mapping(bytes32 delegationHash => bytes32[5] delegationStorage) internal delegations;
+   mapping(bytes32 delegationHash => bytes32[5] delegationStorage) internal _delegations;
```

`Findings Locations`

[DelegateRegistry.sol - Line 18 - 24](https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L18-L24)

[RegistryHashes.sol - Line 23 - 31](https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/libraries/RegistryHashes.sol#L23-L31)

[CreateOfferer.sol - Line 26 - 27](https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L26-L27)

[DelegateToken.sol - Line 33 - 46](https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L33-L46)

[CreateOffererLib.sol - Line 139](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L139)

[DelegateTokenStorageHelpers.sol - Line 09](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenStorageHelpers.sol#L9)

[DelegateTokenStorageHelpers.sol - Line 15 - 16](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenStorageHelpers.sol#L15-L16)

[DelegateTokenStorageHelpers.sol - Line 22 - 24](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenStorageHelpers.sol#L22-L24)

</details>

**Recommended Mitigation Steps**

Apply the convention to these functions and variables.

### [I-02] - Use `natspec` comments for smart contracts, interfaces and libraries.

**Details**

It is recommended by solidity docs that solidity contracts should be fully annotated using NatSpec for all public interfaces (everything in the ABI).

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[RegistryOps.sol - Line 04](https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/libraries/RegistryOps.sol#L4)

[RegistryStorage.sol - Line 04](https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/libraries/RegistryStorage.sol#L4)

[CreateOffererLib.sol - Line 10](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L10)

[CreateOffererLib.sol - Line 33](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L33)

[CreateOffererLib.sol - Line 64](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L64)

[DelegateTokenLib.sol - Line 08](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenLib.sol#L8)

[DelegateTokenLib.sol - Line 42](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenLib.sol#L42)

[DelegateTokenLib.sol - Line 90](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenLib.sol#L90)

[DelegateTokenRegistryHelpers.sol - Line 08](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenRegistryHelpers.sol#L8)

[DelegateTokenStorageHelpers.sol - Line 07](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenStorageHelpers.sol#L7)

[DelegateTokenTransferHelpers.sol - Line 10](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenTransferHelpers.sol#L10)

</details>

### [I-03] - Use `natspec` comments for events, state variables, constructors, functions and all other things which will be included in the ABI.

**Details**

It is recommended by solidity docs that solidity contracts should be fully annotated using NatSpec for all public interfaces (everything in the ABI).

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[PrincipalToken.sol - Line 12 - 57](https://github.com/code-423n4/2023-09-delegate/blob/main/src/PrincipalToken.sol#L12-L57)

[DelegateTokenLib.sol - Line 09 - 115](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenLib.sol#L9-L115)

[DelegateTokenLib.sol - Line 43 - 87](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenLib.sol#L43-L87)

[DelegateTokenLib.sol - Line 91 - 115](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenLib.sol#L91-L115)

[DelegateTokenTransferHelpers.sol - Line 11 - 87](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenTransferHelpers.sol#L11-L87)

</details>

### [I-04] - Set the Internal Layout of `Contracts`, `Interfaces`, and `Libraries`.

**Details**

According to the Solidity [Style Guide](https://docs.soliditylang.org/en/v0.8.21/style-guide.html#style-guide) consistency in the projects layout is very important. If all projects apply the style guides provided by solidity docs then understanding the projects and its code will be much easier for developers, and auditors.

Solidity docs;

> A style guide is about consistency. Consistency with this style guide is important. Consistency within a project is more important. Consistency within one module or function is most important.

According to the solidity styles guide;

Inside each contract, library or interface, this order should be followed:

1. Type declarations
2. State variables
3. Events
4. Errors
5. Modifiers
6. Functions

This should be the order of functions:

-   constructor
-   receive function (if exists)
-   fallback function (if exists)
-   external
-   public
-   internal
-   private
-   View and pure functions last

Apply the recommended layout structure in all mentioned contracts according to the solidity styles guide.

<details>
  <summary>Spotted Findings</summary>
  <p></p>

[PrincipalToken.sol - Line 12 - 57](https://github.com/code-423n4/2023-09-delegate/blob/main/src/PrincipalToken.sol#L12-L57)

[DelegateTokenTransferHelpers.sol - Line 11 - 87](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenTransferHelpers.sol#L11-L87)

</details>