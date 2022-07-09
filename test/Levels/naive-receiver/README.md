# Challenge #2 - Naive receiver
There's a lending pool offering quite expensive flash loans of Ether, which has 1000 ETH in balance.

You also see that a user has deployed a contract with 10 ETH in balance, capable of interacting with the lending pool and receiving flash loans of ETH.

Drain all ETH funds from the user's contract. Doing it in a single transaction is a big plus 😉

[See the contracts](https://github.com/nicolasgarcia214/damn-vulnerable-defi-foundry/tree/master/src/Contracts/naive-receiver)
<br/>
[Complete the challenge](https://github.com/nicolasgarcia214/damn-vulnerable-defi-foundry/blob/master/test/Levels/naive-receiver/NaiveReceiver.t.sol)

# Notes

`NaiveReceiverLEnderPool.sol`

- `ReentrancyGuard` is inherited 
- `using Adress for address` (what does this do??)
- Fee is implemented as a private constant that has a getter method `fixedFee()`
- Contract allows for deposits of ETH through `receive () external payable {}` (is this safe??)

Again looking mainly at the `flashLoan` method

1. Takes two parameters: `address borrower` and `uint256 borrowAmount`
2. It is marked as `external` method (what is the implication of this??)
3. `require(balanceBefore>=borrowAmount)`; checks that there is enough ETH in pool to be flashloaned
4. `borrower.functionCallWithValue`; transfers the loan and give control to the borrower
5. `require(address(this).balance >= balanceBefore + FIXED_FEE)`; checks that flashloan has been returned and that the fee has been paid

Moving to the borrowers contract `FlashLoanReceiver.sol`, zooming into the method attached to `point 4` above. The method's name is `receiveEther`.
1. `require(msg.sender == pool)`; ensures only pool can call the function
2. `uint256 amountToBeRepaid = msg.value + fee`; breaking this down further `msg.value` is the amount loaned and the `fee` is that charged for the flashloan.
3. `require(address(this).balance >= amountToBeRepaid)`; checks that it can repay the loan
4. Calls internal function to used the loaned funds
5. `pool.sendValue(amountToBeRepaid)`; repays the loan + fees

Notice that `FlashLoanReciever` does not check that `fee` passed in is correct. Rather than calling the getter it trusts whatever is passed in as the parameter. This is the exploit.

Upon more thought, `NaiveReceiverLenderPool.sol` the method `flashLoan` does not check that `msg.sender` is actually the borrower. The exploit to drain the `borrower` is simply to call the flashloan method 10 times with the borrower address being that of the user we are trying to drain.
