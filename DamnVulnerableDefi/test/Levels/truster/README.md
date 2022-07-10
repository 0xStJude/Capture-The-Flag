# Challenge #3 - Truster
More and more lending pools are offering flash loans. In this case, a new pool has launched that is offering flash loans of DVT tokens for free.

Currently the pool has 1 million DVT tokens in balance. And you have nothing.

But don't worry, you might be able to take them all from the pool. In a single transaction.

[See the contracts](https://github.com/nicolasgarcia214/damn-vulnerable-defi-foundry/tree/master/src/Contracts/truster)
<br/>
[Complete the challenge](https://github.com/nicolasgarcia214/damn-vulnerable-defi-foundry/blob/master/test/Levels/truster/Truster.t.sol)

# Notes

Immediately notice `target.functionCall(data)` within the `flashLoan()` method on `TrustedLenderPool.sol` contract.

- there are no checks as to what data can be called from within this function
- exploit:
	1. call `flashloan` with a borrow amount of zero and have `functionCall` approve the transfer of all the tokens in the pool to the attacker address
	2. then after `flashloan` method has finished, can simply call `transfer` as we have approved our address using the `functionCall`

