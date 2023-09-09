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

## [G-] State variable should be cached 

Caching of a state variable replaces each Gwarmaccess (100 gas) with a cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

### ``nonce.value``, ``receivers.targetTokenReceiver`` , ``receivers.fulfiller``  should be cached : Saves ``300 GAS``, ``3 SLOD``

https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/libraries/CreateOffererLib.sol#L185

```diff
FILE: 2023-09-delegate/src/libraries/CreateOffererLib.sol

+ uint256 value_ = nonce.value ;
- 185: if (nonce.value != contractNonce) revert CreateOffererErrors.InvalidContractNonce(nonce.value, contractNonce);
+ 185: if (value_ != contractNonce) revert CreateOffererErrors.InvalidContractNonce(value_, contractNonce);



+ address targetTokenReceiver_ = receivers.targetTokenReceiver ;
+ address fulfiller_ = receivers.fulfiller;
289: //slither-disable-start timestamp
300:        if (
            keccak256(
                abi.encode(
                    IDelegateTokenStructs.DelegateInfo({
                        tokenType: tokenType,
-                         principalHolder: decodedContext.targetToken == CreateOffererEnums.TargetToken.principal 
  ? receivers.targetTokenReceiver : receivers.fulfiller,
+                         principalHolder: decodedContext.targetToken == CreateOffererEnums.TargetToken.principal 
  ? targetTokenReceiver_  : fulfiller_ ,
-                        delegateHolder: decodedContext.targetToken == CreateOffererEnums.TargetToken.delegate ? receivers.targetTokenReceiver : receivers.fulfiller,
+                        delegateHolder: decodedContext.targetToken == CreateOffererEnums.TargetToken.delegate ? targetTokenReceiver_  : fulfiller_ ,
                        expiry: CreateOffererHelpers.calculateExpiry(decodedContext.expiryType, decodedContext.expiryLength),
                        rights: decodedContext.rights,
                        tokenContract: consideration.token,
                        tokenId: (tokenType != IDelegateRegistry.DelegationType.ERC20) ? consideration.identifier : 0,
                        amount: (tokenType != IDelegateRegistry.DelegationType.ERC721) ? consideration.amount : 0
                    })
                )
            ) != keccak256(abi.encode(IDelegateToken(delegateToken).getDelegateInfo(DelegateTokenHelpers.delegateIdNoRevert(address(this), identifier))))
        ) revert CreateOffererErrors.DelegateInfoInvariant();

```

##
// NEED TO CHECK THIS
## [G-4] Avoid unwanted state variable write can save gas

The ``burnPrincipal`` function first checks the value of the ``flag`` variable in the ``principalBurnAuthorization`` struct. If the value of the ``flag ``variable is ``BURN_NOT_AUTHORIZED``, then the function sets the value of the flag variable to ``BURN_AUTHORIZED``, burns the delegate token, and then sets the value of the flag variable back to ``BURN_NOT_AUTHORIZED``.

However, the ``flag`` variable is only used to track whether or not the principal token has been authorized to be burned. The value of the ``flag`` variable is ``not actually`` used to ``control`` whether or not the principal token can be burned.

Therefore, the unwanted variable write can be avoided by removing the second assignment to the ``flag`` variable

### 


```diff
FILE: Breadcrumbs2023-09-delegate/src/libraries/DelegateTokenStorageHelpers.sol


```
##

## [G-] If/Require checks should be top of the functions 

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas) in a function that may ultimately revert in the unhappy case.










Save gas by checking against default WETH address

external call la address check pannama check and value aa immutable ls store panni 2100 gas save pannalam 

You can save a Gcoldsload (2100 gas) in the address provider, plus the 100 gas overhead of the external call, for every receive(), by creating an immutable DEFAULT_WETH variable which will store the initial WETH address, and change the require statement to be: require(msg.ender == DEFAULT_WETH || msg.sender == <etc>).







Is this possible to use constants instead of external call


Don't cache with local variables only used once 

Is this possible to avoid extra write 