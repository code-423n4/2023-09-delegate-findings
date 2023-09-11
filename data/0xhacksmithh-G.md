### [Gas-01] Some `State Variables` could be `private` Instead of `public` to save gas cost
```solidity
    address public immutable override delegateRegistry; 
    
    address public immutable override principalToken;

    address public immutable marketMetadata;
```
```solidity
    address public immutable delegateToken; 
    address public immutable marketMetadata;
```
*Instances(5)*
```
File:
https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L21
https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L24
https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L26

https://github.com/code-423n4/2023-09-delegate/blob/main/src/PrincipalToken.sol#L12-L13
```

### [Gas-02] Caching `address(this)` in `immutable` state variable instead calling it again and again can helped in saving gas
```solidity
if (address(this) == operator)

newRegistryHash = RegistryHashes.erc1155Hash(address(this)

newRegistryHash = RegistryHashes.erc20Hash(address(this)

newRegistryHash = RegistryHashes.erc1155Hash(address(this),

IERC721(underlyingContract).transferFrom(address(this)

IERC1155(underlyingContract).safeTransferFrom(address(this),

IERC721(info.tokenContract).transferFrom(address(this)

IERC1155(info.tokenContract).safeTransferFrom(address(this)
```
*Instances()*
```
File:
https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L84
https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L178
https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L182
https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L196
https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L307
https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L312
https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L317
https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L369
https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L384
https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L393
https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L406
```
```
File:
https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L259
https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L280
https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L314
https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L352
https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L366
```

### [Gas-03] Most basic check `modifier` should be at front out of all modifiers 
Here `onlySeaport(msg.sender)` is a basic address validation modifier which checks `caller` is correct or not,
Where as other modifiers perform other function calls, and other operation.

So technically `onlySeaport(msg.sender)` consume less gas than other, so if caller is not allowed caller then function will revert on first modifier with consuming less gas. 
```solidity
    function generateOrder(address fulfiller, SpentItem[] calldata minimumReceived, SpentItem[] calldata 
    maximumSpent, bytes calldata context)
        external
+       onlySeaport(msg.sender)
        checkStage(Enums.Stage.generate, Enums.Stage.transfer)
-       onlySeaport(msg.sender) 
        returns (SpentItem[] memory offer, ReceivedItem[] memory consideration)
```
```solidity
 function ratifyOrder(SpentItem[] calldata offer, ReceivedItem[] calldata consideration, bytes calldata context, bytes32[] calldata, uint256 contractNonce)
        external
+       onlySeaport(msg.sender) 
        checkStage(Enums.Stage.ratify, Enums.Stage.generate)
-       onlySeaport(msg.sender) 
        returns (bytes4)
    {
```
*Instances(2)*
```
File: src/CreateOfferer.sol
https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L55
https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L74
```

### [Gas-04] Use `bytes32` Instead of `strings`
```solidity
string public delegateTokenBaseURI; 
```
*Instances(1)*
```
File:
https://github.com/code-423n4/2023-09-delegate/blob/main/src/MarketMetadata.sol#L14
```

### [Gas-05] `address` variable initialized with default value, which are missed in automated reports.
```solidity
address rightsOwner = address(0); 
```
*Instances(1)*
```
File:
https://github.com/code-423n4/2023-09-delegate/blob/main/src/MarketMetadata.sol#L80
```

### [Gas-06] Gas Can By using `modifier` instead of `internal` function in below case.
According to current senario
`mint()` which call `_checkDelegateTokenCaller()` internal function which verify that `msg.sender` is `delegateToken` or not.

### With internal function `mint()` execution cost = 47942 gas
### With modifier `mint()` execution cost = 688 gas
* Here `IDelegateToken(delegateToken).mintAuthorizedCallback();` excluded while making above test *

A huge gas can save if it(` _checkDelegateTokenCaller()`) convert to a modifier instaed
```solidity
    function _checkDelegateTokenCaller() internal view {
        if (msg.sender == delegateToken) return; 
        revert CallerNotDelegateToken();
    }

    /// @notice Mints a PT if and only if the DT contract calls and has authorized
    function mint(address to, uint256 id) external { 
        _checkDelegateTokenCaller(); // @audit gas cost can saveed with modifier
        _mint(to, id);
        IDelegateToken(delegateToken).mintAuthorizedCallback();
    }


+     modifier _checkDelegateTokenCaller(){
+       if (msg.sender == delegateToken) return; 
+       revert CallerNotDelegateToken();
+     _;
  }


    /// @notice Mints a PT if and only if the DT contract calls and has authorized
+   function mint(address to, uint256 id) external _checkDelegateTokenCaller{ 
       //_checkDelegateTokenCaller();
        _mint(to, id);
    }
``` 
*Instances(1)*
```
File:
https://github.com/code-423n4/2023-09-delegate/blob/main/src/PrincipalToken.sol#L27-L30
```

### [Gas-07] When a `struct` has single variable value in it, instead of using it in a `struct` we could use it as state variable  
```solidity
-   struct Nonce {
-       uint256 value;
-   }

+  uint256 public value;
```
```solidity
-   struct Uint256 { // @audit what rhis name
-       uint256 flag; // @audit
-   }

+   uint256 public flag; 
```
*Instances(2)*
```
File:
https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L72-L74
https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenLib.sol#L9-L11
```

