# Setting up Development Environment

## Install Node.js

Install Node.js by your favorite method, or use Node Version Manager by following directions at https://github.com/creationix/nvm

```bash
nvm install v4
```

## Fork and Download Repositories

To develop bitcore-node:

```bash
cd ~
git clone git@github.com:<yourusername>/bitcore-node.git
git clone git@github.com:<yourusername>/bitcore-lib.git
```

To develop vtorrent or to compile from source:

```bash
git clone git@github.com:<yourusername>/vtorrent.git
git fetch origin <branchname>:<branchname>
git checkout <branchname>
```
**Note**: See bitcoin documentation for building bitcoin on your platform.


## Install Development Dependencies

For Ubuntu:
```bash
sudo apt-get install libzmq3-dev
sudo apt-get install build-essential
```
**Note**: Make sure that libzmq-dev is not installed, it should be removed when installing libzmq3-dev.


For Mac OS X:
```bash
brew install zeromq
```

## Install and Symlink

```bash
cd bitcore-lib
npm install
cd ../bitcore-node
npm install
```
**Note**: If you get a message about not being able to download bitcoin distribution, you'll need to compile vtorrentd from source, and setup your configuration to use that version.


We now will setup symlinks in `bitcore-node` *(repeat this for any other modules you're planning on developing)*:
```bash
cd node_modules
rm -rf bitcore-lib
ln -s ~/bitcore-lib
rm -rf bitcoind-rpc
ln -s ~/bitcoind-rpc
```

And if you're compiling or developing bitcoin:
```bash
cd ../bin
ln -sf ~/vtorrent/src/vtorrentd
```

## Run Tests

If you do not already have mocha installed:
```bash
npm install mocha -g
```

To run all test suites:
```bash
cd bitcore-node
npm run regtest
npm run test
```

To run a specific unit test in watch mode:
```bash
mocha -w -R spec test/services/vtorrentd.unit.js
```

To run a specific regtest:
```bash
mocha -R spec regtest/vtorrentd.js
```

## Running a Development Node

To test running the node, you can setup a configuration that will specify development versions of all of the services:

```bash
cd ~
mkdir devnode
cd devnode
mkdir node_modules
touch bitcore-node.json
touch package.json
```

Edit `bitcore-node.json` with something similar to:
```json
{
  "network": "livenet",
  "port": 3001,
  "services": [
    "vtorrentd",
    "web",
    "insight-api",
    "insight-ui",
    "<additional_service>"
  ],
  "servicesConfig": {
    "vtorrentd": {
      "spawn": {
        "datadir": "/home/<youruser>/.vtorrent",
        "exec": "/home/<youruser>/vtorrent/src/vtorrentd"
      }
    }
  }
}
```

**Note**: To install services [insight-api](https://github.com/vtorrent/insight-api) and [insight-ui](https://github.com/vtorrent/insight-ui) you'll need to clone the repositories locally.

Setup symlinks for all of the services and dependencies:

```bash
cd node_modules
ln -s ~/bitcore-lib
ln -s ~/bitcore-node
ln -s ~/insight-api
ln -s ~/insight-ui
```

Make sure that the `<datadir>/vtorrent.conf` has the necessary settings, for example:
```
server=1
whitelist=127.0.0.1
txindex=1
addressindex=1
timestampindex=1
spentindex=1
zmqpubrawtx=tcp://127.0.0.1:28332
zmqpubhashblock=tcp://127.0.0.1:28332
rpcallowip=127.0.0.1
rpcuser=vtorrent
rpcpassword=local321
```

From within the `devnode` directory with the configuration file, start the node:
```bash
../bitcore-node/bin/bitcore-node start
```