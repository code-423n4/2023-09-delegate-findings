0. 

https://github.com/delegatexyz/delegate-registry/blob/0e0ae905a6692b8f46e9b2a8518b0723396b239f/src/DelegateRegistry.sol#L34-L40

Recommended gas optimizations:
```solidity
        unchecked {
        ++  uint256 dataLength = data.length;
        ++  for (uint256 i; i < dataLength; ++i) {    
                //slither-disable-next-line calls-loop,delegatecall-loop
                (success, results[i]) = address(this).delegatecall(data[i]);    
                if (!success) revert MulticallFailed();     
            }
        }
```

1. 

https://github.com/delegatexyz/delegate-registry/blob/0e0ae905a6692b8f46e9b2a8518b0723396b239f/src/DelegateRegistry.sol#L274-L275

Recommended gas optimizations:
```solidity
        unchecked {
        ++  uint256 hashesLength = hashes.length;
        ++  for (uint256 i; i < hashesLength; ++i) {
        
								/// for loop logic
								
            }
        }
```


2. 

Existing code:
https://github.com/delegatexyz/delegate-registry/blob/0e0ae905a6692b8f46e9b2a8518b0723396b239f/src/DelegateRegistry.sol#L312

Recommendation:
```solidity
        unchecked {
        ++  for (uint256 i; i < length; ++i) {
```

and

Existing code:
https://github.com/delegatexyz/delegate-registry/blob/0e0ae905a6692b8f46e9b2a8518b0723396b239f/src/DelegateRegistry.sol#L386

Recommendation:
```solidity
        unchecked {
        ++  for (uint256 i; i < hashesLength; ++i) {
```

and

Existing code:
https://github.com/delegatexyz/delegate-registry/blob/0e0ae905a6692b8f46e9b2a8518b0723396b239f/src/DelegateRegistry.sol#L393

Recommendation:
```solidity
         ++ for (uint256 i; i < count; ++i) { {
```

and

Existing code:
https://github.com/delegatexyz/delegate-registry/blob/0e0ae905a6692b8f46e9b2a8518b0723396b239f/src/DelegateRegistry.sol#L417

Recommendation:
```solidity
         ++ for (uint256 i; i < hashesLength; ++i) {
```

and

Existing code:
https://github.com/delegatexyz/delegate-registry/blob/0e0ae905a6692b8f46e9b2a8518b0723396b239f/src/DelegateRegistry.sol#L423

Recommendation:
```solidity
         ++ for (uint256 i; i < count; ++i) {
```
