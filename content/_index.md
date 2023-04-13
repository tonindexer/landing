# anton

The project fetches data from the TON blockchain and puts it in PostgreSQL and ClickHouse databases.

## Overview

Our project is building an indexer for the TON blockchain that gathers and analyzes data from contracts and tokens, providing insights into the network's activity. 
Our goal is to help developers and users understand how the blockchain is being used, as well as make it possible for developers to add their own contracts with their own message schemas to our explorer.

If you want to try it, go to [GitHub](https://github.com/tonindexer/anton), [Swagger API documentation](https://anton.tools/api/v0/swagger) and [API query examples](https://github.com/tonindexer/anton/blob/main/docs/API.md).

## How does it work?

Before you start, take a look at the [official docs](https://ton.org/docs/learn/overviews/ton-blockchain).

Consider an arbitrary contract.
It has a state that is updated with any transaction on the contract's account.
This state contains the contract data.
The contract data can be complex,
but developers usually provide [get-methods](https://ton.org/docs/develop/func/functions#specifiers) in the contract.
You can retrieve data by executing these methods and possibly passing them arguments.
By parsing the contract code, you can check any contract for an arbitrary get-method (identified by function name).

TON has some standard tokens, such as
[TEP-62](https://github.com/ton-blockchain/TEPs/blob/master/text/0062-nft-standard.md),
[TEP-74](https://github.com/ton-blockchain/TEPs/blob/master/text/0074-jettons-standard.md).
Standard contracts have predefined get-method names and various types of acceptable incoming messages,
each with a different payload schema.
Standards also specify [tags](https://ton.org/docs/learn/overviews/tl-b-language#constructors) (or operation ids)
as the first 32 bits of the parsed message payload cell.
Therefore, you can attempt to match accounts found in the network to the standards by checking the presence of the get-methods and
matching found messages to these accounts by parsing the first 32 bits of the message payload.

For example, look at NFT standard tokens, which can be found [here](https://github.com/ton-blockchain/token-contract).
NFT item contract has one `get_nft_data` get method and two incoming [messages](https://github.com/ton-blockchain/token-contract/blob/main/nft/op-codes.fc)
(`transfer` with an operation id = `0x5fcc3d14`, `get_static_data` with an operation id = `0x2fcb26a2`).
Transfer payload has the following [schema](https://github.com/xssnick/tonutils-go/blob/master/ton/nft/item.go#L14).
If an arbitrary contract has a `get_nft_data` method, we can parse the operation id of messages sent to and from this contract.
If the operation id matches a known id, such as `0x5fcc3d14`, we attempt to parse the message data using the known schema
(new owner of NFT in the given example).
