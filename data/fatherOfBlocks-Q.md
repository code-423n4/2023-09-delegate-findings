**src/CreateOfferer.sol**	
- L89 - The transferFrom function has 77 lines of code and many conditionals, this reduces the understanding of the code. It would be beneficial for the users' understanding of this contract to have auxiliary functions with names that represent them.


**src/DelegateToken.sol**
- L161/296/353 - The transferFrom function has 49 lines of code and many conditionals within, this reduces the understanding of the code. It would be beneficial for the users' understanding of this contract to have auxiliary functions with names that represent them.
The same thing happens with the create() and withdraw() functions.


**src/libraries/CreateOffererLib.sol**
- L11/12/24/25/26/30 - Multiple errors are created that are never used within the library, therefore they should be eliminated.



