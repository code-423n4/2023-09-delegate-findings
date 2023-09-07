# DELEGATE GAS OPTIMISATIONS


## INTRODUCTION
Please be aware that some code snippets may be shortened to conserve space, and certain code snippets may include @audit tags in comments to facilitate issue explanations."

Emphasizing the significance of our recommendations, we strive to improve the code's efficiency while maintaining its readability. We recognize the importance of code that is easily maintainable and understandable for both developers and auditors.

## TABLE OF FINDINGS

| Number |Issue|Instances|Total Gas Saved|
|-|:-|:-:|:-:|
| G-01| Avoid updating state variable when value has not changed | 1 | 800 |
| G-02| Sort Solidity operations using short-circuit mode | 1 | - |
| G-03| Unbounded Gas Consumption Risk Due to External Call Recipients | 3 |  - |
| G-04| Remove or replace unused state variables | 1 | - |


## [G-01] Avoid updating state variable when value has not changed
Every time you update a state variable, it incurs a gas cost. This cost is primarily associated with writing data to the Ethereum network's distributed ledger.

So when you are writing data to a state variable, you should only do so if the new value is different from the current value stored in that variable. If the new value is the same as the existing one, you should skip the update to save gas.

### 1 Instance
- https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L145
```solidity
file: src/DelegateToken.sol

144:    function setApprovalForAll(address operator, bool approved) external {
145:        accountOperator[msg.sender][operator] = approved;
146:        emit ApprovalForAll(msg.sender, operator, approved);
147:    }
```
If the old value is equal to the new value, not re-storing the value will avoid a `Gsreset (2900 gas)`, potentially at the expense of a `Gcoldsload (2100 gas)` or a `Gwarmaccess (100 gas)`
```diff
diff --git a/lib/seaport b/lib/seaport
--- a/lib/seaport
+++ b/lib/seaport
@@ -1 +1 @@
-Subproject commit ab3b5cb6e10580ea979d63983e409e679935c702
+Subproject commit ab3b5cb6e10580ea979d63983e409e679935c702-dirty
diff --git a/src/DelegateToken.sol b/src/DelegateToken.sol
index ff7fcce..d1090df 100644
--- a/src/DelegateToken.sol
+++ b/src/DelegateToken.sol
@@ -142,8 +142,11 @@ contract DelegateToken is ReentrancyGuard, IDelegateToken {

     /// @inheritdoc IERC721
     function setApprovalForAll(address operator, bool approved) external {
-        accountOperator[msg.sender][operator] = approved;
-        emit ApprovalForAll(msg.sender, operator, approved);
+        if(accountOperator[msg.sender][operator] != approved;) {
+            accountOperator[msg.sender][operator] = approved;
+            emit ApprovalForAll(msg.sender, operator, approved);
+        }
+
```
```
Estimated gas saved: 800 gas units
```


## [G-02] Sort Solidity operations using short-circuit mode
Short-circuiting is a solidity contract development model that uses OR/AND logic to sequence different cost operations. It puts low gas cost operations in the front and high gas cost operations in the back, so that if the front is low If the cost operation is feasible, you can skip (short-circuit) the subsequent high-cost Ethereum virtual machine operation.

f(x) is a low gas cost operation \
g(y) is a high gas cost operation 

Sort operations with different gas costs as follows: \
f(x) || g(y) \
f(x) && g(y)

### 1 Instance
1. #### Refactor `if (erc1155Pulled.flag == ERC1155_PULLED && address(this) == operator)` to use princple of short-circuiting
- https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenTransferHelpers.sol#L78
```solidity
file: src/libraries/DelegateTokenTransferHelpers.sol

77:    function checkERC1155Pulled(Structs.Uint256 storage erc1155Pulled, address operator) internal returns (bool) {
78:        if (erc1155Pulled.flag == ERC1155_PULLED && address(this) == operator) { @audit use short-circuit
.
.
.
83:    }
```
In the `checkERC1155Pulled` function above the if statement ` if (erc1155Pulled.flag == ERC1155_PULLED && address(this) == operator)` could be refactored such that the less gas consuming statement `address(this) == operator` (which is just comparison between stack variable ) comes before the more gas consuming statement `erc1155Pulled.flag == ERC1155_PULLED` (which involves reading a storage variable) so that in scenarios where the `address(this) == operator` results to false the EVM would not need to perform the gas consuming operations of `erc1155Pulled.flag == ERC1155_PULLED`. The diff below shows how the code could be refactored: 
```diff
diff --git a/lib/seaport b/lib/seaport
--- a/lib/seaport
+++ b/lib/seaport
@@ -1 +1 @@
-Subproject commit ab3b5cb6e10580ea979d63983e409e679935c702
+Subproject commit ab3b5cb6e10580ea979d63983e409e679935c702-dirty
diff --git a/src/libraries/DelegateTokenTransferHelpers.sol b/src/libraries/DelegateTokenTransferHelpers.sol
index ad007a9..4641382 100644
--- a/src/libraries/DelegateTokenTransferHelpers.sol
+++ b/src/libraries/DelegateTokenTransferHelpers.sol
@@ -75,7 +75,7 @@ library DelegateTokenTransferHelpers {
     }

     function checkERC1155Pulled(Structs.Uint256 storage erc1155Pulled, address operator) internal returns (bool) {
-        if (erc1155Pulled.flag == ERC1155_PULLED && address(this) == operator) {
+        if (address(this) == operator && erc1155Pulled.flag == ERC1155_PULLED) {
             erc1155Pulled.flag = ERC1155_NOT_PULLED;
             return true;
         }
```

## [G-03] Unbounded Gas Consumption Risk Due to External Call Recipients
In the context of Solidity, external function calls without a specified gas limit present a significant risk. The callee contract has the potential to consume all the gas allocated to the transaction, causing an undesired revert and disrupt the function's execution. To mitigate this, it's recommended to explicitly set a gas limit when performing external calls using addr.call{gas: }. This limits the gas forwarded to the callee, preventing potential pitfalls and offering better control over transaction execution.
{https://gist.github.com/thebrittfactor/c400e0012d0092316699c53843ecad41#gas-24-unbounded-gas-consumption-risk-due-to-external-call-recipients}

### 1 Instance
1. #### Specify gas for a low-level external call
- https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L37
```solidity
file: rc/DelegateRegistry.sol

31:    function multicall(bytes[] calldata data) external payable override returns (bytes[] memory results) {
32:        results = new bytes[](data.length);
33:        bool success;
34:        unchecked {
35:            for (uint256 i = 0; i < data.length; ++i) {
36:                //slither-disable-next-line calls-loop,delegatecall-loop
37:                (success, results[i]) = address(this).delegatecall(data[i]);  @audit specify amount of gas for external call
38:                if (!success) revert MulticallFailed();
39:            }
40:        }
41:    }
```
In the `multicall()` function a low-level call was made without specifing the amount of gas to be used in the low-level call. This is bad practice as the called contract could possibly use up all the gas thereby causing the `multicall()` function to revert and the transaction fails. This scenario could lead to bad consumer experience, you should specify the amount of gas to be used in a low-level call as this would prevent such occurences as only a predefined amount of gas can be used by the called contract. The code could be refactored to:
```diff                                                                                                
diff --git a/src/DelegateRegistry.sol b/src/DelegateRegistry.sol                                          
index 26b1986..b35b6ac 100644                                                                             
--- a/src/DelegateRegistry.sol                                                                            
+++ b/src/DelegateRegistry.sol                                                                            
@@ -34,7 +34,7 @@ contract DelegateRegistry is IDelegateRegistry {                                        
         unchecked {                                                                                      
             for (uint256 i = 0; i < data.length; ++i) {                                                  
                 //slither-disable-next-line calls-loop,delegatecall-loop                                 
-                (success, results[i]) = address(this).delegatecall(data[i]);                             
+                (success, results[i]) = address(this).delegatecall{ gas: /* specfy gas here */}(data[i]);
                 if (!success) revert MulticallFailed();                                                  
             }                                                                                            
         }                                            
```

## [G-04] Remove or replace unused state variables
Saves a storage slot. If the variable is assigned a non-zero value, saves Gsset (20000 gas). If it's assigned a zero value, saves Gsreset (2900 gas). If the variable remains unassigned, there is no gas savings unless the variable is public, in which case the compiler-generated non-payable getter deployment cost is saved. If the state variable is overriding an interface's public function, mark the variable as constant or immutable so that it does not use a storage slot

### 1 Instance
- https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L18
```solidity
file: 

18:    mapping(bytes32 delegationHash => bytes32[5] delegationStorage) internal delegations;
```
The `delegations` state variable was not used in the contract and from my search i could not find other contracts of the protocol that inherited from `DelegateRegistry`contract and used the `delegations` mapping i would recommend the code be refactored and the `delegations` mapping be removed or it should used in the contract.


## CONCLUSION
As you move forward with implementing the suggested optimizations, we urge you to exercise caution and conduct meticulous testing. It is essential to verify that these changes do not introduce any new vulnerabilities and effectively deliver the desired performance improvements. Carefully review the code modifications and perform rigorous testing to validate the security and effectiveness of the refactored code.
