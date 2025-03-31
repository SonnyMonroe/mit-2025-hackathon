# Setup LNBits on MutinyNet

_March 27, 2025_


This will allow you to build Lightning Network Apps ("L-Apps") without putting real funds at risk. And then convert that app to real Bitcoin/Lightning.

Tutorial for the MIT Bitcoin Hackathon 2025. Register [here](https://mitbitcoin.devpost.com/) to participate and head to the discord to ask questions.

**⚠️ Tutorial currently under construction. Will be "official" on April 1, 2025**

### LNBits can be used to:
- Request and Send payments
- Create separate wallet for each user/object
- Every action is available via a simple [API](https://v1.lnbits.com/docs)

### Expectations:
- **In this tutorial you will learn:** 
    - how to get onto a testnet, 
    - transact on the testnet in the browser, 
    - build a simple python app which transacts in a website.
- **Prior Experience:**
    - no prior bitcoin/lightning experience
    - some familiarity with coding is helpful
    - no installs or computer setup needed if you chose the repl.it track
    - traditional localhost install if you chose the local/docker track
- **Duration:** 
    - 45 minute - Setup
    - 45 minute - Hello world demo

>**Tutorial Selection:** If you are doing this tutorial live (online or in-person at the hackathon) you'll have it easier because we can send you testnet coins when you're setup. Which makes your setup easier and you can continue below. 
>If you are following this tutorial before/after the hackathon you'll need a slightly more complicated setup, head over here for those [instructions](./two-node-setup.md)
 

---

# Setup 
_45 mins_
## Create account on voltage
- Sign-up for Voltage: https://www.voltage.cloud/
- We will only be using **free services**
- Later on you can convert this same setup to real bitcoin "mainnet" with a flip of a variable and adding real bitcoin onto your node [0]
### Create a Mutinynet Node
- These should be available for zero compute credits so you can create them for free.
- This will take ten minutes to sync with the chain so let's get that out of the way.
- Note bene: you may notice there is button on this admin page which says "Launch LNBits" which looks appealing. Unfortunately this button doesn't work on testnet
#### Coins & Channels
- Unlock your node via the unlock button in top right and entering your password
- Click 'get coins' and 1M fake sats, wait for confirm
- Click open channel, push 25% across the node [1](#footnotes), wait for confirm
- Repeat for other node
- Here's a guide that's a little out-of-date[2](#footnotes): https://docs.voltage.cloud/dev-sandbox-mutinynet

## Run LNBits locally (or on repl.it)
- Clone down LNBits: https://github.com/lnbits/lnbits
```bash
git clone git@github.com:lnbits/lnbits.git 
```
- The main branch should work fine, current version is ~`1.0.0rc7`

### Run with Docker
- Build the image with docker
```bash
docker build -t local/lnbits .
```
- Make a persistence volume in project root: `mkdir data`
- Modify the env at project root: `.env`:
```bash
comment wallets and update to voidWallet
```
- run with docker per project's [instructions](https://github.com/lnbits/lnbits/blob/main/docs/guide/installation.md#option-4-docker): (Note this assumes you are running this command from the project root.)
```bash
docker run \
    --detach \
    --publish 5000:5000 \
    --volume ${PWD}/.env:/app/.env \
    --volume ${PWD}/data/:/app/data \
    local/lnbits
```
- Verify it works: it should be available on http://localhost:5000

### Connect LNBits to node on voltage
We want to get the following settings to fill for the next section: `LND_REST_ENDPOINT`,`LND_REST_MACAROON`, and `LND_REST_CERT`.
  - the cert is not strictly necessary but you may experience degraded performance without it.
#### Find connection settings on voltage admin
- On Voltage Admin panel go to left sidebar
  - get the macaroon in `base64` format (not `hex`)

>A helpfule guide for this is [here](https://docs.voltage.cloud/dev-sandbox-mutinynet).
#### Modify `.env`
Find where these settings live and comment out the initial values, fill them with your new settings
```bash
LND_REST_ENDPOINT="<subdomain.domain:port>" # my-node-name.u.voltageapp.io:8080
LND_REST_CERT=""
LND_REST_MACAROON="AgEDbG5kAvgBAwo...="
```

#### Re-run the docker with modifications:
- you may need to shutdown the previous run: 
```bash
# view running containers
docker ps
# the run this with the conatiner name of the previous run
docker stop <contianer-name>
```
```bash
docker run \
    --detach \
    --name lnbits-mutiny-1
    --publish 5000:5000 \
    --volume ${PWD}/.env:/app/.env \
    --volume ${PWD}/data/:/app/data \
    local/lnbits
```
- Same command but with a `name` arg. This way we run run `docker stop  lnbits-mutiny-1` or `docker start lnbits-mutiny-1`.
- Remember this command need to be run from project root due to the `PWD` variable.

#### Verify its working:
- Head back to `http://localhost:5000/` Verify the site loads.
- Now you'll perform several actions outlined ion more detail in the section below[`#WebUI`](#webui---using-lnbits-in-the-browser).
    - Now create a wallet and give it a name.
    - Now create an invoice. 
- This verifies that your locally running lnbits is connecting to your Lightning Node running on a Voltage server [3].

#### Verify persistence is working
- Save your `usr` id from TopRight > Account > Settings > usr
- Start and stop the docker container:
```bash
docker stop lnbits-mutiny-1  # should take ~20 secs to shutdown
docker start lnbits-mutiny-1  # should take ~20 secs to startup
```
- Head back to localhost:8000, enter the `usr` id.
    - this might work automatically with your browser saving your login credentials
    - but it's worth practicing 
- Verify the wallet you named is still showing.

#### Debug if nec.
If everything's working, you can move to the next section. Otherwise here's a few things to check:
<details>
<summary>
<b>Logs</b>
</summary>
Running `docker logs mutiny-1` should produce logs and you can search for this section:
- Note how there are multiple attempts to connect to the backend before it succeeds. This is expected behavior and we see in this case it does eventually connect.

```bash
2025-03-28 02:42:48.61 | WARNING | No certificate for LndRestWallet provided! This only wor
ks if you have a publicly issued certificate.
2025-03-28 02:42:48.64 | INFO | Connecting to backend LndRestWallet...
2025-03-28 02:42:53.65 | WARNING | 
2025-03-28 02:42:53.65 | ERROR | The backend for LndRestWallet isn't working properly: 'Una
ble to connect to https://mutiny-1.u.voltageapp.io:8080.'
2025-03-28 02:42:53.66 | WARNING | Retrying connection to backend in 0.5 seconds... (1/4)
2025-03-28 02:42:54.16 | INFO | Connecting to backend LndRestWallet...
2025-03-28 02:42:59.16 | WARNING | 
2025-03-28 02:42:59.16 | ERROR | The backend for LndRestWallet isn't working properly: 'Unable to connect to https://mutiny-1.u.voltageapp.io:8080.'
2025-03-28 02:42:59.16 | WARNING | Retrying connection to backend in 1.0 seconds... (2/4)
2025-03-28 02:43:00.16 | INFO | Connecting to backend LndRestWallet...
2025-03-28 02:43:00.48 | SUCCESS | ✔️ Backend LndRestWallet connected and with a balance of 997476000 msat.
2025-03-28 02:43:00.63 | INFO | Application startup complete.
2025-03-28 02:43:00.63 | INFO | Uvicorn running on http://0.0.0.0:5000 (Press CTRL+C to quit)
```
</details>

## WebUI - Using LNBits in the browser
Now we'll work in the browser seeing how to create 
- Create wallets, 
    - add name
    - logout of account if you're already logged in
- Save your user id from account
- You can test restarting your docker and your coins are still there.

## Recieve & Pay between your wallets
- Create invoice
- Pay an invoice:
    - decode
    - pay
    - view history
    - try to pay too much
## First look at using the API
- View quickstart API tutorial on the right sidebar

---

# Hello World with Python + LNBits
_Pay coins between your nodes programmatically_

_45 mins_


---

## Footnotes
- 1: By pushing some amount of coins across the channel, it enable us to move coins in both directions: into our node and out of our node.
- 2: The official wallet (https://signet-app.mutinywallet.com/) for this testnet is [deprecated](https://blog.mutinywallet.com/mutiny-wallet-is-shutting-down/) but the [network itself](https://mutinynet.com/) is still active so we'll just our two nodes as wallets.
- 3: Actually, Voltage runs on the big clouds like AWS and GCP, not their own servers. They are running the main program you need, LND which is connecting to the broader mutinynet network.

## Hungry for more
- We can use this same application on LNBits for real sats without having to setup our node: https://v1.lnbits.com/wallet
    - you can use login with github
    - don't have real sats on the lightning network, maybe ask someone?
- How to convert from Mutinynet ("test network") to Mainnet ("real bitcoin").
- LNBits Extensions

---

- _Authored March 27, 2025,_
- _Will Sutton [github.com/sutt](https://github.com/sutt)_
- _Pavel [github.com/super-jaba](https://github.com/super-jaba)_

**Special Thanks to:**