---
search:
  boost: 0.5
---

# FTSO v1

!!! warning "Archived content"

    This is an archived page corresponding to a phased-out version of the FTSO.

    This page is only maintained for historical reasons.
    You can find the latest version in the [FTSO concept page](../ftso/index.md).

The **Flare Time Series Oracle** (FTSO) is a smart contract running on the Flare network that **provides continuous estimations for different types of data**. It does so in a **decentralized manner** (no single party is in control of the process) and **securely** (it takes a lot of effort to disrupt the process).

To achieve a secure, decentralized system, a set of **independent data providers** retrieves data from external sources, like centralized and decentralized exchanges, and supplies the data to the FTSO system. Then, this information is weighted according to each provider's **vote power**, and a **median** is calculated to produce the final estimate.

!!! note "Important"

    When FTSOs were initially designed, they supported only cryptocurrency [price pairs](glossary.md#price_pair). Now, they support all types of data. However, contract names and methods still refer to prices and price epochs, and price pairs are used in the following information to show how FTSOs work.

The following diagram shows how price pairs are submitted to and filtered by the FTSO system.

<figure markdown>
  ![FTSO summary](ftso-summary.png){ loading=lazy .allow-zoom }
  <figcaption>FTSO summary.</figcaption>
</figure>

Data providers that supply **useful information**, such as price pairs that are not removed as outliers because they are too far away from the median value, are **rewarded**, and the resulting data estimates are finally **published on-chain**.

The following information describes:

* [The FTSO workflow](#procedure-overview)
* [How results are calculated](#how-results-are-calculated)
* [Vote power](#vote-power)
* [Delegation](#delegation)

## Procedure Overview

Using price data as an example, the procedure in the following diagram runs continuously. It produces new data estimates during every **price epoch**, which is **3 minutes long**.

<figure markdown>
  ![FTSO workflow](ftso-workflow.png){ loading=lazy .allow-zoom }
  <figcaption>FTSO workflow.</figcaption>
</figure>

1. **Any user** with an account (address) on the Flare network can act as an FTSO data provider, **submit data**, and **collect rewards**.

    During each epoch, only submissions from the **100** data providers with the most **vote power** are considered.
    An account's vote power is based on its wrapped `$FLR` or `$SGB` balance and the delegations made to it (see [Vote Power](#vote-power) below).

    Submitted data must be the current price (in `$USD`) for one or more of the supported price pairs, currently: `$ADA`, `$ALGO`, `$ARB`, `$AVAX`, `$BNB`, `$BTC`, `$DOGE`, `$ETH`, `$FIL`, `$FLR`, `$LTC`, `$MATIC`, `$SOL`, `$USDC`, `$USDT`, `$XDC`, `$XLM`, and `$XRP`.
    On Songbird, replace `$FLR` with `$SGB`.

    More general data types might be added in the future.

2. FTSO data providers submit data in rounds in a **commit-and-reveal** process, so they cannot see each other's submissions until a round is over.

    This process is like submitting data in a closed envelope, and when the round is over, all envelopes are opened.

    During a **3-minute** price epoch, providers fetch the information, run their algorithms, and **submit a hash** of the data (_commit_).
    Then, during the first half of the following price epoch (**1.5 minutes**), providers submit the actual data (_reveal_).

    See technical details about the FTSO in the [Flare Dev Hub](https://dev.flare.network/ftso/overview).

3. The FTSO system calculates the **resulting median**, taking into account each provider's vote power (see [How Results are Calculated](#how-results-are-calculated) below).

    Results are **publicly available for 5 price epochs** for any app or contract to read.
    Previous epochs can always be retrieved from an archival node.

4. For each price epoch in which the submitted data is close enough to the median value, data providers and their [delegators](#delegation) are **rewarded**.

    Rewards are accumulated in **reward epochs**, which last **3.5 days** on the Flare network and **7 days** on Songbird, and you can claim them after the epoch finishes.
{ #reward-epoch }

    See [Rewards](#rewards) below.

## How Results are Calculated

The following example uses price pairs to show the filtering process that turns all submitted data into a single estimate.
See all details [in the Flare whitepaper](https://flare.network/whitepapers/).

<figure markdown>
  ![FTSO price calculation](ftso-price.png){ loading=lazy .allow-zoom }
  <figcaption>FTSO price calculation.</figcaption>
</figure>

* The contract in charge of each price pair calculates the resulting price for a price epoch using the submissions received from all data providers during that epoch.
  Price epochs are 3 minutes long.

* Each submission has a **price** and a **weight**.
  Weight is based on the data provider's [vote power](#vote-power), as explained below.

* The [weighted median](https://en.wikipedia.org/wiki/Weighted_median) of the prices is the resulting price for the price epoch.

* Submissions in the top and bottom 25% range are not [rewarded](#rewards).

## Vote Power

<figure markdown>
  ![FTSO delegation weight calculation](ftso-weight.png){ loading=lazy .allow-zoom }
  <figcaption>FTSO delegation weight calculation.</figcaption>
</figure>

* As explained above, an FTSO data provider's submissions are weighted by its **vote power**.
  A data provider's vote power is proportional to the amount of **wrapped Flare or Songbird tokens** (`$WFLR` or `$WSGB`) it holds, plus any amount [delegated to it](#delegation).

    !!! note "A data provider's influence is limited"

        A **vote-power cap** limits the influence of individual data providers to **2.5% of the total vote power** on both Flare and Songbird.

        Any vote power above this cap is ignored. If vote power exceeds the limit, consider delegating those `$WFLR` or `$WSGB` to a different data provider.

* A **snapshot** of each data provider's vote power is taken once per reward epoch, and the resulting weight is then used **throughout the next reward epoch**.
* The actual snapshot block is called the **vote-power block**, and it is **randomly chosen** from the last blocks of the previous epoch.
  On Flare, the vote-power block is randomly chosen from roughly the last 50% of the blocks, and on Songbird, it is randomly chosen from roughly the last 25%.
  The random selection only roughly corresponds to the last 50% or 25% of the _time_ because block production times are not constant.

!!! note "Reward epochs"

    The first reward epoch on **Songbird** started on Saturday, 18 September 2021 08:41:39 (GMT), 1631954499 in Unix time and repeats every 7 days.
    Therefore, **all Songbird reward epochs start on Saturday morning (GMT)**.

    The first reward epoch on **Flare** started on Thursday, 21 July 2022 19:00:05 (GMT), 1658430005 in Unix time and repeats every 3.5 days.
    Therefore, **all Flare reward epochs start on Thursday evening (GMT) and Monday morning (GMT)**.

## Delegation

If you hold `$FLR` or `$SGB` tokens, you can **delegate** them to an FTSO data provider to increase its [vote power](#vote-power) and earn a share of its [rewards](#rewards), resulting in a **mutually beneficial arrangement**.
When you delegate your vote power, you not only earn rewards but also support reliable data providers, which strengthens the stability of the FTSO and the whole ecosystem.

Before you can delegate your native `$FLR` and `$SGB` tokens, you must [wrap these tokens](../../user/wrapping-tokens.md) into **ERC-20** `$WFLR` and `$WSGB` tokens, an operation you can reverse at any time.

After you wrap your tokens, you will have the vote power that is equivalent to the wrapped token balance, and you can delegate 100% of this vote power to 1 or 2 data providers.
Delegating 100% of your vote power to reliable data providers committed to providing accurate data maximizes your rewards and enhances the stability of the ecosystem.

!!! example "The reward rate (for advanced users)"

    As you explore data providers, consider the expected reward rate each one offers.
    The reward rate describes how many tokens were earned by a data provider during a reward epoch for every 100 tokens delegated.

    The reward rate is calculated as $total\_reward / vote\_power * (100 - fee)$, where:

    * $total\_reward$: All accumulated rewards for the data provider and its delegators in the reward epoch.
    * $vote\_power$: All the data provider's `$WFLR` and all the `$WFLR` delegated to it in the vote-power block selected for the reward epoch.
    * $fee$: The amount kept by the data provider as compensation for the service it provides. The value is specified as a percentage. For example, if the data provider's fee is 21.3%, specify 21.3 to calculate the reward rate.

    Because rewards are distributed in units of `$FLR`, the reward rate is calculated in units of `$FLR`.

For the duration of the delegation, you will earn rewards that are commensurate with vote power and the performance of the chosen data providers.
Rewards accumulate, and they become claimable for each reward epoch that is finalized.

[Inflation](glossary.md#inflation) is distributed to everyone who participates in the FTSO system, which includes data providers and entities that delegate their vote power to the data providers.
Delegated tokens are **not locked**, meaning that they remain in the user's control and the delegation can be removed at any time.

Any `$WFLR` or `$WSGB` that is newly wrapped, sent, or received will automatically update your actual delegated vote power.
However, if you receive native tokens, you must [wrap them](../../user/wrapping-tokens.md) before you contribute to existing delegations.

### Immediate Delegation Revocation

Sometimes, a data provider might maliciously attack the FTSO system to skew the reported data. If this type of attack occurs, the vote power of a data provider can be revoked immediately instead of in the next reward epoch.

In this situation, an off-chain process, such as a Twitter storm, calls for users to revoke vote power from the data provider that has attacked the system.
When vote power is revoked, the revocation occurs immediately.

[Learn how to perform this operation](../../user/delegation/managing-delegations.md#revoking-vote-power) from the block explorer.

### Effects of the Vote-Power Block Snapshot on Delegations

The following table shows when new, changed, and revoked delegations take effect in relation to the vote-power block snapshot.

| Delegation Type | Before or After Vote-Power Block Snapshot | When Delegation Takes Effect     |
| --------------- | ----------------------------------------- | -------------------------------- |
| New or changed  | Before                                    | In the next reward epoch         |
|                 | After                                     | After the next reward epoch ends |
| Revoked         | N/A                                       | Immediately                      |

### Delegation Procedure

You can [delegate your tokens using the Flare Portal](../../user/delegation/managing-delegations.md), a supported wallet like [Bifrost](../../user/wallets/bifrost-wallet.md), or a [dapp](glossary.md#dapp).
Some FTSO data providers have already started providing these dapps as a convenience.
Take a look at [flaremetrics.io](https://flaremetrics.io/) and pick the one you prefer.

## Rewards

A percentage of the annual network **[inflation](glossary.md#inflation)** is reserved to reward FTSO data providers and distributed uniformly among the year's reward epochs.
The mechanism that distributes rewards to data providers consists of several bands:

* **Primary reward band**: This band rewards [50% of submitted data, weighted by vote power and centered around the median price](#how-results-are-calculated).
That is, the primary reward band fixes the rewarded vote power at 50%, which makes the width of the primary reward band in each epoch variable.
* **Secondary reward band**: This band rewards [submitted data that falls within a fixed percentage around the calculated median](https://proposals.flare.network/STP/STP_2.html#annex-a).
That is, the width of the secondary reward band is fixed, which makes the rewarded vote power in each epoch variable.

Submitted data in each reward epoch belongs to one of the following:

* Primary reward band
* Primary and secondary reward band
* Neither reward band

On Flare, reward epochs are 3.5 days.
On Songbird, reward epochs are 7 days.
In each reward epoch, rewards are distributed to providers whose submission falls within the primary or secondary reward bands.

Because the secondary reward band is wider, it rewards more data providers than the primary band. However, submissions still must be close enough to the median to be included.
If a submission falls within both bands, it receives both rewards because each reward band is independent.

The secondary reward band receives 30% of all FTSO rewards, and the primary reward band receives the remaining 70%.
As the FTSO system evolves, these reward percentages might be revised later, in accordance with an accepted proposal that requests changes to the secondary reward band.

After the band rewards are distributed, each provider can take an **optional, configurable fee**, which is set to 20% by default, and distributes the rest of the reward among **all contributors to its vote power**, i.e., itself and all its delegators, according to the delegated amounts.

If you delegated to a data provider, the amount of your rewards depends on multiple factors:

* The percentage of vote power you delegated
* The data providers to which you delegated your vote power
* The performance of those data providers
* The fee charged by those data providers
* Whether the total vote power of one or both of those data providers exceeded the vote power cap

You can claim your rewards at the end of each reward epoch.

You must claim your rewards within 90 days of their availability.
After 90 days, unclaimed rewards on Flare are [burned](glossary.md#burn), and on Songbird, they are reallocated.

### Reward-Claiming Procedure

FTSO rewards are not automatically transferred to their recipients.
Instead, the amounts are accumulated in a smart contract and must be claimed **once the reward epoch is finished**.

You can [claim your rewards using the Flare Portal](../../user/delegation/managing-rewards.md), a supported wallet like [Bifrost](../../user/wallets/bifrost-wallet.md), or a [dapp](glossary.md#dapp).
Take a look at [flaremetrics.io](https://flaremetrics.io/) and pick the one you prefer.

To save on gas costs, rewards from multiple reward epochs are claimed simultaneously when you use the Portal. However, be aware that **rewards expire after 90 days**.
Moreover, you probably want to claim soon, to redelegate the received amount and obtain compounded rewards.

It is also worth noting that:

* Rewards are paid in the network's native currency. On Flare, the native token is `$FLR`, and on Songbird, the native token is `$SGB`.
* Data providers and their delegators must claim independently.

## Related User Guides

* [Managing delegations](../../user/delegation/managing-delegations.md)
* [Managing rewards](../../user/delegation/managing-rewards.md)
* [Wrapping tokens](../../user/wrapping-tokens.md)
