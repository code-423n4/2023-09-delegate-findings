[G-01] array length not cached
Vulnerability details
Impact
In function multicall() ,data.length is used in the for loop which is gas heavy as length is accessed
again and again
In function geDelegationsFromHashes() ,hashes.length is used in the for loop which is gas heavy as
length is accessed again and again
Proof of Concept
https://github.com/delegatexyz/delegateregistry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L30
https://github.com/delegatexyz/delegateregistry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L272
Recommended Mitigation Steps
Declare a variable and cache the length of data
```
function multicall(bytes[] calldata data) external payable override returns (bytes[] memory results) {
results = new bytes[](data.length);
uint datalength = data.length
bool success;
unchecked {
for (uint256 i = 0; i < datalength; ++i) {
//slither-disable-next-line calls-loop,delegatecall-loop
(success, results[i]) = address(this).delegatecall(data[i]);
if (!success) revert MulticallFailed();
}
}
}
```

``` 
function getDelegationsFromHashes(bytes32[] calldata hashes) external view returns (Delegation[] memory delegations_) {
delegations_ = new Delegation[](hashes.length); uint hasheslength = hashes.length;
unchecked {
for (uint256 i = 0; i < hasheslength; ++i) {
bytes32 location = Hashes.location(hashes[i]);
address from = _loadFrom(location);
if (_invalidFrom(from)) {
delegations_[i] = Delegation({type_: DelegationType.NONE, to: address(0), from: address(0), rights: "", amount: 0, contract_: address(0), tokenId: 0});
} else {
(, address to, address contract_) = _loadDelegationAddresses(location);
delegations_[i] = Delegation({
type_: Hashes.decodeType(hashes[i]),
to: to,
from: from,
rights: _loadDelegationBytes32(location, Storage.POSITIONS_RIGHTS),
amount: _loadDelegationUint(location, Storage.POSITIONS_AMOUNT),
contract_: contract_,
tokenId: _loadDelegationUint(location, Storage.POSITIONS_TOKEN_ID)
});
}
}
}
}
```
[G-02] Loop variable can be only initialized
Vulnerability details
Impact Loop variable can only be initialized and not be equivalent to 0 to save gas
Proof of Concept
https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L30 https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L272
Recommended Mitigation Steps 
``` 
function multicall(bytes[] calldata data) external payable override returns (bytes[] memory results) {
results = new bytes[](data.length);
uint datalength = data.length
bool success;
unchecked {
for (uint256 i; i < datalength; ++i) 
………… 
```
 ``` 
function getDelegationsFromHashes(bytes32[] calldata hashes) external view returns (Delegation[] memory delegations_) {
delegations_ = new Delegation[](hashes.length); uint hasheslength = hashes.length;
unchecked {
for (uint256 i ; i < hasheslength; ++i) 
………. 
```