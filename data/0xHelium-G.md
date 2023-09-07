Struct can be packed to save gas .
There is the code concerned:
https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/libraries/DelegateTokenLib.sol#L31-L39

Instead of having this struct , it can be packed to save gas:
// @audit: struct can be packed to save gas, 
   // bytes = bytes1 = 1 byte. 
   // uint256(32), uint256(32), address(20) 
   // address(20), address(20), bytes(1), IDelegateRegistry.DelegationType(1) 
   // 5 storage slot instead of current 6
    struct FlashInfo {
        address receiver; // The address to receive the loaned assets
        address delegateHolder; // The holder of the delegation
        IDelegateRegistry.DelegationType tokenType; // The type of contract, e.g. ERC20
        address tokenContract; // The contract of the underlying being loaned
        uint256 tokenId; // The tokenId of the underlying being loaned, if applicable
        uint256 amount; // The amount being lent, if applicable
        bytes data; // Arbitrary data structure, intended to contain user-defined parameters
    }