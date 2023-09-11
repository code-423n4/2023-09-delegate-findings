 
# Gas Optimizations Report

This report focuses on Delegate contest, in context of various improvements that can be made in terms of gas cost.

Some of the opportunities identified for improving gas efficiency throughout the codebase of Delegate are categorised into 06 main areas; with further multiple instances in each of the category.

## Summary
[G-01] Storage over memory (07 Instances)
[G 02] Consolidating library functions can save gas by preventing external calls (09 Instances)
[G-03] Revert inside a loop (01 Instances)
[G 04] Using private rather than public for constant/immutable, saves gas (06 Instances)
[G 05] abi.encode() is less efficient than abi.encodepacked()(09 Instances)
[G-06] It's cheaper to declare the variable outside the loop (02 Instances)


# [G-01] Storage over memory (07 Instances)
Some functions are using memory to read state variables while using storage is more gas efficient.
When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read.

Link to the Code:
1.	https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L34
2.	https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L59
3.	https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L94
4.	https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L116
5.	https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L142
6.	https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L157
7.	https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L298


# [G 02] Consolidating library functions can save gas by preventing external calls (09 Instances)
Libraries of functions commonly used across different contracts come at the increased cost of gas usage at runtime because of the external calls.
Consider moving all the library functions internal to contract or to a single library to save gas from external calls.

Link to the code:
1.	https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L5
2.	https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L6
3.	https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L10
4.	https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L11
5.	https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L12
6.	https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L13
7.	https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L6
8.	https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L7
9.	https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L8


# [G-03] Revert inside a loop (01 Instances)
Condition inside for loop will carry on, unless it finds one condition that is not met, it will revert the whole transaction. 
It is recommended that instead of reverting if condition is not true, to just break the loop. This way we don't need to run again the transaction.
Link to the Code: 
1.	https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L38


# [G-04] Using private rather than public for constant/immutable, saves gas (06 Instances)
Private over public saves around 21,000 for immutable and around 14,000 for constant in gas during deployment.
Gas consumption also increases with number of variable created with public visibility for both constant and immutable.
Link to the code:
1.	https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L21
2.	https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L24
3.	https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L26
4.	https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L24
5.	https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L25
6.	https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L137


# [G 05] abi.encode() is less efficient than abi.encodepacked()(09 Instances)
Refer to this [link]( https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison).

Link to the Code:
1.	https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L98
2.	https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L120
3.	https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L146
4.	https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L201
5.	https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L219
6.	https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L237
7.	https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L302
8.	https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L314
9.	https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L398



# [G-06] It's cheaper to declare the variable outside the loop (02 Instances)
Declaring a variable inside a loop result in variable being redeclared during each loop iteration which consume higher gas.
The variable gets reallocated when declared outside loop making it more gas efficient.

Link to the Code:
1.	https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L276
2.	https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L277
