### L1 - Overly complex Modifiers with side effects 

Modifiers should be used to only make checks for conditions and not have any side effects. The following modifiers have side effects 

[CreateOffererLib.sol#L156](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/libraries/CreateOffererLib.sol#L156)
```solidity 
modifier checkStage(CreateOffererEnums.Stage currentStage, CreateOffererEnums.Stage nextStage) {
        CreateOffererStructs.Stage memory cacheStage = stage;
        if (cacheStage.lock != CreateOffererEnums.Lock.unlocked) revert CreateOffererErrors.Locked();
        if (cacheStage.flag != currentStage) revert CreateOffererErrors.WrongStage(currentStage, cacheStage.flag);
        stage.lock = CreateOffererEnums.Lock.locked;
        _;
        stage = CreateOffererStructs.Stage({flag: nextStage, lock: CreateOffererEnums.Lock.unlocked});
    }
```

**Impact**-> It affects the code  maintainability, readability, organization, auditability, can introduce errors, result in code complexity or even cause unintended effects in code. 

**Recommendation** -> It is recommended modifiers be kept simple, complex logic with side effects can be put into internal functions  

### NC1 - Usage of different Solidity versions 

Contracts are making use of different versions of Solidity 
DelegateRegistry.sol ^0.8.21
RegistryHashes.sol ^0.8.21
RegistryStorage.sol ^0.8.21
RegistryOps.sol ^0.8.21
DelegateToken.sol ^0.8.21
PrincipalToken.sol ^0.8.21
CreateOfferer.sol ^0.8.21
CreateOffererLib.sol ^0.8.4
DelegateTokenLib.sol ^0.8.4
DelegateTokenRegistryHelpers.sol ^0.8.4
DelegateTokenStorageHelpers.sol ^0.8.4
DelegateTokenTransferHelpers.sol ^0.8.4

**Impact**-> It affects the code  maintainability, readability, auditability and most importantly can lead to issues and different versions have different features, bugs, errors, fixes which can result in differences amongst the compiled contracts 

**Recommendation** -> It is recommended to use the same Solidity version across all contracts e.g 0.8.21

### NC2 - Add interfaces to own folder

[DelegateRegistry.sol in /src/ folder](https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/IDelegateRegistry.sol)

**Impact**-> It affects the code maintainability, readability, organization, auditability and cleanliness of folders and structure

**Recommendation** -> It is recommended add all interfaces into their own /interface folders

### NC3 - Custom Errors should be prefixed by contract name

[PrincipalToken.sol line 15-18](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/PrincipalToken.sol#L15)
```solidity
 error DelegateTokenZero();
 error MarketMetadataZero();
 error CallerNotDelegateToken();
 error NotApproved(address spender, uint256 id);
```

[CreateOfferLib.sol errors line 11-30](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/libraries/CreateOffererLib.sol#L11C9-L11C9) 
e.g 
```
 error DelegateTokenIsZero();
 error PrincipalTokenIsZero();
```

[DelegateTokenLib.sol lines 43-58](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/libraries/DelegateTokenLib.sol#L43)

Error naming is not indicative of the contract name in which the errors will be thrown. It is best practise for errors to have a prefix of the contract they are applied in.

**Impact**-> Not naming errors with indication of contract can make error tracking, debuggin, analysis difficult as error data bubbles through chain of calls so it may not be clear if error contract receives is from where

**Recommendation** -> It is recommended the errors be named in accordance to the name of the contract in which they will be thrown e. g error
```
error PrinciplaToken__DelegateTokenZero();
error PrincipalToken__MarketMetadataZero();
error CreateOffer__DelegateTokenIsZero();
```
 used

#### NC-4 Capitalize immutable variables 

-Description => It is considered best practise to capitalize constants and immutables in code as these behave the same as they are part of the bytecode as added to parts of code where used, only difference is immutables are constants set in constructor.

[src/DelegateToken.sol line 21-26](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/DelegateToken.sol#L20C5-L26C45)
```solidity
/// @inheritdoc IDelegateToken
    address public immutable override delegateRegistry;

    /// @inheritdoc IDelegateToken
    address public immutable override principalToken;

    address public immutable marketMetadata;
```
[src/CreateOfferer.sol lines 24,25](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/CreateOfferer.sol#L24)
```
 address public immutable delegateToken;
 address public immutable principalToken;
```
[CreateOffererLib.sol line 137 address public immutable seaport;](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/libraries/CreateOffererLib.sol#L137)

-Recommendation => It is recommended to capitalize the immutable variables in instances above or any other relevant cases e.g delegateToken => DELEGATE_TOKEN


