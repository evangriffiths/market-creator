# Market creator

MArket creator is an autonomous service that processes worlwide news using an LLM and creates bets on prediction markets on the Gnosis chain.

## Developers

- Clone the repository:

      git clone https://github.com/valory-xyz/market-creator.git

- System requirements:

    - Python `== 3.10`
    - [Tendermint](https://docs.tendermint.com/v0.34/introduction/install.html) `==0.34.19`
    - [Poetry](https://python-poetry.org/docs/) `>=1.4.0`
    - [Docker Engine](https://docs.docker.com/engine/install/)
    - [Docker Compose](https://docs.docker.com/compose/install/)
 
- Create development environment:

      poetry run pip install "cython<3"
      poetry run pip install wheel==0.40.0
      poetry run pip install --no-build-isolation pyyaml==5.4.1
      poetry install && poetry shell
  
- Configure command line:

      autonomy init --reset --author valory --remote --ipfs --ipfs-node "/dns/registry.autonolas.tech/tcp/443/https"

- Pull packages:

      autonomy packages sync --update-packages

## Market maker runtime paramaters

Market creator has following configurable parameters for creating markets and these can be configured at both agent and service level.

- `num_markets`: Number of markets an agent to allowed to create before, default is 1
- `market_fee`: Fees for creating a market, default is 2 unit (Eth, xDAI, etc...)
- `initial_funds`: Initial funds for the market, default is 1 unit (Eth, xDAI, WxDAI, etc...)
- `market_timeout`: Time for which the market should be active after opening, default is 7 days
- `realitio_contract`: Address of the realitio oracle contract, [default](https://gnosisscan.io/address/0x79e32aE03fb27B07C89c0c568F80287C01ca2E57/)
- `realitio_oracle_proxy_contract`: Address of the realitio oracle proxy contract, [default](https://gnosisscan.io/address/0x2bf1BFb0eB6276a4F4B60044068Cb8CdEB89f79B/)
- `conditional_tokens_contract`: Address of the conditional tokens that are going to be used, [default](https://gnosisscan.io/address/0xCeAfDD6bc0bEF976fdCd1112955828E00543c0Ce/)
- `fpmm_deterministic_factory_contract`: Address of the fixed product marker maker contract, [default](https://gnosisscan.io/address/0x9083A2B699c0a4AD06F63580BDE2635d26a3eeF0)
- `collateral_tokens_contract`: Address of the collateral token to be used for market, default is [WxDAI](https://gnosisscan.io/address/0xe91d153e0b41518a2ce8dd3d7944fa863463a97d)
- `arbitrator_contract`: Address of the arbitration provider contract, default is [kleros](https://gnosisscan.io/address/0xe40DD83a262da3f56976038F1554Fe541Fa75ecd)


The market maker agent is configured to work with the gnosis chain by default, if you want to use the agent with other chains you can figure out what contracts to use from [here](https://github.com/protofire/omen-exchange/blob/a98fff28a71fa53b43e7ae069924564dd597d9ba/README.md)

## Testing a single agent locally

Run a tendermint node using

```
bash run_tm.sh
```

and in a separate terminal, run 

```
bash run_agent.sh
```

Now running an agent this way, will not create any market since the agent depends on gnosis multisig contract to execute the final transactions which actually creates the agent. So if you want to run an agent end 2 end you will require a gnosis multisig safe with 1 registered member. To do this run following in your terminal

```
aea generate-key ethereum
```

This will generate a ethereum private in your working directory in a file named `ethereum_private_key.txt`. You can add some funds to this key and create a gnosis multisig using their [app](https://app.safe.global/welcome). Once you create a multisig, update the [safe_contract_address](https://github.com/valory-xyz/market-creator/blob/0bab9ff6b41c2f024cc1f0d2aa149347fd0f47a9/packages/valory/agents/market_maker/aea-config.yaml#L149) parameter on the `aea-config.yaml` and use the `ethereum_private_key.txt` to run the agent using the script mentioned above. 

Also make sure your multisig safe account holds some amount of the tokens which you're planning on using as collateral. By default the agent uses `WxDAI` as collateral.


## Testing the service against Gnosis Mainnet

* Prepare the agent keys:
    ```
    cat > keys.json << EOF
    [
    {
        "address": "<your_agent_address>",
        "private_key": "<your_agent_private_key>"
    }
    ]
    EOF
    ```

* Prepare an .env file with the following environment variables:
    ```
    NUM_MARKETS=1
    NEWSAPI_ENDPOINT=https://newsapi.org/v2/everything
    NEWSAPI_API_KEY=<your_news_api_key>
    MARKET_FEE=1
    INITIAL_FUNDS=1
    MARKET_TIMEOUT=7
    REALITIO_CONTRACT=<realitio_oracle_contract>
    REALITIO_ORACLE_PROXY_CONTRACT=<realitio_oracle_procy_contract>
    CONDITIONAL_TOKENS_CONTRACT=<conditional_tokens_contract>
    FPMM_DETERMINISTIC_FACTORY_CONTRACT=<fpmm_deterministic_factory_contract>
    COLLATERAL_TOKENS_CONTRACT=<collateral_tokens_contract>
    ARBITRATOR_CONTRACT=<arbitrator_contract>
    MULTISEND_ADDRESS=<multisend_address>
    OPENAI_API_KEY=<your_openai_api_key>
    ETHEREUM_LEDGER_RPC=<chain_rpc_endpoint>
    ETHEREUM_LEDGER_CHAIN_ID=<chain_id>
    ALL_PARTICIPANTS='["<your_agent_address>"]'
    RESET_PAUSE_DURATION=10
    KEY_DIR=<path_to_the_folder_containing_your_keys>
    ```

* Build and run the service:
    ```
    bash run_service.sh
    ```
