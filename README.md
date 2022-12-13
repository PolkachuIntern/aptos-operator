# Aptos Node Operator Ansible Scripts

## Design Philosophy

1. Cover scenarios with multiple playbooks (initial setup, node upgrade, validator migration, etc)
2. Support both testnet and mainnet
3. Support both validator, validator fullnode and public fullnode

## TL/DR

Once all set, you run one playbook to do a task, such as:

```bash
ansible-playbook main.yml -e "target=fullnode_mainnet"
ansible-playbook main_upgrade.yml  -e "target=fullnode_mainnet"
```

The target can be:

- validator_mainnet
- vfn_mainnet
- fullnode_mainnet
- indexer_mainnet
- validator_testnet
- vfn_testnet

## Set up your own inventory file

First create your own inventory file by copying the sample one. Then you customize the file with your own.

```bash
cp inventory.sample.yaml inventory.yaml
```

## Playbooks

| Playbook           | Description                                  |
| ------------------ | -------------------------------------------- |
| `main.yml`         | Set up a new node                            |
| `main_upgrade.yml` | Upgrade the node version and restart service |

## Port management

| Node                 | Validator                          | Validator Fullnode     | Public Fullnode        |
| -------------------- | ---------------------------------- | ---------------------- | ---------------------- |
| P2P port             | 6180 Open to all, 6181 Open to vfn | None open              | 6182 Open to all       |
| Prometheus port 9101 | Open to monitor server             | Open to monitor server | Open to monitor server |
| REST PORT 8080       | Closed                             | Closed                 | Reverse proxy to 80    |

## Limitation

These playbooks do not touch any key files. We already do it manually. We think you should too.
