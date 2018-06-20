# ElectrumX - Reimplementation of electrum-server

For a future network with bigger blocks.

>   - Licence  
>     MIT
> 
>   - Language  
>     Python (\>= 3.6)
> 
>   - Author  
>     Neil Booth

## Getting Started

See [docs/HOWTO.rst](). There is also an
[installer](https://github.com/seci-coin/electrumx-installer) available
that simplifies the installation on various Linux-based distributions.
This is the preferred method.

## Features

  - Efficient, lightweight reimplementation of electrum-server
  - Fast synchronization of bitcoin mainnet from Genesis. Recent
    hardware should synchronize in well under 24 hours. The fastest time
    to height 448k (mid January 2017) reported is under 4h 30m. On the
    same hardware JElectrum would take around 4 days and electrum-server
    probably around 1 month.
  - The full Electrum protocol is implemented. The only exception is the
    blockchain.address.get\_proof RPC call, which is not used by
    Electrum GUI clients, and can only be invoked from the command line.
  - Various configurable means of controlling resource consumption and
    handling denial of service attacks. These include maximum connection
    counts, subscription limits per-connection and across all
    connections, maximum response size, per-session bandwidth limits,
    and session timeouts.
  - Minimal resource usage once caught up and serving clients; tracking
    the transaction mempool appears to be the most expensive part.
  - Fully asynchronous processing of new blocks, mempool updates, and
    client requests. Busy clients should not noticeably impede other
    clients' requests and notifications, nor the processing of incoming
    blocks and mempool updates.
  - Daemon failover. More than one daemon can be specified, and
    ElectrumX will failover round-robin style if the current one fails
    for any reason.
  - Peer discovery protocol removes need for IRC
  - Coin abstraction makes compatible altcoin and testnet support easy.

## Implementation

ElectrumX does not do any pruning or throwing away of history. I want to
retain this property for as long as it is feasible, and it appears
efficiently achievable for the forseeable future with plain Python.

The following all play a part in making ElectrumX very efficient as a
Python blockchain indexer:

  - aggressive caching and batching of DB writes
  - more compact and efficient representation of UTXOs, address index,
    and history. Electrum Server stores full transaction hash and height
    for each UTXO, and does the same in its pruned history. In contrast
    ElectrumX just stores the transaction number in the linear history
    of transactions. For at least another 5 years this transaction
    number will fit in a 4-byte integer, and when necessary expanding to
    5 or 6 bytes is trivial. ElectrumX can determine block height from a
    simple binary search of tx counts stored on disk. ElectrumX stores
    historical