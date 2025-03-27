# Setup Two LNBits accounts on MutinyNet

_Authored March 27, 2025, Will Sutton [github.com/sutt](https://github.com/sutt)_

This will allow you to build Lightning Network Apps ("L-Apps") without putting real funds at-risk.

LNBits can be used to:
- Request and Send payments
- Create separate wallet for each user/object
- Has multiple extensions for custom activities
- Everything action is available via a traditional REST-API

## Create account on voltage
- Sign-up for Voltage: https://www.voltage.cloud/
- We will only be using **free services**
- Later on you can convert this same setup to real bitcoin "mainnet" with a flip of a variable and adding real bitcoin onto your node [0]
### Create two Mutinynet nodes
- These should be available for zero compute credits so you can create them for free.
- These will take ten minutes to sync so let's get that out of the way
- Note bene: we are creating two nodes so:
    - one node can act as bank of the app your building and
    - the other node can act as the customer paying or recieving funds from the app.
- Note bene: you may notice there is button on this admin page
#### Coins & Channels
- Unlock your node via the unlock button in top right and entering your password
- Click 'get coins' and 1M fake sats, wait for confirm
- Click open channel, push 25% across the node [1](#footnotes), wait for confirm
- Repeat for other node
- Here's a guide that's a little out-of-date[2](#footnotes): https://docs.voltage.cloud/dev-sandbox-mutinynet

## Run LNBits locally (or on repl.it)
- Clone down LNBits: https://github.com/lnbits/lnbits
> git clone git@github.com:lnbits/lnbits.git 
- The main branch should work fine, current version is ~`1.0.0rc7`
### Run with Docker
- Create an env at project root: `cp .env .env.node.1`
- Make a persistence volume: `mkdir data && mkdir data/node.1`
- Build the image with docker
```bash
docker build -t local/lnbits .
```
- run with docker per project's [instructions](https://github.com/lnbits/lnbits/blob/main/docs/guide/installation.md#option-4-docker): (Note this assumes you are running this command from the project root.)
```bash
docker run \
    --detach \
    --publish 5000:5000 \
    --name mutiny-2 \
    --volume ${PWD}/.env.node.1:/app/.env \
    --volume ${PWD}/data/node.1:/app/data \
    local/lnbits
```
- Verify it works: it should be available on http://localhost:5000
### Configure - Connect LNBits to nodes on voltage
- On Voltage Admin panel go to left sidebar
  - get the macaroon in `base64` format (not `hex`)


- run the docker image:
```bash

```
- Verify it works by generating an invoice

### Run two instances on different ports
- Now repeat for the second node. Modifications include:
  - Create a second env file `cp .env.node.1 .env.node.2` and modify:
    - `LND_REST_ENDPOINT`
    - `LND_REST_CERT`
    - `LND_REST_MACAROON`
  - modify docker run command for:
    - system port `5001`
    - persistence volume to `data/node.2`
    - container name to `mutiny-2`
  ```bash


### WebUI - setup wallets
- Create wallets, logout of account if you're already logged in
- Save your user id from account

## Recieve coins & Pay between your nodes manually
- Use two separate browser profiles for your two nodes
- View quickstart API tutorial on the right sidebar

## Pay coins between your nodes programmatically

## Footnotes
- 1: By pushing some amount of coins across the channel, we move coins in both directions: into our node and out of our node.
- 2: The official wallet (https://signet-app.mutinywallet.com/) for this testnet is [deprecated](https://blog.mutinywallet.com/mutiny-wallet-is-shutting-down/) but the [network itself](https://mutinynet.com/) is still active so we'll just our two nodes as wallets.

## Optional - Create your own seed payment to LNBits
So if you are following this tutorial and you're not at the hackathon you won't have 


