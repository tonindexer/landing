# Anton

Written in Go, Anton is an open source [The Open Network](https://ton.org) blockchain indexer available under the [Apache Licence 2.0](https://github.com/tonindexer/anton/blob/master/LICENSE).
Anton is designed to provide a scalable and flexible solution for developers to access and analyze blockchain data.

## Overview

Our project is building an indexer for the TON blockchain that gathers and analyzes data from contracts and tokens, providing insights into the network's activity. 
Our goal is to help developers and users understand how the blockchain is being used, as well as make it possible for developers to add their own contracts with their own message schemas to our explorer.

If you want to hack it, go to [GitHub](https://github.com/tonindexer/anton).

If you want to use it, go to [Swagger API documentation](https://anton.tools/api/v0/swagger) and [API query examples](https://github.com/tonindexer/anton/blob/main/docs/API.md).

If you want to look at the data, go to [Apache Superset](https://superset.anton.tools).

## How does it work?

Before you start, take a look at the [official docs](https://ton.org/docs/learn/overviews/ton-blockchain).

Consider an arbitrary contract.
It has a state that is updated with any transaction on the contract's account.
Each state has the contract code and data.
The contract data can be complex, but developers typically provide [get-methods](https://ton.org/docs/develop/func/functions#specifiers) in the contract, which can be executed to retrieve the necessary data.
The TON has standard contracts (such as [TEP-62](https://github.com/ton-blockchain/TEPs/blob/master/text/0062-nft-standard.md), [TEP-74](https://github.com/ton-blockchain/TEPs/blob/master/text/0074-jettons-standard.md)), and they have predefined get-method names.
Therefore, you can attempt to match accounts found in the network to these standards by checking the presence of the get-methods.
Contract standards also specify [TL-B constructor tags](https://ton.org/docs/learn/overviews/tl-b-language#constructors) (or operation ids) for each acceptable message to contract, defined as the first 32 bits of the parsed message payload cell.
So you if you know standard of a given contract, you can determine the type of message to it (for example, NFT item transfer) by parsing the first 32 bits of message body.

Anton allows you to define the contract interface in just one JSON schema.
Format of every schema is described in detail in [abi/README.md](abi/README.md).
Every schema comprises contract get-methods, as well as incoming and outgoing message schemas for the contract.
Once contract interfaces are defined and stored in the database, Anton begins scanning new blocks on the network.
The tool stores every account state, transaction, and message in the database.
For get-methods without arguments in the contract interface, Anton emulates these methods and saves the returned values to the database.
When a message is sent to a known contract interface, Anton attempts to match the message to a known schema by comparing the parsed operation ID.
If the message is successfully parsed using the identified schema, Anton also stores the parsed data.

To explore contract interfaces known to this project, visit the [abi/known](/abi/known) directory.
This will provide you with an understanding of the various contract interfaces already supported and serve as examples for adding your own.
