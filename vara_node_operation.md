# 

## **Security**

**1. Server Security**

* Use the Ubuntu account for single sign-on.
* SSH port is set to 22.
* Firewall settings allow login only through the jump server bastion host.
* Disable root account login; use sudo if necessary.
* Business port 30333 has full firewall access.
* Monitoring ports 9100 and 9090 are open for the monitoring host.

**2. Private Key Management**

* Separate management of Stash and controller private keys. [Understand stash and controller](https://wiki.polkadot.network/docs/en/learn-staking#accounts).
* Stash is stored in a hardware wallet.
* Controller mnemonic appears in the claim script. Ensure server security is maintained.

## Hardware requirements

* `OS:` Ubuntu 20.04 or later
* `CPU:` 4vCPUs @ 3.4GHz; (it coud be Intel Ice Lake, Xeon or Core series, even AMD Zen3)
* `Memory:` 16GB RAM
* `Storage:` minimum 160GB SSD storage. Should be increased as the blockchain grows.

## **Set Up**

**1、Download the latest stable release of the `gear` node from the builds repo and unpack**

```
curl https://get.gear.rs/gear-v1.0.2-x86_64-unknown-linux-gnu.tar.xz | tar xJ
```

**2、Run Vara node as service**

Copy the `gear` executable to the `/usr/local/bin` directory:

```
sudo cp gear /usr/local/bin
```

configure the systemd file:

```
vim /etc/systemd/system/vara-node.service
```

Configure and save:

```
[Unit]
Description=Vara Node
After=network.target

[Service]
Type=simple
User=ubuntu
Group=ubuntu

ExecStart=/usr/local/bin/gear \
    --chain vara  \
    --rpc-cors all \
    --name "VALIDATOR_NAME" \
    --telemetry-url "wss://telemetry.rs/submit 1" \
    --validator
Restart=always
RestartSec=3
LimitNOFILE=10000

[Install]
WantedBy=multi-user.target
```

Save & Exit.. Enable Launch at Startup and start the service.

```
sudo systemctl enable vara-node.service
sudo systemctl start vara-node.service
```

**3、Syncing the Blockchain**

follow all the instructions, the node will require time to synchronize with the blockchain. To check the service status in real time use:

```
sudo journalctl --follow -u vara-node.service
```

You can also see your running node in telemetry portal: [https://telemetry.rs](https://telemetry.rs/)

## **Create Stash and Controller accounts**

We recommend creating two accounts: Stash and Controller, for security reasons. Ensure each of them has enough funds to pay the transaction fee. Store most of the funds on the Stash accounts, which are the optimal location for saving staking funds safely and securely.

Use the prompt to generate a new seed phrase:

```
gear key generate --network vara
```

Save both seed phrases in a secure place. Skip the previous step if you intend to use your seed phrase. If you want to use your seed phrase, skip this step. You can use [Polkadot.{js}](https://polkadot.js.org/apps/#/accounts) or [Polkadot extension](https://polkadot.js.org/extension/)
Get session keys

You need to tell the chain your Session keys. If you are on a remote server, it is easier to run this command on the same machine (while the node is running with the default HTTP RPC port configured):

```
curl -H "Content-Type: application/json" -d '{"id":1, "jsonrpc":"2.0", "method": "author_rotateKeys", "params":[]}' http://localhost:9933
```

Output:

```
{“jsonrpc”:”2.0",”result”:”0x5e977ddcc0c69a6aed067052d5bd8f6bd365fae03562fd447d434e9814ac415d7c9ffe722364922bda314e44654f5c0cdc00d152470d5433f12cb73d078061863ac769d5f17b5460f042d221edf0099d2ce4c23edbe96ac943452cc4d3ad6d72”,”id”:1}
```

The output will have a hex-encoded `result` field. Copy and save it!

## Setup Validator

Once your node is live, synchronized, and appears in telemetry, and session keys are prepared, it's time to set up the validator.

Go to Polkadot.{js} app and navigate to Network → Staking → [Account actions](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Frpc.vara.network#/staking/actions) section and click `+Validator`:

Choose your stash and controller accounts and specify your stake amount. It's recommended to utilize distinct accounts both stash and controller.

![5237745d-7fa6-4398-af7a-75cf178c495e](https://github.com/nodeplusio/vara_node_operation/blob/main/img/5237745d-7fa6-4398-af7a-75cf178c495e.png)

Set the `session key` and reward commission.

![a7c5bf20-8b70-4fad-b68e-5a576b91d658](https://github.com/nodeplusio/vara_node_operation/blob/main/img/a7c5bf20-8b70-4fad-b68e-5a576b91d658.png)

Sign the transaction. Ensure you're added to the stash account and wait for the next Era to start. Once it begins, the network will add your validator.

![c10df94f-1d97-4194-8b7a-5a07adf55aa9](https://github.com/nodeplusio/vara_node_operation/blob/main/img/c10df94f-1d97-4194-8b7a-5a07adf55aa9.png)

## **Update Validator**

Regularly updating your validator is critical for keeping up with current maintenance requirements.

The fastest and simplest way to update the validator:

1. Download the latest binary version
2. Swap binaries
3. Restart your service

## Migration Validator

1. Assuming the original host is Host A, and the migrated host is Host B.

2. Host B goes through the installation steps, and after synchronization, starts as a validator. (**Note: Do not use Host A's disk snapshot for Host B's installation, as it may lead to double-signing and result in slashing.**)

3. Host B generates a new session key.

4. Change the session key to the one generated by Host B, and the display will show the next session key as the updated session key from Host B.

5. Wait for the next epoch (1 epoch equals 2 hours). The session key will be updated to Host B's session key. At this point, Host A needs to continue running.

6. Wait for the next era (1 era equals 12 hours). Host A can be shut down, and the migration is complete.

## **Monitoring**

* Use Grafana + Prometheus for monitoring.
* Basic monitoring (CPU, memory, disk, etc.) + specific metrics monitoring (uptime, block height, validator status, etc.).
* Receive alerts through PagerDuty, Telegram, and DingTalk. Handle alerts promptly upon occurrence.
* Check the Grafana dashboard at least once a day to detect any abnormal monitoring metrics.
* Subscribe to new version release information via the company email group dev@eosio.sg and upgrade promptly.

## **Reference Documents**

* Official site: [https://vara.network/](https://vara.network/)
* Block Explorer: [https://explorer.vara-network.io/](https://explorer.vara-network.io/)
* Vara Wiki: [https://wiki.vara.network/](https://wiki.vara.network/)
* Node Telemetry: [https://telemetry.rs/](https://telemetry.rs/)
* Tokenomics:[ https://wiki.vara.network/docs/tokenomics/]( https://wiki.vara.network/docs/tokenomics/)
* Whitepaper: [https://whitepaper.gear.foundation/](https://whitepaper.gear.foundation/)
* Gear Wiki: [https://wiki.gear-tech.io/](https://wiki.gear-tech.io/)
* Gear Foundation: [https://gear.foundation/](https://gear.foundation/)
