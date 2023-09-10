### Issue 1:
The condition validation at L95 basically checks if erc721Order.info.targetToken is delegate or principal, a more simpler and readable approach is to simply check if it is not none of the two, this is more direct, readable and saves gas.
https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L95
```solidity
        ---    if (!(erc721Order.info.targetToken == Enums.TargetToken.delegate || erc721Order.info.targetToken == Enums.TargetToken.none)) {
        +++    if (erc721Order.info.targetToken == Enums.TargetToken.none) {
                revert Errors.TargetTokenInvalid(erc721Order.info.targetToken);
            }
```
A track down of enum TargetToken below shows that there are 3 possibilities, so instead of checking for the opposite of the 2nd & 3rd possibilities, it is more direct to simply check for the 1st possibility (none).
https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L42-L46
```solidity
    enum TargetToken {
        none,
        principal,
        delegate
    }
```
Two Other instances of this can be found at [L117](https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L117) & [L143](https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L143) of the same contract
https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L117
https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L143
### Issue 2:
Quality of return data in L230-L235 of tokenURI(...) function in DelegateToken.sol contract would be affected due to Typographical Error in L47 of delegateTokenURI(...) function in MarketMetadata.sol contract, It should be "time period" not "timeperiod"
https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L230-L235
```solidity
 function tokenURI(uint256 delegateTokenId) external view returns (string memory) {
        ....
        return MarketMetadata(marketMetadata).delegateTokenURI(
            RegistryHelpers.loadContract(delegateRegistry, registryHash),
            RegistryHelpers.loadTokenId(delegateRegistry, registryHash),
            StorageHelpers.readExpiry(delegateTokenInfo, delegateTokenId),
            IERC721(principalToken).ownerOf(delegateTokenId)
        );
    }
```
https://github.com/code-423n4/2023-09-delegate/blob/main/src/MarketMetadata.sol#L47
```solidity
37. function delegateTokenURI(address tokenContract, uint256 delegateTokenId, uint256 expiry, address principalOwner) external view returns (string memory) {
...
46.            idstr,
--->47.            '","description":"DelegateMarket lets you escrow your token for a chosen timeperiod and receive a token representing the associated delegation rights. This collection represents the tokenized delegation rights.","attributes":[{"trait_type":"Collection Address","value":"',
            Strings.toHexString(tokenContract),
```
Another Instance of this error is in L54-L57 of tokenURI(...) function in PrincipalToken.sol contract from principalTokenURI(...) function in MarketMetadata.sol contract
https://github.com/code-423n4/2023-09-delegate/blob/main/src/PrincipalToken.sol#L56
```solidity
  function tokenURI(uint256 id) public view override returns (string memory) {
        _requireMinted(id);
        return MarketMetadata(marketMetadata).principalTokenURI(delegateToken, id);
    }
}
```
https://github.com/code-423n4/2023-09-delegate/blob/main/src/MarketMetadata.sol#L92
```solidity
72. function principalTokenURI(address delegateToken, uint256 id) external view returns (string memory) {
...
91.            string.concat(dt.name(), " #", idstr),
--->92.            '","description":"DelegateMarket lets you escrow your token for a chosen timeperiod and receive a token representing its delegation rights. This collection represents the principal i.e. the future right to claim the underlying token once the associated delegate token expires.","attributes":[{"trait_type":"Collection Address","value":"',
```

