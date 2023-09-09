Any comments for the judge to contextualize your findings:
I found integer overflow on this project.

Approach taken in evaluating the codebase:
1. Use Mythx to search for bugs.
2. Once medium or high bug found the write exploit code for it.
3. Then submit.

Architecture recommendations:
Utilize AI more.

Codebase quality analysis:
There are some floating pragmas.

Centralization risks:
When depositing avoid monopoly by capping individual limits as a percentage, but over time the limit can increase.

Mechanism review:
Most of the code files seem secure enough.  There are recurring low security issues with floating pragmas. And rarely an integer overflow.  But mostly, the issues are floating pragmas and out of bounds array access.

Systemic risks:
I do not see a risk of an overall breakdown. Just the odd integer overlow on DelegateToken.sol

### Time spent:
15 hours