**Gas Optimizations**
| count | Title |
|-------|-------|

| [G-01]   | use assembly to check for address 0 |
| [G-02]   | Functions guaranteed to revert when called by normal users can be marked `payable` |
| [G-03]   | Optimizing Storage Layout for Gas Efficiency |
| [G-04]   | avoid emitting storage variables |
| [G-05]   | Refactor internal function to avoid unnecessary SLOAD|


| Total Gas Optimization Issues| 7 |
|-------|-------|

[G-01]  use assembly to check for address 0
-----------------------------------------------


Recommendation:
--------------

```solidity
  if (setDelegateToken == address(0)) revert DelegateTokenZero();
 if (setMarketMetadata == address(0)) revert MarketMetadataZero();
```

Using assembly to check for the zero address can result in significant gas savings compared to using a Solidity expression; especially if the check is performed frequently or in a loop. However, it's important to note that using assembly can make the code less readable and harder to maintain, so it should be used judiciously and with caution.

[G-02]  Functions guaranteed to revert when called by normal users can be marked `payable`
-----------------------------------------------

In cases where  a function modifier such as onlyOwner is used, the function will revert incase a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include the necessary checks for checking if a payment was provided.

```solidity
 function setDefaultRoyalty(address receiver, uint96 feeNumerator) external onlyOwner {
        _setDefaultRoyalty(receiver, feeNumerator);
    }

    function deleteDefaultRoyalty() external onlyOwner {
        _deleteDefaultRoyalty();
    }

```

The above code is found in MarketMetadata.sol

[G-03]  Optimizing Storage Layout for Gas Efficiency
-----------------------------------------------

The contract has been improved by optimizing storage layout for gas efficiency. This optimization can lead to reduced gas costs, improved contract performance, and more efficient storage slot usage.
Optimizing storage layout is crucial for ensuring that smart contracts run efficiently on the Ethereum network. By grouping related data into structs and combining mappings into a single mapping, gas costs associated with storage operations can be reduced. This change contributes to better gas efficiency and a more economical use of storage slots.

[G-04] avoid emitting storage variables
-----------------------------------------

Avoiding emission of storage variables can have an impact on gas, it saves gas because emitting storage variables will  typically result in an additional gas costs making your contracts more expensive. When you emit a storage variable in contract, this involves reading the data from storage and then copying it to the logs. This operation can be relatively expensive in terms of gas costs.

[G-05]  Refactor internal function to avoid unnecessary SLOAD
-----------------------------------------
```solidity
  function name() external pure returns (string memory) {  
        return "Delegate Token";
    }

    /// @inheritdoc IERC721Metadata
    function symbol() external pure returns (string memory) {
        return "DT";
    }
```
you can use named returns for this.



