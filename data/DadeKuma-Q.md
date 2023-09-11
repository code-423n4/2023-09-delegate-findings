# Summary

| Id     | Title                                                                           |
|--------|---------------------------------------------------------------------------------|
| [L-01] | `previewOrder` should not revert                                                |
| [L-02] | Withdraw should revert with a not supported `delegationType`                    |
| [L-03] | Lack of `data` on flashloan could make some ERC1155 unusable                    |
| [L-04] | Using `delegatecall` inside a loop may cause issues with `payable` functions    |
| [L-05] | `CreateOfferer` uses a custom context implementation instead of an existing SIP |

## Low Risk

### [L-01] `previewOrder` should not revert

The official [documentation](https://github.com/ProjectOpenSea/seaport/blob/main/docs/SeaportDocumentation.md#arguments-and-basic-functionality) says that `previewOrder` should not revert, as it may cause issues to the caller:

> An optimal Seaport app should return an order with penalties when its previewOrder function is called with unacceptable minimumReceived and maximumSpent arrays, so that the caller can learn what the Seaport app expects. But it should revert when its generateOrder is called with unacceptable minimumReceived and maximumSpent arrays, so the function fails fast, gets skipped, and avoids wasting gas by leaving the validation to Seaport.


```solidity
function previewOrder(address caller, address, SpentItem[] calldata minimumReceived, SpentItem[] calldata maximumSpent, bytes calldata context)
    external
    view
    onlySeaport(caller)
    returns (SpentItem[] memory offer, ReceivedItem[] memory consideration)
{
    if (context.length != 160) revert Errors.InvalidContextLength();
    (offer, consideration) = Helpers.processSpentItems(minimumReceived, maximumSpent);
}
```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L176-L184

### [L-02] Withdraw should revert with non-supported `delegationType`

There seems to be no way to create a DelegateToken that is not ERC721/ERC20/ERC1155, but due to the fact that `DelegationType` supports also `ALL` and `CONTRACT`:

```solidity
enum DelegationType {
    NONE,
    ALL,
    CONTRACT,
    ERC721,
    ERC20,
    ERC1155
}
```

And there are multiple ways to create DelegateToken (e.g. `DelegateToken.create` and `CreateOfferer.transferFrom`, but more contracts could be created in the future), it would be safer to revert the transaction on withdraw to avoid burning unsupported tokens:

```solidity
function withdraw(uint256 delegateTokenId) external nonReentrant {
    ...
    } else if (delegationType == IDelegateRegistry.DelegationType.ERC1155) {
        uint256 erc1155UnderlyingAmount = StorageHelpers.readUnderlyingAmount(delegateTokenInfo, delegateTokenId);
        StorageHelpers.writeUnderlyingAmount(delegateTokenInfo, delegateTokenId, 0); // Deletes amount
        uint256 erc11551UnderlyingTokenId = RegistryHelpers.loadTokenId(delegateRegistry, registryHash);
        RegistryHelpers.decrementERC1155(
            delegateRegistry, registryHash, delegateTokenHolder, underlyingContract, erc11551UnderlyingTokenId, erc1155UnderlyingAmount, underlyingRights
        );
        StorageHelpers.burnPrincipal(principalToken, principalBurnAuthorization, delegateTokenId);
        IERC1155(underlyingContract).safeTransferFrom(address(this), msg.sender, erc11551UnderlyingTokenId, erc1155UnderlyingAmount, "");
    }
/* @audit Should revert here to avoid burning a not supported delegate token
    else {
        revert Errors.InvalidTokenType(delegationType);
    }
*/
}
```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L353-L386

### [L-03] Lack of `data` on flashloan could make some ERC1155 unusable

When doing an ERC1155 flashloan, the data parameter is always empty:

```solidity
IERC1155(info.tokenContract).safeTransferFrom(address(this), info.receiver, info.tokenId, info.amount, "");
```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L406

Some collections might need this parameter to be usable. This is an [example](https://etherscan.io/tx/0xd067dbef589678ad1c138e007fee644f1b5ed350043551983c90b9e46071fed2) of such case:

1. The sender transfers ERC1155 from the OpenSea contract to the ApeGang contract
2. In the same transaction the ApeGang's onERC1155BatchReceived hook is called
3. This allows the ApeGang to react to the token being received
4. The data includes merkle proofs that are needed to validate the transaction

Consider adding a `flashloanData` parameter to `Structs.FlashInfo` and pass it while transfering.


### [L-04] Using `delegatecall` inside a loop may cause issues with `payable` functions

If one of the `delegatecall` consumes part of the `msg.value`, other calls might fail, if they expect the full `msg.value`. Consider using a different design, or fully document this decision to avoid potential issues.

```solidity
function multicall(bytes[] calldata data) external payable override returns (bytes[] memory results) {
    results = new bytes[](data.length);
    bool success;
    unchecked {
        for (uint256 i = 0; i < data.length; ++i) {
            //slither-disable-next-line calls-loop,delegatecall-loop
            (success, results[i]) = address(this).delegatecall(data[i]);
            if (!success) revert MulticallFailed();
        }
    }
}
```

https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L37

### [L-05] `CreateOfferer` uses a custom context implementation instead of an existing SIP

It is suggested to use an existing [SIP implementation](https://github.com/ProjectOpenSea/SIPs/blob/main/SIPS/sip-7.md) instead of creating a new standard from scratch, which might be prone to errors:

```solidity
Structs.Context memory decodedContext = abi.decode(context, (Structs.Context));
```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L59

Please note that the contract needs to be SIP compliant before it's possible to implement SIP-7, as it requires SIP-5 and SIP-6. 

The issue describing non-compliance is described here: #280