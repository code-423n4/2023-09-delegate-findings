# Use of delegatecall in a payable function inside a loop

**Severity**: Low

**Effected Contract:** DelegateRegistry.sol

## Summary
The payable `DelegateRegistry.multicall()` function uses delegatecall, which takes user-provided call
data, within a loop. Each delegatecall within the `for` loop will retain the msg.value of the initial transaction.

```solidity
/// @inheritdoc IDelegateRegistry
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

## Tools Used
Manual Review

## Recommendations
1. Fortunately, none of the payable functions within the DelegateRegistry use msg.value in any way, therefore this is not a current attack vector. Moving into the future, it's worth documenting the risks associated with the use of msg.value in this context in code to ensure that all devs are aware of the potential attack vector, especially as development of new versions of the DelegateRegistry eventually begins.

## References
1. [”Two Rights Might Make a Wrong,” Paradigm](https://www.paradigm.xyz/2021/08/two-rights-might-make-a-wrong)
2. [Yield V2 - Trail of Bits](https://solodit.xyz/issues/use-of-delegatecall-in-a-payable-function-inside-a-loop-trailofbits-yield-v2-pdf)
