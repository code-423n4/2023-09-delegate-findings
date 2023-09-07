# `flashloan` should reclaim assets from the receiver rather than from msg.sender.

https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L389-L396
```solidity
    function flashloan(Structs.FlashInfo calldata info) external payable nonReentrant {
        StorageHelpers.revertNotOperator(accountOperator, info.delegateHolder);
        if (info.tokenType == IDelegateRegistry.DelegationType.ERC721) {
            RegistryHelpers.revertERC721FlashUnavailable(delegateRegistry, info);
            IERC721(info.tokenContract).transferFrom(address(this), info.receiver, info.tokenId);
            Helpers.revertOnCallingInvalidFlashloan(info);
            TransferHelpers.checkERC721BeforePull(info.amount, info.tokenContract, info.tokenId);
            TransferHelpers.pullERC721AfterCheck(info.tokenContract, info.tokenId);
```

In `flashloan`, the token transfer path is as follows: contract -> receiver -> msg.sender -> contract


The receiver currently has to first transfer the tokens to the `msg.sender` and then back to the contract. I think this could be simplified by having the receiver directly transfer the tokens back to the contract with `transferFrom(receiver, contract)`, thereby saving on the gas costs for both the receiver-to-msg.sender transfer and the msg.sender's approval of the contract.