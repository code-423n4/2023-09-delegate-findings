## Summary

### Gas Optimization

no | Issue |Instances||
|-|:-|:-:|:-:|
| [G-01] | Using fixed bytes is cheaper than using string | 2 | - |
| [G-02] | State variables should be cached in stack variables rather than re-reading them from storage | 2 | - |
| [G-03] | Not using the named return variable when a function returns, wastes deployment gas | 6 | - |
| [G-04] | Can make the variable outside the loop to save gas | 2 | - |
| [G-05] | Public function not called by the contract should be declared external instead | 1 | - |
| [G-06] | Using PRIVATE rather than PUBLIC FOR Constants/Immutable, Saves Gas  | 8 | - |
| [G-07] | Before transfer of  some functions, we should check some variables for possible gas save | 2 | - |
| [G-08] | Don't initialize variables with default value | 7 | - |
| [G-09] | State varaibles only set in the Constructor should be declared Immutable | 2 | - |
| [G-10] | Using storage instead of memory for structs/arrays saves gas | 4 | - |
| [G-11] | Duplicated require()/if() checks should be refactored to a modifier or function | 1 | - |
| [G-12] | Use constants instead of type(uintx).max | 4 | - |
| [G-13] | A modifier used only once and not being inherited should be inlined to save gas | 1 | - |
| [G-14] | abi.encode() is less efficient than  abi.encodepacked() | 4 | - |
| [G-15] | Use hardcode address instead address(this) | 8 | - |
| [G-16] | Refactor event to avoid emitting empty data | 2 | - |
| [G-17] | Non efficient zero initialization | 4 | - |
| [G-18] | Shorten the array rather than copying to a new one | 2 | - |
| [G-19] | Using this to access functions results in an external call, wasting gas | 1 | - |
| [G-20] | Using assembly to check for zero can save gas | 4 | - |
| [G-21] | Use assembly to validate msg.sender | 8 | - |

## Gas Optimizations  

## [G-01] Using fixed bytes is cheaper than using string

As a rule of thumb, use bytes for arbitrary-length raw byte data and string for arbitrary-length string (UTF-8) data.
If you can limit the length to a certain number of bytes, always use one of bytes1 to bytes32 because they are much cheaper.

```solidity
file: /src/DelegateToken.sol

@audit use ' bytesX ' isteade of ' string ' , because the return string is 14 bytes ( "Delegate Token" = 14 bytes).

217    function name() external pure returns (string memory) {    // FOUND 
218        return "Delegate Token";


222    function symbol() external pure returns (string memory) {   // FOUND  
223        return "DT";        ///@audit it's 2 bytes .

```
https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L217-L218


## [G-02] State variables should be cached in stack variables rather than re-reading them from storage

The instances below point to the second+ access of a state variable within a function.
Caching of a state variable replace each Gwarmaccess (100 gas) with a much cheaper stack read.
Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.
Most of the times this if statement will be true and we will save 100 gas at a small possibility of 3 gas loss ,

```solidity
file: /src/DelegateToken.sol

///@audit ' delegateRegistry ' is state variable, it is called in one function, on lines: 174, 177, 180,185, 195,199
///@audit and also called multiple time in another next function on line: 358, 364, 366, 367, 373, 381

165        (address delegateTokenHolder, address underlyingContract) = RegistryHelpers.loadTokenHolderAndContract(delegateRegistry, registryHash);

```
https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L165


```solidity
file: /src/CreateOfferer.sol

///@audit ' delegateToken ' is state variable, its called on lines: 101, 114, 121, 125, 138, 147, 149, 162 ,
///@audit first should cache to stack after that will use. 

99            IERC721(erc721Order.info.tokenContract).setApprovalForAll(address(delegateToken), true);

```
https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L99


## [G-03] Not using the named return variable when a function returns, wastes deployment gas

When you execute a function that returns values in Solidity, the EVM still performs the necessary operations to execute and return those values. This includes the cost of allocating memory and packing the return values. If the returned values are not utilized, it can be seen as wasteful since you are incurring gas costs for operations that have no effect.

```solidity
file: /src/libraries/DelegateTokenLib.sol

/// ' return uint256(keccak256(abi.encode(caller, salt))); ' using of uint256 is west extra gas its declared one time.

113    function delegateIdNoRevert(address caller, uint256 salt) internal pure returns (uint256) {
114        return uint256(keccak256(abi.encode(caller, salt)));             // FOUND

```
https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenLib.sol#L113-L114


### Recommended code 

```solditiy 

    function delegateIdNoRevert(address caller, uint256 salt) internal pure returns (uint256) {
        return (keccak256(abi.encode(caller, salt)));        

```

 Instances:
```solidity
file: /src/libraries/DelegateTokenRegistryHelpers.sol

82    function loadAmount(address delegateRegistry, bytes32 registryHash) internal view returns (uint256) {
        unchecked {            
        return uint256(IDelegateRegistry(delegateRegistry).readSlot(bytes32(uint256(RegistryHashes.location  //FOUND  (registryHash)) + RegistryStorage.POSITIONS_AMOUNT)));
85        }


108            return uint256(IDelegateRegistry(delegateRegistry).readSlot(bytes32(uint256(RegistryHashes.location(registryHash)) + RegistryStorage.POSITIONS_TOKEN_ID))); 



136                uint256(IDelegateRegistry(delegateRegistry).readSlot(bytes32(uint256(RegistryHashes.location(registryHash)) + RegistryStorage.POSITIONS_AMOUNT))) + increaseAmount;

```
https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenRegistryHelpers.sol#L82-L85


```solidity
file: /src/libraries/DelegateTokenStorageHelpers.sol
 
127    function readApproved(mapping(uint256 delegateTokenId => uint256[3] info) storage delegateTokenInfo, uint256 delegateTokenId) internal view returns (address) {
128        return address(uint160(delegateTokenInfo[delegateTokenId][PACKED_INFO_POSITION] >> 96));


135    function readRegistryHash(mapping(uint256 delegateTokenId => uint256[3] info) storage delegateTokenInfo, uint256 delegateTokenId) internal view returns (bytes32) {
136        return bytes32(delegateTokenInfo[delegateTokenId][REGISTRY_HASH_POSITION]);

```
https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenStorageHelpers.sol#L127-L128


## [G-04] Can make the variable outside the loop to save gas

Consider making the stack variables before the loop which gonna save gas.

```solidity
file: /src/DelegateRegistry.so

276                bytes32 location = Hashes.location(hashes[i]);

277                address from = _loadFrom(location);

```
https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L276-L277


## [G-05] Public function not called by the contract should be declared external instead 

Contracts are allowed to override their parents' functions and change the visibility from external to public and can save gas by doing so.

```solidity
file: /src/PrincipalToken.sol

54    function tokenURI(uint256 id) public view override returns (string memory) {

```
https://github.com/code-423n4/2023-09-delegate/blob/main/src/PrincipalToken.sol#L54


## [G-06] Using PRIVATE rather than PUBLIC FOR Constants/Immutable, Saves Gas        

If needed, the value can be read from the verified contract source code. Savings are due to the compiler not having to create non-payable getter functions for deployment calldata, and not adding another entry to the method ID table

Recommended Example:

```solidity

address private immutable token;

```

Instances:
```solidity
file: /src/DelegateToken.sol

21    address public immutable override delegateRegistry;

24    address public immutable override principalToken;

26    address public immutable marketMetadata;

```
https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L21-L26


```solidity
file: /src/PrincipalToken.sol

12    address public immutable delegateToken;

13    address public immutable marketMetadata;

```
https://github.com/code-423n4/2023-09-delegate/blob/main/src/PrincipalToken.sol#L12-L13


```solidity 
file: /src/CreateOfferer.sol

24    address public immutable delegateToken;

25    address public immutable principalToken;

```
https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L24-L25


```solidity
file: /src/libraries/CreateOffererLib.sol

137    address public immutable seaport;

```
https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L137


## [G-07] Before transfer of  some functions, we should check some variables for possible gas save

Before transfer, we should check for amount being 0 so the function doesn't run when its not gonna do anything:

```solidity
file: /src/DelegateToken.sol
 
362        emit Transfer(delegateTokenHolder, address(0), delegateTokenId);

```
https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L362


```solidity
file: /src/libraries/DelegateTokenTransferHelpers.sol

41        IERC721(underlyingContract).transferFrom(msg.sender, address(this), underlyingTokenId);

```
https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenTransferHelpers.sol#L41


## [G-08] Don't initialize variables with default value

Uninitialized variables are assigned with the types default value. Explicitly initializing a variable with it's default value costs unnecesary gas.

```solidity
file: /src/DelegateRegistry.sol

381        uint256 count = 0;

412        uint256 count = 0;

```
https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L381


```solidity
file: /src/libraries/RegistryHashes.sol

31    uint256 internal constant DELEGATION_SLOT = 0;

```
https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/libraries/RegistryHashes.sol#L31


```solidity
file: /src/DelegateToken.sol

175        bytes32 newRegistryHash = 0;

305        bytes32 newRegistryHash = 0;

```
https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L175


```solidity
file: /src/libraries/DelegateTokenRegistryHelpers.sol

153        uint256 availableAmount = 0;

163        uint256 availableAmount = 0;

```
https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenRegistryHelpers.sol#L153


## [G-09] State varaibles only set in the Constructor should be declared Immutable

Avoids a Gsset (20000 gas) in the constructor, and replaces the first access in each transaction (Gcoldsload - 2100 gas) and each access thereafter (Gwarmacces - 100 gas) with a PUSH32 (3 gas).

```solidity
file: /src/CreateOfferer.sol

///@audit ' transientState ' is state var , its write in constructor should be declare immutabel.

26    Structs.TransientState internal transientState;

```
https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L26


```solidity
file: /src/libraries/CreateOffererLib.sol

139    CreateOffererStructs.Stage private stage;

```
https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L139


## [G-10] Using storage instead of memory for structs/arrays saves gas

When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declearing the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct.

```solidity
file: /src/DelegateRegistry.sol

415        bytes32[] memory filteredHashes = new bytes32[](hashesLength);

```
https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L415


```solidity
file: /src/CreateOfferer.sol

59        Structs.Context memory decodedContext = abi.decode(context, (Structs.Context));

94            Structs.ERC721Order memory erc721Order = transientState.erc721Order;

```
https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L59


```solidity
file: /src/libraries/CreateOffererLib.sol

349        returns (SpentItem[] memory offer, ReceivedItem[] memory consideration)

```
https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L349


## [G-11] Duplicated require()/if() checks should be refactored to a modifier or function   

to reduce code duplication and improve readability.
•  Modifiers can be used to perform additional checks on the function inputs or state before it is executed. By defining a modifier to perform a specific check, we can reuse it across multiple functions that require the same check.
• A function can also be used to perform a specific check and return a boolean value indicating whether the check has passed or failed. This can be useful when the check is more complex and cannot be performed easily in a modifier.

```solidity
file: /src/DelegateRegistry.sol

///@audit the function come duplicated on lines: 71, 91, 111, 136

52                if (rights != "") _writeDelegation(location, Storage.POSITIONS_RIGHTS, rights);

```
https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L52


## [G-12] Use constants instead of type(uintx).max

type(uint120).max or type(uint128).max, etc. it uses more gas in the distribution process and also for each transaction than constant usage.

```solidity
file: /src/libraries/DelegateTokenStorageHelpers.sol

9    uint256 internal constant MAX_EXPIRY = type(uint96).max;

```
https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenStorageHelpers.sol#L9


```solidity 
file: /src/DelegateRegistry.sol
  
213                ? type(uint256).max

215            if (!Ops.or(rights == "", amount == type(uint256).max)) {

372        uint256 cleanUpper12Bytes = type(uint256).max << 160;    

```
https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L213


## [G-13] A modifier used only once and not being inherited should be inlined to save gas

```solidity
file: /src/libraries/CreateOffererLib.sol

169    modifier onlySeaport(address caller) {
        if (caller != seaport) revert CreateOffererErrors.CallerNotSeaport(caller);
        _;
172    }

```
https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L169-L172


## [G-14] abi.encode() is less efficient than  abi.encodepacked()

In terms of efficiency, abi.encodePacked() is generally considered to be more gas-efficient than abi.encode(), because it skips the step of adding function signatures and other metadata to the encoded data. However, this comes at the cost of reduced safety, as abi.encodePacked() does not perform any type checking or padding of data.

Refference: https://github.com/ConnorBlockchain/Solidity-Encode-Gas-Comparison

```solidity
file: /src/CreateOfferer.sol

///@audit ' abi.encode(erc721Order) '

98            Helpers.validateCreateOrderHash(targetTokenReceiver, createOrderHashAsTokenId, abi.encode(erc721Order), tokenType);


120            Helpers.validateCreateOrderHash(targetTokenReceiver, createOrderHashAsTokenId, abi.encode(erc20Order), tokenType);

```
https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L98


```solidity
file: /src/libraries/CreateOffererLib.sol

302                abi.encode(

```
https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L302


```solidity
file: /src/libraries/DelegateTokenLib.sol
 
114        return uint256(keccak256(abi.encode(caller, salt)));

```
https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenLib.sol#L114


## [G-15] Use hardcode address instead address(this)
 
 Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.
References: https://book.getfoundry.sh/reference/forge-std/compute-create-address

an example :
```solidity
contract MyContract {
    address constant public CONTRACT_ADDRESS = 0x1234567890123456789012345678901234567890;
    
    function getContractAddress() public view returns (address) {
        return CONTRACT_ADDRESS;
    }
}
```

```solidity
file: /src/DelegateRegistry.sol

37                (success, results[i]) = address(this).delegatecall(data[i]);

```
https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L37


```solidity
file: /src/DelegateToken.sol

84        if (address(this) == operator) return IERC721Receiver.onERC721Received.selector;


178            newRegistryHash = RegistryHashes.erc721Hash(address(this), underlyingRights, to, underlyingTokenId, underlyingContract);


317            newRegistryHash = RegistryHashes.erc1155Hash(address(this), delegateInfo.rights, delegateInfo.delegateHolder, delegateInfo.tokenId, delegateInfo.tokenContract);

```
https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L84


```solidity
file: /src/CreateOfferer.sol

90        if (from != address(this)) revert Errors.FromNotCreateOfferer(from);

138            if (IERC20(erc20Order.info.tokenContract).allowance(address(this), address(delegateToken)) != 0) {

```
https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L90


```solidity
file: /src/libraries/CreateOffererLib.sol

352        if (minimumReceived[0].itemType != ItemType.ERC721 || minimumReceived[0].token != address(this) || minimumReceived[0].amount != 1) {

```
https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L352


```solidity
file: /src/libraries/DelegateTokenTransferHelpers.sol

52        if (IERC20(underlyingContract).allowance(msg.sender, address(this)) < underlyingAmount) {

```
https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenTransferHelpers.sol#L52

## [G-16] Refactor event to avoid emitting empty data

```solidity
file: /src/DelegateRegistry.sol

/// @audit '   ""   '

195            valid = _validateFrom(Hashes.allLocation(from, "", to), from) || _validateFrom(Hashes.contractLocation(from, "", to, contract_), from)

212            amount = (_validateFrom(Hashes.allLocation(from, "", to), from) || _validateFrom(Hashes.contractLocation(from, "", to, contract_), from))

```
https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L195


## [G-17] Non efficient zero initialization
 
Solidity does not recognize null as a value, so uint variables are initialized to zero. Setting a uint variable to zero is redundant and can waste gas.

```solidity
file: /src/DelegateRegistry.sol

275            for (uint256 i = 0; i < hashes.length; ++i) {

312            for (uint256 i = 0; i < length; ++i) {

386            for (uint256 i = 0; i < hashesLength; ++i) {

423            for (uint256 i = 0; i < count; ++i) {    

```
https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L275



## [G-18] Shorten the array rather than copying to a new one

Inline-assembly can be used to shorten the array by changing the length slot, so that the entries don't have to be copied to a new, shorter array

```solidity
file: /src/DelegateRegistry.sol

384        bytes32[] memory filteredHashes = new bytes32[](hashesLength);

415        bytes32[] memory filteredHashes = new bytes32[](hashesLength);

```
https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L384


## [G-19] Using this to access functions results in an external call, wasting gas

External calls have an overhead of 100 gas, which can be avoided by not referencing the function using this. Contracts are allowed to override their parents' functions and change the visibility from external to public, so make this change if it's required in order to call the function internally.

```solidity
file: /src/CreateOfferer.sol

79        return this.ratifyOrder.selector;

```
https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L79


## [G-20] Using assembly to check for zero can save gas

Using assembly to check for zero can save gas by allowing more direct access to the evm and reducing some of the overhead associated with high-level operations in solidity.

```solidity
file: /src/libraries/DelegateTokenLib.sol

97        if (to.code.length == 0 || IERC721Receiver(to).onERC721Received(msg.sender, from, delegateTokenId, data) == IERC721Receiver.onERC721Received.selector) return;


102        if (to.code.length == 0 || IERC721Receiver(to).onERC721Received(msg.sender, from, delegateTokenId, "") == IERC721Receiver.onERC721Received.selector) return;

```
https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenLib.sol#L97


```solidity
file: /src/libraries/DelegateTokenTransferHelpers.sol

49        if (underlyingAmount == 0) {

62        if (pullAmount == 0) revert Errors.WrongAmountForType(IDelegateRegistry.DelegationType.ERC1155, pullAmount);    

```
https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenTransferHelpers.sol#L49


## [G-21] Use assembly to validate msg.sender

We can use assembly to efficiently validate msg.sender for the didPay and uniswapV3SwapCallback functions with the least amount of opcodes necessary. Additionally, we can use xor() instead of iszero(eq()), saving 3 gas. We can also potentially save gas on the unhappy path by using scratch space to store the error selector, potentially avoiding memory expansion costs.

```solidity
file: /src/PrincipalToken.sol

28        if (msg.sender == delegateToken) return;

```
https://github.com/code-423n4/2023-09-delegate/blob/main/src/PrincipalToken.sol#L28


```solidity
file: /src/libraries/DelegateTokenStorageHelpers.sol

113        if (msg.sender == principalToken) return;

123        if (msg.sender == account || accountOperator[account][msg.sender]) return;


149        if (msg.sender == account || accountOperator[account][msg.sender] || msg.sender == readApproved(delegateTokenInfo, delegateTokenId)) return;


163            if (delegateTokenHolder == msg.sender || msg.sender == readApproved(delegateTokenInfo, delegateTokenId) || accountOperator[delegateTokenHolder][msg.sender]) {

```
https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenStorageHelpers.sol#L113


```solidity
file: /src/DelegateToken.sol

330        if (PrincipalToken(principalToken).isApprovedOrOwner(msg.sender, delegateTokenId)) {

```
https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L330


```solidity
file: /src/libraries/DelegateTokenLib.sol

92        if (IDelegateFlashloan(info.receiver).onFlashloan{value: msg.value}(msg.sender, info) == IDelegateFlashloan.onFlashloan.selector) return;

```
https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenLib.sol#L92


```solidity
file: /src/libraries/DelegateTokenTransferHelpers.sol

52        if (IERC20(underlyingContract).allowance(msg.sender, address(this)) < underlyingAmount) {

```
https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenTransferHelpers.sol#L52