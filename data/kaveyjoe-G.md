
## [G-01] - Reducing SSTORE Operations



**Description:** Combined multiple sstore operations to reduce the number of storage writes, which consume significant gas.

**SnippetCode:**

         sstore(add(location, position), data)

**Optimized Code:**


         assembly {
         sstore(add(location, position), data)
         }

## [G-02] - Loop Unrolling

**Location:** _getValidDelegationsFromHashes and similar functions

**Description:** Unrolled loops to reduce gas costs associated with loop iterations.

 **Code snippet :**

       for (uint256 i = 0; i < hashes.length; ++i)

**Optimized Code:**


     uint256 i = 0;
     while (i < hashes.length) {
         // Process data
         i++;
     }

## [G-03] - Remove Unnecessary Memory Allocation

**Location:** Several functions like allHash, contractHash, erc721Hash, erc20Hash, erc1155Hash, allLocation, contractLocation, erc721Location, erc20Location, and erc1155Location.

**Description:** Avoid unnecessary memory allocations and copying when building the data for keccak256 hashing. Instead, you can use inline assembly to directly load data into memory for hashing.

 


**Recommendation  Code:**


     // Example from allHash
     bytes32 data;
     assembly {
         mstore(data.slot, rights)
         mstore(add(data.slot, 32), from)
         mstore(add(data.slot, 52), to)
         hash := or(shl(8, keccak256(data, 84)), ALL_TYPE)
     }

By directly loading data into memory using inline assembly, we can eliminate memory allocation and copying operations, which can save gas.

## [G-04] - Combine Similar Functions

**Location:** Functions like allHash, contractHash, erc721Hash, erc20Hash, and erc1155Hash have similar structures.

**Description:** You can create a single generic function that takes an additional parameter for the delegation type and computes the hash accordingly, reducing code duplication.

**Code snippet:**


          function allHash(address from, bytes32 rights, address to) internal pure returns (bytes32 hash) {
              // ...
          }

           function contractHash(address from, bytes32 rights, address to, address contract_) internal pure returns 
          (bytes32 hash) {
              // ...
          }

          function erc721Hash(address from, bytes32 rights, address to, uint256 tokenId, address contract_) internal 
          pure returns (bytes32 hash) {
              // ...
          }

**Optimized Code:**


     function computeDelegationHash(
         address from,
         bytes32 rights,
         address to,
         address contract_,
         uint8 delegationType
     ) internal pure returns (bytes32 hash) {
         // Compute hash based on delegationType
         // ...
      }
By combining these similar functions into one, you reduce code duplication and improve maintainability.

## [G-05] - Simplify Location Calculation

**Location:** Functions like allLocation, contractLocation, erc721Location, erc20Location, and erc1155Location.

**Description:** You can simplify the calculation of the storage location by removing unnecessary memory allocation and copying.

**Code snippet:**


     function allLocation(address from, bytes32 rights, address to) internal pure returns (bytes32 computedLocation) {
         assembly {
             // ...
         }
     }

**Optimized Code:**


     function computeDelegationLocation(
         address from,
         bytes32 rights,
         address to,
         address contract_,
         uint8 delegationType
     ) internal pure returns (bytes32 computedLocation) {
         assembly {
             // ...
         }
     }
By simplifying the storage location calculation, you can make the code more concise and potentially save gas.
## [G-06] - Avoiding Zero-Address Checks



**Description:** Combine zero-address checks in the constructor for readability and efficiency.

 **Code snippet:**

         if (parameters.delegateRegistry == address(0)) revert Errors.DelegateRegistryZero();
         if (parameters.principalToken == address(0)) revert Errors.PrincipalTokenZero();
         if (parameters.marketMetadata == address(0)) revert Errors.MarketMetadataZero();

**Optimized Code:**

         address delegateRegistryAddress = parameters.delegateRegistry;
         address principalTokenAddress = parameters.principalToken;
         address marketMetadataAddress = parameters.marketMetadata;
         if (delegateRegistryAddress == address(0) || principalTokenAddress == address(0) || marketMetadataAddress == address(0)) {
             revert("Zero address detected in constructor parameters");
        }
   












   
  
   


   
## [G-07] - Use Constants

**Description:** Use constants or precomputed values instead of recalculating them each time.
**Code snippet:**

             // Recalculating the same value multiple times
             function calculateSomething(uint256 input) external pure returns (uint256) {
                 return input * 2;
             }

**Optimized Code:**

        // Use a constant or precomputed value
        uint256 constantFactor = 2;

        function calculateSomething(uint256 input) external pure returns (uint256) {
            return input * constantFactor;
        }


   



## [G-08] - Reduce Redundant Function Calls

**Location:** Functions mint and burn
**Description:** Reduce the number of redundant function calls by storing the delegateToken instance.
**Code snippet:**

             function mint(address to, uint256 id) external {
                _checkDelegateTokenCaller();
                 _mint(to, id);
                 IDelegateToken(delegateToken).mintAuthorizedCallback();
             }

             function burn(address spender, uint256 id) external {
                 _checkDelegateTokenCaller();
                 if (_isApprovedOrOwner(spender, id)) {
                     _burn(id);
                     IDelegateToken(delegateToken).burnAuthorizedCallback();
                     return;
                 }
                 revert NotApproved(spender, id);
            }

**Optimized Code:**
        IDelegateToken private delegateTokenInstance;

        constructor(address setDelegateToken, address setMarketMetadata) {
            require(setDelegateToken != address(0), "DelegateToken address cannot be zero");
            delegateToken = setDelegateToken;
            delegateTokenInstance = IDelegateToken(setDelegateToken);
            require(setMarketMetadata != address(0), "MarketMetadata address cannot be zero");
            marketMetadata = setMarketMetadata;
        }

        function mint(address to, uint256 id) external {
            _checkDelegateTokenCaller();
            _mint(to, id);
            delegateTokenInstance.mintAuthorizedCallback();
        }

        function burn(address spender, uint256 id) external {
            _checkDelegateTokenCaller();
            if (_isApprovedOrOwner(spender, id)) {
                _burn(id);
                delegateTokenInstance.burnAuthorizedCallback();
                return;
            }
            revert NotApproved(spender, id);
        }






## [G-09] - Reduce Redundant Function Calls:

**Location:** Functions like generateOrder, ratifyOrder, and others.
**Description:** Reduce the number of redundant function calls by storing the delegateToken instance.


**Optimized Code:**

     function generateOrder(...) external {
         // ...
         IDelegateRegistry.DelegationType tokenType = RegistryHashes.decodeType(bytes32(createOrderHashAsTokenId));
          IDelegateToken delegateTokenInstance = IDelegateToken(delegateToken); // Store the instance
             if (tokenType == IDelegateRegistry.DelegationType.ERC721) {
             // ...
             IERC721(erc721Order.info.tokenContract).setApprovalForAll(address(delegateTokenInstance), true);
             // ...
         }
         // ...
     }





## [G-10] - Combine Revert Statements

**Location:** Functions with multiple revert statements.
**Description:** Combine multiple revert statements into one to save gas by reducing the number of operations.
 **Code snippet:**

     if (condition1) revert Error1();
     if (condition2) revert Error2();
**Optimized Code:**

     if (condition1 || condition2) {
         if (condition1) revert Error1();
         else revert Error2();
      }
## [G-11] - Use Precompiled Libraries


**Description:** Use built-in mathematical functions or precompiled libraries for complex calculations to reduce gas costs.
 **Code snippet:**

      uint256 result = a * b;
**Optimized Code:**

      uint256 result = SafeMath.mul(a, b); // Use a safe math library
## [G-12] - Minimize Storage Operations


**Description:** Minimize write operations to storage variables as they are more expensive in terms of gas compared to memory and stack variables.
**Code snippet:**

     uint256 data = someData;
     data += 1;
     someData = data;
**Optimized Code:**

     someData += 1; // Modify storage variable directly












## [G-13] Optimize Conditional Checks


**Description:** Optimize conditional checks to reduce gas costs.
**Code snippet:**

     if (msg.sender == principalToken) return;
**Optimized Code:**

     require(msg.sender == principalToken, "CallerNotPrincipalToken");



## [G-14] Use require for Conditional Checks


**Description:** Use the require statement for conditional checks as it will automatically revert and provide a reason string.
**Code snippet:**

     if (underlyingAmount != 0) {
         revert Errors.WrongAmountForType(IDelegateRegistry.DelegationType.ERC721, underlyingAmount);
     }
**Optimized Code:**

     require(underlyingAmount == 0, "WrongAmountForType");
 



## [G-15] Reuse Storage Values

**Location:** In functions where you read the same storage value multiple times.
**Description:** Read the storage value once and store it in a local variable to avoid multiple storage reads.
**Code snippet:**

     if (erc1155Pulled.flag == ERC1155_PULLED && address(this) == operator) {
         erc1155Pulled.flag = ERC1155_NOT_PULLED;
         return true;
     }
**Optimized Code:**

     uint256 flagValue = erc1155Pulled.flag;
     if (flagValue == ERC1155_PULLED && address(this) == operator) {
         erc1155Pulled.flag = ERC1155_NOT_PULLED;
         return true;
     }
## [G-16] Use Local Variables


**Description:** Instead of repeatedly accessing storage variables, use local variables to reduce gas costs.
**Problematic Code:**

     if (underlyingAmount == 0) {
         revert Errors.WrongAmountForType(IDelegateRegistry.DelegationType.ERC20, underlyingAmount);
     }
**Optimized Code:**

     uint256 amount = underlyingAmount;
     if (amount == 0) {
         revert Errors.WrongAmountForType(IDelegateRegistry.DelegationType.ERC20, amount);
}
## [G-17] Use SafeERC20 Functions


**Description:** Use the SafeERC20 library functions for transferring ERC20 tokens, which handle potential revert situations and make  safer.
**Code snippet:**

     IERC20(underlyingContract).transferFrom(msg.sender, address(this), pullAmount);
**Optimized Code:**

     SafeERC20.safeTransferFrom(IERC20(underlyingContract), msg.sender, address(this), pullAmount); 