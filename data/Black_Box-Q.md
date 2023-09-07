## Issues in comments

### Line #92

This comment line is false: https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/libraries/RegistryHashes.sol#L92

````
* @dev gives the same location hash as location(allHash(rights, from, to)) would 
````

should be replaced by

````
* @dev gives the same location hash as location(allHash(from, rights, to)) would
````

### Line #140

This comment line is false: https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/libraries/RegistryHashes.sol#L140

````
* @dev gives the same location hash as location(contractHash(rights, from, to, contract_)) would
````

should be replaced by

````
* @dev gives the same location hash as location(contractHash(from, rights, to, contract_)) would
````

### Line #192

This comment line is false: https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/libraries/RegistryHashes.sol#L192

````
* @dev gives the same location hash as location(erc721Hash(rights, from, to, contract_, tokenId)) would
````

should be replaced by

````
* @dev gives the same location hash as location(erc721Hash(from, rights, to, contract_, tokenId)) would
````

### Line #242

This comment line is false: https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/libraries/RegistryHashes.sol#L242

````
* @dev gives the same location hash as location(erc20Hash(rights, from, to, contract_)) would
````

should be replaced by

````
* @dev gives the same location hash as location(erc20Hash(from, rights, to, contract_)) would
````

### Line #294

This comment line is false: https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/libraries/RegistryHashes.sol#L294

````
* @dev gives the same location hash as location(erc1155Hash(rights, from, to, contract_, tokenId)) would
````

should be replaced by

````
* @dev gives the same location hash as location(erc1155Hash(from, rights, to, contract_, tokenId)) would
````