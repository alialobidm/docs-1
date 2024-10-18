---
title: Minting
search:
  boost: 2
---

# FAssets Minting

Minting [FAssets](./index.md) is the process of wrapping underlying tokens from connected blockchains into FAssets to be used on the Flare blockchain.
Any [user](./index.md#users) can mint FAssets.

## Minting Process

This is the summary of the minting process:

<figure markdown>
  ![Minting process](fassets-minting.png){ loading=lazy .allow-zoom }
  <figcaption>Minting process.</figcaption>
</figure>

1. The minter chooses an agent from the publicly available [agent list](./index.md#agents).
    The choice is based on the minting fee or the amount of free collateral, which must be enough to back the amount to be minted.
2. The minter sends to the Asset Manager contract a collateral reservation transaction (CRT). The CRT includes:

    * The address of the chosen agent
    * The amount to mint, which must be a whole number of [lots](#lots)
    * The [collateral reservation fee (CRF)](#fees) to compensate for the locked collateral

3. The Asset Manager contract locks the agent's collateral in the amount needed to back the whole minting until the underlying payment is proved or disproved.
    The collateral reservation response is an event issued by the contract, which includes:

    * The agent's address to which the minter must send funds on the underlying chain.
    * The amount to be paid on the underlying chain, which corresponds to the amount to be minted plus the agent's fee.
    * The payment reference, which is a unique 32-byte number the minter must include as a memo in the payment on the underlying chain.
    * The last underlying block and the last underlying timestamp to pay.
        Valid payments occur either before the last block or before the last timestamp, both inclusive.

        The time to pay is measured both in the underlying chain's block numbers and block times because the underlying chain might halt for a long time.
      In this situation, the block numbers do not increment but the block timestamps do.

4. After this event is emitted, the minter must pay the full underlying amount plus the fee to the agent on the underlying chain in a certain amount of time.
5. Using the Flare Data Connector, the minter proves the payment on Flare.
6. After the payment is proved, the minter executes the minting process, which sends FAssets to the minter's account.

When minting is executed, the [minting fee](#fees) is split between the agent and the pool:

* The percentage split is set by the agent.
* The agent's share increases the free balance on the agent's underlying address.
  The free balance is the part of the balance in an agent's underlying address that the agent can withdraw.
  It is composed of minting fees, redemption fees, and self-closed FAssets.
* The pool share gets minted as FAssets and credited to the collateral pool contract.

After minting is complete, the Asset Manager creates a [redemption ticket](#redemption-tickets-and-the-redemption-queue), which includes the mint amount and the name of the agent backing the minting.

## Fees

The following fees are paid to mint FAssets:

* The **collateral reservation fee (CRF)** is paid in native tokens by the minter at the same time the [CRT](#minting-process) is made.
  The CRF is defined by governance as a percentage of the minted value, and the same fee applies to all agents.
  { #crf }

    When the minter does not pay on the underlying chain, this fee compensates the agent and the CPT holders for the time their collateral was locked while the mint processed.
    If the minter pays on the underlying chain, the CRF is [burned](glossary.md#burn).

    For underlying chains on which proving payments takes a long time, the fee might be higher than the fee on chains that quickly prove payments.

* The **minting fee** is paid by the minter with the underlying currency as a percentage of the minted amount, and each agent can declare a different fee value.
  This fee is the main source of revenue for the agent and the CPT holders.
  { #minting-fee }

    The minting fee is further divided in two shares:

    <figure markdown>
      ![Minting fees](fassets-fees.png){ loading=lazy .allow-zoom }
      <figcaption>Minting fees.</figcaption>
    </figure>

    * **Agent's share**: This share remains in the agent's underlying account but is not marked as being in use.
        The agent can use this balance freely.
    * **Pool's share**: This share is minted as FAssets and sent to the [collateral pool](./collateral.md#pool-collateral).
        The percentage of this share is defined by the agent and can be changed by the agent after a delay that provides time for minters to notice the change.
        { #pool-share }

    The [Collateral page](./collateral.md#fasset-minting-fees-and-debt) contains more information about this fee.

## Payment Failure

To finalize the minting, the minter must pay the agent on the underlying chain and prove the payment was received.
If the payment is not completed in the time frame defined by the underlying chain block and timestamp, the agent must prove nonpayment to release the locked collateral.
After nonpayment is proved, the agent's collateral that was reserved by the [CRT](#minting-process) is released, and the agent receives the [CRF](#crf).

The [agent's registration process](./index.md#agents) verifies that the agent's underlying address does not purposefully block payments and illegally collects the CRF.

!!! example "Example: Proof of Nonpayment"
    The following example shows how the nonpayment proof works.

    The [Flare Connector](../data-connector.md)'s [payment nonexistence attestation type](https://gitlab.com/flarenetwork/state-connector-protocol/-/blob/main/specs/attestations/active-types/ReferencedPaymentNonexistence.md?ref_type=heads) proves nonpayment.

    1. The minter sends a request to mint `$FBTC`.
        At the time the request is received, the last mined block on the Bitcoin chain is number 92, with timestamp 09:00 AM.

        The Asset Manager answers with the following threshold settings to complete the payment:

        * Block 100
        * Timestamp 11:00 AM

    3. Block 101 is mined with timestamp 10:59 AM.
       At this point, the payment can still happen.
    4. Block 102 is mined with timestamp 11:04 AM.
       Payment did not occur.
       After this block is finalized, nonpayment can be proved.

    5. Block 109 is mined.
       In this case, 7 blocks on the Bitcoin blockchain are enough blocks to assume finality.
    6. The agent sends a nonpayment attestation request, which includes the payment reference, the underlying amount that was expected, the last block (100), and the last timestamp (11:00).
    7. Attestation providers attest to the following facts:

        * Block 102 is finalized and has both the number and timestamp larger than required.
        * Until this block, the required payment either was not made or was not sufficient.

    Now, the mint-payment failure and the nonpayment proof can be submitted to the FAssets system.

## Edge Cases

* **Unresponsive minter**: After a successful payment, the minter might not provide the payment proof needed to complete the minting process.
    In this case, the agent can present the payment proof and execute minting at any time.
    FAssets are still transferred to the minter's account, and the agent's collateral becomes redeemable.

* **Expired proof**: Proofs provided by the Flare Data Connector are available for only 24 hours, approximately.
    If neither the minter nor the agent presents the proof of payment or nonpayment within 24 hours, the regular minting process cannot continue, and the agent's collateral could be locked indefinitely.

    In this case, the agent can still recover the collateral by buying it back with native tokens.
    The recovery is accomplished with the following procedure:

    1. Request the proof from the time when the deposit should have happened.
        The Data Connector's answer will indicate that payments proofs are no longer available for that time.
    2. Provide the amount of `$FLR` collateral equivalent to the price of the underlying assets that should have been deposited.
    3. Present the proof.

    Because a successful deposit cannot be proven, the FAssets system burns the amount of collateral in native tokens provided by the agent.
    After the burn is complete, the rest of the agent's collateral is released, both from his vault and the collateral pool.

    !!! warning
        Note that this procedure should be used only in rare cases because providing timely payment or nonpayment proofs is always more advantageous for agents.

## Duration of the Minting Process

The duration of the minting process depends mainly on the speed of the underlying chain.
The maximum duration of the process is the sum of:

* A system-defined maximum time for deposit.
    It is either a few blocks on the underlying chain or a few minutes, whichever is longer.
* The underlying chain's finalization time.
* The Data Connector proof time, which is approximately 3 - 5 minutes, independent of the underlying chain.

On fast chains like XRPL, the maximum total time is less than 10 minutes, while on Bitcoin it is approximately 1.5 hours.
For payment failures, the agent needs to wait the maximum time, as defined above, before the nonpayment proof can be retrieved.

## Minting Payment Reference

The system generates a unique payment reference at the time of the collateral reservation request.
The minter must include the payment reference in a memo field when the underlying payment transaction is made.

The payment reference ensures the payment transaction cannot be used by another entity that might claim to have made the payment on the underlying chain and receive the minted FAssets in return.
Additionally, if the payment time expires before payment is done, the agent can prove that no payment with that reference was made.

A similar payment reference for the same purposes is generated for [redemptions](./redemption.md).

## Redemption Tickets and the Redemption Queue

For every minting operation, a redemption ticket is created.
This ticket references the minted amount and the agent that is backing the minting.

The redemption tickets are ordered in a queue that determines the next agent to be [redeemed](./redemption.md) against according to the first in, first out method (FIFO).
In other words, the first redemption ticket created will be the first redemption ticket processed.
The FIFO queue impartially ensures that all agents have the opportunity to fulfill the duties of their role.

!!! example "Example: Redemption Queue"
    The following example shows how the redemption queue works.

    1. Bob mints 10 `$FXRP` with Agent 1.
    2. Alice mints 20 `$FXRP` with Agent 2.
    3. Charlie mints 5 `$FXRP` with Agent 1.

        After Bob, Alice, and Charlie have minted their FAssets, the redemption queue according to the FIFO method is:

         1. Agent 1 with 10 `$FXRP`.
         2. Agent 2 with 20 `$FXRP`.
         3. Agent 1 with 5 `$FXRP`.

    4. Dana redeems 25 `$FXRP`.
       To redeem 25 `$FXRP`:

        1. Agent 1 pays 10 `$FXRP`.
        2. Agent 2 pays 15 `$FXPR`.

        Now, the redemption queue according to the FIFO method is:

        1. Agent 2 with 5 `$FXRP`.
        2. Agent 1 with 5 `$FXRP`.

## Lots

Every minting and redemption must be made in a whole number of lots.
Lots serve the following purposes:

* They prevent underlying transaction fees from exceeding minting or redemption fees.
* They restrict large numbers of very small redemption tickets from being submitted, which would increase gas costs.

Therefore, the amount of tokens in a lot (the _lot size_) varies for each underlying chain.
For example, on the XRPL chain, a lot can be as small as 10 `$XRP` because transaction fees are low.
On the other hand, on the Bitcoin chain, lots might need to be as big as 0.25 `$BTC` or more because transactions are far more expensive.

Over time, the lot size can be updated to reflect price fluctuations of the underlying asset.
Only a governance call can update the lot size, and it can be updated only by a limited amount per day.

!!! note "Dust"

    Some processes generate a fractional number of lots:

    * On minting, part of the minting fee is minted as the FAsset fee to the collateral pool.
      This value is usually less than 1 lot.
    * When the lot size is changed, redemptions close only a whole number of lots of each redemption ticket, which leaves the remainder unredeemed.

    These amounts, known as dust, cannot be redeemed directly because redemption requires a whole number of lots.

    In such cases, the generated dust is not included in any redemption ticket.
    Instead, each agent's dust is accumulated until the dust amounts to a whole lot.
    When that happens, another redemption ticket is automatically created.

    Therefore, the dust can be recovered or destroyed in the following ways:

    * If the dust exceeds 1 lot during minting, the part that is a whole multiple of a lot is automatically added to the created redemption ticket.
    * If an agent does not mint any FAssets for a while but the lot size changes and several redemptions occur, enough dust might accumulate to more than 1 lot.

        In this case, the part that is a whole multiple of a lot can be converted to a redemption ticket by request.
        To prevent an inactive agent making FAssets less fungible, this request can be made by any address.

    * [Self-closing](./redemption.md#self-closing) can work with fractional lots, so it can be used to remove dust.
    * [Liquidation](./liquidation.md) can work with fractional lots too, so it can also be used to remove dust.

## Self-Minting

Agents can also act as minters and mint FAssets from their own vaults.
This process is called self-minting and is simpler than regular minting because neither the [CRT](#minting-process) nor the agent's fee are necessary.

When an agent self-mints FAssets:

* The agent still needs to pay the amount to mint on the underlying chain and execute the minting.
* The self-minting operation also adds a [ticket to the redemption queue](#redemption-tickets-and-the-redemption-queue), alongside tickets added by mints done by other users.
    All tickets are processed by the FIFO queue.
* Only the [pool's share of the fee](#fees) must be paid.

Because self-minting is done without a collateral reservation request, in some cases, a change between the underlying deposit and the execution, such as another collateral reservation, price change which reduces the amount of free [lots](#lots), or lot-size change, might prohibit the intended number of lots to be minted.
If one of these changes occurs, the agent can self-mint a smaller number of lots, even 0 lots, and the remainder of the deposited underlying assets is added to the free underlying balance.

Additionally, when agents create a vault, they can choose not to make it public, so the vault can only be used to self-mint.
