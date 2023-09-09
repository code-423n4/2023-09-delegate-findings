# GAS OPTIMIZATIONS

Structs can be packed into fewer storage slots

Each slot saved can avoid an extra Gsset (20000 gas) for the first setting of the struct.

Subsequent reads as well as writes have smaller gas savings.

### ``expiry`` can be uint96 instead of ``uint256`` :  Saves ``2000 GAS`` , ``1 SLOT``

https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/libraries/DelegateTokenLib.sol#L20-L29

A ``uint96`` can store values from ``0 to 2^96 - 1``, which is a very large range. However, it's important to note that Ethereum's ``block.timestamp`` is a ``Unix timestamp``, which represents time in seconds.
A ``uint96`` would overflow after approximately ``2,508,149,904,626,209 years`` when storing time in seconds.  This is an extremely long time frame, and it's highly unlikely that Ethereum or any blockchain system will remain unchanged for such an extended period.


```diff
FILE: 2023-09-delegate/src/libraries/DelegateTokenLib.sol

20: struct DelegateInfo {
21:        address principalHolder;
22:        IDelegateRegistry.DelegationType tokenType;
23:        address delegateHolder;
24:        uint256 amount;
25:        address tokenContract;
+ 28:        uint96 expiry;
26:        uint256 tokenId;
27:        bytes32 rights;
- 28:        uint256 expiry;
29:    }

```

### ``signerSalt``, ``expiryLength`` can be ``uint128`` instead of ``uint256`` : Saves ``4000 GAS`` , ``2 SLOT``

https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/libraries/CreateOffererLib.sol#L89-L105

In many blockchain protocols, the usage of ``salt`` often involves values within the range of ``uint32``, as they provide a sufficient numeric space for generating unique salts. Therefore, adopting a uint32 data type for the signerSalt field is a reasonable choice, as it aligns with common practices and conserves storage resources.

For the ``expiryLength`` field, which is intended to store the ``duration`` or ``length`` of an ``expiration period``, using ``uint128`` is more than adequate. This choice allows for a vast range of possible expiration lengths, accommodating a wide spectrum of use cases without incurring unnecessary storage overhead

```diff
FILE: 2023-09-delegate/src/libraries/CreateOffererLib.sol

89: struct Context {
90:        bytes32 rights;
+ 91:        uint128 signerSalt;
+ 92:        uint128 expiryLength;
- 91:        uint256 signerSalt;
- 92:        uint256 expiryLength;
93:        CreateOffererEnums.ExpiryType expiryType;
94:        CreateOffererEnums.TargetToken targetToken;
95:    }


98: struct Order {
99:        bytes32 rights;
- 100:        uint256 expiryLength;
- 101:        uint256 signerSalt;
+ 100:        uint128 expiryLength;
+ 101:        uint128 signerSalt;
102:        address tokenContract;
103:        CreateOffererEnums.ExpiryType expiryType;
104:        CreateOffererEnums.TargetToken targetToken;
105:    }

```

## [G-2] Remove or replace unused state variables

Saves a storage slot. If the variable is assigned a non-zero value, saves ``Gsset (20000 gas)``. If it's assigned a zero value, saves ``Gsreset (2900 gas)``. If the variable remains unassigned, there is no gas savings unless the variable is public, in which case the compiler-generated non-payable getter deployment cost is saved. If the state variable is overriding an interface's public function, mark the variable as constant or immutable so that it does not use a storage slot

```solidity
FILE: delegate-registry/src/DelegateRegistry.sol

18: mapping(bytes32 delegationHash => bytes32[5] delegationStorage) internal delegations;

```
https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L18

##

[G-] Multiple accesses of a mapping/array should use a local variable cache

The instances below point to the second+ access of a value inside a mapping/array, within a function. Caching a mapping’s value in a local storage or calldata variable when the value is accessed [multiple times](https://gist.github.com/IllIllI000/ec23a57daa30a8f8ca8b9681c8ccefb0), saves ~42 gas per access due to not having to recalculate the key’s keccak256 hash (Gkeccak256 - 30 gas) and that calculation’s associated stack operations. Caching an array’s struct avoids recalculating the array offsets into memory/calldata








Save gas by checking against default WETH address

external call la address check pannama check and value aa immutable ls store panni 2100 gas save pannalam 

You can save a Gcoldsload (2100 gas) in the address provider, plus the 100 gas overhead of the external call, for every receive(), by creating an immutable DEFAULT_WETH variable which will store the initial WETH address, and change the require statement to be: require(msg.ender == DEFAULT_WETH || msg.sender == <etc>).

Avoid emitting state variables 

Use assembly for loops to gas 

If/Require checks should be top of the functions 

Is this possible to use constants instead of external call

Cache loops inside the loops 

Cache the external calls inside the loops

Don't cache with local variables only used once 

Is this possible to avoid extra write 





