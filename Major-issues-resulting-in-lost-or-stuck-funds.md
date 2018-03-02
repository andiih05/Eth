Ethereum has had expensive bugs, such as the following:

## The DAO
[the DAO vulnerability](https://en.wikipedia.org/wiki/The_DAO_(organization)) and Ethereum Classic replay attacks that occurred from the ensuing hard fork;

## Parity multisig library contract issue 1 [[1](https://paritytech.io/the-multi-sig-hack-a-postmortem/), [2](https://paritytech.io/security-alert/), [3](https://paritytech.io/security-update/)] 
the first bug was fixed in [this pull request](https://github.com/paritytech/parity/pull/6103/files). "On Wednesday 19th July, 2017 a bug found in the multi-signature wallet ("multi-sig") code used as part of Parity Wallet software was exploited by parties unknown... The bug was in a pair of extremely sensitive functions designed to allow the set-up of "multi-sig" wallets in the Parity Wallet software. The functions should have been protected in order that they be usable only in one specific circumstance, as the contract was being created. However, they were entirely unguarded, which allowed the attacker to reset the ownership and usage parameters of existing wallets arbitrarily." 


## Parity multisig library contract issue 2 [[4](https://paritytech.io/security-alert-2/), [5](https://paritytech.io/parity-technologies-multi-sig-wallet-issue-update/), [6](https://paritytech.io/a-postmortem-on-the-parity-multi-sig-library-self-destruct/), [7](https://paritytech.io/on-classes-of-stuck-ether-and-potential-solutions/)]  

<p>Following the fix for the <a href="https://paritytech.io/blog/security-alert-high-2.html">original multi-sig vulnerability</a> that had been exploited on 19th of July (function visibility), a new version of the Parity Wallet library contract was deployed on 20th of July. Unfortunately, that code contained another vulnerability which was undiscovered at the time - it was possible to turn the Parity Wallet library contract into a regular multi-sig wallet and become an owner of it by calling the initWallet function. It is our current understanding that this vulnerability <a href="https://github.com/paritytech/parity/issues/6995">was triggered</a> accidentally on 6th Nov 2017 02:33:47 PM +UTC and subsequently a user deleted the <a href="https://etherscan.io/address/0x863df6bfa4469f3ead0be8f9f2aae51c91a907b4">library-turned-into-wallet</a>, wiping out the library code which in turn rendered all multi-sig contracts unusable and funds frozen since their logic (any state-modifying function) was inside the library.</p>
<p>All dependent multi-sig wallets that were deployed after 20th July functionally now look as follows:</p>

    contract Wallet {
        function () payable {
        Deposit(...)
        }
    }

<p>This means that currently no funds can be moved out of the multi-sig wallets.</p>

## Cases covered in [EIP-156](https://github.com/ethereum/EIPs/issues/156)
(We should split these out and have a section for each distinct issue)

EIP-156 gives more examples such as sending to an empty address, e.g. [1](https://github.com/ethereum/EIPs/issues/156#issuecomment-2766829920) and [2](https://github.com/ethereum/EIPs/issues/156#issuecomment-307015852);

## Geth Consensus Bug 
"On 2016-11-24, a consensus bug occurred due to two implementations having different behavior in the case of state reverts. [3](https://blog.ethereum.org/2016/11/25/security-alert-11242016-consensus-bug-geth-v1-4-19-v1-5-2/). The specification was amended to clarify that empty account deletions are reverted when the state is reverted... Details: Geth was failing to revert empty account deletions when the transaction causing the deletions of empty accounts ended with an an out-of-gas exception. An additional issue was found in Parity, where the Parity client incorrectly failed to revert empty account deletions in a more limited set of contexts involving out-of-gas calls to precompiled contracts; the new Geth behavior matches Parity’s, and empty accounts will cease to be a source of concern in general in about one week once the state clearing process finishes." [The source is here.](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-161.md#addendum-2017-08-15)

## "0x" prefix enforcement (QuadrigaCX)
(this is my understanding, needs to be verified)

QuadrigaCX attempted to send funds shortly after ethereum switched over to require the "0x" prefix. Quadriga's sweeper daemon attempted to collect funds from deposit accounts and then send them through the `SafeConditionalHFTransfer` contract to their collection account.  However, the sweeper daemon did not include the "0x" prefix on addresses which resulted in malformed input send to the contract.  The contract's fallback function was invoked which had no default logic.  The contract does not include any mechanism to retrieve ETH (or tokens) sent to it.

https://github.com/bokkypoobah/BadBeef/blob/master/README.md
https://www.reddit.com/r/ethereum/comments/6ettq5/statement_on_quadrigacx_ether_contract_error/

Contract with stuck funds: https://etherscan.io/address/0x1e143b2588705dfea63a17f2032ca123df995ce0#code
$58M in stuck ETH. Presumably, more than just QuadrigaCX is affected.  

## EthereumJS Padding Bug
(this is my understanding, needs to be verified)

A bug in EthereumJS caused the public key to be incorrectly computed from the private key.  So, users created an account and the utility would generate an address.  However, it was not the address that corresponded to the private key.  

Sources are e.g. [here](https://forum.ethereum.org/discussion/3988/bug-in-ethereumjs-util) and [here](https://www.reddit.com/r/ethereum/comments/6chqyk/trying_to_recover_my_121_eth_from_2015_js_bug/).