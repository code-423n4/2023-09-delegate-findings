## [QA-01] Use ERC1155P instead of ERC1155 for better gas.

ERC1155 is implemented throughout the codebase. ERC1155P is a new implementation of ERC1155 that aims to reduce gas costs for people that are minting, swapping through a burn process, and buying multiple tokens.
Using ERC1155P can significantly reduce the gas costs. 
Since implementing ERC1155P throughout the whole codebase and running gas benchmarks is not feasible in terms of contest audit, this is given as a QA report.

The gas benchmarks and comparison of ERC1155P can be seen from this twitter [thread](https://twitter.com/0xjustadev/status/1620571591653605377) by `justadev`.

The repo for is ERC1155P  [here](https://github.com/0xth0mas/ERC1155P).

