 **Low risk**
| count | Title |
|-------|-------|
| 
| [L-01]   | Enhance Access Controll using Modifiers |
| [L-02]   | Minting to address(0) |
| [L-03]   | Comment Clarification in checkMintAuthorized Function |
| [L-04]   | Unnecessary Default Initialization of uint256|


| Total Low Risk Issues| 4 |
|-------|-------|

 **Non-Critical**
| count | Title |
|-------|-------|
| [NC-01]   | conflicting variables Naming convention |
| [NC-02]   |Correction of typo in the delegateTokenURI |
| [NC-03]   | Improved variable naming |
| [NC-04]   |Use name error function to enhance clarity and Debugging |
| [NC-05]   | Function Orders |


| Total Non-Critical Issues| 5 |
|-------|-------|


Low Risk
---------
 [L-01]   Access Controll:Enhance Access controll using Modifiers 
------------------------------------------------------------------

in the current code specifically on PrincipalToken.sol file access controll on certain functions specifically the mint and burn functions
are implemented through inline checks, while this approach is functional it is missing the organization and readeability from modifiers
you have the following code which checks  the delegate token caller.

```solidity
 function _checkDelegateTokenCaller() internal view {
        if (msg.sender == delegateToken) return;  
        revert CallerNotDelegateToken();
    }
```
This code above can easily be  found [here](https://github.com/code-423n4/2023-09-delegate/blob/main/src/PrincipalToken.sol#L27C4-L30C6)

Recommendation:
----------

use the modifier instead

```solidity
modifier onlyDelegateTokenCaller() {
    require(msg.sender == delegateToken, "Caller is not the delegateToken");
    _;
}
```
there after you can add the modifier to the mint and burn function as follows

```solidity
function mint(address to, uint256 id) external onlyDelegateTokenCaller {
    _mint(to, id);
    IDelegateToken(delegateToken).mintAuthorizedCallback();
}

```

Using modifiers for access controll is best practice in solidity, this approach will enhance code readability and will elimintate code duplication.

 [L-02]    Minting to address(0) 
------------------------------------------------------------------

in your mint function in the PrincipalToken.sol you are defining a mint function as follows:

```solidity
 function mint(address to, uint256 id) external {
        _checkDelegateTokenCaller(); 
        _mint(to, id);
        IDelegateToken(delegateToken).mintAuthorizedCallback();
    }

```

you might want to consider adding cheks in the function to ensure you are not minting to address(0).
This may result to loss of tokens, To prevent this you have to check to ensure that
you are not minting to address(0). This will protect the integrity of your contract by ensuring that tokens are only minted to valid addresses.

 [L-03]  Comment Clarification in checkMintAuthorized Function 
------------------------------------------------------------------
In the checkMintAuthorized function, the comment provided incorrectly states what the function is acctually doing which can be confusing, 
in your comment "must revert if delegate token did not call burn on the Principal Token for the delegateTokenId". link to this comment can be found [here](https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenStorageHelpers.sol#L102):

the function should revert if delegate token did not call mint on the principal Token for the delegateTokenId. based on this function

```solidity

  function checkBurnAuthorized(address principalToken, Structs.Uint256 storage principalBurnAuthorization) internal view {
        principalIsCaller(principalToken);
        if (principalBurnAuthorization.flag == BURN_AUTHORIZED) return;
        revert Errors.BurnNotAuthorized();
    }

```

Recommendation:
-----------

the comment inaccurately describes the logic behind this function. to maintain code clarity update the burn to should revert if delegate token did not call mint on the principal Token. to accurately reflect the functionality of the function.



 [L-05]   Unnecessary Default Initialization of uint256
------------------------------------------------------------------

```solidity
 function revertERC20FlashAmountUnavailable(address delegateRegistry, Structs.FlashInfo calldata info) internal view {
        uint256 availableAmount = 0;
        unchecked {
            // We sum the delegation amounts for "flashloan" and "" rights since liquid delegate doesn't allow double spends for different rights
            availableAmount = loadAmount(delegateRegistry, RegistryHashes.erc20Hash(address(this), "flashloan", info.delegateHolder, info.tokenContract))
                + loadAmount(delegateRegistry, RegistryHashes.erc20Hash(address(this), "", info.delegateHolder, info.tokenContract));
        } // Unreasonable that this block will overflow
        if (info.amount > availableAmount) revert Errors.ERC20FlashAmountUnavailable();
    }

```
Recommendation:
-----------

uint256 availableAmount;

Non-Critical
---------
 [NC-01]    conflicting variables Naming convention 
---------------------------------------------------

Try to maintain naming consistency for your state variables thoroughout your code base so it will be easier to know what 
you are dealing with , knowing out of the box you are dealing with a storage variable while using it will ensure you are aware of
gas optimization techniques

in your code base all storage immutables and constants are using the same naming convention constants and immutables are all uppercase insome cases lowecase immutables
you might want to consider using this. to make it easier 

for constants use

```solidity
uint256 public constant  MY_CONSTANT = 1;

```

for storage variables  use

```solidity

uint256 public  s_simpleStorage;

```

for immutable use

```solidity

uint256 private immutable i_owner;

```

by doing this whenever you want to use any of the variable in your code and you see s_simpleStorage this will bring gas optimizations to your attention.

[NC-02]    Correction of typo in the delegateTokenURI
---------------------------------------------------

in MaketMetadata.sol  there is this an instance of a typographical error in the  firstPartOfMetadataString where in the description timeperiod is used istead of time period.
Accurate spelling and grammar in comments are essential for maintaining code professionalism and clarity.

To address this issue, we recommend replacing the typo "timeperiod" with the correct term "time period" in code comments throughout the codebase.

code can be located here [here](https://github.com/code-423n4/2023-09-delegate/blob/main/src/MarketMetadata.sol#L47)

[NC-03]    Improved variable naming 
---------------------------------------------------

in your MarketMetadata.sol in the following line:
[view line on github](https://github.com/code-423n4/2023-09-delegate/blob/main/src/MarketMetadata.sol#L38)

in the line:

```solidity
string memory idstr = Strings.toString(delegateTokenId);

```

The is a variable named idstr is used to store the string representation of delegateTokenId.
However, the variable name idstr is not sufficiently descriptive and does not convey its purpose clearly.
you might want to change that to:

```solidity
string memory tokenIdString = Strings.toString(delegateTokenId);
```
same instance can be found on line:

[view on github](https://github.com/code-423n4/2023-09-delegate/blob/main/src/MarketMetadata.sol#L40C8-L40C110)

```solidity

 string memory pownerstr = principalOwner == address(0) ? "N/A" : Strings.toHexString(principalOwner);

```
using pownerstr is not very clear you can change this to 

```solidity

 string memory principalOwnerString = principalOwner == address(0) ? "N/A" : Strings.toHexString(principalOwner);
```
Using descriptive naming will make your code more clear and self-explanatory and can be especially helpful for you and other developers who may work with or review your code in the future.
It's a matter of code readability and maintainability.


**Explanation:**

This issue is classified as "Low" and "Informational" because it primarily pertains to naming conventions and code readability.
While it doesn't pose a security risk, clear and descriptive variable names are a best practice that contributes to code maintainability and understanding.

[NC-04]    Use name error function to enhance clarity and Debugging 
---------------------------------------------------
throughout your code base you are defining your error function using the syntax:

error ErrorName();

in this case lets say an error occurs you will not know which file is throwing what error as they are too many.
for a better and improved clarity and debugging it is a good practice to prefix your error with the contract name, such as:

```solidity
error PrincipalToken__DelegateTokenZero();
error DelegateToken__ToIsZero();
```
This syntax will make it easier to identify the source of an error, and will facilitate error tracking during code execution.

 [NC-05]    Function Orders  
------------------------------------------------------------------
functions in a contract should be grouped and ordered as follows with the view and pure functions being placed last in each group:

constructor
receive function (if exists)
fallback function (if exists)
external
public
internal
private

in the PrincipalToken.sol your order is as follows:
internal
external
external
external
public 

please update it to above for better convention

