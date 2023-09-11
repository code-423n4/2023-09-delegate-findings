## 1. Audit Approach

|     |     |     |
| --- | --- | --- |
| Step | Task | Details |
| 1   | Run Tests | Tests run successfully |
| 2   | Coverage | 80% test coverage for contracts in audit scope |
| 3   | Slither | Reviewed Slither results, no vulnerabilities discovered |
| 4   | Surya | Generate graphs to understand the overall project structure. Provided an initial insight to the contract inheritance and function call flow |
| 5   | Solidity Metrics | Generate metrics reports to obtain initial insight on the codebase, noting areas of potential concern |
| 6   | Code Review | Line by line code review |
| 7   | Test Review | Review of each test and it's purpose |

## 2. Mechanism Summary

`DelegateRegistry` is used to create delegations and contains methods for various types of contracts including ERC20, ERC721 and ERC1155. Generic contracts are also supported. Two ERC721 tokens are minted to the user when depositing tokens into a delegate escrow. These are the `DelegateToken` which receives delegation rights for a given time period, and the `PrincipalToken` which is used to represent ownership of the escrowed token and can be used to reedem it at the end of the time period. `CreateOfferer` provides functionality to create delegate tokens directly from Seaport.

## 3. Centralisation Risks

### 3.1 Owner is able to change the DelegateTokenURI and royalties

The deployer owns the `MarketMetadata` contract and is able to change the `DelegateTokenURI` and royalties. This could lead to unexpected losses for users if royalties are changed without them knowing, or mislead users if the `DelegateTokenURI` is changed.

## 4. Quality Analysis

### 4.1 Codebase

The code is well structured and organised into separate contracts, each with a clear purpose. In depth functionality is logically separated into libraries. NatSpec comments are well used throughout the codebase. The [automated findings](https://github.com/code-423n4/2023-09-delegate/blob/main/bot-report.md) show that some styling conventions such as line lengths and underscores before internal/private functions are not adhered to.

### 4.2 Documentation

The code is well commented with NatSpec comments, which are clear and informative. This could be improved by adding supplementary external documentation with diagrams outlining the workings of the core functionality.

### 4.3 Tests

Tests are clearly structured and cover most of the core functionality but coverage could be better than 80%. Testing could be further improved with the use of fuzz testing, or formal verification to give users the extra assurance that their funds are safe.

## 5. Architecture Improvements

### 5.1 Combine DelegateRegistry and MarketMetadata into a single factory contract

`DelegateRegistry` is used to create and view delegation hashes, and `MarketMetadata` is used to set the `DelegateTokenURI` and royalties. Combining these into a single deployed contract simplifies the system and makes this easier to read when using block explorers and tools using live data.

### Time spent:
25 hours