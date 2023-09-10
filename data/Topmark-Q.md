### Issue 1:
Typographical Error in L47 & L92 of MarketMetadata.sol, It should be "time period" not "timeperiod"
https://github.com/code-423n4/2023-09-delegate/blob/main/src/MarketMetadata.sol#L47
https://github.com/code-423n4/2023-09-delegate/blob/main/src/MarketMetadata.sol#L92
```solidity
46.            idstr,
47.            '","description":"DelegateMarket lets you escrow your token for a chosen timeperiod and receive a token representing the associated delegation rights. This collection represents the tokenized delegation rights.","attributes":[{"trait_type":"Collection Address","value":"',
            Strings.toHexString(tokenContract),
...
91.            string.concat(dt.name(), " #", idstr),
92.            '","description":"DelegateMarket lets you escrow your token for a chosen timeperiod and receive a token representing its delegation rights. This collection represents the principal i.e. the future right to claim the underlying token once the associated delegate token expires.","attributes":[{"trait_type":"Collection Address","value":"',
```

