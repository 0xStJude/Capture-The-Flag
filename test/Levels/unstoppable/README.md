# Challenge #1 - Unstoppable
There's a lending pool with a million DVT tokens in balance, offering flash loans for free.

If only there was a way to attack and stop the pool from offering flash loans ...

You start with 100 DVT tokens in balance.

[See the contracts](https://github.com/nicolasgarcia214/damn-vulnerable-defi-foundry/tree/master/src/Contracts/unstoppable)
<br/>
[Complete the challenge](https://github.com/nicolasgarcia214/damn-vulnerable-defi-foundry/blob/master/test/Levels/unstoppable/Unstoppable.t.sol)

# Notes

`UnstoppableLender.sol`

- Contract using Solidity >0.8; so no underflow/overflow errors
- Contract inherit from OpenZeppelin's ReentrancyGuard contract; so reentrancy not the problem
- DVT uses ERC20 standard; so is safe
- Constructor correctly checks that DVT token is not an empty address

Attention is drawn to the `flashLoan` method
1. `require(borrowAmount>0)`; checks that non-negative/zero borrowed
2. `require(balanceBefore >= borrowAmount)`; checks pool has enough tokens to be borrowed
3. `assert(poolBalance == balanceBefore)`; checks `poolBalance` which is a state variable that tracks the deposits by users
4. `dvt.transfer(msg.sender, borrowAmount)`; performs the flashloan
5. `IRceiver(msg.sender).receiveTokens(address(dvt), borrowAmount)`; recieves the flashloan back from the user
6. `require(balanceAfter >= balanceBefore)`; checks that the flashloan has been paid back

Immediately obvious that if a user sends funds directly to the pool, rather than calling `depositTokens` then the state variable `poolBalance` will not be updated. This will cause `point 3` above to fail above and thus stopping the pool from offering a flash loan.
