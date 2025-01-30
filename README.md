# twrap-smart-contracts
A bridge to wrap Telos EVM tokens to Telos native
## Operating sequence example
* User1e, a tEVM user, acquires MUSD, a USD stablecoin on tEVM, for fiat
* User1e wraps the MUSD into a bridgeable equivalent WMUSD using a standard ERC20 wrap contract
* User1e submits a "deposit-to" transfer action to the bridge contract, specifying User2n (a native account) as recipient
* User2n receives the native wrapped token TMUSD into their account
* User2n buys something from a merchant and pays witl TMUSD. The merchant receives TMUSD into account User3n
* User3n submits a "redeem-to" action to the Native bridge contract with metadata specifying the recipient EVM account. In this case it is their own account, User3e.
* The bridge contract sends WMUSD to User3e, 1:1 with the TMUSD transfer in.
* User3e unwraps the WMUSD to MUSD
* User3e exchanges MUSD for fiat via Meridian
* 
# Outline
## Solidity-side bridge contract
Write a Solidity contract which accepts ERC20 "deposit-to" transfers containing metadata for recipient Telos native account. Executing such a transfer deposits the ERC20 into a vault and places the transaction in a queue. The contract also has an action to send tokens out from the vault.

## Native-side bridge contract
Write a Telos native contract which reads the queue and mints wrapped tokens at 1:1 to deposited ERC20's and delivers them to the recipient account. The contract also has a "redeem-to" action which burns the wrapped tokens and generates an EVM transaction to deliver original tokens to from the vault to the recipient.

## Solidity-side wrap contract (e.g. for EVM stablecoin)
This can use a standard OpenZeppelin wrap contract to convert from a recognized stablecoin (e.g. Meridian MUSD) to a wrapped version which is an extended ERC20 able to participate in the Solidity-side bridge contract (i.e. has the "deposit-to" action).


# References
## Jesse on Telos Dev Chat
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

