




open TODO 

Dividing an integer by another integer will often result in loss of precision. When the result is multiplied by another number, the loss of precision is magnified, often to material magnitudes. (X / Z) * Y should be re-written as (X * Y) / Z

There are 8 instances of this issue:

File: contracts/markets/Market.sol

290          uint256 denominator = ((10 ** ratesPrecision) -
291              (collateralizationRate *
292                  ((10 ** ratesPrecision) + liquidationMultiplier)) /
293:             (10 ** ratesPrecision)) * (10 ** (18 - ratesPrecision));

392                  (userCollateralShare[user] *
393                      (EXCHANGE_RATE_PRECISION / FEE_PRECISION) *
394:                     collateralizationRate),

392                  (userCollateralShare[user] *
393:                     (EXCHANGE_RATE_PRECISION / FEE_PRECISION) *

417                  collateralShare *
418                      (EXCHANGE_RATE_PRECISION / FEE_PRECISION) *
419:                     collateralizationRate,

417                  collateralShare *
418:                     (EXCHANGE_RATE_PRECISION / FEE_PRECISION) *

External calls in an un-bounded for-loop may result in a DOS

Missing checks for address(0x0) when assigning values to address state variables

Draft imports may break in new minor versions

5:    import {IERC20Permit} from "@openzeppelin/contracts/token/ERC20/extensions/draft-ERC20Permit.sol";

Consider implementing two-step procedure for updating protocol addresses

Array lengths not checked
If the length of the arrays are not required to be of the same length, user operations may not be fully executed due to a mismatch in the number of items iterated over, versus the number of items provided in the second array

Signature use at deadlines should be allowed
According to EIP-2612, signatures used on exactly the deadline timestamp are supposed to be allowed. While the signature may or may not be used for the exact EIP-2612 use case (transfer approvals), for consistency's sake, all deadlines should follow this semantic. If the timestamp is an expiration rather than a deadline, consider whether it makes more sense to include the expiration timestamp as a valid timestamp, as is done for deadlines.

There are 3 instances of this issue:

File: contracts/governance/twTAP.sol

159:         if (participant.expiry < block.timestamp) {

Open TODOs
Code architecture, incentives, and error handling/reporting questions/issues should be resolved before deployment

There are 10 instances of this issue:

File: tap-token-audit/contracts/governance/twTAP.sol

233:              //    (TODO: Word better?)

Calls to _get() will revert when totalSupply() returns zero

totalSupply() being zero will result in a division by zero, causing the transaction to fail. The function should instead special-case this scenario, and avoid reverting.

There is one instance of this issue:

File: contracts/oracle/implementations/SGOracle.sol

/// @audit _get()
50           uint256 lpPrice = (SG_POOL.totalLiquidity() *
51:              uint256(UNDERLYING.latestAnswer())) / SG_POOL.totalSupply();

latestAnswer() is deprecated
Use latestRoundData() instead so that you can tell whether the answer is stale or not. The latestAnswer() function returns zero if it is unable to fetch data, which may be the case if ChainLink stops supporting this API. The API and its deprecation message no longer even appear on the ChainLink website, so it is dangerous to continue using it.


latestRoundData() gives stale data







SafeApprove deprecated. Use safeAllowance

approve status not checked 

1.Revert on Approval To Zero Address

Some tokens (e.g. OpenZeppelin) will revert if trying to approve the zero address to spend tokens (i.e. a call to approve(address(0), amt)).

Integrators may need to add special cases to handle this logic if working with such a token.

2.Low Decimals

Some tokens have low decimals (e.g. USDC has 6). Even more extreme, some tokens like Gemini USD only have 2 decimals.

This may result in larger than expected precision loss.

Array length not checked 

3.High Decimals

Some tokens have more than 18 decimals (e.g. YAM-V2 has 24).

This may trigger unexpected reverts due to overflow, posing a liveness risk to the contract.

4.Non string metadata
Some tokens (e.g. MKR) have metadata fields (name / symbol) encoded as bytes32 instead of the string prescribed by the ERC20 specification.

This may cause issues when trying to consume metadata from these tokens.

5.No Revert on Failure
Some tokens do not revert on failure, but instead return false (e.g. ZRX, EURS).

While this is technically compliant with the ERC20 standard, it goes against common solidity coding practices and may be overlooked by developers who forget to wrap their calls to transfer in a require.

6. push 0 problems when version more than 0.8.19

7. Divide by zero should be avoided 

8. latest openzepelin version 

9. All proxy contracts initialized in initialize function

10. initializer could be front run 

11) All hard coded values right ?

12. add blocklist function for NFT 

13) Is timeclock function implemented ?

14) is there any swap function . Slippage protection, deadline, Hardcoded Slippage ?, 

15. Can the 1st deposit raise a problem ?

16) The contract implement a white/blacklist ? or some kind of addresses check ? is blocklist and whitelist tokens checked

17) Solmate ERC20.sageTransferLib do not check the contract existence

18) msg.value not checked can have result in unexpected behaviour

19) Is the function refunds the extra amount paid ? when using msg.value 

Was disableInitializers() called ?

if any contract inheritance has a constructor (erc20, reentrancyGuard, Pausable…) is used : use the upgreadable version for initialize

Signature Malleability : do not use escrevover() but use the openzepplin/ECDSA.sol (The last version should be used here)

External calls in an un-bounded for-loop may result in a DOS

Take care if (receiver == caller) can have unexpected behaviour

Hash collisions are possible with abi.encodePacked (here)

Is this possible the oracle returns state data. LatestAnswer() 

divide by zero should be avoided 

Use safetranfer function instead of transfer/trasferFrom

status of  transfer/trasferFrom not checked

There is possible that chainlink stale values 


Avoid double casting
Consider refactoring the following code, as double casting may introduce unexpected truncations and/or rounding issues.

Furthermore, double type casting can make the code less readable and harder to maintain, increasing the likelihood of errors and misunderstandings during development and debugging.

There are 3 instances of this issue.

File: src/libraries/DelegateTokenStorageHelpers.sol

// @audit uint256(uint160)
39: 		        delegateTokenInfo[delegateTokenId][PACKED_INFO_POSITION] = (uint256(uint160(approved)) << 96) | expiry;

// @audit uint256(uint160)
47: 		        delegateTokenInfo[delegateTokenId][PACKED_INFO_POSITION] = (uint256(uint160(approved)) << 96) | expiry;


Is there any possibility unchecked blocks underflow ?



(, int256 totalETHXSupplyInInt, , , ) = AggregatorV3Interface(staderConfig.getETHXSupplyPORFeedProxy())
	        .latestRoundData();

latestRoundData() returns stale data. return values not checked 

https://github.com/code-423n4/2022-06-stader-findings/issues/225







