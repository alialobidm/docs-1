---
description: How to use your Brave wallet to access the Songbird and Flare networks
---

# Brave Wallet

Brave Browser now offers a noncustodial software wallet on both Windows and macOS for Ethereum Virtual Machine (EVM) integrated chains such as Flare and Songbird.

## Getting Started

To use Brave Wallet with Flare or Songbird, ensure you have:

* Downloaded Brave Browser to your computer, version 1.42.88 or later.
* Initialized a Brave wallet or restored an existing one.
* Protected your Brave wallet with a password.
* Backed up your crypto wallet with a 12-word recovery phrase.

## Adding Flare Tokens

After your wallet is set up, you need to connect to Flare's networks, which will add each network's native token to your listed assets.

1. Open Brave browser on your computer.
2. Navigate to **Settings**, and select **Web3** from the list of options.
3. In the box on the right, click **Wallet Networks**, and then click **Add**.
4. Complete the following steps to set up the Flare network and Songbird network:

    === "Flare"

         1. In the **Search network field**, select `0xe(14) Flare Mainnet`, and verify that the displayed values match the values in this table:

            | Network Setting               | Value                                          |
            | ----------------------------- | ---------------------------------------------- |
            | **The id of chain**           | 0xe                                            |
            | **The name of chain**         | Flare Mainnet                                  |
            | **Chain's currency name**     | Flare                                          |
            | **Chain's currency symbol**   | FLR                                            |
            | **Chain's currency decimals** | 18                                             |
            | **RPC URLs**                  | <https://flare-api.flare.network/ext/bc/C/rpc> |
            | **Icon URLs**                 | _(leave blank)_                                |
            | **Block explorer URLs**       | <https://flare-explorer.flare.network>         |

         2. Click **Submit**.

    === "Songbird"

         1. In the **Search network field**, select `0x13(19) Songbird Canary-Network`, and verify that the displayed values match the values in the following table.

            !!! tip
                Make sure the RPC node URL is `https://songbird-api.flare.network/ext/bc/C/rpc`.

            | Network Setting               | Value                                             |
            | ----------------------------- | ------------------------------------------------- |
            | **The id of chain**           | 0x13                                              |
            | **The name of chain**         | Songbird Canary-Network                           |
            | **Chain's currency name**     | Songbird                                          |
            | **Chain's currency symbol**   | SGB                                               |
            | **Chain's currency decimals** | 18                                                |
            | **RPC URLs**                  | <https://songbird-api.flare.network/ext/bc/C/rpc> |
            | **Icon URLs**                 | _(leave blank)_                                   |
            | **Block explorer URLs**       | <https://songbird-explorer.flare.network>         |

         2. Click **Submit**.

5. Click **Wallet**.
6. Specify your password, and click **Unlock**.
7. On the left side of the screen beside **Balance**, click the drop-down menu and select **Flare** or **Songbird**.
   Connection to the network is complete, and your balance of native tokens on the selected network is displayed.

With the previous steps completed, you now have access to the Flare network and Songbird Network and each network's native token, but you must [complete an extra step](./../wrapping-tokens.md#automatically) for the wallet to recognize wrapped tokens, which are needed in a lot of operations.

## Wrap and Delegate

When you delegate your vote power to FTSO data providers, you not only support the Flare ecosystem but also earn monetary rewards.

Wrap and delegate your `$FLR` or `$SGB` tokens using the [Flare Portal](https://portal.flare.network/). First, [wrap your tokens](../wrapping-tokens.md), and then [delegate them](../delegation/managing-delegations.md#delegating-your-vote-power).

--8<-- "./include/wallet-warning.md"
