!!! warning

    If you are using Windows, it is strongly recommended to use [Windows Subsystem for Linux, version 2](https://docs.microsoft.com/en-us/windows/wsl/about) (WSL 2).

## Setting up the FAssets tools

### Setting up After Testnet XRP Reset

!!! info

    This section is only for users using FAssets before the testnet XRP reset, so please [read more](https://flare.network/important-update-xrp-testnet-reset-fassets-beta-reset/).

Suppose you previously ran the FAssets agent before the XRP testnet reset. You will need to skip the whitelisting part but still need to:

* pull the latest changes from the repository by `git pull`;
* build then with `yarn && yarn build`;
* installing the MySQL database and setting it up [following the guide](#setting-up-mysql-database) later in this document;
* create a new agent using the existing management address following the guide.

### Clone and Setup the Tools Repository

!!! info

    If you set up an FAssets agent, bot or user, please use a separate directory for each role.

1. Clone the repository and enter the working directory:

    ```bash
    git clone https://github.com/flare-labs-ltd/fasset-bots.git
    cd fasset-bots
    ```

2. Switch to the `open_beta` branch:

    ```bash
    git checkout open_beta
    ```

3. Install dependencies and build the project:

    ```bash
    yarn && yarn build
    ```

    !!! info

        Fresh installation can take more than 10 minutes, depending on if you have cached dependencies before.

4. Copy the environment file from a template `.env.template` to `.env`:

    ```bash
    cp .env.template .env
    ```