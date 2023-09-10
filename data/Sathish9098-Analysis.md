# Analysis - Delegate

## Overview

> Securing onchain identities by linking cold and hot wallets

 - v2 of the delegate registry
 - v2 of the delegate marketplace

### Delegate registry

The ``delegate registry`` is a standalone ``singleton`` database that aggregates onchain programmable ``access control``. Users can link ``cold wallets`` to hot wallets, or specify individual token rights to delegate to other wallets.

### Delegate marketplace

The ``delegate marketplace`` consists of three core contracts: 
 - DelegateToken  
 - PrincipalToken 
 - CreateOfferer. 
  
  Users will deposit a token, such as a bored ape ``NFT``, into smart contract escrow using the ``DelegateToken.sol::create()`` function. They will receive back two ``ERC721s``: a bored ape DelegateToken, and a bored ape PrincipalToken.
  
  
## Approach taken in evaluating the codebase

### Steps:

- Read all documents with very fast forward way
- Then read the ``Readme.md`` file understood following things 
 - Composition
 - Test Coverage only ``80%``
 - Delegate is ``upgrade`` of existing system 
 - ``ERC-20`` Token, Non ``ERC-20`` Token

 - Then clone the repo and setup test environment to make sure all written tests are working correct
- Then Jump into the codebase analysis. In first iteration just identified the important ``GAS`` and ``QA`` related findings. 
#### GAS
  1. Structs can be packed into fewer storage slots
  2. Remove or replace unused state variables
  3. State variable should be cached 

- Then ``deep dig`` to the ``documents`` thoroughly and ``understood protocols`` flow
-Deep dive into code base with line by ``line analysis``. The ``weak`` and vulnerable areas are marked with ``@audit tags``
- Then analyzing each and every ``@audit tags`` more deep. Then performed all possible ``unit`` and ``fuzzing tests`` to ensured the functions are working ``indented way``.
- Then i reported final reports as per ``C4`` Formats 
#### M/H Risks
[Insecure use of low-level ``.call`` function could allow attacker to exploit error]()

## Architecture recommendations

The current architecture of your project and believe it to be ``very good`` and ``feasible``. A ``well-designed`` and ``robust architecture`` is indeed a valuable asset, and maintaining stability and security in your system is crucial.

## Codebase quality analysis 

- Incossistant solidity versions used ( some contracts using 0.8.21 and some using 0.8.4)
- Make sure to ``add comments`` explaining the ``purpose`` and ``functionality`` of each function and variable. Include details about the ``data structures`` and how they are used.
- Ensure that error handling is ``robust`` in your contract. For example, the ``multicall`` function ``delegates calls`` and ``reverts`` the entire transaction if any ``delegatecall fails``. Consider whether a more ``graceful error handling`` approach is appropriate, such as ``returning an error code`` instead of ``reverting``
- Instead of using ``bytes32`` to represent ``delegation types``, consider using an ``enum`` to make the code more ``self-explanatory`` and reduce the ``risk of errors`` when specifying ``delegation types``
- Ensure consistent ``naming conventions`` for function ``parameters`` and ``local variables``. This helps improve code readability. 
  - Internal variable and function parameters always starts with ``_``
- Some functions ``(delegateERC20, delegateERC721, delegateERC1155)`` have ``similar logic``. Consider ``consolidating common functionality`` into ``reusable`` internal functions to ``reduce redundancy``
- While assembly can be used for ``gas optimization``, it can also make the ``code less readable`` and ``harder to maintain``. Use assembly sparingly and only ``when necessary`` for optimization
- Lack of ``input validation`` to ensure that function parameters meet expected criteria before processing. This helps prevent ``unexpected behavior``
- In functions like ``delegateERC20``, there are ``multiple storage writes``. Consolidate these ``writes`` to ``optimize gas`` usage and ``readability``. For example, you can gather ``multiple write operations`` into a ``single assembly`` block to ``reduce gas costs``
- Instead of using custom checks with ``if`` statements, consider using the ``require`` statement for ``input validation``. It provides ``more clarity`` and ``automatically reverts`` the transaction on failure
- Consider implementing ``access control`` to restrict certain functions to authorized ``users`` or ``roles``
- If certain functions do not modify ``contract state``, mark them as ``view`` or ``pure`` to indicate their immutability
- ``if (!_invalidFrom(from))`` consider ``modifiers`` for repeated checks 
- Consolidate ``mapping`` into ``structs``
- Array lengths not checked 
- Think about how contract ``upgrades`` will be managed in the ``future``. If necessary, implement a mechanism for ``upgrading`` the contract's ``logic`` while preserving its ``storage data``
- The contract relies on external contracts and libraries ``(e.g., IDelegateRegistry, RegistryHashes, RegistryStorage, RegistryOps)``. Ensure that these dependencies are secure and ``up-to-date``
- When using ``inline assembly``, ensure that you include ``appropriate error`` handling mechanisms. ``Inline assembly`` can be ``error-prone``, so it's crucial to ``handle exceptions`` or ``errors gracefully``
- Lack of documents about inline assembly 
-  Replace ``magic numbers`` (e.g., 32, 64, 92) with named constants to improve ``code readability`` and make it easier for others to understand the purpose of these values

## Centralization risks
Contracts does not include any functions with "onlyOwner" access controls, it typically means that the contract does not grant exclusive ``control`` or ``privileges`` to a ``single centralized entity`` or ``owner``.

## Systemic Risks

1. The function ``is_compatible()`` checks if the version ``0.8.21`` is compatible with the chains ``chain1``, ``chain2``, and ``chain3``. The code returns False because the ``version 0.8.21`` is not included in ``any of the chains``
2. Low level ``call`` function return value must be checked 
3. Don't use ``payable`` when function not using ``msg.value `` or ``funds``
4. In the ``multicall`` function, consider implementing a ``gas limit`` to prevent ``individual`` delegate calls from ``consuming excessive gas`` and causing the entire transaction to ``fail``. This can help protect against potential ``denial-of-service attacks``
5. When dealing with dynamic arrays like ``delegationHashes``, be mindful of ``potential gas costs``. If these ``arrays`` could ``grow indefinitely``, consider implementing ``pagination`` or limiting the ``maximum array`` size to manage gas consumption
6. Implement a ``fallback function`` (fallback or receive) if necessary to handle ``ether`` sent to the contract without a ``specific function call``. Ensure it has ``appropriate logic`` or reverts to prevent ``accidental loss`` of funds
7. The contract uses ``delegate calls`` (delegatecall) in the multicall function. Delegate calls can be ``risky`` as they execute code in the context of the ``caller``. Ensure that the called ``contract`` is ``trusted`` and ``secure``
8. Ensure that the ``external contracts`` behave as expected and handle ``unexpected cases gracefully``
9. The code lacks explicit ``reentrancy protection``. Without proper ``checks`` and ``guards``, malicious contracts or ``attackers`` could exploit reentrancy vulnerabilities and manipulate the ``contract's state`` or ``funds``
10. The ``contract's`` may be susceptible to ``front-running attacks``, particularly when handling transactions with ``variable gas prices``. ``Malicious actors`` can insert their transactions ``before`` or ``after`` users' transactions, potentially ``affecting the contract's behavior``
11. The absence of rate limiting or gas limits in certain operations can expose the contract to ``denial-of-service (DoS)`` attacks, where malicious actors flood the contract with transactions to disrupt its functionality.
12. The code may need to handle data types consistently between ``Solidity`` and ``assembly``, especially when interacting with ``Ethereum addresses`` (e.g., using address as uint160). Inconsistencies could lead to ``unexpected behavior``
13. The ``decodeType`` function relies on ``assembly`` to extract the last byte of a hash, which represents the ``delegation type``. If the encoding ``format changesm`` it may lead to incorrect ``delegation type decoding``
14. It's a good practice to place modifiers like ``nonReentrant`` before function definitions for better readability.
15. The contract interacts with external contracts (e.g., ERC721, ERC20, ERC1155) through calls like ``transferFrom`` and ``safeTransferFrom``. Ensure that these external contracts are trusted and that their behaviors are well-understood to prevent ``unexpected issues``
16. The flashloan function appears to allow ``flash loans``. this creates the ``potential exploits`` or ``vulnerabilities`` related to flash loans.

## Time Spend
10 hours 

### Time spent:
10 hours