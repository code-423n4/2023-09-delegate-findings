# Issues in comments

## Arguments order make the comment false

````location(allHash(rights, from, to))```` has a different result than ````location(allHash(from, rights, to))````. So the comments need to be fixed.

#### Line #92
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

#### Line #294
This comment line is false: https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/libraries/RegistryHashes.sol#L294
````
* @dev gives the same location hash as location(erc1155Hash(rights, from, to, contract_, tokenId)) would
````
should be replaced by
````
* @dev gives the same location hash as location(erc1155Hash(from, rights, to, contract_, tokenId)) would
````

## The function is not equivalent to hash data with the last byte overwritten with delegate type

Let's take a hexadecimal number ````a = 0x1234````
If ````a```` is the data hash, and ````y```` the delegate type byte, then following comments, the returned value should be ````0x123y````.
The returned value by the function is ````0x234y```` which is different.

#### Line #117
This comment line is false: https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/libraries/RegistryHashes.sol#L117
````
* @dev returned hash should be equivalent to keccak256(abi.encodePacked(rights, from, to, contract_)) with the last byte overwritten with CONTRACT_TYPE
````
should be replaced by
````
* @dev returned hash should be equivalent to keccak256(abi.encodePacked(rights, from, to, contract_)) followed by a shift left by 1 byte and writing the CONTRACT_TYPE byte to the cleaned last byte
````

### Line #167
This comment line is false: https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/libraries/RegistryHashes.sol#L167
````
* @dev returned hash should be equivalent to keccak256(abi.encodePacked(rights, from, to, contract_, tokenId)) with the last byte overwritten with ERC721_TYPE
````
should be replaced by
````
* @dev returned hash should be equivalent to keccak256(abi.encodePacked(rights, from, to, contract_, tokenId)) followed by a shift left by 1 byte and writing the ERC721_TYPE to the cleaned last byte
````

### Line 219
This comment line is false: https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/libraries/RegistryHashes.sol#L219
````
* @dev returned hash should be equivalent to keccak256(abi.encodePacked(rights, from, to, contract_)) with the last byte overwritten with ERC20_TYPE
````
should be replaced by
````
* @dev returned hash should be equivalent to keccak256(abi.encodePacked(rights, from, to, contract_)) followed by a shift left by 1 byte and writing the ERC20_TYPE to the cleaned last byte
````

### Line 269

This comment line is false: https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/libraries/RegistryHashes.sol#L269
````
* @dev returned hash should be equivalent to keccak256(abi.encodePacked(rights, from, to, contract_, tokenId)) with the last byte overwritten with ERC1155_TYPE
````
should be replaced by
````
* @dev returned hash should be equivalent to keccak256(abi.encodePacked(rights, from, to, contract_, tokenId)) followed by a shift left by 1 byte and writing the ERC1155_TYPE to the cleaned last byte
````