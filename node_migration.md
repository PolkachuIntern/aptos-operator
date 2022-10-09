# Polkachu's notes on Aptos validator migration

Aptos validators run a validator node (thereafter `validator`) and a validator fullnode (thereafter `vnf`). At times, we need to migrate validator or vfn.

## Our practice in key/config file management

The vnf provides a full-synced redundant node in case we need to switch over the validator. We keep the following keys/configs on both validator and vfn, even if some of the files are not used based on the node type.

For one thing, we have one fewer thing to worry about when we need to switch. For another, we get a backup copy of the critical files. Combined with a local backup we have encrypted with the strongest math and buried under the deepest ocean, we have the peace of mind with 3 copies.

```
config
├── fullnode.yaml
└── validator.yaml
```

```
keys
├── validator-full-node-identity.yaml
└── validator-identity.yaml
```

## Our Practice in Provisioned Servers

We have 4 servers provisioned by Aptos currently.

1. Mainnet validator: Intel and 4TB
1. Mainnet vfd: AMD and 2TB
1. Testnet validator: Intel and 1TB
1. Testnet vfd: AMD and 1TB

We have a mix of Intel and AMD chips, just to keep it interesting. Intel chips are proved to perform during the testnet, so we keep them as validator servers. AMD chips are more common in cloud data centers, so we want to run them just for the sake of it.

Finally, we have a spare server, ready to step in at any moment.

1. Spare Server: AMD and 1TB

## Scenarios of Node Migration

We will cover 3 scenarios:

1. Planned validator migration
1. Emergency validator migration
1. Routine management of node storage

### Planned Validator Migration

For one reason or another, we might want to move validator to a new server while the current validator and vfn are both functioning.

The key idea in this scenario is to fast-sync the new server to the block height as a 2nd vfn first. We use our own [Ansible deployment script](https://github.com/polkachu/aptos-operator) to do so. The current script does not have the fast-sync config, so we make sure to manually add it before starting the node. Based on our experiences, we can fast-sync to the block height within 30 minutes on both mainnet and testnet. While the new server is doing its work, both validator and existing vfn are functioning without disruption.

Once the new server is fully synced, here is a checklist to migrate the validator:

1. Double-check on the new server to ensure all key files and config files are in place.
1. Stop the current validator
1. Modify the new server's systemd service file and run it as a validator
1. Change DNS to point validator URL to the new server's IP
1. Point the current vfn to the new validator and the new validator's port 6181 needs to be open to the vfn
1. Do whatever we need to do to ensure the old validator is done for good so it does not come back by accident to confuse the network with two validators, which will be bad!

### Emergency Validator Migration

In emergency situations, our validator is not functioning properly and we do not have time to sync a new node. Two situations here:

1. We are sure that the validator will not come back. For example, we accidentally did `sudo rm -rf /`
2. We are not sure whether the validator will come back. For example, the Internet to the data center is cut off so we have no access to the server abd we are not sure whether or when the server will be connected again. In this situation, make sure you monitor the situation and properly shut down one validator if the new validator has started but the old validator comes back to life.

In both situations, we will promote the current vfn to validator (important node: make sure that validator and vfn are in two different data centers). After the "promotion", we will be without a vfn for a short period.

Because the vfn is fully synced, we will follow the checklist:

1. Double-check on the vfn to ensure all key files and config files as a validator are in place.
1. Modify the vfn's systemd service file and run it as a validator
1. Change DNS to point validator URL to the vfn's IP
1. Once the new validator is functioning, we will provision a new server as the new vfn. The normal procedure of starting a vfn applies here.

### Routine Management of Node Storage

Depending on the pruning setting, it is quite possible that we will eventually run out of disk space.

If it is the validator running out of space, just follow "Planned Validator Migration" above. Since the new validator is fast-synced, it will have a much smaller disk requirement, so the problem is solved.

If it is the vfn running out of space, just follow "Planned Validator Migration" too. We just stop once the second vfn is synced then phased out the original vfn. Since the new vfn is fast-synced, it will have a much smaller disk requirement, so the problem is solved.
