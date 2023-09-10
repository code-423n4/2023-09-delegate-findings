Writing magic values manually is always error-prone. 
The contract has the interfaceId values manually written https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L65. 

A typo in these values could cause integration issues to the protocol. 

Recommendation(s): Consider using the syntax type(Type).interfaceId instead of manually writing the values.
e.g. type(IERC721).interfaceId, type(IERC1155).interfaceId, etc.