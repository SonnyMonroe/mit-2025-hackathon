

### Fork on Repl.it
- Sign up for repl.it if you haven't already. We suggest using GitHub OAuth, but other methods should work as well.
- Then head to this page 
    - https://replit.com/@sutt/LNBits-Tutorial-2025
    - and click "Remix this app". This will create the app
        - you can only have 3 apps on replit with the free plan so delete existing apps if you're over the limit.
- Note bene: this is based off the main [lnbits github repo](https://github.com/lnbits/lnbits) but with several things modified in my fork to help in run in replit's environment.

### Setup Admin
- Change `HOST=0.0.0.0`: this will allow you to view the webpage.
- Leave `LNBITS_ADMIN_UI=true`
- Create a master user in the browser
- Open the "VoidWallet" (just the UI, no backend connected)
- Open the preview, something like: https://4e6a61f2-fab4-4a86-b223-08c56049495c-00-ixaxv8wabsgw.riker.replit.dev/
    - sign in on this page so your browser remembers you when we reload.

### Connect to Voltage
#### Find these variable in Admin UI
- Modify the `.env`
    - Set `LNBITS_ADMIN_UI=false`. This will force lnbits to try to connect to the backend.
    - Modify `LND_REST_` values to hard coded or secrets.
        - You can add these in replit secrets or just paste them here. Just pasting should be fine for the duration of the hackathon since these are testnet coins.
```bash
LNBITS_ADMIN_UI=false
LND_REST_ENDPOINT="<subdomain.domain:port>" # my-node-name.u.voltageapp.io:8080
LND_REST_CERT=""  # deliberate blank string
LND_REST_MACAROON="AgEDbG5kAvgBAwo...="  # paste your full one here
```
- ⚠️ Never use this setup with real bitcoin lightning node, instead run lnbits on your own private server.

#### Verify it works
- VoidWallet warning should be gone,
- Generate an invoice to prove you are connected.
- Debug if your browser doesn't sign you in: logout, then "sign is in with usr id" and enter the `usr` id you copied.

### Done with repl.it specific setup
Head back to the [main tutorial](./readme.md#webui---using-lnbits-in-the-browser) here to complete the walk thru.

One note: you can send yourself the link to the replit preview url (and your usr id) and login with your phone to have a mobile wallet, which you can use to scan the qr code and pay invoices.


### Replit specific using the API
- Head into repl.it's _Shell_ window 
- you can do your `curl`'s from there using both `curl http://127.0.0.1:5000/...` or your apps preview url, e.g. something like this `curl https://4e6a61f2-fab4-4a86-b223-08c56049495c-00-ixaxv8wabsgw.riker.replit.dev/...`

### Replit Debugging 
You don't need to be here, but if something went wrong, your probably are:

#### To reset/refresh in order of increasing severity:
- try run/stop/run the replit app.
- try deleting the `data/` folder. This will lose any wallets and admin user you have setup but create a way to start fresh.
- delete the repl.it "app" in https://replit.com/repls then go back to the [base repo](https://replit.com/@sutt/LNBits-Tutorial-2025) and "remix" it again.
- reach out to us on discord.

#### Modifications + explanations
- The nix settings are updated to accomodate the difficult of building the secp256k1 package on replit vm. Instead this is loaded in system nix settings as a system python package and then added to the packages inside of the poetry environment when running the app.

`.pyproject.toml` removed secp256k1 as a dependency to prevent install error.

```
# secp256k1 = "0.14.0"
```

`.replit` settings:
```
compile = "poetry install --only main"
run = "PYTHONPATH='/nix/store/x3wnmi3zvckmn0xf80ma8cwqwkbhys66-python3.12-secp256k1-0.14.0/lib/python3.12/site-packages:$PYTHONPATH' poetry run lnbits"
```

`replit.nix` settings:
```
deps = [
    pkgs.python312Packages.secp256k1
    ...
```

### full packages

```
{pkgs}: {
  deps = [
    pkgs.python312Packages.secp256k1
    pkgs.python311Packages.python
    pkgs.zlib
    pkgs.openssl
    pkgs.grpc
    pkgs.c-ares
    pkgs.rustc
    pkgs.libiconv
    pkgs.cargo
    pkgs.libuv
    pkgs.pkg-config
    pkgs.postgresql
    pkgs.libxcrypt
    # pkgs.python3       # ensures Python is available (use python3_12 if needed for 3.12.x)
    # pkgs.python3-dev       # ensures Python is available (use python3_12 if needed for 3.12.x)
    pkgs.gcc           # C compiler
    pkgs.pkg-config    # pkg-config for finding libraries
    pkgs.autoconf      # autoconf (often needed for ./autogen.sh)
    pkgs.automake      # automake (for generating Makefiles)
    pkgs.libtool       # libtool (for linking)
    pkgs.libffi        # libffi library (includes headers)
    pkgs.gmp           # GNU MP library (for big numbers, includes headers)
    # pkgs.libsecp256k1-dev
  ];
}

```