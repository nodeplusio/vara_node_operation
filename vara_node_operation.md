# 

## **Security**

**1. Server Security**

* Use the Ubuntu account for single sign-on.
* Firewall settings allow login only through the jump server bastion host.
* Disable root account login; use sudo if necessary.
* Allow only port 30333 with full firewall access.
* Monitoring ports 9100 and 9090 are only allowed to be opened to the monitoring host.

**2. Private Key Management**

Two different accounts can be used to securely manage your funds while staking.

* **Stash:** This account holds funds bonded for staking, but delegates all staking functions to a staking proxy account. You may actively participate in staking with a stash private key kept in a cold wallet like Ledger, meaning it stays offline all the time. Having a staking proxy will allow you to sign all staking-related transactions with the proxy instead of using your Ledger device. This will allow you:
  
  * to avoid carrying around your Ledger device just to sign staking-related transactions, and
  * to and to keep the transaction history of your stash clean

* **Staking Proxy(Controller):** This account acts on behalf of the stash account, signaling decisions about nominating and validating. It can set preferences like commission (for validators) and the staking rewards payout account. The earned rewards can be bonded (locked) immediately for bonding on your stash account, which would effectively compound the rewards you receive over time. You could also choose to have them deposited to a different account as a free (transferable) balance. If you are a validator, it can also be used to set your [session keys](https://wiki.polkadot.network/docs/learn-cryptography). Staking proxies only need sufficient funds to pay for the transaction fees.

* Separate management of Stash and controller private keys. [Understand stash and controller](https://wiki.polkadot.network/docs/en/learn-staking#accounts).

* Stash is stored in a hardware wallet.

* Controller mnemonic appears in the claim script. Ensure server security is maintained.

**3. Protect Yourself from Scams**

Scams are an unfortuante reality of the crypto industry. It's important to stay alert and protect yourself and your non-refundable crypto assets from scammers.

- Never, ever, ever share your seed phrase or account password.

- Do not trust anyone online. It is trivial for them to lie and change their identities.

- If you are scammed, there is likely nothing that can be done to recover your funds. If a scammer gets a hold of your seed phrase, they can transfer all of your funds to their account in seconds. It is better to be safe than to risk all of your tokens.

- If it sounds too good to be true, it probably is. People, especially celebrities, do not give away crypto for free. Even if they wanted to, they could just ask for your address as opposed to having you send them tokens.

- Scams are absolutely rife in this space. It is easy and cheap to set a scam up, and hard to shut one down. Therefore, the onus is on the user to be as diligent as possible in avoiding them.

- If you can, try to always verify new information that you see with an official source. Often scammers will fake a website or a blog post, but if you check it against a secondary source you will reduce the chances of being scammed.

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

Validators perform critical functions for the network, and as such, have strict uptime requirements. Validators may have to go offline for short-periods of time to upgrade client software or to upgrade the host machine. Usually, standard client upgrades will only require you to stop the service, replace the binary and restart the service. This operation can be executed within a session and if performed correctly will not produce a slashable event.

Validators may also need to perform long-lead maintenance tasks that will span more than one session. Under these circumstances, an active validator may chill their stash and be removed from the active validator set. Alternatively, the validator may substitute the active validator server with another allowing the former to undergo maintenance activities.

This guide will provide an option for validators to seamlessly substitute an active validator server to allow for maintenance operations.

The process can take several hours, so make sure you understand the instructions first and plan accordingly.

**Session Keys**

Session keys are stored in the client and used to sign validator operations. They are what link your validator node to your staking proxy. If you change them within a session you will have to wait for the current session to finish and a further two sessions to elapse before they are applied.

**Keystore**

Each validator server contains essential private keys in a folder called the _keystore_. These keys are used by a validator to sign transactions at the network level. If two or more validators sign certain transactions using the same keys, it can lead to an [equivocation slash](https://wiki.polkadot.network/docs/learn-staking#slashing).

For this reason, it is advised that validators DO NOT CLONE or COPY the _keystore_ folder and instead generate session keys for each new validator instance.

Default keystore path - `~/.local/share/gear/chains/vara_network/keystore`

1. Assuming the original host is Host A, and the migrated host is Host B.

2. Host B goes through the installation steps, and after synchronization, starts as a validator. (**Note: Do not use Host A's disk snapshot for Host B's installation, as it may lead to an equivocation slash.**)

3. Host B generates a new session key.

4. Change the session key to the one generated by Host B, and the display will show the next session key as the updated session key from Host B.

5. Wait for the next epoch (1 epoch equals 2 hours). The session key will be updated to Host B's session key. At this point, Host A needs to continue running.

6. Wait for the next era (1 era equals 12 hours). Host A can be shut down, and the migration is complete.
   
   

## Introduction to Slashing

Slashing will happen if a validator misbehaves (e.g. goes offline, attacks the network, or runs modified software) in the network. They and their nominators will get slashed by losing a percentage of their bonded/staked VARA.

Any slashed VARA will be added to the [Treasury](https://wiki.polkadot.network/docs/learn-treasury). The rationale for this (rather than burning or distributing them as rewards) is that slashes may then be reverted by the Council by simply paying out from the Treasury. This would be useful in situations such as faulty slashes. In the case of legitimate slashing, it moves tokens away from malicious validators to those building the ecosystem through the normal Treasury process.

Validators with a larger total stake backing them will get slashed more harshly than less popular ones, so we encourage nominators to shift their nominations to less popular validators to reduce their possible losses.

It is important to realize that slashing only occurs for active validations for a given nominator, and slashes are not mitigated by having other inactive or waiting nominations. They are also not mitigated by the validator operator running separate validators; each validator is considered its own entity for purposes of slashing, just as they are for staking rewards.

In rare instances, a nominator may be actively nominating several validators in a single era. In this case, the slash is proportionate to the amount staked to that specific validator. With very large bonds, such as parachain liquid staking accounts, a nominator has multiple active nominations per era. Note that you cannot control the percentage of stake you have allocated to each validator or choose who your active validator will be (except in the trivial case of nominating a single validator). Staking allocations are controlled by the [Phragmén algorithm](https://wiki.polkadot.network/docs/learn-phragmen).

Once a validator gets slashed, it goes into the state as an "unapplied slash". You can check this via [Polkadot-JS UI](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Frpc.polkadot.io#/staking/slashes). The UI shows it per validator and then all the affected nominators along with the amounts. While unapplied, a governance proposal can be made to reverse it during this period (7 days). After the grace period, the slashes are applied.

The following levels of offense are [defined](https://research.web3.foundation/Polkadot/security/slashing/amounts). However, these particular levels are not implemented or referred to in the code or in the system; they are meant as guidelines for different levels of severity for offenses. To understand how slash amounts are calculated, see the equations in the section below.

* Level 1: isolated [unresponsiveness](https://wiki.polkadot.network/docs/learn-staking-advanced#unresponsiveness), i.e. being offline for an entire session. Generally no slashing, only [chilling](https://wiki.polkadot.network/docs/learn-staking#chilling).

* Level 2: concurrent unresponsiveness or isolated [equivocation](https://wiki.polkadot.network/docs/learn-staking-advanced#equivocation), slashes a very small amount of the stake and chills.

* Level 3: misconducts unlikely to be accidental, but which do not harm the network's security to any large extent. Examples include concurrent equivocation or isolated cases of unjustified voting in [GRANDPA](https://wiki.polkadot.network/docs/learn-consensus). Slashes a moderately small amount of the stake and chills.

* Level 4: misconduct that poses serious security or monetary risk to the system, or mass collusion. Slashes all or most of the stake behind the validator and chills.

## Best practices to prevent slashing

A slash may occur under four circumstances:

1. Unresponsiveness

2. Equivocation

3. Malicious action

4. Application related (bug or otherwise)

This article provides some best practices to prevent slashing based on lessons learned from previous slashes. It provides comments and guidance for all circumstances except for malicious action by the node operator.

##### Unresponsiveness

The following are recommendations to validators to avoid slashing under liveliness for servers that have historically functioned:

1. Utilize systems to host your validator instance. Systemd should be configured to auto reboot the service with a minimum 60-second delay. This configuration should aid with re-establishing the instance under isolated failures with the binary.
2. A validator instance can demonstrate un-lively behaviour if it cannot sync new blocks. This may result from insufficient disk space or a corrupt database.
3. Monitoring should be implemented that allows node operators to monitor connectivity network connectivity to the peer-to-peer port of the validator instance. Monitoring should also be implemented to ensure that there is <50 Block ‘drift’ between the target and best blocks. If either event produces a failure, the node operator should be notified. The following are recommendations to validators to avoid liveliness for new servers / migrated servers:
4. Ensure that the `--validator` flag is used when starting the validator instance
5. If switching keys, ensure that the correct session keys are applied
6. If migrating using a two-server approach, ensure that you don’t switch off the original server too soon.
7. Ensure that the database on the new server is fully synchronized.
8. It is highly recommended to avoid hosting on providers that other validators may also utilize. If the provider fails, there is a probability that one or more other validators would also fail due to liveliness building to a slash.  
   There is a precedent that a slash may be forgiven if a single validator faces an offline event when a larger operator also faces multiple offline events, resulting in a slash.

##### Equivocation

The following are scenarios that build towards slashes under equivocation,the following scenarios should be avoided:

1. Cloning a server, i.e., copying all contents when migrating to new hardware. This action should be avoided. If an image is desired, it should be taken before keys are generated.
2. High Availability (HA) Systems – Equivocation can occur if there are any concurrent operations, either when a failed server restarts or if false positive event results in both servers being online simultaneously. HA systems are to be treated with extreme caution and are not advised.
3. The keystore folder is copied when attempting to copy a database from one instance to another.  
   It is important to note that equivocation slashes occur with a single incident. This can happen if duplicated keystores are used for only a few seconds.

##### Application Related

In the past, there have been releases with bugs that lead to slashes; these issues are not as prevalent in current releases. The following are advised to node operators to ensure that they obtain pristine binaries or source code and to ensure the security of their node:

1. Always download either source files or binaries from the official Parity repository
2. Verify the hash of downloaded files.
3. Use the W3F secure validator setup or adhere to its principles
4. Ensure essential security items are checked, use a firewall, manage user access, use SSH certificates
5. Avoid using your server as a general-purpose system. Hosting a validator on your workstation or one that hosts other services increases the risk of maleficence.

## Rewards management

Use the claim script to schedule rewards claim at the end of each era,after the claim script runs normally, the number of rewards will be notified through monitoring alarms.

![8c375517-4e70-4fa9-96b1-7bd79c6bb317](https://github.com/nodeplusio/vara_node_operation/blob/main/img/8c375517-4e70-4fa9-96b1-7bd79c6bb317.png)

## **Monitoring**

* Use Grafana + Prometheus for monitoring.

* Basic monitoring (CPU, memory, disk, etc.) + specific metrics monitoring (uptime, block height, peers,validator status, etc.).
  ![3a5773ae-1115-4d1b-b00f-f53bdb32ce61](https://github.com/nodeplusio/vara_node_operation/blob/main/img/3a5773ae-1115-4d1b-b00f-f53bdb32ce61.png)

* Receive alerts through PagerDuty, Telegram, and DingTalk. Handle alerts promptly upon occurrence.

* Check the Grafana dashboard at least once a day to detect any abnormal monitoring metrics.

* Subscribe to new version release information via the company email group and upgrade promptly.

## **Reference Documents**

* Official site: [https://vara.network/](https://vara.network/)
* Block Explorer: [https://explorer.vara-network.io/](https://explorer.vara-network.io/)
* Vara Wiki: [https://wiki.vara.network/](https://wiki.vara.network/)
* Node Telemetry: [https://telemetry.rs/](https://telemetry.rs/)
* Tokenomics:[ https://wiki.vara.network/docs/tokenomics/](https://wiki.vara.network/docs/tokenomics/)
* Whitepaper: [https://whitepaper.gear.foundation/](https://whitepaper.gear.foundation/)
* Gear Wiki: [https://wiki.gear-tech.io/](https://wiki.gear-tech.io/)
* Gear Foundation: [https://gear.foundation/](https://gear.foundation/)
* Polkadot Wiki: [https://wiki.polkadot.network/](https://wiki.polkadot.network/)
