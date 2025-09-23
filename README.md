## Autoclock Validator
* A fast and straightforward way to spin up a Solana validator running the Jito client. 
* The Autoclock Validator Ansible script 

### Follow these steps:

#### If you don't already have a validator, start with the Solana docs:
1) Install Solana Tool Suite https://docs.solana.com/cli/install-solana-cli-tools
2) Create validator identity keypair https://docs.solana.com/running-validator/validator-start#generate-identity
3) Create withdrawer account keypair https://docs.solana.com/running-validator/validator-start#create-authorized-withdrawer-account
4) Create vote account keypair https://docs.solana.com/running-validator/validator-start#create-vote-account

#### 1) Install Ansible locally
```
https://docs.ansible.com/ansible/latest/installation_guide/installation_distros.html
```

#### 2) Create a hosts.yaml file in the root location using hosts.example.yaml as template
* https://github.com/Cap-repository-hub/autoclock-validator/blob/master/hosts.example.yaml
* Edit hosts.yaml so that it points to your validator's IP address and needed ssh parameters

#### 3) Edit and configure the Ansible command and run it
* This step assumes that validator-keypair.json and vote-account-keypair.json have been generated using solana-keygen and that the vote-account has already been created. The Ansible playbook executes the vote-account command to see that vote-account-keypair.json actually exists and is associated with validator-keypair.json. It will fail before starting the validator if that is not the case.
* Make sure that the solana_version is up-to-date and that you specify the correct cluster
```
ansible-playbook setup.yaml -i hosts.yaml -e id_path=./keys/validator-keypair.json -e vote_path=./keys/vote-account-keypair.json -e region=ny -e cluster=mainnet -e rpc_address=https://api.mainnet-beta.solana.com -e repo_version=v1.13.6-jito
```
#### 4) Edit and configure the jito main.yaml file
* https://github.com/Cap-repository-hub/autoclock-validator/blob/master/roles/jito/defaults/main.yaml
* Inside this file you will see (and may need to edit):
```
# 1. Supply a valid cluster
# testnet, mainnet
cluster: "testnet"

# 2. Supply a valid jito block engine location
# mainnet: amsterdam, frankfurt, ny, tokyo 
# testnet: dallas, ny
location: "ny"

# Commission is in basis points (bps). 100 bps = 1%
commission_bps: 800
# Optional. This sends metricss to Solana's public influx and it is encouraged to set to true since it helps Solana Labs and others debug your validator as well as network issues.
metrics: true
org_name: "jito-foundation"
repo_name: "jito-solana"
repo_dir: "jito-solana"
repo_version: "v1.13.6-jito"
```
* The location as mentioned in the comment needs to be one of the 4 (for mainnet) or one of 2 (for testnet) - the validator parameters for block engine, relayer, etc are set based on the nearest location to your validator server.
* Other parameters can be left as is (most validators set commission to 800 basis points atm, but you can adjust that if you want to).

#### 5) SSH into your new server

#### 6) Start a tmux session
```
tmux
```

#### 7) Switch to the solana user with:
```
sudo su - solana
```

#### 8) Check the status
```
source ~/.profile
solana-validator --ledger /mnt/operations/solana/ledger monitor

Ledger location: /mnt/operaions/solana/ledger
⠉ Validator startup: SearchingForRpcService...
```

#### Initially the monitor should just show the below message which will last for a few minutes and is normal:

```
⠉ Validator startup: SearchingForRpcService...
```

#### After a while, the message at the terminal should change to something similar to this:

```
⠐ 00:08:26 | Processed Slot: 156831951 | Confirmed Slot: 156831951 | Finalized Slot: 156831917 | Full Snapshot Slot: 156813730 |
```

#### Check whether the validator is caught up with the rest of the cluster with:

```
solana catchup --our-localhost
```

If you see the message above, then everything is working fine! gratz. You have a new validator server and you can visit the URL at http://xx.xx.xx.xx:8899/

#### Check if vote transactions are landing
* Search your validator's identity on a block explorer to see if votes from your validator are landing.
* If this is your first time starting a validator, your Identity account needs enough Sol for voting fees.
* Example: https://solana.fm/address/C1ocKDYMCm2ooWptMMnpd5VEB2Nx4UMJgRuYofysyzcA

### Helpful Links
* [Jito Discord](https://discord.gg/jito)
* [Jito Docs](https://jito.notion.site/Jito-Resources-76fac1863c23457198f46657b54d7a6a)
* [Solana Discord](https://discord.gg/ZmYnApcbTj) (validator-support channel)

