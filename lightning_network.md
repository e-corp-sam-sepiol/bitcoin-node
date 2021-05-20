# 🌩️ Lightning Network w/ myNode 

The Lightning Network is a decentralized system for instant, high-volume micropayments that removes the risk of delegating custody of funds to trusted third parties.

Funds are placed into a two-party, multisignature "channel" bitcoin address. This channel is represented as an entry on the bitcoin public ledger. In order to spend funds from the channel, both parties must agree on the new balance. The current balance is stored as the most recent transaction signed by both parties, spending from the channel address. To make a payment, both parties sign a new exit transaction spending from the channel address. All old exit transactions are invalidated by doing so. 

The Lightning Network does not require cooperation from the counterparty to exit the channel.  Both parties have the option to unilaterally close the channel, ending their relationship. Since all parties have multiple multisignature channels with many different users on this network, one can send a payment to any other party across this network.

By embedding the payment conditional upon knowledge of a secure cryptographic hash, payments can be made across a network of channels without the need for any party to have unilateral custodial ownership of funds. The Lightning Network enables what was previously not possible with trusted financial systems vulnerable to monopolies—without the need for custodial trust and ownership, participation on the network can be dynamic and open for all.

## Helpful Resources To Get You Started

* [http://lightning.network/docs/](http://lightning.network/docs/ "http://lightning.network/docs/")

* [https://www.ellemouton.com/blog](https://www.ellemouton.com/blog "https://www.ellemouton.com/blog")

* [Bitcoin Kindergarten - #82 Lightning Routing w/beeforbitcoin1](https://youtu.be/qnj-ix45tVw?t=137)

@beeforbitcoin1 gives a presentation on the steps someone who is trying to setup their own Bitcoin Lightning routing node should consider and use as a guide.

* [BTC Sessions - How To Run A Bitcoin Lightning Network Node](https://youtu.be/KItleddMYFU?t=265)

BTC Sessions gives a presentation and tutorial on how to run your Bitcoin Lightning node using the Umbrel software stack, however you can easily perform the same tasks shown in the video in myNode. 

------------

# Getting Started With The Lightning Network
#### 1. [Create Lightning Wallet](https://mynodebtc.github.io/lightning/create.html "Create Lightning Wallet")
Create a lightning wallet and initalize the 	`lnd` daemon, syncing your node to the lightning network.
#### 2. [Ride the Lightning](https://mynodebtc.github.io/lightning/rtl.html#usage "Ride the Lightning")
Ride the Lightning is a Lightning wallet and node management tool accessible via a web interface, and is built into myNode.
You can use RTL from any browser that is able to access your myNode installation. Useful for graphical management and monitoring of your lightning node.
#### 3. [ThunderHub](https://mynodebtc.github.io/lightning/thunderhub.html#introduction "Thunderhub")
ThunderHub is an open-source LND node manager to monitor your node and manage channels via a web-interface. It allows you to take control of the Lightning network with a simple and intuitive UX. Useful for graphical management and monitoring of your lightning node.
### 4. [Creating a Channel](https://www.ellemouton.com/blog/view/3)
In order to send or recieve payments over the lightning network, you must create channel(s) with peers on the lightning network. Without active channels, you cannot route payments to others, and no one can route a payment to you. More information regarding peers, channels and channel sizes is covered below.

------------

# Lightning Node Management - Overview

### Peers & Channels
- Peers are Lightning nodes which are connected to each other over the internet (TCP/IP).
- Channel(s) are a payment channel, established between two peers on the Lightning Network.

### Receiving Payments
Receiving incoming lightning payments on your node requires the following:

- An active channel with a well connected node on the Lightning Network. 
- Inbound Liquidity, i.e active channel(s) that have a remote balance. This means that there needs to be satoshis on the remote side of an open channel. The size of the remote balance affects the size of lightning payments your node can recieve.  The larger the remote balance, the greater amount of satoshis you can recieve in payments. 

The maximum amount of incoming payment you can recieve is determined by the highest inbound liquidity of a single channel. 

### Channel Sizes
As the price of Bitcoin increases overtime, a smaller amount of satoshis represents a greater amount of value, thus these general guidelines may change.

- It is recommended to avoid opening channels below `200,000` - `500,000` satoshis. I tend not to open channels smaller than `1,000,000` satoshis, and no larger than `3,000,000` satoshis.
- If a channel is too small, it may result in the inability to close that channel when `on-chain` fees are high. This can leave a channel vulnerable when a counterparty tries to close with a previous channel state.
- The maximum size of payments made or routed is limited by the highest directional liquidity of a single active channel.
- One big channel to a well connected and stable node (high uptime) is much more useful to the Lightning Network than many small ones. Allocate your liquidity with forethought, and a desire to build useful links in the Lightning Network. Creating these useful links (payment paths) between nodes is what sets up your node to be a desirable payment path in the future as the network recognizes your nodes stablity and maturity. 
- Consider opening channels with nodes who's operator can be contacted in case of a problem. Channels with large amounts of local and remote liquidity, established with stable (high uptime), well connected nodes are the goal.

### Opening Channels & On-Chain Bitcoin Fees

- Opening or closing a lightning channel is an `on-chain` transaction.

- The time it takes to open or close a channel depends on the congestion of the bitcoin mempool, and the sats/byte fee used when opening or closing that channel. [mempool.space - Bitcoin Mempool & Fee Estimates](https://mempool.space).

### Tor Nodes
The Tor (The Onion Router) network is designed to conceal the participant's IP address and geographical location. 

- Lightning Network nodes who's traffic is routed through Tor can open a channel with any node.

- Lightning Network nodes that route their traffic through the "clearnet" are not able to initate opening a channel with a node behind Tor if they are not already added as a peer to the node running behind Tor. Where as, the Tor node can initate the opening of a channel with a clearnet node. 

### Routing Payments
A conceptual illustration of how payment routing works on lightning.

- Consider a lightning node `B` in a serial connection of nodes `A` `<->` `B` `<->` `C`
- If node `A` wants to make a payment to node `C`, one hop will be required through node `B` to make that payment. 
- Node `A` pays node `B` the amount of satoshis it wants to send to node `C`. Node `B` then pushes the same amount of satoshis it recieved from node `A` to node `C`.
- The capacity of channels never changes, only moves from one side of a channel to the other and vice versa. 
- The payment process is all or nothing, meaning there is no risk of a payment getting stuck in the network on route to it's destination.

### Lightning Network Routing Fees

`On-Chain` Bitcoin transaction fees are dictated by a free market demand of desired settlement times. If you want your payment to be included in the next mined block, a fee larger than the majority of transactions in the mempool will help ensure your transaction is included in the next block and is settled quickly. 

`Off-Chain` Bitcoin transactions on the Lightning Network correspond to the amount of satoshis that are being routed in a payment. When a payment is routed, the fee is calculated using the following components:

- Base Fee: The default base fee is `1,000` millisat, meaning `1` satoshi fee per every routed payment. 

- Proportional Fee: The default proportional fee is `.000001`, meaning there is an additional `1` satoshi charged for every million satoshis routed in the payment.

- There are no Lightning Network fees for payments made in a direct channel between two connected peers. 

Avoid setting a zero routing fee, associating a zero cost of routing payments through your node can leave you open for liquidity Denial of Service attacks on your node, quickly draining the balance of your local side of channel(s). 

When you open new channel(s), leave the fee rates at their default value. Observe their activity over the course of a month or two, and identify which channels are being used for routing to help inform your decisions going forward in respect to the fees you set for each channel. 

### Finding Peers To Connect With
Opening your first channel on the Lightning Network should prioritize network reach, by selecting a very well connected node. This ensures we will be able to route payments to as many nodes as possible on the network with our first channel.

We should also avoid connecting with an exchanges' node at first while we learn to operate a lightning node. Due to the nature of an exchange, they can quickly imbalance a channel.  

- [1ML - Lightning Nodes - Top Network Reach](https://1ml.com/node?order=nodeconnectednodecount)

 1ML is a handy Lightning Network directory of nodes, which grades nodes by their capacity, number of open channels, age, growth metrics, and availability (uptime). The smaller the rankings are for a node listed on 1ML, the better it is.

- [Moneni - Lightning Node Match](https://moneni.com/mcb/nodematch)

 Lightning Node Match is another handy tool which searches the network for public nodes which would give you the best additional reach, taking into consideration the channel(s) you already have open. This tool does not consider capacity when recommending nodes and prioritizes network reach only (maximizing the number of nodes reached in a minimal number of hops).
 
- [lnrouter.app/graph](https://lnrouter.app/graph)

 lnrouter.app provides a graph of the lightning network that can help visualize the connectivity between peers. 

[Lightning Node Management - OpenOMS](https://openoms.gitbook.io/lightning-node-management/)

------------

## Payment Routing
#### [Routing vs. Path Finding](https://github.com/lnbook/lnbook/blob/develop/routing.asciidoc#routing-vs-path-finding "Routing vs. Path Finding")
Path Finding, which is covered in [[path_finding]](https://github.com/lnbook/lnbook/blob/develop/routing.asciidoc#path_finding "[path_finding]") is the process of finding and choosing a contiguous path made of payment channels which connects the sender `A` to the recipient `B`. The sender of a payment does the path finding, by examining the channel graph which they have assembled from channel announcements gossiped by other nodes.

Routing refers to the series of interactions across the network that allow a payment to flow from A to B, across the path previously selected by path finding. Routing is the active process of sending a payment on a path, which involves the cooperation of all the intermediary nodes along that path.

An important rule of thumb is that it is possible for a path to exist between `Alice` and `Bob`, yet there may not be an active route on which to send the payment.

- [Exploring Lightning Network Routing](https://blog.lightning.engineering/posts/2018/05/30/routing.html "Exploring Lightning Network Routing")

Due to the use of [onion routing](https://github.com/lnbook/lnbook/blob/develop/routing.asciidoc#routing-a-payment "onion routing"), intermediary nodes are only explicitly aware of the one node preceding them and the one node following them in the route. They will not necessarily know who is the sender and recipient of the payment. This enables fans to use intermediary nodes to pay Dina, without leaking private information and without risking theft.

-----------------

## 🔑 Balance of Satoshis

[Balance of Satoshis](https://github.com/alexbosworth/balanceofsatoshis "Balance of Satoshis"), written by Alex Bosworth, can help us simulate lightning network payments without actually paying anything. This can help us probe the network, and acquire information regarding our capability of routing payments to others and ourselves.

*This section assumes you have active channel(s) open with peers.*

As of myNode version v0.2.33, users are able to install Balance of Satoshis via the "Applications" manager on the home page of mynode.local

### Help File
```
bos help
```

[https://github.com/alexbosworth/balanceofsatoshis](https://github.com/alexbosworth/balanceofsatoshis "https://github.com/alexbosworth/balanceofsatoshis")

------------

# ⚖️ Managing Channel Balances 
[![channel_1](https://i.imgur.com/HQI3THY.png "channel_1")](https://i.imgur.com/HQI3THY.png "channel_1")

When a new channel is opened, it is likely that the complete balance will remain on the local side of the channel. This means that we can send satoshis, but cannot recieve any satoshis to `channel_1`.

Routing transactions for the lightning network will require fairly balanced channels on your node. This ensures throughput can be sent and recieved via all of the active channels.

There are two options to help you manage your lightning channel balance(s) addressed below:

* 1.) Acquiring inbound liquidity to your channel(s)

* 2.) Using Balance of Satoshis to circularly rebalance your channels. 

## 1. Acquiring Inbound Liquidity

### Loop (Submarine Swaps)
#### [https://github.com/lightninglabs/loop](https://github.com/lightninglabs/loop "https://github.com/lightninglabs/loop")
Lightning Loop is a non-custodial service offered by [Lightning Labs](https://lightning.engineering/) to bridge on-chain and off-chain Bitcoin using submarine swaps. This repository is home to the Loop client and depends on the Lightning Network daemon [lnd](https://github.com/lightningnetwork/lnd). All of LNDs supported chain backends are fully supported when using the Loop client: Neutrino, Bitcoin Core, and btcd.

-  [A Closer Look at Submarine Swaps in the Lightning Network](https://blog.muun.com/a-closer-look-at-submarine-swaps-in-the-lightning-network/ "A Closer Look at Submarine Swaps in the Lightning Network")

In the current iteration of the Loop software, two swap types are supported:
#### Loop Out
  * `off-chain` to `on-chain`, where the Loop client sends funds from the local balance of an active channel to an `on-chain` recieve address.

#### Why Loop Out?
- Acquiring inbound channel liquidity on the Lightning network.
- Depositing funds to a Bitcoin `on-chain` address without closing active channels.
- Balancing active channel(s) to optimize your nodes ability to forward payments.

#### Loop In
  * `on-chain` to `off-chain`, where the Loop client sends funds from an `on-chain` `UTXO` to the local balance of an active channel.

#### Why Loop In?
- Refilling depleted channels with funds from cold-wallets or exchange withdrawals.
- Servicing `off-chain` Lightning withdrawals using `on-chain` payments, with no funds in channels required.

Looping out from an active channel will require a path to the [LOOP public node](https://1ml.com/node/021c97a90a411ff2b10dc2a8e32de2f29d2fa49d41bfbb52bd416e460db0747d0d "LOOP public node"), if you want to be able to re-balance your channels with Loop make sure the channels you open have have a path to the LOOP public node.

The fees to exchange lightning network Bitcoin to `on-chain` Bitcoin are larger than what you would pay to circular rebalance a channel, and while looping out can be useful for initally balancing a channel, looping out can also be considered a way to "withdraw" some Bitcoin from an active channel without ever having to close the channel. This can help you withdraw `on-chain` Bitcoin from your large capacity channel, and then use the "withdrawn" looped out `on-chain` Bitcoin to then form another new channel.

## 2. Circular Rebalance Channels w/ Balance of Satoshis

* Circular payments are a completely off-chain rebalancing strategy where a node makes a payment to itself across a circular path of chained payment channels. For the route to be circular, there should be at least 3 nodes involved.

* Circular payments are a complete off-chain rebalancing strategy; meaning Carol can successfully rebalance her channels without broadcasting a single transaction to the blockchain. She doesn’t need to pay for on-chain fees or wait for confirmation times.

* Circular payments are not free. Since there are at least three nodes involved, one being herself, Carol must pay at least two other nodes for routing her payment. The bigger the loop, the more nodes she has to pay. 

* This strategy requires the existence of a circular and well funded route. The size of the attempted rebalancing payment is capped by the route at the moment of rebalancing. The longer the route, the more chances of getting a smaller cap. 

[Muun Wallet - Rebalancing Strategies](https://blog.muun.com/rebalancing-strategies-overview/)

## Cost / Benefit Analysis - Looping Out vs. Circular Rebalance

|    Tools that Rebalance Channels    |                              Pros                             |                                                                   Cons                                                                  |
|:-----------------------------------:|:-------------------------------------------------------------:|:---------------------------------------------------------------------------------------------------------------------------------------:|
|         Balance of Satoshis         | Cheaper rebalancing fees than Looping Out.                    | Requires the use of the CLI (Command Line Interface).                                                                                   |
|                                     | Faster than Looping Out.                                      | Amount you can circular rebalance depends on the current state of liquidity in your peer's channel and the peers they are connected to. |
| Lightning Loop (Lightning Terminal) | Non-custodial and trustless way to acquire inbound liquidity. | Fees are much higher than circular rebalancing, Loop Out while fees are cheap to help.                                                  |
|                                     | Lightning Terminal is an easy to use UI, no CLI needed.       | Takes some time to complete (approximately 30 minutes), and it is not always successful.                                                |

Lightning Loop and Circular Rebalancing are two different techniques you can use to rebalance a channel, both serving a different purpose. The choice of which tool you want to use is based on your discretion.

------------

### Looping Out (Create Inbound Liquidity) w/ myNode

As of myNode version v0.2.30, Lighting Terminal has been added to the myNode software stack. 

* [Announcing Lightning Terminal](https://lightning.engineering/posts/2020-08-04-lightning-terminal/)

#### 🎉 Channel rebalancing successful! You now have inbound liqudity.


------------

### Circular Rebalance w/ Balance of Satoshis

------------

### 🔙 [Back to Read Me](https://github.com/e-corp-sam-sepiol/bitcoin-node/blob/main/README.md "readme")
