# ElectrumX for DebitMyData

The server side of the DebitMyData Electrum wallet. It indexes the DebitMyData (DMD) chain from a
full node, so that lightweight wallets can find their addresses' history and balances in an
instant, without downloading the chain. The servers the
[DebitMyData Electrum wallet](https://github.com/DebitMyData-PoW/electrum) connects to run it.

It is a fork of [ElectrumX](https://github.com/spesmilo/electrumx), the Electrum server most of
the public Electrum network runs, kept up to date with it, with the DebitMyData chain added
(`DebitMyData` in [`src/electrumx/lib/coins.py`](src/electrumx/lib/coins.py): its addresses,
extended keys, genesis block and node port).

## What it does

- **Indexes every address** of the chain, from a DebitMyData Core node, into its own database.
- **Answers wallets** over TCP, SSL and WebSocket, with the Electrum protocol: balances, history,
  headers, fee estimates, and broadcasting their transactions.
- **Joins other servers** through peer discovery, if its operator chooses.
- Holds **no keys and no coins**: wallets sign their transactions themselves.

## Installation

Running a DebitMyData Electrum server, from the node to the wallets connecting to it: see
**[INSTALL.md](INSTALL.md)**. ElectrumX's own documentation, with every setting, is in
[docs/](docs/) and on [readthedocs](https://electrumx-spesmilo.readthedocs.io).

## License

MIT ([LICENCE](LICENCE)). ElectrumX was written by Neil Booth and is maintained by the Electrum
developers.
