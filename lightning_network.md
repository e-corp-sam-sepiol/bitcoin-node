# Lightning Network 🌩️
#### [http://lightning.network/docs/](http://lightning.network/docs/ "http://lightning.network/docs/")

The Lightning Network is a decentralized system for instant, high-volume micropayments that removes the risk of delegating custody of funds to trustedthird parties.

Funds are placed into a two-party, multisignature "channel" bitcoin address. This channel is represented as an entry on the bitcoin public ledger. In order to spend funds from the channel, both parties must agree on the new balance. The current balance is stored as the most recent transaction signed by both parties, spending from the channel address. To make a payment, both parties sign a new exit transaction spending from the channel address. All old exit transactions are invalidated by doing so. 

The Lightning Network does not require cooperation from the counterparty to exit the channel.  Both parties have the option to unilaterally close the channel, ending their relationship. Since all parties have multiple multisignature channels with many different users on this network, one can send a payment to any other party across this network.

By embedding the payment conditional upon knowledge of a secure cryptographic hash, payments can be made across a network of channels without the need for any party to have unilateral custodial ownership of funds. The Lightning Network enables what was previously not possible with trusted financial systems vulnerable to monopolies—without the need for custodial trust and ownership, participation on the network can be dynamic and open for all.

# myNode 🔗
#### 1. [Create Lightning Wallet](https://mynodebtc.github.io/lightning/create.html "Create Lightning Wallet")
#### 2. [Logging into Ride the Lightning](https://mynodebtc.github.io/lightning/rtl.html#usage "Logging into Ride the Lightning")
Ride the Lightning is a Lightning wallet and node management tool accessible via a web interface, and is built into myNode.
You can use RTL from any browser that is able to access your myNode installation.
#### 3. [Thunderhub](https://mynodebtc.github.io/lightning/thunderhub.html#introduction "Thunderhub")
ThunderHub is an open-source LND node manager to monitor your node and manage channels via a web-interface. It allows you to take control of the Lightning network with a simple and intuitive UX.

# Managing Channel Balances ⚖️
[![channel_1](https://i.imgur.com/HQI3THY.png "channel_1")](https://i.imgur.com/HQI3THY.png "channel_1")

When a new channel is opened, it is likely that the complete balance will remain on the local side of the channel. This means that we can send satoshis, but cannot recieve any satoshis to `channel_1`.

Routing transactions for the lightning network will require fairly balanced channels on your node. This ensures throughput can be sent and recieved via all of the active channels.

## Loop (Submarine Swaps)
#### [https://github.com/lightninglabs/loop](https://github.com/lightninglabs/loop "https://github.com/lightninglabs/loop")
Lightning Loop is a non-custodial service offered by [Lightning Labs](https://lightning.engineering/) to bridge on-chain and off-chain Bitcoin using submarine swaps. This repository is home to the Loop client and depends on the Lightning Network daemon [lnd](https://github.com/lightningnetwork/lnd). All of LNDs supported chain backends are fully supported when using the Loop client: Neutrino, Bitcoin Core, and btcd.

-  [A Closer Look at Submarine Swaps in the Lightning Network](https://blog.muun.com/a-closer-look-at-submarine-swaps-in-the-lightning-network/ "A Closer Look at Submarine Swaps in the Lightning Network")

In the current iteration of the Loop software, two swap types are supported:
### Loop Out
  * `off-chain` to `on-chain`, where the Loop client sends funds from the local balance of an active channel to an `on-chain` recieve address.

#### Why Loop Out?
- Acquiring inbound channel liquidity on the Lightning network.
- Depositing funds to a Bitcoin `on-chain` address without closing active channels.
- Balancing active channel(s) to optimize your nodes ability to forward payments.

### Loop In
  * `on-chain` to `off-chain`, where the Loop client sends funds from an `on-chain` `UTXO` to the local balance of an active channel.

#### Why Loop In?
- Refilling depleted channels with funds from cold-wallets or exchange withdrawals.
- Servicing `off-chain` Lightning withdrawals using `on-chain` payments, with no funds in channels required.

#### Considerations When Using Loop
- Looping out from an active channel will require a path to the [LOOP public node](https://1ml.com/node/021c97a90a411ff2b10dc2a8e32de2f29d2fa49d41bfbb52bd416e460db0747d0d "LOOP public node"), if you want to be able to re-balance your channels with Loop make sure the channels you open have have a path to the LOOP public node.


### How to Loop Out

Log into your node via `SSH`
```
ssh mynode.local -l admin
```
[![https://i.imgur.com/ZRmBccm.png](https://i.imgur.com/ZRmBccm.png "https://i.imgur.com/ZRmBccm.png")](https://i.imgur.com/ZRmBccm.png "https://i.imgur.com/ZRmBccm.png")

Change directories to the `LND` folder.
```
cd /mnt/hdd/mynode/lnd
```

View the `LND` logs in realtime.
```
sudo tail -f logs/bitcoin/mainnet/lnd.log
```

Monitor Loop status. View the realtime status of your `Loop Out` attempts. 
```
loop monitor
```

Identify the channel you want to rebalance.

[![https://i.imgur.com/hPLvkZB.png](https://i.imgur.com/hPLvkZB.png "https://i.imgur.com/hPLvkZB.png")](https://i.imgur.com/hPLvkZB.png "https://i.imgur.com/hPLvkZB.png")

`channel_2` is evenly balanced, however `channel_1` is heavily weighted to the local balance side of the channel. 
- This means more satoshis can be sent from `channel_1` than can be recieved into `channel_1`.
- `channel_1` should be rebalanced so that it is closer to an even split of balance on both ends of the channel.

Loop Out allows a portion of the local balance of an active channel to be sent out from the channel, pushing that amount of satoshis to the remote side. However, instead of simply purchasing something to facilitate moving funds to the remote side of the channel, we are going to loop it back to an `on-chain` recieve address we control.


Find the Channel ID of the channel you want to target for Loop Out.
```
lncli listchannels
```
[![https://i.imgur.com/1UKvN0g.png](https://i.imgur.com/1UKvN0g.png "https://i.imgur.com/1UKvN0g.png")](https://i.imgur.com/1UKvN0g.png "https://i.imgur.com/1UKvN0g.png")

`lncli` `listchannels` allows us to find the `chan_id` for the channel we want to rebalance with Loop Out. 

```
loop out --help
```

```
OPTIONS:
   --channel value               the comma-separated list of short channel IDs of the channels to loop out
   --addr value                  the optional address that the looped out funds should be sent to, if let blank the funds will go to lnd's wallet
   --amt value                   the amount in satoshis to loop out (default: 0)
   --htlc_confs value            the number of of confirmations, in blocks that we require for the htlc extended by the server before we reveal the preimage. (default: 1)
   --conf_target value           the number of blocks from the swap initiation height that the on-chain HTLC should be swept within (default: 9)
   --max_swap_routing_fee value  the max off-chain swap routing fee in satoshis, if not specified, a default max fee will be used (default: 0)
   --fast                        Indicate you want to swap immediately, paying potentially a higher fee. If not set the swap server might choose to wait up to 30 minutes before publishing the swap HTLC on-chain, to save on its chain fees. Not setting this flag therefore might result in a lower swap fee.
   --label value                 an optional label for this swap,limited to 500 characters. The label may not start with our reserved prefix: [reserved].
```

Construct the Loop Out request.
```
loop out --channel <chan_id> --conf_target <# of blocks> --max_swap_routing_fee <max fee amount> <loop out amount>
```
[![https://i.imgur.com/hPLvkZB.png](https://i.imgur.com/hPLvkZB.png "https://i.imgur.com/hPLvkZB.png")](https://i.imgur.com/hPLvkZB.png "https://i.imgur.com/hPLvkZB.png")

Rebalancing `channel_1` to an even split between the local balance and remote balance will require `368,514` satoshis to be looped out from the channel.
```
loop out --channel <chan_id> --conf_target 100 --max_swap_routing_fee 1000 368514
```
[![https://i.imgur.com/2GDAs61.png](https://i.imgur.com/2GDAs61.png "https://i.imgur.com/2GDAs61.png")](https://i.imgur.com/2GDAs61.png "https://i.imgur.com/2GDAs61.png")
[![https://i.imgur.com/fcEHgmR.png](https://i.imgur.com/fcEHgmR.png "https://i.imgur.com/fcEHgmR.png")](https://i.imgur.com/fcEHgmR.png "https://i.imgur.com/fcEHgmR.png")
[![https://i.imgur.com/MHpzEVk.png](https://i.imgur.com/MHpzEVk.png "https://i.imgur.com/MHpzEVk.png")](https://i.imgur.com/MHpzEVk.png "https://i.imgur.com/MHpzEVk.png")

We can confirm the loop request has been submitted by checking the `RTL` web dashboard, under `Lightning > Peers/Channels > Active HTLCs`. There are active HTLCs in `RTL`, this confirms our loop out request has been sent.

[![https://i.imgur.com/6butY5F.png](https://i.imgur.com/6butY5F.png "https://i.imgur.com/6butY5F.png")](https://i.imgur.com/6butY5F.png "https://i.imgur.com/6butY5F.png")

The loop out process will typically take about 30 minutes until you will see further progress from `loop monitor`. Once we see `LOOP_OUT PREIMAGE_REVEALED` in the `loop monitor` output, we will know the Loop Out attempt will succeed.

[![https://i.imgur.com/ugwanXq.png](https://i.imgur.com/ugwanXq.png "https://i.imgur.com/ugwanXq.png")](https://i.imgur.com/ugwanXq.png "https://i.imgur.com/ugwanXq.png")
[![https://i.imgur.com/T8lcNj7.png](https://i.imgur.com/T8lcNj7.png "https://i.imgur.com/T8lcNj7.png")](https://i.imgur.com/T8lcNj7.png "https://i.imgur.com/T8lcNj7.png")

### Channel rebalancing successful!

[![https://i.imgur.com/dQPJmy7.png](https://i.imgur.com/dQPJmy7.png "https://i.imgur.com/dQPJmy7.png")](https://i.imgur.com/dQPJmy7.png "https://i.imgur.com/dQPJmy7.png")

# Balance of Satoshis
#### [https://github.com/alexbosworth/balanceofsatoshis](https://github.com/alexbosworth/balanceofsatoshis "https://github.com/alexbosworth/balanceofsatoshis")
Tool for working with the balance of your satoshis on LND.

### Installation
```
sudo npm install -g balanceofsatoshis
```
If you recieve an error message such as this:
```
npm WARN ws@7.4.4 requires a peer of bufferutil@^4.0.1 but none is installed. You must install peer dependencies yourself.
npm WARN ws@7.4.4 requires a peer of utf-8-validate@^5.0.2 but none is installed. You must install peer dependencies yourself.
```
You can solve it by doing the following:
```
npm install --save-dev "bufferutil@^4.0.1"
npm install --save-dev "utf-8-validate@^5.0.2"
```
`--save-dev` saves the dependency as a development dependency to your [package.json](https://docs.npmjs.com/specifying-dependencies-and-devdependencies-in-a-package-json-file#adding-dependencies-to-a-packagejson-file-from-the-command-line "package.json").

```
bos help
```

```
   bos 8.0.2 

   USAGE
     bos <command> [options]

   COMMANDS
     accounting <category>               Get an accounting rundown                         
     advertise                           Broadcast advertisement                           
     balance                             Get total tokens                                  
     broadcast <tx>                      Submit a signed transaction to the mempool        
     cert-validity-days                  Number of days until the cert is invalid          
     chain-deposit [amount]              Deposit coins in the on-chain wallet              
     chain-receive [amount]              Receive funds on-chain via submarine swap         
     chainfees                           Get the current chain fee estimates               
     chart-fees-earned [via_peer]        Get a chart of earned routing fees                
     chart-chain-fees                    Get a chart of chain fee expenses                 
     chart-fees-paid                     Get a chart of paid routing fees                  
     chart-payments-received             Get a chart of received payments                  
     closed                              Get the status of a channel closings              
     credentials                         Export local credentials                          
     fanout <size> <count>               Fan out utxos                                     
     fees                                Show and adjust outbound fee rates                
     find <query>                        Find a record                                     
     forwards                            Get forwards                                      
     fund <address_amount...>            Make a signed transaction spending on-chain funds 
     gateway                             Request gateway for https://ln-operator.github.io/
     inbound-channel-rules               Enforce rules for inbound channels                
     inbound-liquidity                   Get inbound liquidity size                        
     increase-inbound-liquidity          Increase node inbound liquidity                   
     increase-outbound-liquidity         Move on-chain funds off-chain                     
     market [pair] [exchange]            Get the history of prices on a market             
     nodes [node]                        List and edit saved nodes                         
     open <peer_public_keys...>          Open channels using an external wallet for funding
     open-balanced-channel               Open a dual-funded channel with a node            
     outbound-liquidity                  Get outbound liquidity size                       
     pay <request>                       Pay a payment request, probing first              
     peers                               Get a list of channel-connected peers             
     price [symbols...]                  Get the price                                     
     probe <to> [amount]                 Check if a payment request is sendable            
     purchase-ping <to>                  Request a ping payment response from a node       
     rebalance                           Rebalance funds between peers                     
     reconnect                           Reconnect to disconnected channel partners        
     remove-peer [public_key]            Close out with a channel-connected peer           
     report                              Report about the node                             
     send <to>                           Send funds to a node                              
     service-keysend-requests            Respond to keysend service requests               
     swap-api-key                        Purchase a swap API key or inspect a swap API key 
     tags [tag]                          View or adjust the set of tagged nodes            
     telegram                            Post updates to a Telegram bot                    
     unlock <path_to_password_file>      Unlock wallet if locked                           
     utxos                               Get a list of utxos                               
     help <command>                      Display help for a specific command               

   GLOBAL OPTIONS
     -h, --help         Display help                                      
     -V, --version      Display version                                   
     --no-color         Disable colors                                    
     --quiet            Quiet mode - only displays warn and error messages
     -v, --verbose      Verbose mode - will also output debug messages    

```
------------

### Payment Routing 🧅
[![Multi Hop Payment](https://blog.lightning.engineering/assets/images/multihop.png "Multi Hop Payment")](https://blog.lightning.engineering/assets/images/multihop.png "Multi Hop Payment")
#### [Routing vs. Path Finding](https://github.com/lnbook/lnbook/blob/develop/routing.asciidoc#routing-vs-path-finding "Routing vs. Path Finding")
Path Finding, which is covered in [[path_finding]](https://github.com/lnbook/lnbook/blob/develop/routing.asciidoc#path_finding "[path_finding]") is the process of finding and choosing a contiguous path made of payment channels which connects the sender `A` to the recipient `B`. The sender of a payment does the path finding, by examining the channel graph which they have assembled from channel announcements gossiped by other nodes.

Routing refers to the series of interactions across the network that allow a payment to flow from A to B, across the path previously selected by path finding. Routing is the active process of sending a payment on a path, which involves the cooperation of all the intermediary nodes along that path.

An important rule of thumb is that it is possible for a path to exist between `Alice` and `Bob`, yet there may not be an active route on which to send the payment.

- [Exploring Lightning Network Routing](https://blog.lightning.engineering/posts/2018/05/30/routing.html "Exploring Lightning Network Routing")

------------

[Balance of Satoshis](https://github.com/alexbosworth/balanceofsatoshis "Balance of Satoshis"), written by Alex Bosworth, can help us simulate lightning network payments without actually paying anything. This can help us probe the network, and acquire information regarding our capability of routing payments.

```
bos help probe
```
```
   bos 8.0.2 
   USAGE
     bos probe <to> [amount]
     Simulate paying a payment request without actually paying it

   ARGUMENTS
     <to>          Payment request or node public key            required      
     [amount]      Amount to probe, default: request amount      optional      

   OPTIONS
     --avoid <pubkey>        Avoid forwarding through node                           optional                    
     --find-max              Find the maximum routeable amount on success route      optional      default: false
     --in <public_key>       Route through specific peer of destination              optional                    
     --max-paths <max>       Maximum paths to use for find-max                       optional      default: 1    
     --no-color              Mute all colors                                         optional      default: false
     --node <node_name>      Node to use for payment request check                   optional                    
     --out <public_key>      Make first hop through peer                             optional                    
```

[1ml.com](https://1ml.com/ "1ml.com") is a Lightning Network search and analysis engine, which provides useful network metrics associated with nodes that can help inform our decisions to open channel(s) with one node vs another node. We can use [1ml.com](https://1ml.com/ "1ml.com") to find the `public id` of lightning network nodes when learning how to use the `bos` command `probe`.

My bitcoin node currently has three active channels with other lightning nodes on the network. When I use the `probe` command, the process will run by using one of the three channels I have open, as these channels are my payment gateway to the rest of the lightning network.

------------

Let's `probe` a payment route to [Bitrefill.com](http://bitrefill.com "Bitrefill.com")'s lightning node:
- [Public Key: 030c3f19d742ca294a55c00376b3b355c3c90d61c6b6b39554dbc7ac19b141c14f](https://1ml.com/node/030c3f19d742ca294a55c00376b3b355c3c90d61c6b6b39554dbc7ac19b141c14f "Public Key: 030c3f19d742ca294a55c00376b3b355c3c90d61c6b6b39554dbc7ac19b141c14f")

```
bos probe <public id of node we want to route to> --find-max
```

`--find-max` Find the maximum routeable amount on success route.

```
bos probe 030c3f19d742ca294a55c00376b3b355c3c90d61c6b6b39554dbc7ac19b141c14f --find-max
```
[![bos probe](https://i.imgur.com/VkVB2m9.png "bos probe")](https://i.imgur.com/VkVB2m9.png "bos probe")

The maximum payment I can route on the lightning network to Bitrefill.com's node is `0.01230734` or `1,230,734` satoshis. The payment would be routed through `LightningTo.Me`.

`--max-paths` can help us `probe` multiple paths to the same payment destination.

Let's see if I can find three different payment routes to Bitrefill.com:
```
bos probe 030c3f19d742ca294a55c00376b3b355c3c90d61c6b6b39554dbc7ac19b141c14f --find-max --max-paths 3
```
[![payment routing](https://i.imgur.com/gsgqaHJ.png "payment routing")](https://i.imgur.com/gsgqaHJ.png "payment routing")
