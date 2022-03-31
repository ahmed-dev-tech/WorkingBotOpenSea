
## Some tips to successfully run automatic operations on OpenSea
- Have an OpenSea [API key][api-key-link].
- Use proxies. In case you don't have a good private proxy, Tor is currently better than any bad or public proxy.
- Specify Cookies and User-Agent.
- Specify random delays.
- You can run multiple bot instances simultaneously with different configurations.
- Don't do too many requests in a short time.

You can use this bot alongside with [tor-multiproxy][tor-multiproxy-link].

Be very precautious with automatic trading!
Have a separate account with small balance for testing unknown bots and services.

## Configuration

### Required settings
- `network` - network name (use `mainnet` or `rinkeby`).
- `infura_key` or `alchemy_key` - Infura or Alchemy node API key.
- `mnemonic` or `private_keys` - MetaMask mnemonic phrase or array with private keys.
- `wallet_address` - buyer wallet address.
- `opensea_key` - OpenSea API key.

### Optional settings
- `expiration` - expiration time for offer in hours. Default: `24`.
- `discard_threshold` - how much consecutive fails to discard an asset. Default: `10`.
- `price_auto` - enable auto price calculation. Default: `true`.
  - `price_floor` - minimum price in `wETH`. Default: `0.0001`.
  - `price_roof` - maximum price in `wETH`. Default: `1`.
  - `price_epsilon` - price increment in `wETH`. Default: `0.0001`.
  - `price_increment_factor`- minimum price increment factor. Default: `0.1` (10%).
- Delay options:
  - `delay` - delay between buy orders in milliseconds, *including processing time*. Default: `5000`.
  - `random_factor` - additional random delay factor. Default: `0.5`.
  - `random_delay` - additional random delay roof in milliseconds. Default: `0`.
  - Actual delay between offers will be in range:
    - from `delay`;
    - to `delay * (1 + random_factor) + random_delay`.
  - `process_timeout` - timeout for SDK calls in milliseconds. Default: `10000`.
- Skipping options:
  - `skip_if_have_bid` - skip offer duplicates. Default: `true`.
  - `skip_if_too_low` - skip an asset if the floor price is higher than the current highest bid. Default: `false`.
  - `skip_if_too_high` - skip an asset if the roof price is lower than the current highest bid. Default: `true`.
  - `skip_if_owner_is_buyer` - skip an asset if you already own it. Default: `true`.
  - `skip_if_order_created` - skip an asset if an error occured but order was created. Default: `false`.
- Logging options:
  - `log_opensea` - print OpenSea log messages. Default: `false`.
  - `log_fetch` - log fetch calls. Default: `false`.
    - `log_fetch_all` - log fetch headers and body. Default: `false`.
  - `log_full` - log full error messages. Default: `false`.
- Proxy settings:
  - `proxy_list` - proxies list file. No proxy by default.
  - `proxy_protocol` - proxy protocol. Should be `http://` or `socks://`. Default: `http://`.
  - `proxy_checking` - enable proxy checking. Default: `true`.
  - `switch_threshold` - how much fails to switch proxy. Default: `10`.
  - `switch_delay` - proxy switching delay in milliseconds. Default: `2000`.
- HTTP request options:
  - `cookie` - Cookie data. No Cookie by default.
  - `user_agent` - User-Agent data. No User-Agent by default.
  - `fetch_timeout` - fetch request timeout in milliseconds. Default: `10000`.
  - `cache_timeout` - fetch cache timeout in milliseconds. Default: `60000`.
  - `clear_cache_threshold` - how much consecutive fails to clear cache. Default: `5`.

Values `floor`, `roof`, `epsilon` for price calculation will be taken from the assets list file if specified, or from the config otherwise.

If auto price calculation is disabled, only the `floor` value will be used to create a buy order.

Default config file: `config.json`.

**Example**
```json
{
  "network":        "rinkeby",
  "infura_key":     "<your Infura API key>",
  "private_keys":   [ "<your private key>" ],
  "wallet_address": "<your wallet address>",

  "price_auto":     true,
  "price_floor":    0.001,
  "price_roof":     100,
  "price_epsilon":  0.001,

  "expiration":     4,

  "delay":          500
}
```

## Assets list file
Each line contains a link to the asset on OpenSea and floor, roof, epsilon price values in `wETH` as floating-point numbers.
If there is no value specified, it will be taken from the config. First number is `floor`, second is `roof` and last is `epsilon`.

Asset line notation: `<link to asset> [floor price] [roof price] [epsilon]`

Default list file: `list.txt`.

**Example**
```
https://testnets.opensea.io/assets/0x08a62684d8d609dcc7cfb0664cf9aabec86504e5/100 0.01 100
https://testnets.opensea.io/assets/0x08a62684d8d609dcc7cfb0664cf9aabec86504e5/200
https://testnets.opensea.io/assets/0x08a62684d8d609dcc7cfb0664cf9aabec86504e5/300 0.2 200 0.1
```

## Proxies list file
Available proxy notation:
- `protocol://user:pass@host:port`
- `protocol://host:port:user:pass`

Available protocols:
- `http://` - for HTTP proxy.
- `socks://` - for SOCKS 4/5 proxy.

Host should be an **IPv4** address.
Protocol can be omitted, in which case the value `proxy_protocol` from the configuration will be used.

**Example**
```
socks://127.0.0.1:9050
127.0.0.1:8080
```

## Command line arguments
- `--file=<file name>` - assets list file. Default: `list.txt`.
- `--output=<file name>` - output log file. Default: `log.txt`.
- `--verbose` - print all messages to the console.
- `--config=<file name>` - config file. Default: `config.json`.
  - You can override some config options with these arguments:
  - `--api_key=<API key>` - OpenSea API key.
  - `--wallet=<address>` - buyer wallet address.
  - `--proxy=<file name>` - proxies list file.
  - `--log_opensea` - print OpenSea log messages.
  - `--log_fetch` - log fetch calls.
  - `--log_fetch_all` - log fetch headers and body.
  - `--log_full` - log full error messages.
- `--resume=<line>` - resume progress from specified line.
- `--stop` - properly stop currently running bot instance.

## Auto price calculation formula
For each asset:

`bid price = max(floor, min(max(highest bid + epsilon, highest bid * (1 + increment factor)), roof)`
- `highest bid` is the current highest bid price for the asset.
- `floor`, `roof`, `epsilon` values taken from assets list file if specified, or from config otherwise.
- `increment factor` value taken from config.

## Usage
You should have an Infura or Alchemy API key, an OpenSea API key, an OpenSea account and a MetaMask account.

Make sure to have installed recent version of **Node.js**
with **Git**, **Python** and **C/C++** build tools (**npm** may require this to install dependencies).
- Install the package.
- Create a config file.
- Run `offers.js`.

**Example**
```shell
npm install
node offers.js --config=config.json --file=list.txt
```

## Troubleshooting
If the bot don't work with recent Node.js version, try to use **v16**.
You can use **NVM** to easily switch versions.

If you getting error `0308010C:digital envelope routines::unsupported`,
you should use Node.js **v16** or set environment variable `NODE_OPTIONS=--openssl-legacy-provider`.
See also - https://github.com/webpack/webpack/issues/14532

If you getting error `FetchError: request to ... failed, reason: certificate has expired`,
and you are using an old Node.js version, set environment variable `NODE_TLS_REJECT_UNAUTHORIZED=0`.

[api-key-link]:         https://docs.opensea.io/reference/request-an-api-key
