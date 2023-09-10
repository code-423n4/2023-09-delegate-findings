## [L-01]  Unsafe ERC20 Operation(s)
Description
ERC20 operations can be unsafe due to different implementations and vulnerabilities in the standard.

It is therefore recommended to always either use OpenZeppelin's SafeERC20 library or at least to wrap each operation in a require statement.

To circumvent ERC20's approve functions race-condition vulnerability use OpenZeppelin's SafeERC20 library's safe{Increase|Decrease}Allowance functions.

In case the vulnerability is of no danger for your implementation, provide enough documentation explaining the reasonings.

Common Unsafe ERC20 Operations:

Missing Check for transfer Return Value:

Impact: If the transfer function does not check the return value for success, a malicious contract can call it, causing a loss of tokens without notifying the sender.
Mitigation: Always check the return value of the transfer function and handle failed transfers appropriately.
Reentrancy Vulnerabilities:

Impact: If the ERC20 contract implements custom logic in the transfer function, it might be vulnerable to reentrancy attacks, allowing malicious contracts to manipulate the state and steal tokens.
Mitigation: Implement the checks-effects-interactions pattern and use revert or require to revert the transaction on failure before making any external calls.
Incorrect Balance Updates:

Impact: Incorrectly updating token balances within the contract can lead to inconsistencies and enable attackers to mint or burn tokens maliciously.
Mitigation: Ensure that balance updates are atomic and validate them thoroughly to prevent unauthorized minting or burning of tokens.
Lack of Approval Checks:

Impact: Failing to check for approval before performing token transfers can lead to unauthorized transfers and loss of tokens.
Mitigation: Always check the allowance of the spender using the allowance function before executing a transferFrom operation.
Overflow and Underflow:

Impact: Integer overflow and underflow vulnerabilities can be exploited to manipulate token balances or perform unauthorized operations.
Mitigation: Use SafeMath or similar libraries to prevent arithmetic overflows and underflows.
Uninitialized Variables:

Impact: Using uninitialized variables can lead to unpredictable behavior and potential security vulnerabilities.
Mitigation: Ensure that all variables are properly initialized before use, especially when dealing with token balances and state changes.
```txt
2023-09-delegate/src/CreateOfferer.sol::121 => if (!IERC20(erc20Order.info.tokenContract).approve(address(delegateToken), erc20Order.amount)) {
2023-09-delegate/src/DelegateToken.sol::369 => IERC721(underlyingContract).transferFrom(address(this), msg.sender, erc721UnderlyingTokenId);
2023-09-delegate/src/DelegateToken.sol::393 => IERC721(info.tokenContract).transferFrom(address(this), info.receiver, info.tokenId);
2023-09-delegate/src/libraries/DelegateTokenTransferHelpers.sol::41 => IERC721(underlyingContract).transferFrom(msg.sender, address(this), underlyingTokenId);
```
## [L-02] Unspecific Compiler Version Pragma
Description
Avoid floating pragmas for non-library contracts.

While floating pragmas make sense for libraries to allow them to be included with multiple different versions of applications, it may be a security risk for application implementations.

A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up checking a different EVM compilation that is ultimately deployed on the blockchain.

It is recommended to pin to a concrete compiler version.

One common practice is to use a fixed pragma version to ensure code stability and security. However, some developers opt for a "floating pragma," which allows the compiler to automatically use the latest version. While this may seem convenient, it can have significant security implications. In this article, we will explore the security impact of using a floating pragma in Solidity smart contracts.

Security Impact:

Compiler Vulnerabilities: Solidity is continuously evolving, and each new version typically includes bug fixes, optimizations, and security improvements. By using a floating pragma, smart contracts may unintentionally rely on the latest compiler version. This can introduce vulnerabilities if the latest compiler version contains undiscovered bugs or security flaws. Developers who use a fixed pragma can audit and test their code with a known compiler version, reducing the risk of relying on potentially flawed compiler releases.

Dependency Risks: Smart contracts often depend on external libraries or contracts. These dependencies can be sensitive to changes in the Solidity compiler. A floating pragma might inadvertently trigger updates to these dependencies, introducing compatibility issues, or worse, exposing security vulnerabilities. Smart contract developers need to carefully manage and update dependencies to avoid unexpected issues when using a floating pragma.

Incompatibility with Legacy Code: Smart contracts can interact with each other, and sometimes, a smart contract written with a floating pragma may need to interact with a legacy contract written using an older compiler version. This can lead to compatibility issues, as the two contracts may have different storage layouts, function call behaviors, or other incompatibilities. Ensuring interoperability between contracts becomes challenging when using a floating pragma.

Lack of Code Review: A fixed pragma version encourages developers to review their code thoroughly and test it with a specific compiler version. In contrast, a floating pragma may lead to complacency, as developers might assume that their code is always up-to-date and secure. This can result in inadequate code review and testing, increasing the likelihood of introducing vulnerabilities.

Unexpected Behavior: The introduction of new features or changes in compiler behavior can lead to unexpected behavior in smart contracts using a floating pragma. This can result in unintended functionality or vulnerabilities that are difficult to anticipate. Developers should have a clear understanding of how changes in the compiler version might impact their contracts.
```txt
2023-09-delegate/src/CreateOfferer.sol::2 => pragma solidity ^0.8.21;
2023-09-delegate/src/DelegateToken.sol::2 => pragma solidity ^0.8.21;
2023-09-delegate/src/PrincipalToken.sol::2 => pragma solidity ^0.8.21;
2023-09-delegate/src/libraries/CreateOffererLib.sol::2 => pragma solidity ^0.8.4;
2023-09-delegate/src/libraries/DelegateTokenLib.sol::2 => pragma solidity ^0.8.4;
2023-09-delegate/src/libraries/DelegateTokenRegistryHelpers.sol::2 => pragma solidity ^0.8.4;
2023-09-delegate/src/libraries/DelegateTokenStorageHelpers.sol::2 => pragma solidity ^0.8.4;
2023-09-delegate/src/libraries/DelegateTokenTransferHelpers.sol::2 => pragma solidity ^0.8.4;
```