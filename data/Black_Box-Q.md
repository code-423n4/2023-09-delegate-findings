This comment line is false: https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/libraries/RegistryHashes.sol#L92-L92

````
* @dev gives the same location hash as location(allHash(rights, from, to)) would 
````

should be replaced by

````
* @dev gives the same location hash as location(allHash(from, rights, to)) would
````
