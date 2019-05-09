# Build and run this app

```
yarn
rm -rf node_modules/eosjs && bash -c "cd external/eosjs && yarn" && yarn add file:external/eosjs
rm -rf dist && yarn server
```

Connect to http://localhost:8000

# Limitations

* server only supports a single client (browser tab) connecting to it
* server only supports a single user; it assumes all stored keys belong to that user
* server only supports a single hardware key; it assumes all stored keys came from that key
* `createKey` in `ClientRoot.tsx`
    * assumes it's served from localhost; other domains will break
    * hard codes the `user` field
    * hard codes the `challenge` field; this should be randomly generated by the server then checked by the server
* If `keys.json` (maintained by the server) is lost, then the keypairs become inaccessible. Webauthn doesn't provide a way to recover from this.

# Create test chain with webauthn support

This starts a minimal test chain with some upgrades activated. This guide assumes you're already familiar with creating and using test chains.

## Prerequisites

* Install eosio webauthn_wip branch
* Install eosio.cdt v1.6.x
* Use the eosio.cdt to build eosio.contracts v1.7.0 RC1

## Create a wallet, or unlock existing

Use `cleos wallet create` or `cleos wallet unlock`

## Add the default key

```
cleos wallet import --private-key 5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3
```

## Create a test chain

This will create a fresh chain in directory `my-chain`. `chain_api_plugin` and `producer_api_plugin` enable the commands which follow.

```
mkdir my-chain
cd my-chain
nodeos -d ./data --config-dir ./config --plugin eosio::chain_api_plugin --plugin eosio::producer_api_plugin --http-validate-host 0 --access-control-allow-origin "*" -p eosio -e 2>stderr &
```

## Preactivate feature

Do this before installing eosio.bios:

```
curl -X POST http://127.0.0.1:8888/v1/producer/schedule_protocol_feature_activations -d '{"protocol_features_to_activate": ["0ec7e080177b2c02b278d5088611686b49d739925a92d9bfcacd7fc6b74053bd"]}' | jq
```

## Install contracts, activate webauthn keys, and create tokens

```
cleos create account eosio eosio.token EOS6MRyAjQq8ud7hVNYcfnVPJqcVpscN5So8BhtHuGYqET5GDW5CV
cleos set contract eosio .../path/to/eosio.contracts/build/contracts/eosio.bios -p eosio
cleos set contract eosio.token .../path/to/eosio.contracts/build/contracts/eosio.token -p eosio.token
cleos push action eosio activate '["4fca8bd82bbd181e714e283f83e1b45d95ca5af40fb89ad3977b653c448f78c2"]' -p eosio
cleos push action eosio.token create '["eosio","10000000.0000 SYS"]' -p eosio.token
```

## Use app to generate 2 keys. Use them below:

```
cleos create account eosio usera PUB_WA_.....
cleos create account eosio userb PUB_WA_.....
cleos push action eosio.token issue '["usera","1000.0000 SYS",""]' -p eosio
cleos push action eosio.token issue '["userb","1000.0000 SYS",""]' -p eosio
```
