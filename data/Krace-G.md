# Use `Ops.or` instead of ||
In DelegateRegistry, there are many uses of `||`, maybe you should use `Ops.or`, it's implemented in `RegistryOps` with assembly, which could save more gas.

[checkDelegateForContract](https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L180)
```
            valid = _validateFrom(Hashes.allLocation(from, "", to), from) || _validateFrom(Hashes.contractLocation(from, "", to, contract_), from);
            if (!Ops.or(rights == "", valid)) {
                valid = _validateFrom(Hashes.allLocation(from, rights, to), from) || _validateFrom(Hashes.contractLocation(from, rights, to, contract_), from);
            }
```

[checkDelegateForERC721](https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L195)
```
            valid = _validateFrom(Hashes.allLocation(from, "", to), from) || _validateFrom(Hashes.contractLocation(from, "", to, contract_), from)
                || _validateFrom(Hashes.erc721Location(from, "", to, tokenId, contract_), from);
            if (!Ops.or(rights == "", valid)) {
                valid = _validateFrom(Hashes.allLocation(from, rights, to), from) || _validateFrom(Hashes.contractLocation(from, rights, to, contract_), from)
                    || _validateFrom(Hashes.erc721Location(from, rights, to, tokenId, contract_), from);
```

[checkDelegateForERC20](https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L212)
```
            amount = (_validateFrom(Hashes.allLocation(from, "", to), from) || _validateFrom(Hashes.contractLocation(from, "", to, contract_), from))
                ? type(uint256).max
                : _loadDelegationUint(Hashes.erc20Location(from, "", to, contract_), Storage.POSITIONS_AMOUNT);
            if (!Ops.or(rights == "", amount == type(uint256).max)) {
                uint256 rightsBalance = (_validateFrom(Hashes.allLocation(from, rights, to), from) || _validateFrom(Hashes.contractLocation(from, rights, to, contract_), from))
```

[checkDelegateForERC1155](https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L231)
```
            amount = (_validateFrom(Hashes.allLocation(from, "", to), from) || _validateFrom(Hashes.contractLocation(from, "", to, contract_), from))
                ? type(uint256).max
                : _loadDelegationUint(Hashes.erc1155Location(from, "", to, tokenId, contract_), Storage.POSITIONS_AMOUNT);
            if (!Ops.or(rights == "", amount == type(uint256).max)) {
                uint256 rightsBalance = (_validateFrom(Hashes.allLocation(from, rights, to), from) || _validateFrom(Hashes.contractLocation(from, rights, to, contract_), from))
```
