# Running a DebitMyData Electrum server

These steps run ElectrumX for the DebitMyData chain on an Ubuntu 24.04 server of your own, next to
its DebitMyData node. Below, `electrum.example.com` stands for your server's name.

**Install it from this repository.** `pip install e-x` gives upstream ElectrumX, which does not know
the DebitMyData chain.

## What you need

- **An Ubuntu 24.04 server** (x86_64): 2 vCPU, 4 GB of memory, an SSD with room for the node's
  chain and the index (40 GB is plenty today). Root access with an SSH key.
- **A name** pointing to its IP address (for example `electrum.example.com`), **DNS only**: wallets
  connect to it directly, not through a web proxy.

## Install

1. **The DebitMyData node.** Download DebitMyData Core's Linux release from
   [GitHub](https://github.com/DebitMyData-PoW/DebitMyData-PoW-Core/releases) and check it: the
   SHA-256 that `sha256sum` prints must be the one published for that release. For a newer
   version, change `1.3.5` everywhere below:

   ```
   cd /tmp
   curl -LO https://github.com/DebitMyData-PoW/DebitMyData-PoW-Core/releases/download/v1.3.5/debitmydata-1.3.5-x86_64-linux-gnu.tar.gz
   sha256sum debitmydata-1.3.5-x86_64-linux-gnu.tar.gz
   tar -xzf debitmydata-1.3.5-x86_64-linux-gnu.tar.gz
   sudo install -m 755 debitmydata-1.3.5-x86_64-linux-gnu/debitmydatad debitmydata-1.3.5-x86_64-linux-gnu/debitmydata-cli /usr/local/bin/
   rm -rf debitmydata-1.3.5-x86_64-linux-gnu*
   ```

   ElectrumX needs the node's **transaction index**, and its RPC on this server only. In the node's
   `debitmydata.conf` (in its data folder, `~/.debitmydata` by default), with a user name and a
   long random password of your own:

   ```
   server=1
   txindex=1
   rpcbind=127.0.0.1
   rpcallowip=127.0.0.1
   rpcuser=electrumx
   rpcpassword=<a long random password>
   ```

   Start the node (`debitmydatad -daemon`) and let it sync: `debitmydata-cli getblockchaininfo`
   shows its progress. A node that already ran without `txindex=1` needs one start with `-reindex`.

2. **ElectrumX**, from this repository, in its own folder and under its own user:

   ```
   sudo apt-get install -y git python3-venv python3-dev build-essential libleveldb-dev
   sudo useradd --system --create-home --home-dir /var/lib/electrumx electrumx
   sudo git clone https://github.com/DebitMyData-PoW/electrumx.git /opt/electrumx
   sudo python3 -m venv /opt/electrumx/.venv
   sudo /opt/electrumx/.venv/bin/pip install '/opt/electrumx[leveldb]'
   ```

3. **A certificate for SSL.** Wallets accept a self-signed one (they remember it):

   ```
   sudo -u electrumx openssl req -newkey rsa:2048 -sha256 -nodes -x509 -days 3650 \
       -subj "/CN=electrum.example.com" \
       -keyout /var/lib/electrumx/server.key -out /var/lib/electrumx/server.crt
   ```

4. **The settings**, in `/etc/electrumx.conf` (`sudo chmod 600 /etc/electrumx.conf`), with the
   node's user and password from step 1:

   ```
   COIN = DebitMyData
   DB_ENGINE = leveldb
   DB_DIRECTORY = /var/lib/electrumx/db
   DAEMON_URL = http://electrumx:<the node's RPC password>@127.0.0.1:18557/
   SERVICES = tcp://:50001,ssl://:50002,rpc://127.0.0.1:8000
   SSL_CERTFILE = /var/lib/electrumx/server.crt
   SSL_KEYFILE = /var/lib/electrumx/server.key
   REPORT_SERVICES = tcp://electrum.example.com:50001,ssl://electrum.example.com:50002
   ```

   Ports 50001 (TCP) and 50002 (SSL) are the ones the DebitMyData Electrum wallet expects.
   `REPORT_SERVICES` is how other servers learn about yours; leave it out to keep the server
   private. Every other setting is in [docs/environment.rst](docs/environment.rst).

5. **The service.** Make the database folder, then start ElectrumX with the server:

   ```
   sudo -u electrumx mkdir -p /var/lib/electrumx/db
   sudo sed -e 's#/usr/local/bin/#/opt/electrumx/.venv/bin/#g' \
       /opt/electrumx/contrib/systemd/electrumx.service | sudo tee /etc/systemd/system/electrumx.service
   sudo systemctl daemon-reload
   sudo systemctl enable --now electrumx
   ```

6. **The firewall.** Open the wallets' ports, with SSH:

   ```
   sudo ufw allow OpenSSH
   sudo ufw allow 50001/tcp
   sudo ufw allow 50002/tcp
   sudo ufw enable
   ```

7. **Check.** The first time, ElectrumX indexes the whole chain (from minutes to an hour):

   ```
   journalctl -u electrumx -f
   /opt/electrumx/.venv/bin/electrumx_rpc getinfo
   ```

   Once it says it is caught up, connect a DebitMyData Electrum wallet to it: **Network**, then the
   server `electrum.example.com:50002:s` (SSL), or `:50001:t` (TCP).

## Updating

```
cd /opt/electrumx && sudo git pull
sudo /opt/electrumx/.venv/bin/pip install '/opt/electrumx[leveldb]'
sudo systemctl restart electrumx
```

Read the [changelog](docs/changelog.rst) first: a new version may index the chain again.

## Development

```
python3 -m venv .venv && .venv/bin/pip install -e '.[leveldb]' pytest
.venv/bin/pytest tests
```
