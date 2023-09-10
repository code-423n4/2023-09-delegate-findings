## 1. Use do-while loop for simple operation instead of for-loop() :-
Each iteration do-while loop saves 5-10 gas.

Scenario 1 
**Before**

```solidity
for (uint256 i = 0; i < data.length; ++i) {
                //slither-disable-next-line calls-loop,delegatecall-loop
                (success, results[i]) = address(this).delegatecall(data[i]);
                if (!success) revert MulticallFailed();
            }
```

**After**
```solidity
uint i;


do{ 

                //slither-disable-next-line calls-loop,delegatecall-loop
                (success, results[i]) = address(this).delegatecall(data[i]);
                if (!success) revert MulticallFailed();
++i;}
while(i < data.length);

Look into `//@audit changed here` comment in above loop

code snippet:-
https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L35

Scenario 

**Before**
```solidity
for (uint256 i = 0; i < length; ++i) {
                tempLocation = locations[i];
                assembly {
                    tempValue := sload(tempLocation)
                }
                contents[i] = tempValue;
            }

```

**After**
```solidity

uint i;//@audit changed here
do{//@audit changed here
tempLocation = locations[i];
                assembly {
                    tempValue := sload(tempLocation)
                }
                contents[i] = tempValue;
++i;
}while( i < length);//@audit changed here
```
Look into `//@audit changed here` comment in above loop

code snippet:-
https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L312C13-L318C14


scenario 3 :-
https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L312C13-L318C14

scenario 4 ;-
https://github.com/delegatexyz/delegate-registry/blob/6d1254de793ccc40134f9bec0b7cb3d9c3632bc1/src/DelegateRegistry.sol#L417C13-L425C14


## 2. Check IF statement and memory pointer should be in top of the function  :-

In below function we can move check IF statement and memory pointer `decodedContext` variable into top of the function to avoid offset calculation and check should be made on top of the function .

**Before**
```solidity
    function verifyCreate(address delegateToken, uint256 identifier, CreateOffererStructs.Receivers storage receivers, ReceivedItem calldata consideration, bytes calldata context)
        internal
        view
    {
        IDelegateRegistry.DelegationType tokenType = RegistryHashes.decodeType(bytes32(identifier));
        if (context.length != 160) revert CreateOffererErrors.InvalidContextLength(); //@audit
        CreateOffererStructs.Context memory decodedContext = abi.decode(context, (CreateOffererStructs.Context));
        //slither-disable-start timestamp
```

Look into `//@audit` comment in above function

**After**
```solidity
function verifyCreate(address delegateToken, uint256 identifier, CreateOffererStructs.Receivers storage receivers, ReceivedItem calldata consideration, bytes calldata context)
        internal
        view
    {
        if (context.length != 160) revert CreateOffererErrors.InvalidContextLength(); //@audit changed here
        CreateOffererStructs.Context memory decodedContext = abi.decode(context, (CreateOffererStructs.Context));//@audit changed here
        IDelegateRegistry.DelegationType tokenType = RegistryHashes.decodeType(bytes32(identifier));
        //slither-disable-start timestamp
```

Look into `//@audit changed here` comment in above function ...

code snippet :-
https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L297

## 3. Gas savings can be achieved by changing the model for assigning value to the structure
By changing the pattern of assigning value to the structure, gas savings of ~130 per instance are achieved. In addition, this use will provide significant savings in distribution costs.

**Before**
```solidity
        consideration[0] = ReceivedItem({ //@audit
            itemType: maximumSpent[0].itemType,
            token: maximumSpent[0].token,    //change struct align
            identifier: maximumSpent[0].identifier,//@audit
            amount: maximumSpent[0].amount,//@audit
            recipient: payable(address(this))
        });
```
Look into`//@audit` comment in above code snippet

**After**
```solidity
        consideration[0].itemType = maximumSpent[0].itemType;//@audit changed here
        consideration[0].token = maximumSpent[0].token;//@audit changed here
        consideration[0].identifier = maximumSpent[0].identifier;//@audit changed here
        consideration[0].amount = maximumSpent[0].amount;//@audit changed here
        consideration[0].recipient = payable(address(this));//@audit changed here
```
Look into `//@audit changed here` in above code snippet 

code snippet:-
https://github.com/code-423n4/2023-09-delegate/blob/main/src/libraries/CreateOffererLib.sol#L361C1-L366C13