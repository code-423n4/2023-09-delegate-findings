src/libraries/DelegateTokenTransferHelpers.sol

   /// @dev Should revert for a typical 721 / 1155 and pass for a typical 20
    function checkERC20BeforePull(uint256 underlyingAmount, address underlyingContract, uint256 underlyingTokenId) internal view {
        if (underlyingTokenId != 0) {
            revert Errors.WrongTokenIdForType(IDelegateRegistry.DelegationType.ERC20, underlyingTokenId);
        }
        if (underlyingAmount == 0) {
            revert Errors.WrongAmountForType(IDelegateRegistry.DelegationType.ERC20, underlyingAmount);
        }
        if (IERC20(underlyingContract).allowance(msg.sender, address(this)) < underlyingAmount) {
            revert Errors.InsufficientAllowanceOrInvalidToken();
        }
    }
