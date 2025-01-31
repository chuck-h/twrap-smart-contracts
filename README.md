# twrap-smart-contracts
A bridge to wrap Telos EVM tokens to Telos native

Because we need a Telos Native stablecoin.
## Operating sequence example
* User1e, a tEVM user, acquires USDM, a USD stablecoin on tEVM, via DEX or for fiat
* User1e wraps the MUSD into a bridgeable equivalent WUSDM, which is an ERC20 wrap contract with additional call functions supporting bridging.
* User1e submits an EVM "bridge" call to the WUSDM contract, specifying User1n (a native account owned by the same individual) as recipient.
* User1n receives the bridged Telos native token TUSDM, just minted by the contract, into their account
* User1n buys something from a merchant and pays with TUSDM. The merchant receives TUSDM into account User2n
* User2n transfers TUSDM to the Native bridge contract with the recipient EVM account specified in the memo. In this case the recipient is an EVM account they also own, User2e.
* The bridge contract sends WUSDM to User2e, 1:1 with the TUSDM transfer in, and burns the TUSDM
* User2e unwraps the WUSDM to USDM
* User2e exchanges USDM for fiat via Meridian
* 
# Outline
## Solidity-side bridge contract
### Wrapped ERC20 functionality
This Solidity contract provides ERC20 "depositFor" and "WithdrawTo" functions to wrap and unwrap the underlying token (e.g. USDM). It supports the conventional ERC20 "transfer", "transferFrom" and "approve" functions so that the wrapped tokens can be exchanged as any ERC20 token.
### EVM-to-Native bridge transfer
In addition, this contract provides  "bridge" and "bridgeFrom" calls which place tx data (including a recipient address on the Telos native chain) in a pending queue. The matched native bridge contract responds to the queue data. When the native side completes, it calls a "cleanup" function on the Solidity contract.
### Native-to-EVM bridge transfer
The native bridge contract calls an "issue" function on the Solidity contract, minting and delivering tokens to the recipient.
## Native-side bridge contract
### EVM-to-Native bridge transfer
This Telos native contract, when triggered, reads the queue and mints native tokens at 1:1 to bridged ERC20's for delivery to the recipient account. If successful, it generates an "cleanup" call to the Solidity-side contract which burns the EVM tokens and erases the pending-queue data.
### Native-to-EVM bridge transfer
When native tokens are transferred into this contract, the memo data is validated and the contract calls an "issue" function on the Solidity bridge contract. If successful, the native contract burns the submitted tokens.
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

## ERC20 Token transfer guidelines
https://mixbytes.io/blog/defi-patterns-erc20-token-transfers-howto
