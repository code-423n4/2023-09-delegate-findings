DelegateTokenStorageHelpers constant variables consumes more gas due to usage of uint256 on very low numbers, example :

```
 uint256 internal constant ID_AVAILABLE = 0;
    uint256 internal constant ID_USED = 1
```

it's recommended to use uint8 for low constant number as it is costlier than uint256, because when the uint8 value is read by EVM it reads the whole slot first and performs extra masking operations, while the uint256 value is returned directly. thus any extra opcode operations mean extra gas consumption.