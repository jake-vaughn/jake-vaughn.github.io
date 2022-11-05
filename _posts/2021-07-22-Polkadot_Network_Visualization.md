---
title: Polkadot Network Visualization
date: 2021-07-22 8:00:00 +/-1111
author: Jake Vaughn
categories: [Projects]
tags: [project, portfolio, Polkadot, SQL, Database, Gephi]
---

![30k](/images/Polkadot_Network_Visualization/30k.svg)


## Overview:

A current project I am working on is called Visualizing-Substrate which can access database .db files and use RPC/API calls to substrate-based chains (currently Polkadot and Kusama) to create nodes and edges based on addresses, balances, and transaction data for the entire history of the chain. Addresses and transactions converted to nodes can be chosen based on current address holding and or transaction sizes. The project is currently written in typescript utilizing npm SQLite packages to access the database. Output from my program is designed to be used with Gephi (an open-source network analysis and visualization software) to visualize and organize the nodes and edges with layout algorithms such as the [Yifan Hu algorithm](http://yifanhu.net/PUB/graph_draw_small.pdf) an example of which above.

## Description
I came up with the idea out of an interest to identify connected accounts and be able to visualize where native tokens on the Kusama blockchain were and who had them. With this program it is easy to identify where assets originate on chain, where they are currently, and the path they took to get there. Using the community detection algorithm named Modularity also allows me to color nodes by groups depending on their connections. A good example of this is the green, orange, and teal color groups below that all interact with seemingly central nodes. These different colored groups are based on the exchanges that the nodes interact with.

![GroupingExample](/images/Polkadot_Network_Visualization/NodeGrouping2021-07-15.png)

The database that I used to create the nodes and edges in this project was created using [polka-store](https://github.com/TheGoldenEye/polka-store). Polka-store is a Node.js program written in typescript which scans a Polkadot chain (Polkadot/Kusama/Westend) and stores balance-relevant transactions in a SQLite database. This is needed because Polkadot itself does not directly store transaction data, and requires a API or block explorer to get transaction data which can be slow. With an SQLite database accessing transaction data is fast and can be done with SELECT cases. The database structure looks as follows.

|  Column            | Description                                               |
|:-------------------|:----------------------------------------------------------|
| chain              | chain name                                                |
| id                 | unique id                                                 |
| height             | bock height                                               |
| blockHash          | block hash                                                |
| type               | extrinsic method                                          |
| subType            | extrinsic submethod (e.g. in a 'utility.batch' extrinsic) |
| event              | the event triggered by the extrinsic                      |
| timestamp          | unix timestamp (in ms since Jan 1, 1970)                  |
| specVersion        | runtime version                                           |
| transactionVersion | transaction version                                       |
| authorId           | the account id of the block validator                     |
| senderId           | the account id of the block signer / transaction sender   |
| recipientId        | the account id of the transaction recipient               |
| amount             | the amount which was sent or rewarded                     |
| totalFee           | the fee which was paid by the block signer                |
| feeBalances        | the part of the totalFee that passed to the block author  |
| feeTreasury        | the part of the totalFee that passed to the treasury      |
| tip                | an additional tip paid by the block signer (is part of feeBalances)|
| success            | the transaction was successfull                           |

From this database I need the transactions where the event was a "balances.Transfer" event. From those transaction I then need the senderId, recipientId, amount, and height values to create my nodes and edges. To do this a created a function to get rows with the desired values using a SELECT FROM WHERE statement. This function also enables me to request transfer transactions after a certain block height and greater than a chosen value in Dot.

``` typescript
  // returns all rows with requested query in database
  GetRows(height: number, amount: number): any {
    const sql = `SELECT senderId,
                 recipientId,
                 amount,
                 height
                 FROM transactions
                 WHERE event = ?
                 AND height > ?
                 AND amount > ?`;

    const rows = db().query(sql, ["balances.Transfer"], height, amount);
    return rows;
  }
```

> Note: Staking rewards and other niche transfers of native assets are not tracked only balance transfers for now.

With the transaction data I am able to create CSV files for nodes and edges. In this case the nodes are nonrepeating senderIds and recipientIds also known as user addresses. For this project I also use API calls to a polkadot node to get the current balance of the address for each node. Node.csv file example:

``` csv

```


## Challenges
In making to project the biggest challenge I faced was being able to access blockchain data from the Polkadot RPC. As the concept was relatively new to me and examples were far and few between I found myself using Doc pages to build up an understanding of how to connect to endpoints and use different methods and queries to obtain the data that I needed.

## Future Ambitions
The current outcome of the project is a functioning software that allows me to track tokens and transactions and use Gephi to visualize the information. I have future plans to expand the functionality of the project as the substrate chains Polkadot and Kusama have gained a large amount of complexity with the addition of Parachains. To do so, I will most likely have to explore and/or create new databases with para-chain transaction data.


