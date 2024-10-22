# Wallets

Choose your wallet:

* [Bifrost Wallet](./bifrost-wallet.md)
* [Brave Wallet](./brave-wallet.md)
* [D'CENT Wallet](./dcent-wallet.md)
* [Enkrypt Wallet](./enkrypt-wallet.md)
* [Ledger Nano X and Nano S](./how-to-access-flare-network-with-a-ledger-device.md)
* [MetaMask](./how-to-access-flare-network-with-metamask.md)
* [SafePal S1 Wallet](./safepal-s1-wallet.md)
* [Trezor T](./how-to-access-flare-network-with-a-trezor-device.md)

If your wallet is not in the list, you might be able to configure it to connect to the Flare and Songbird networks by specifying the following configuration parameters in your wallet's settings:

=== "Flare"

    | Configuration Parameter | Value                                          |
    | ----------------------- | ---------------------------------------------- |
    | **Chain ID**            | 14                                             |
    | **Asset Ticker**        | `FLR`                                          |
    | **RPC endpoint**        | `https://flare-api.flare.network/ext/bc/C/rpc` |
    | **Block Explorer**      | <https://flare-explorer.flare.network>         |

=== "Songbird"

    | Configuration Parameter | Value                                             |
    | ----------------------- | ------------------------------------------------- |
    | **Chain ID**            | 19                                                |
    | **Asset Ticker**        | `SGB`                                             |
    | **RPC endpoint**        | `https://songbird-api.flare.network/ext/bc/C/rpc` |
    | **Block Explorer**      | <https://songbird-explorer.flare.network>         |

!!! warning "Caution"
    When you connect your wallet to a dapp such as the [Flare Portal](https://portal.flare.network/), always double-check that the address shown in the dapp is the one you intend.

    For some wallets, you must explicitly grant dapps access to each account on your wallet.
    This is typically done through a "Connect to Wallet" button.

    If you did not grant this access, the account you select on your wallet might not even be visible to the dapp.
    Therefore, always double-check that the address shown in the dapp is the one you intend to use.
    { #unconnected_account }
