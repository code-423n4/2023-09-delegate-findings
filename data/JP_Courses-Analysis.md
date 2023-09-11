Another speedrun of mine, to train my speed.

Difficult codebase due to all the inline/yul assembly blocks being used all over the place. I guess it's partially for gas optimizations etc but it could potentially increase attack surface when using so much assembly throughout your codebase. Unless properly implemented I guess?

I wont be able to make it through all the in scope contracts, but I've mostly covered the registry related contracts and libraries. Only found gas optimizations, QA and some LOW, that's it. Cant deep dive due to time constraints.

I like the idea of delegating marketplace, seems pretty innovative and eliminates most/all risks by the looks of it. I think this approach has great potential and most likely will be applied widely in the future throughout web3.

I wasnt sure whether delegatecall() from multicall() function in DelegateRegistry contract could be a risky external call, needed to understand the codebase/protocol better, but not sure if reentrancy could be done, or some other attack vector related to delegatecall, however I left this for now as time is limited.

### Time spent:
10 hours