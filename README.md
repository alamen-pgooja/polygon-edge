![Banner](.github/banner.jpg)

## Update: Edge v0.9.0 is here!

Developers at Polygon Labs have been hard at work gathering and incorporating community feedback into the Edge client and a new version of Edge, v0.9.0, is here with several new features! Check out the Release Notes included with the release to find out more!

## Polygon Edge

Polygon Edge is a modular and extensible framework for building Ethereum-compatible blockchain networks.

To find out more about Polygon, visit the [official website](https://polygon.technology/).

WARNING: This is a work in progress so architectural changes may happen in the future. The code is still being audited, so please contact the Polygon team if you would like to use it in production.

## Documentation 📝

If you'd like to learn more about the Polygon Edge, how it works and how you can use it for your project,
please check out the **[Polygon Supernets Documentation](https://wiki.polygon.technology/docs/supernets/get-started/what-are-supernets)**.

## Disclaimer

As this project evolves, the Polygon Labs developer team will focus on the latest version of the Edge client and does not plan to support Edge 0.6 or lower. It is highly recommended that you upgrade to the newest version with the most up-to-date features and fixes. Users that want to stay on 0.6 or below, can continue to do so. The repo will continue to exist and users can fork it and do with it as they please, subject to applicable open-source license terms.

---

Copyright 2022 Polygon Technology

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

# Polygon Edge - Production Deployment Guide

This document lists out the commands used for deploying a production grade Polygon Edge network with secure secrets.

## Clone & Build

```bash
git clone https://github.com/0xPolygon/polygon-edge.git

make
make build
```

## Create A Vault

The following is a docker-compose file used to create a docker container for the vault (Hashicorp). The vault is exposed on `http://127.0.0.1:8200`.

```bash
version: '3'
services:
  myvault:
    image: vault
    container_name: myvault
    ports:
      - 8200:8200
    environment:
       VAULT_SERVER: "http://127.0.0.1:8200"
       VAULT_DEV_ROOT_TOKEN_ID: "my-token"
```

## Create Secrets

Now that the vault is accessible, we'll supply the url & token to the "init" command to store the secrets in the vault after creation.

```bash
./polygon-edge secrets generate --name secretMan --token my-token --server-url http://127.0.0.1:8200
```

```bash
./polygon-edge secrets init --config ./secretsManagerConfig.json
```

#### The above commands will store the secrets in the specified vault and print the following.

```bash
[SECRETS INIT]
Public key (address) = 0xf0b581F4256B8801D8e397a0024883eeBdEe2e38
BLS Public key       = 0x87b756961fa6304bfcf5156177a782e22f0b077ad2bef01f0b175a76ca4928fd0637704fe724073cd64dbd2c919d0ba8
Node ID              = 16Uiu2HAm6CVzf6VfHqR5WnFwZCdBrKiGaTsqU2McXBVTjqfzUTe7
```

## Create Genesis File

```bash
./polygon-edge genesis --consensus ibft --ibft-validator 0xf0b581F4256B8801D8e397a0024883eeBdEe2e38:0x87b756961fa6304bfcf5156177a782e22f0b077ad2bef01f0b175a76ca4928fd0637704fe724073cd64dbd2c919d0ba8 --bootnode /ip4/127.0.0.1/tcp/10001/p2p/16Uiu2HAm6CVzf6VfHqR5WnFwZCdBrKiGaTsqU2McXBVTjqfzUTe7
```

## Start Node

```bash
./polygon-edge server --data-dir chain-data --secrets-config ./secretsManagerConfig.json --chain ./genesis.json --grpc-address :10000 --libp2p :30301 --jsonrpc :10002 --seal
```
