## [G-01] Use hardcode address instead address(this)
Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.

```
 if (address(this) == operator) return IERC721Receiver.onERC721Received.selector;
```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L84C8-L84C89

```
 uint256 requestedDelegateId = DelegateTokenHelpers.delegateIdNoRevert(address(this), createOrderHash);
```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L259C8-L259C111

```
 delegateTokenId = IDelegateToken(delegateToken).getDelegateId(address(this),
```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L280C8-L280C85

```
 newRegistryHash = RegistryHashes.erc20Hash(address(this), delegateInfo.rights, delegateInfo.delegateHolder, delegateInfo.tokenContract);
```


https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L312C12-L312C149

```
newRegistryHash = RegistryHashes.erc721Hash(address(this), delegateInfo.rights, delegateInfo.delegateHolder, delegateInfo.tokenId, delegateInfo.tokenContract);
```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L307C13-L307C172

```
 newRegistryHash = RegistryHashes.erc1155Hash(address(this), delegateInfo.rights, delegateInfo.delegateHolder, delegateInfo.tokenId, delegateInfo.tokenContract);
```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L317C13-L317C173

```
IERC721(info.tokenContract).transferFrom(address(this), info.receiver, info.tokenId);
```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L393C13-L393C98

```
 IERC1155(info.tokenContract).safeTransferFrom(address(this), info.receiver, info.tokenId, info.amount, "");
```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L406C12-L406C120

```
SafeERC20.safeTransferFrom(IERC20(underlyingContract), msg.sender, address(this), pullAmount);
```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenTransferHelpers.sol#L58C8-L58C103

```
 IERC1155(underlyingContract).safeTransferFrom(msg.sender, address(this), underlyingTokenId, pullAmount, "");
```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenTransferHelpers.sol#L71C8-L71C117

```
if (erc1155Pulled.flag == ERC1155_PULLED && address(this) == operator) {
```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenTransferHelpers.sol#L78C9-L78C81
## [G-02] Public Functions To External
The following functions could be set external to save gas and improve code quality.
External call cost is less expensive than of public functions.

```
 function transferFrom(address from, address to, uint256 delegateTokenId) public {
```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L161C4-L161C86

## [G-03] Using unchecked blocks to save gas
Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers. When an overflow or an underflow isnâ€™t possible (as an example, when a comparison is made before the arithmetic operation), some gas can be saved by using an unchecked block.

```
 return block.timestamp + expiryLength;
```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L328C12-L328C51

## [G-04] Use functions instead of modifiers

```
  if (to.code.length == 0 || IERC721Receiver(to).onERC721Received(msg.sender, from, delegateTokenId, data) == IERC721Receiver.onERC721Received.selector) return;
```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenLib.sol#L97

```
 if (to.code.length == 0 || IERC721Receiver(to).onERC721Received(msg.sender, from, delegateTokenId, "") == IERC721Receiver.onERC721Received.selector) return;
```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenLib.sol#L102C8-L102C165

## [G-05] Expensive operation inside a for loop

```
 for (uint256 i = 0; i < count; ++i) {
                hash = filteredHashes[i];
                location = Hashes.location(hash);
                (address from, address to, address contract_) = _loadDelegationAddresses(location);
                delegations_[i] = Delegation({
                    type_: Hashes.decodeType(hash),
                    to: to,
                    from: from,
                    rights: _loadDelegationBytes32(location, Storage.POSITIONS_RIGHTS),
                    amount: _loadDelegationUint(location, Storage.POSITIONS_AMOUNT),
                    contract_: contract_,
                    tokenId: _loadDelegationUint(location, Storage.POSITIONS_TOKEN_ID)
                });
            }
```

https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L393

## [G-06] Redundant zero initialization

Solidity does not recognize null as a value, so uint variables are initialized to zero. Setting a uint variable to zero is redundant and can waste gas.

There are several places where an int is initialized to zero, which looks like:

```
 uint256 internal constant POSITIONS_FIRST_PACKED = 0;
```

https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/libraries/RegistryStorage.sol#L10C4-L10C58


```
    uint256 count = 0;
```

https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L412C5-L412C27

```
        bytes32 newRegistryHash = 0;
```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L175C1-L175C37

## [G-07] Rearranging order of state variable declarations to pack 
them will save storage slots and gas
put them in follwing order
    uint64
    bool
    address
	
```
 address delegateHolder;
        uint256 amount;
        address tokenContract;
        uint256 tokenId;
        bytes32 rights;
        uint256 expiry;
```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenLib.sol#L23C8-L28C24

```
 address delegateRegistry,
        bytes32 registryHash,
        address from,
        bytes32 newRegistryHash,
        address to,
        bytes32 underlyingRights,
        address underlyingContract,
        uint256 underlyingTokenId
```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenRegistryHelpers.sol#L175C8-L182C34

```
   address delegateRegistry,
        bytes32 registryHash,
        address from,
        bytes32 newRegistryHash,
        address to,
        uint256 underlyingAmount,
        bytes32 underlyingRights,
        address underlyingContract
```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenRegistryHelpers.sol#L195C6-L202C35

And so on...
