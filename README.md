# liqwid-batching-bot docker image

## Overview

The batching bot docker image. The bot will query the marketState for the next available bathing slot. If available the bot will first run `marketBatch` and then `marketBatch` final.

> Note: As there may well be multiple bots batching the same market contention is expected. The bots will do their best to catch error messages and only submit them if another bot has not succesfully submited the batch. However some errors are to be expected around UTxO contention.

Each batching bot will batch a single market only.
To batch multiple markets multiple batching bots must be run and their env-files set up (details below) accordingly.

## Usage

Running:

```
docker load < ghcr.io/liqwid-labs/liqwid-batching-bot
docker run --env-file batchingBotEnv ghcr.io/liqwid-labs/liqwid-batching-bot:latest
```

> NOTE: Due to the static parameterization of the protocol two docker images of the bot are provided. One image intended for the mainnet and the other intended for the preview testnet. Using the incorrect image for the network will result in the bot not being able to identify the protocol and subsequently failing.

To interact with the image, modify the environment variables (as described below) in the `batchingBotEnv` file.

### Required keys

> NOTE: The key generation process described below will produce _sensitive data_. Make sure you properly store the keys, and generate them on a machine you trust.

Users must provide a wallet key in order to sign transactions.

You can generate a new key with `cardano-cli`. For example:

```sh
# Enter a nix shell with the cardano cli available
nix shell 'github:input-output-hk/cardano-node#cardano-cli'

# Generate signing and verification keys
cardano-cli address key-gen --signing-key-file key.skey --verification-key-file key.vkey

# Extract the address for initial funding of the wallet; choose either the mainnet or `preview` testnet via the last option.
cardano-cli address build --payment-verification-key-file key.vkey --out-file payment.addr [--mainnet | --testnet-magic 2]
```

> NOTE: Once generated the payment.addr must be funded in order to run the bot.

The environment variable expected by the docker image is then `SKEY`:

> NOTE: No environment variables require quotes. `cborHex_signing_key` and NOT `"cborHex_signing_key"`.

```
SKEY=cborHex_signing_key
```

### Optional services

CTL needs kupo, ogmios.
Note: docker must expose connections to them

```
OGMIOS_PORT=1337
OGMIOS_HOST=localhost
OGMIOS_SECURE=false
OGMIOS_PATH=

KUPO_PORT=1442
KUPO_HOST=localhost
KUPO_SECURE=false
KUPO_PATH=
```

> Note: if no services are provided the default Liqwid sevices will be used

### Optional params

### Verbosity

Set the level of logging for debugging.

```
VERBOSE=false
```

default: false (No debug logging)

### Market

Set the Market with which the bot should run on.

```
MARKET=DJED
```

default: Ada

> NOTE: When batching the Ada market a 30 minute pause window is enforced 15 minutes prior to every epoch boundary.

### Heartbeat

The batching bot exposes a heartbeat which will respond with the string "beat" if the service
is active.

```
HEARTBEAT_ADDR=127.0.0.1
HEARTBEAT_PORT=2001
```

default: `127.0.0.1`, port `2001`
