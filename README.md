# Interledger test network in a box

Sets up an Interledger network on top of the [Ethereum test network in a box](https://github.com/adaburrows/ethereum-testnet-docker).

## Overview

This sets up two Interledger Connectors and two Interledger Ethereum Settlement
Engines on top of a docker-compose Ethereum test network. This has some pretty
tight coupling with the above mentioned docker-compose Ethereum test network, but
it does make layering the services easy. For instance, a Ripple or Bitcoin test
network could be set up in a separate docker-compose file and yet another
docker-compose file could have the rest of the Interledger network settling over
the other cryptocurrency network.

Since it uses shared externally managed networks, it is easy to connect it to
another docker-compose managed project in the same manner this connects with the
docker-compose Ethereum test network. It uses `ilp-test-net` and `eth-test-net`
networks which are set up with the `create_docker_network` scripts in each repo.

## Getting started

Make sure you've cloned both repos:

```
git clone git@github.com:adaburrows/ethereum-testnet-docker.git
git clone git@github.com:adaburrows/interledger-testnet-docker.git
```

Get four terminal tabs open. Two of them should be in the
`ethereum-testnet-docker` and two of them should be in
`interledger-testnet-docker`.

Let's step through getting the test networks set up:

1. Use the scripts to set up the required docker networks.
2. Bring up the Ethereum test network.
3. Use the facet to distribute some Wei to a few wallets.
4. Bring up the Interledger test network.
5. Run the script to set up the Interledger accounts.
6. Send a transaction and watch the transaction get settled.
7. Start playing around with more configuration options.

### Set up the networks

In your first `ethereum-testnet-docker` terminal run:

```
./ethereum/scripts/create_docker_network
```

In your first `interledger-testnet-docker` terminal run:

```
./interledger/scripts/create_docker_network
```

### Bring the Ethereum testnet online

In your first `ethereum-testnet-docker` terminal run:

```
docker-compose up
```

Wait for a few minutes looking at the logs until the nodes are reporting they all
have 4 peers.

### Distribute some funds

In your second `ethereum-testnet-docker` terminal, execute the following:

Let's give Xochitl and Cuauhtli 7 ETH.

```
./ethereum/scripts/drip 075DDc7E2f96e65a7835e55FB3DbfD438c4A26E0 7000000000000000000 # Xochitl
```

It will respond with a transaction number like this:

```
"0x59b4534074b6b495724cec6694856bc9c754326a4334d9964241f07234eb4b77"
```

Then run:

```
./ethereum/scripts/drip 65F5641480B861815E2C188F6097D519A8c48c98 7000000000000000000 # Cuauhtli
```

It will also return a transaction number like:

```
"0x132ef54cbdee6f8862441a9e9a419a0f0446ac7e95cf429c1c801a5db6e0008d"
```

Once those transactions are confirmed on the network and the new block replicates
out to the nodes, we can check the balances to confirm:

```
./ethereum/scripts/balance eth-xochitl
```

```
7000000000000000000
```

```
./ethereum/scripts/balance eth-cuauhtli
```

```
7000000000000000000
```

### Bring up the ILP network

In your first `interledger-testnet-docker` terminal run:

```
docker-compose up
```

That should actually come up really quickly and start displaying the logs.

### Set up the ILP accounts

In your second `interledger-testnet-docker` terminal run:

```
./interledger/scripts/create-accounts
```
That should display output like this:

```
{"id":"0bf95a1b-c5a6-468d-93df-e545133d085f","username":"xochitl","ilp_address":"test.xochitl","asset_code":"ETH","asset_scale":18,"max_packet_amount":18446744073709551615,"min_balance":null,"ilp_over_http_url":null,"ilp_over_http_incoming_token":"SECRET","ilp_over_http_outgoing_token":"SECRET","ilp_over_btp_url":null,"ilp_over_btp_incoming_token":"SECRET","ilp_over_btp_outgoing_token":"SECRET","settle_threshold":null,"settle_to":null,"routing_relation":"NonRoutingAccount","round_trip_time":500,"packets_per_minute_limit":null,"amount_per_minute_limit":null,"settlement_engine_url":null}
{"id":"aa733d7e-ca2f-413f-8bef-59b62bab8f81","username":"cuauhtli","ilp_address":"test.cuauhtli","asset_code":"ETH","asset_scale":18,"max_packet_amount":18446744073709551615,"min_balance":null,"ilp_over_http_url":null,"ilp_over_http_incoming_token":"SECRET","ilp_over_http_outgoing_token":"SECRET","ilp_over_btp_url":null,"ilp_over_btp_incoming_token":"SECRET","ilp_over_btp_outgoing_token":"SECRET","settle_threshold":null,"settle_to":null,"routing_relation":"NonRoutingAccount","round_trip_time":500,"packets_per_minute_limit":null,"amount_per_minute_limit":null,"settlement_engine_url":null}
{"id":"6ff1f9d8-3f12-4d27-8d8f-431283d31c31","username":"cuauhtli","ilp_address":"test.cuauhtli","asset_code":"ETH","asset_scale":18,"max_packet_amount":18446744073709551615,"min_balance":null,"ilp_over_http_url":"http://ilp-cuauhtli-node:7770/accounts/xochitl/ilp","ilp_over_http_incoming_token":"SECRET","ilp_over_http_outgoing_token":"SECRET","ilp_over_btp_url":null,"ilp_over_btp_incoming_token":"SECRET","ilp_over_btp_outgoing_token":"SECRET","settle_threshold":7,"settle_to":0,"routing_relation":"Peer","round_trip_time":500,"packets_per_minute_limit":null,"amount_per_minute_limit":null,"settlement_engine_url":"http://ilp-xochitl-eth:3000/"}
{"id":"b9d145d3-7cdb-464e-bd98-06d2668cb6a7","username":"xochitl","ilp_address":"test.xochitl","asset_code":"ETH","asset_scale":18,"max_packet_amount":100000,"min_balance":-150000,"ilp_over_http_url":"http://ilp-xochitl-node:7770/accounts/cuauhtli/ilp","ilp_over_http_incoming_token":"SECRET","ilp_over_http_outgoing_token":"SECRET","ilp_over_btp_url":null,"ilp_over_btp_incoming_token":"SECRET","ilp_over_btp_outgoing_token":"SECRET","settle_threshold":7,"settle_to":0,"routing_relation":"Peer","round_trip_time":500,"packets_per_minute_limit":null,"amount_per_minute_limit":null,"settlement_engine_url":"http://ilp-cuauhtli-eth:3000/"}
0
0
```

This script went through and created:

* a `xochitl` ilp account on `ilp-xochitl-node`
* a `cuauhtli` ilp account on `ilp-cuauhtli-node`
* a peering relationship for for settling between `xochitl` and `cuauhtli`
* a peering relationship for for settling between `cuauhtli` and `xochitl`

It also set the balances of the accounts equal to the balances of the reserve
Ethereum wallets. The output from these actions are the two `0`s. This will lead
to an interesting outcome about reserve accounts.

Set up some aliases, so we don't have to type nearly as much.

```
. ./interledger/scripts/aliases
```
You can verify the balances by running:

```
xochitl-cli accounts balance xochitl --auth token.xochitl
```

```
{"asset_code":"ETH","balance":7.0}
```

Examine Cuauhtli's new ILP account balance with:

```
cuauhtli-cli accounts balance cuauhtli --auth token.cuauhtli
```

```
{"asset_code":"ETH","balance":7.0}
```

### Send a transaction

In your second `interledger-testnet-docker` terminal:

Send a payment from Xochitl to Cuauhtli over ILP.

```
xochitl-cli pay xochitl --auth token.xochitl --amount 1000 --to http://ilp-cuauhtli-node:7770/accounts/cuauhtli/spsp
```

```
{"delivered_amount":1000,"destination_asset_code":"ETH","destination_asset_scale":18,"from":"test.xochitl","in_flight_amount":0,"sent_amount":1000,"source_amount":1000,"source_asset_code":"ETH","source_asset_scale":18,"to":"test.cuauhtli.09tfBVYWdMV3kgJHdTBHwPlD"}
```

Examine the new balance Xochitl has on the ILP network with:

```
xochitl-cli accounts balance xochitl --auth token.xochitl
```

```
{"asset_code":"ETH","balance":6.999999999999999}
```

Examine Cuauhtli's new ILP account balance with:

```
cuauhtli-cli accounts balance cuauhtli --auth token.cuauhtli
```

```
{"asset_code":"ETH","balance":7.000000000000001}
```

Once you see the next confirmation in the Ethereum network, you can check the
Ethereum balances in the corresponding accounts.

In your second `ethereum-testnet-docker` terminal, execute the following:

```
./ethereum/scripts/balance eth-xochitl
```

```
6999978999999999000
```

```
./ethereum/scripts/balance eth-cuauhtli
```

```
7000000000000001000
```

Notice how these numbers are "wrong": Xochitl's balance is much lower than one
would expect and it no longer matches what's in the Interledger account. This is
because every Ethereum transaction costs "gas" which is paid by the sender. This
will be mentioned again below along with ways to mitigate this.

That's the basic demo. Congratulations! You have just sent a payment over your
own ILP network. Now you can change things around to see what happens. Also,
the various script files are well documented in hopes that it makes understanding
how to configure ILP a little easier.

### Tear down the networks

In each terminal displaying docker-compose logs, hit Ctrl-C and wait for it to
stop. Then in each of those terminals run:

```
docker-compose rm
```

That's it. The configuration is ready to change and be brought back up.

### Play with the configuration

There's some interesting things going on here. The primary one is how much gas is
used for confirming transactions on the Ethereum network. In our example, the
cost is so much more than the single transaction that it's not worth settling
that little because it eats up the reserve Ethereum account. How would we change
this? We could edit the account configuration to have a much higher settlement
threshold. Also, since this is our own network we could just lower the amount of
gas required in our Ethereum network.

There's lots of other things which can be played with. For instance, we could set
up a multitude of accounts all backed by Xochitl's reserve. This, however, is
currently left as an exercise to the reader.

Here are some very important pointers on where to find the configuration options:

* Some of the configuration is based in the config files. These options are found
  in the source code for both the connector and the settlment engine:
  * [Connector](https://github.com/interledger-rs/interledger-rs/blob/master/crates/ilp-node/src/main.rs)
  * [Settlement Engine](https://github.com/interledger-rs/settlement-engines/blob/master/crates/ethereum-engine/src/main.rs)
* Some of the configuration is set on accounts and other entities through the API.
  One of the best sets of documentation for this is found in the:
  * [CLI tool](https://github.com/interledger-rs/interledger-rs/blob/master/crates/ilp-cli/src/main.rs)
  * [Open API Specification](https://github.com/interledger-rs/interledger-rs/blob/master/docs/api.yml)
  * [interledger-rs/docs/peering.md](https://github.com/interledger-rs/interledger-rs/blob/master/docs/peering.md)
