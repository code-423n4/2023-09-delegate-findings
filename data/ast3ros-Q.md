# [L-1] The flashloan is not followed ERC-3156: Flash Loans

When implementing the flashloan, the lender should follow the ERC-3156: Flash Loans standard by implementing the interface `IERC3156FlashLender`. It will details the max amount and the fees for the users. 

https://eips.ethereum.org/EIPS/eip-3156

And in `flashLoan` function, the result of the callback function `receiver.onFlashLoan` should be the compare to the `keccak256("ERC3156FlashBorrower.onFlashLoan")` instead of `IDelegateFlashloan.onFlashloan.selector`.

https://github.com/code-423n4/2023-09-delegate/blob/bf8240bc3403f07c5466d6a35c20b77bb6dac49a/src/DelegateToken.sol#L389-L410
https://github.com/code-423n4/2023-09-delegate/blob/bf8240bc3403f07c5466d6a35c20b77bb6dac49a/src/libraries/DelegateTokenLib.sol#L92

## Impact

Not following the standard will make the flashloan not compatible with other flashloan contracts and it leads to revert of the transaction. Other protocol will needs to implement a specific flashloan contract for the Delegate protocol.

## Recommended Mitigation Steps

Implement the flashloan following the ERC-3156

# [L-2] The check `checkERC721BeforePull` is insufficient

When DelegateToken contract pulls the token from the user, it perform check before pull. In ERC20 and ERC1155, it checks the allowance of the user to the contract. However, in ERC721, it only checks if the user is the owner of the token. It does not check if the user has approved the token to the contract. It is inconsistent and insufficient.

https://github.com/code-423n4/2023-09-delegate/blob/bf8240bc3403f07c5466d6a35c20b77bb6dac49a/src/libraries/DelegateTokenTransferHelpers.sol#L35-L37

## Recommended Mitigation Steps

The function should check: 

        require(IERC721(info.tokenContract).getApproved(info.tokenId) == address(this));

# [NC-1] No check for address 0 when setApprovalForAll

When setting the approval, the function does not check if the address of the operator is 0 or not.

Instance:
https://github.com/code-423n4/2023-09-delegate/blob/bf8240bc3403f07c5466d6a35c20b77bb6dac49a/src/DelegateToken.sol#L144-L147

Please check OZ implementation.
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/9ef69c03d13230aeff24d91cb54c9d24c4de7c8b/contracts/token/ERC721/ERC721.sol#L438

# [NC-2] checkERC1155BeforePull should follow pullERC1155AfterCheck function

`checkERC1155BeforePull` will change the erc1155Pulled.flag from `ERC1155_NOT_PULLED` to `ERC1155_PULLED` so the ERC1155 token can be pulled to the Delegate Token. 
However between the check and the pull, external calls are made and users can controll the code flow in between flags. It may lead to unexpected behaviour.

        IERC1155(info.tokenContract).safeTransferFrom(address(this), info.receiver, info.tokenId, info.amount, "");
        Helpers.revertOnCallingInvalidFlashloan(info);

https://github.com/code-423n4/2023-09-delegate/blob/bf8240bc3403f07c5466d6a35c20b77bb6dac49a/src/DelegateToken.sol#L405-L408

## Recommended Mitigation Steps

Follow the  `checkERC1155BeforePull` by `pullERC1155AfterCheck` function to avoid unexpected behaviour.



```diff
-       TransferHelpers.checkERC1155BeforePull(erc1155PullAuthorization, info.amount);
        IERC1155(info.tokenContract).safeTransferFrom(address(this), info.receiver, info.tokenId, info.amount, "");
        Helpers.revertOnCallingInvalidFlashloan(info);
+       TransferHelpers.checkERC1155BeforePull(erc1155PullAuthorization, info.amount);
        TransferHelpers.pullERC1155AfterCheck(erc1155PullAuthorization, info.amount, info.tokenContract, info.tokenId);
```
