# twrap-smart-contracts
A bridge to wrap Telos EVM tokens to Telos native

Because we need a Telos Native stablecoin.
## Operating sequence example
* User1e, a tEVM user, acquires MUSD, a USD stablecoin on tEVM, for fiat
* User1e wraps the MUSD into a bridgeable equivalent WMUSD, which is an ERC20 wrap contract with an extra call function for bridging.
* User1e submits an EVM "bridge-to" call to the WMUSD contract, specifying User1n (a native account owned by the same individual) as recipient.
* User1n receives the bridged Telos native token TMUSD, just minted by the contract, into their account
* User1n buys something from a merchant and pays with TMUSD. The merchant receives TMUSD into account User2n
* User2n transfers TMUSD to the Native bridge contract with the recipient EVM account specified in the memo. In this case the recipient is an EVM account they also own, User2e.
* The bridge contract sends WMUSD to User2e, 1:1 with the TMUSD transfer in, and burns the TMUSD
* User2e unwraps the WMUSD to MUSD
* User2e exchanges MUSD for fiat via Meridian
* 
# Outline
## Solidity-side bridge contract
This Solidity contract provides ERC20 "depositFor" and "WithdrawTo" functions to wrap and unwrap the underlying token (e.g. MUSD). It supports the conventional ERC20 "transfer", "transferFrom" and "approve" functions so that the wrapped tokens can be exchanged as any ERC20 token. In addition, this contract provides  "bridge" and "bridgeFrom" calls which place tx data (including a recipient address on the Telos native chain) in a pending queue. In the case of success, the recipient will get native tokens in 1:1 ratio to the EVM wrapped tokens bridged in, and those EVM tokens will be burned. 

## Native-side bridge contract
This Telos native contract reads the queue and mints native tokens at 1:1 to bridged ERC20's for delivery to the recipient account. If successful, it generates an EVM call to the Solidity-side contract which burns EVM tokens and erases the pending-queue data.
When native tokens are transferred into this contract,  action which burns the wrapped tokens and generates an EVM transaction to deliver original tokens to from the vault to the recipient.

## Solidity-side wrap contract (e.g. for EVM stablecoin)
This can use a standard OpenZeppelin wrap contract to convert from a recognized stablecoin (e.g. Meridian MUSD) to a wrapped version which is an extended ERC20 able to participate in the Solidity-side bridge contract (i.e. has the "deposit-to" action).


# References
## Jesse on Telos Dev Chat
https://t.me/HelloTelos/649436/685622

It’s the other direction that’s missing
The best option until the message bridge is available is to queue some request/message on the solidity/EVM side and have an off chain oracle notify the native side that it’s there
Then the native side can read the storage of the solidity/EVM contract and fulfill the request or consume the message
Still trustless, but the off chain “oracle” has to call the native contract so it can check the EVM side and do whatever the contract is programmed to do

## RNG example
https://github.com/telosnetwork/rng-oracle-bridge/blob/main/antelope/src/rng.bridge.cpp
https://github.com/telosnetwork/telos-oracle-scripts?tab=readme-ov-file

## native/evm example
https://github.com/telosnetwork/native-to-evm-transaction

## OpenZeppelin Wrap contract
https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/extensions/ERC20Wrapper.sol

## Add metadata to EVM transfers
https://github.com/stephenhodgkiss/erc20-with-onchain-data/tree/main

Maybe not applicable?
https://ethereum.stackexchange.com/questions/138779/is-there-a-way-to-attach-a-memo-to-an-erc-20-approve-transaction

## Token transfer guidelines
You probably know all this but I don't

https://mixbytes.io/blog/defi-patterns-erc20-token-transfers-howto
