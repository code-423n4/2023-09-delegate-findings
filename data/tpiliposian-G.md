## Gas Optimizations
| |Issue|Instances|
|-|:-|:-:|
| [GAS-01] | Default Value Initialization | 4 | 
| [GAS-02] | Increments can be `unchecked` | 2 | 
| [GAS-03] | Setting the `constructor` to `payable` | 4 | 
| [GAS-04] | Using fixed bytes is cheaper than using `string` | 7 | 

### [GAS-01] Default Value Initialization
Uninitialized variables are assigned with the types default value.<br>Explicitly initializing a variable with it's default value costs unnecessary gas.

*Instances (4)*:
```solidity
File: 
153:        uint256 availableAmount = 0;

163:        uint256 availableAmount = 0;

```

```solidity
File: 
15:    uint256 internal constant ID_AVAILABLE = 0;

22:    uint256 internal constant REGISTRY_HASH_POSITION = 0;

```

### [GAS-02] Increments can be `unchecked`
Increments in for loops as well as some uint256 iterators cannot realistically overflow as this would require too many iterations, so this can be `unchecked`.
		The `unchecked` keyword is new in solidity version 0.8.0, so this only applies to that version or higher, which these instances are. This saves 30-40 gas PER LOOP.

*Instances (2)*:
```solidity
File: 
187:            ++nonce.value;

```

```solidity
File: 
60:            ++balances[delegateTokenHolder];

```

### [GAS-03] Setting the `constructor` to `payable`
Saves ~13 gas per instance

*Instances (4)*:
```solidity
File: 
52:    constructor(Structs.DelegateTokenParameters memory parameters) {

```

```solidity
File: 
29:    constructor(Structs.Parameters memory parameters) Modifiers(parameters.seaport, Enums.Stage.generate) {

```

```solidity
File: 
20:    constructor(address setDelegateToken, address setMarketMetadata) {

```

```solidity
File: 
145:    constructor(address setSeaport, CreateOffererEnums.Stage firstStage) {

```

### [GAS-04] Using fixed bytes is cheaper than using `string`
Use `bytes` for arbitrary-length raw byte data and string for arbitrary-length `string` (UTF-8) data. If you can limit the length to a certain number of bytes, always use one of `bytes1` to `bytes32` because they are much cheaper. Example:
```solidity
// Before
string a;
function add(string str) {
	a = str;
}

// After
bytes32 a;
function add(bytes32 str) public {
	a = str;
}
```

*Instances (7)*:
```solidity
File: 
217:    function name() external pure returns (string memory) {

222:    function symbol() external pure returns (string memory) {

227:    function tokenURI(uint256 delegateTokenId) external view returns (string memory) {

247:    function baseURI() external view returns (string memory) {

252:    function contractURI() external view returns (string memory) {

```

```solidity
File: 
241:    function getSeaportMetadata() external pure returns (string memory, Schema[] memory) {

```

```solidity
File: 
54:    function tokenURI(uint256 id) public view override returns (string memory) {

```

