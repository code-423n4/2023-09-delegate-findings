# Gas Optimizations Report

| Gas Optimizations | Issues                                                                      | Instances |
|-------------------|-----------------------------------------------------------------------------|-----------|
| [G-01]            | processSpentItems should have non-array input to save gas | 1         

**Total issues: 1 instances across 1 issue
Total gas saved: 1189 gas**

## [G-01] processSpentItems should have non-array input to save gas

**Total gas saved: 1189 gas**

https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L346

Before VS After
Function execution cost: 45583 - 44394 = 1189 gas saved (min.)

Instead of this:
```solidity
    function processSpentItems(SpentItem[] calldata minimumReceived, SpentItem[] calldata maximumSpent)
        internal
        view
        returns (SpentItem[] memory offer, ReceivedItem[] memory consideration)
    {
        if (!(minimumReceived.length == 1 && maximumSpent.length == 1)) revert CreateOffererErrors.NoBatchWrapping();
        if (minimumReceived[0].itemType != ItemType.ERC721 || minimumReceived[0].token != address(this) || minimumReceived[0].amount != 1) {
            revert CreateOffererErrors.MinimumReceivedInvalid(minimumReceived[0]);
        }
        if (maximumSpent[0].itemType != ItemType.ERC721 && maximumSpent[0].itemType != ItemType.ERC20 && maximumSpent[0].itemType != ItemType.ERC1155) {
            revert CreateOffererErrors.MaximumSpentInvalid(maximumSpent[0]);
        }
        offer = new SpentItem[](1);
        offer[0] = SpentItem({itemType: minimumReceived[0].itemType, token: minimumReceived[0].token, identifier: minimumReceived[0].identifier, amount: minimumReceived[0].amount});
        consideration = new ReceivedItem[](1);
        consideration[0] = ReceivedItem({
            itemType: maximumSpent[0].itemType,
            token: maximumSpent[0].token,
            identifier: maximumSpent[0].identifier,
            amount: maximumSpent[0].amount,
            recipient: payable(address(this))
        });
    }
```
Use this:
```solidity
    function processSpentItems(SpentItem memory minimumReceived, SpentItem memory maximumSpent)
        internal
        view
        returns (SpentItem[] memory offer, ReceivedItem[] memory consideration)
    {
        if (minimumReceived.itemType != ItemType.ERC721 || minimumReceived.token != address(this) || minimumReceived.amount != 1) {
            revert CreateOffererErrors.MinimumReceivedInvalid(minimumReceived);
        }
        if (maximumSpent.itemType != ItemType.ERC721 && maximumSpent.itemType != ItemType.ERC20 && maximumSpent.itemType != ItemType.ERC1155) {
            revert CreateOffererErrors.MaximumSpentInvalid(maximumSpent);
        }
        offer = new SpentItem[](1);
        offer[0] = SpentItem({itemType: minimumReceived.itemType, token: minimumReceived.token, identifier: minimumReceived.identifier, amount: minimumReceived.amount});
        consideration = new ReceivedItem[](1);
        consideration[0] = ReceivedItem({
            itemType: maximumSpent.itemType,
            token: maximumSpent.token,
            identifier: maximumSpent.identifier,
            amount: maximumSpent.amount,
            recipient: payable(address(this))
        });
    }
```
