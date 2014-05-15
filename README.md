node-cryptonote-pool
====================

High performance Node.js (with native C addons) mining pool for CryptoNote based coins such as Bytecoin and Monero.
Comes with lightweight example front-end script which uses the pool's AJAX API.



#### Table of Contents
* [Features](#features)
* [Under Development](#under-development)
* [Community Support](#community--support)
* [Usage](#usage)
  * [Requirements](#requirements)
  * [Downloading & Installing](#1-downloading--installing)
  * [Configuration](#2-configuration)
  * [Configure Easyminer](#3-options-configure-cryptonote-easy-miner-for-your-pool)
  * [Starting the Pool](#4-start-the-portal)
  * [Upgrading](#upgrading)
* [Setting up Testnet](#setting-up-testnet)
* [Donations](#donations)
* [Credits](#credits)
* [License](#license)


#### Features

* Variable difficulty / share limiter
* IP banning to prevent low-diff share attacks
* Socket flooding detection
* Payment processing
* Detailed logging
* Clustering for vertical scaling
* Live stats API (using CORS with AJAX and HTML5 EventSource)
  * Currency network/block difficulty
  * Current block height
  * Network hashrate
  * Pool hashrate
  * Each miners' hashrate
  * Blocks found (pending, confirmed, and orphaned)
  * Total paid out
* Light-weight front-end using API to display pool data


#### Under Development

* Worker login validation (make sure miners are using proper wallet addresses for mining)
* New tab showing blocks (pending, unlocked, and orphaned)
* More stats for individual worker (total shares submitted, total paid out)
* Ability to configure multiple ports - each with their own diff and vardiff settings

### Community / Support

* Support / general discussion join #monero: https://webchat.freenode.net/?channels=#monero
* Development discussion join #monero-dev: https://webchat.freenode.net/?channels=#monero-dev

Usage
===

#### Requirements
* Coin daemon(s) (find the coin's repo and build latest version from source)
* [Node.js](http://nodejs.org/) v0.10+ ([follow these installation instructions](https://github.com/joyent/node/wiki/Installing-Node.js-via-package-manager))
* [Redis](http://redis.io/) key-value store v2.6+ ([follow these instructions](http://redis.io/topics/quickstart))

##### Seriously
Those are legitimate requirements. If you use old versions of Node.js or Redis that may come with your system package manager then you will have problems. Follow the linked instructions to get the last stable versions.


#### 1) Downloading & Installing

[**Redis security warning**](http://redis.io/topics/security): be sure firewall access to redis - an easy way is to include `bind 127.0.0.1` in your `redis.conf` file

Clone the repository and run `npm update` for all the dependencies to be installed:

```bash
git clone https://github.com/zone117x/node-cryptonote-pool.git pool
cd pool
npm update
```

#### 2) Configuration

Explanation for each field:
```javascript
{
    /* Used for storage in redis so multiple coins can share the same redis instance. */
    "coin": "monero",

    "coinUnits": 100000000,
    "transferFee": 1000000,

    /* Port that simpleminer is pointed to. */
    "poolPort": 5555,

    /* Host that simpleminer is pointed to.  */
    "poolHost": "example.com",

    /* Contact email address. */
    "email": "support@cryppit.com",

    /* Address where block rewards go, and miner payments come from. */
    "poolAddress": "4AsBy39rpUMTmgTUARGq2bFQWhDhdQNekK5v4uaLU699NPAnx9CubEJ82AkvD5ScoAZNYRwBxybayainhyThHAZWCdKmPYn"

    /* Initial difficulty miners are set to. */
    "difficulty": 200,

    /* Variable difficulty is a feature that will automatically adjust difficulty for
       individual miners based on their hashrate in order to lower networking and CPU
       overhead. */
    "varDiff": {
        "minDiff": 2, //Minimum difficulty
        "maxDiff": 10000,
        "targetTime": 100, //Try to get 1 share per this many seconds
        "retargetTime": 15, //Check to see if we should retarget every this many seconds
        "variancePercent": 30 //Allow time to very this % from target without retargeting
    },

    /* Download link to cryptonote-easy-miner for Windows users. */
    "easyminerDownload": "https://github.com/zone117x/cryptonote-easy-miner/raw/master/CryptoNoteMiner/bin/Release/cryptnote-easy-miner-latest.zip",

    /* Set to "auto" by default which will spawn one process/fork/worker for each CPU
       core in your system. Each of these workers will run a separate instance of your pool(s),
       and the kernel will load balance miners using these forks. Optionally, the 'forks' field
       can be a number for how many forks will be spawned. */
    "clusterForks": "auto",

    /* Specifies the level of log output verbosity. Anything more severe than the level specified
       will also be logged. */
    "logLevel": "debug", //or "warn", "error"

    /* By default the pool logs to console and gives pretty colors. If you direct that output to a
       log file then disable this feature to avoid nasty characters in your log file. */
    "logColors": true,

    /* Poll RPC daemons for new blocks every this many milliseconds. */
    "blockRefreshInterval": 1000,

    /* How many seconds until we consider a miner disconnected. */
    "minerTimeout": 900,

    /* Only works with the new simpleminer with longpolling enabled. */
    "longPolling": {
        "enabled": true,
        "timeout": 8500
    },

    /* If under low-diff share attack we can ban their IP to reduce system/network load. */
    "banning": {
        "enabled": true,
        "time": 600, //How many seconds to ban worker for
        "invalidPercent": 25, //What percent of invalid shares triggers ban
        "checkThreshold": 30 //Perform check when this many shares have been submitted
    },

    /* Module that sends payments to miners according to their submitted shares. */
    "payments": {
        "enabled": true,
        "interval": 30, //how often to run in seconds
        "poolFee": 2, //2% pool fee
        "depth": 60 //block depth required to send payments (CRYPTONOTE_MINED_MONEY_UNLOCK_WINDOW)
    }

    /* AJAX/EventSource API used for front-end website. */
    "api": {
        "enabled": true,
        "hashrateWindow": 600, //how many second worth of shares used to estimate hash rate
        "updateInterval": 3, //gather stats and broadcast every this many seconds
        "port": 8117
    },

    /* Coin daemon connection details. */
    "daemon": {
        "host": "127.0.0.1",
        "port": 18081
    },

    /* Wallet daemon connection details. */
    "wallet": {
        "host": "127.0.0.1",
        "port": 8082
    },

    /* Redis connection into. */
    "redis": {
        "host": "127.0.0.1",
        "port": 6379
    }
}
```

#### 3) [Options] Configure cryptonote-easy-miner for your pool
Your miners that are Windows users can use [cryptonote-easy-miner](https://github.com/zone117x/cryptonote-easy-miner)
which will automatically generate their wallet address and stratup multiple threads of simpleminer. You can download
it and edit the `config.ini` file to point to your own pool.
Inside the `easyminer` folder, edit `config.init` to point to your pool details
```ini
pool_host=example.com
pool_port=5555
```

Rezip and upload to your server or a file host. Then change the `easyminerDownload` link in your `config.json` file to
point to your zip file.

#### 4) Start the pool

```bash
node init.js
```


#### 5) Host the front-end

Edit `index.html` to use your pool API configuration

```html
    <script>

        var api = 'http://poolhost:8117';

    </script>
```

Then simply serve the file via nginx, Apache, Google Drive, or anything that can host static content.


#### Upgrading
When updating to the latest code its important to not only `git pull` the latest from this repo, but to also update
the Node.js modules, and any config files that may have been changed.
* Inside your pool directory (where the init.js script is) do `git pull` to get the latest code.
* Remove the dependencies by deleting the `node_modules` directory with `rm -r node_modules`.
* Run `npm update` to force updating/reinstalling of the dependencies.
* Compare your `config.json` to the latest example ones in this repo or the ones in the setup instructions where each config field is explained. You may need to modify or add any new changes.

### Setting up Testnet

No cryptonote based coins have a testnet mode (yet) but you can effectively create a testnet with the following steps:

* Open `/src/p2p/net_node.inl` and remove lines with `ADD_HARDCODED_SEED_NODE` to prevent it from connecting to mainnet (Monero example: http://git.io/0a12_Q)
* Build the coin from source
* You now need to run two instance of the daemon and connect them to each other (without a connection to another instance the daemon will not accept RPC requests)
  * Run first instance with `./coind --p2p-bind-port 28080 --allow-local-ip`
  * Run second instance with `./coind --p2p-bind-port 5011 --rpc-bind-port 5010 --add-peer 0.0.0.0:28080 --allow-local-ip`
* You should now have a local testnet setup. The ports can be changes as long as the second instance is pointed to the first instance, obviously

*Credit to surfer43 for these instructions*

Donations
---------
* MRO: `asdf`
* BCN: `asdf`

Credits
===

* [LucasJones](//github.com/LucasJones) - Co-dev on this project; did tons of debugging for binary structures and fixing them. Pool couldn't have been made without him.
* [surfer43](//github.com/iamasupernova) - Did lots of testing during development to help figure out bugs and get them fixed
* [Wolf0](https://bitcointalk.org/index.php?action=profile;u=80740) - Helped try to deobfuscate some of the daemon code for getting a bug fixed
* [Tacotime](https://bitcointalk.org/index.php?action=profile;u=19270) - helping with figuring out certain problems and lead the bounty for this project's creation

License
-------
Released under the GNU General Public License v2

http://www.gnu.org/licenses/gpl-2.0.html