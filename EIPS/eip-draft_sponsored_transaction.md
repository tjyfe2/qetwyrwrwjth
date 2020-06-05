---
eip: <to be assigned>
title: Sponsored Transactions
author: Matt Garnett (@lightclient)
discussions-to: <URL>
status: Draft
type: Standards Track
category Core
created: 2020-06-04
---

## Simple Summary
Add support for transactions which are paid for (sponsored) by an account other than the sender.

## Abstract
An optional ECDSA signature is added to transactions to authorize value transfers and gas payments.
The validity logic is amended to verify that the sponsor can afford to pay for the transaction.

## Motivation
Many projects currently rely on a contract-level concept known as **meta-transactions**, which allow
EOAs to authorize a relayer to submit a transaction on their behalf. This technique usually relies
on the ability to catch out-of-gas errors from untrusted calls to facilitate some accounting that
must be done, such as updating the EOAs meta-transaction nonce and (if applicable) paying a fee to
the relayer. Although this has been an anti-pattern for some time, there is no simple alternative to
ensure state updates are persisted when making untrusted calls. With the increasing interest in
protocol changes that remove the observability of the gas schedule, it is imperative that a simple
alternative is introduced.

## Specification
If `block.number >= FORK_BLOCK`, accept transactions of the shape:

```
{
    nonce,
    gasPrice,
    gasLimit,
    to,
    value,
    data

    // sender signature
    v, r, s

    // sponsor signature (optional)
    s_v, s_r, s_s,
}
```

The transaction validation process is also amended to accept a transaction as valid if:

1. Verify transaction invariants (intrinsic gas, block gas limit, etc.)
2. Validate sender signature
3. Validate the sponsor signature (if present)
4. Ensure the payer's balance (sponsor if the sponsor signature is present, sender otherwise) is 
greater than `value + gasPrice * gasLimit`

## Rationale

Smart contracts regularly use `CALLER (0x33)` to authenticate a call. Meta-transactions force
developers to support authentication via both `CALLER` and EIP-1271. This burden leads to more
expensive transactions and contract deployments. This EIP is a **primitive** that can be used to create
a more interoperable meta-transaction system that is resilient to changes in the gas schedule.

Here is one example of how it might be used in practice.

#### Relaying an EOA transaction

1. An EOA finds a relayer willing sponsor a transaction and be paid in `DAI`. 
2. The relayer altruistically sponsors a transaction that moves some `DAI` to a special accounting
   contract that will be used to pay the relayer.
3. The EOA negotiates a fair `DAI` gas price for an arbitrary transaction.
4. The EOA signs a acknowledgement that if a transaction with matching `tx_hash` appears on chain,
   it will transfer some amount of DAI to the relayer.
5. The relayer signs the arbitrary transaction as a sponsor and submits it on-chain. It holds onto
   the acknowledgement from the EOA that it will transfer the agreed upon `DAI`.
6. The relayer submits the acknowledgement to the accounting contract, along with a witness to the
   transaction receipt proving i) the `tx_hash` and ii) the amount of gas expended.*

*\* This could be rather expensive and should be batched with other acknowledgements.*

## Backwards Compatibility
Because the sponsor signature is a new, optional field it is fully backwards compatible.

## Test Cases
TBA

## Implementation
TBA

## Security Considerations
TBA

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
