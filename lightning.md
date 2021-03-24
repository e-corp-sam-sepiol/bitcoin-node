# Lightning Network (MyNode)

#### [http://lightning.network/docs/](http://lightning.network/docs/ "http://lightning.network/docs/")

The Lightning Network is a decentralized system for instant, high-volume micropayments that removes the risk of delegating custody of funds to trustedthird parties.

Funds are placed into a two-party, multisignature "channel" bitcoin address. This channel is represented as an entry on the bitcoin public ledger. In order to spend funds from the channel, both parties must agree on the new balance. The current balance is stored as the most recent transaction signed by both parties, spending from the channel address. To make a payment, both parties sign a new exit transaction spending from the channel address. All old exit transactions are invalidated by doing so. 

The Lightning Network does not require cooperation from the counterparty to exit the channel.  Both parties have the option to unilaterally close the channel, ending their relationship. Since all parties have multiple multisignature channels with many different users on this network, one can send a payment to any other party across this network.

By embedding the payment conditional upon knowledge of a secure cryptographic hash, payments can be made across a network of channels without the need for any party to have unilateral custodial ownership of funds. The Lightning Network enables what was previously not possible with trusted financial systems vulnerable to monopolies—without the need for custodial trust and ownership, participation on the network can be dynamic and open for all.

# Managing Channel Balances
[![channel_1](https://i.imgur.com/HQI3THY.png "channel_1")](https://i.imgur.com/HQI3THY.png "channel_1")

When a new channel is opened, it is likely that the complete balance will remain on the local side of the channel. This means that we can send satoshis, but cannot recieve any satoshis to `channel_1`.

Routing transactions for the lightning network will require fairly balanced channels on your node. This ensures throughput can be sent and recieved via all of the active channels.

## Loop
#### [https://github.com/lightninglabs/loop](https://github.com/lightninglabs/loop "https://github.com/lightninglabs/loop")
Lightning Loop is a non-custodial service offered by [Lightning Labs](https://lightning.engineering/) to bridge on-chain and off-chain Bitcoin using submarine swaps. This repository is home to the Loop client and depends on the Lightning Network daemon [lnd](https://github.com/lightningnetwork/lnd). All of lnd’s supported chain backends are fully supported when using the Loop client: Neutrino, Bitcoin Core, and btcd.

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

We can confirm the loop request has been submitted by checking the `RTL` web dashboard, under Lightning > Peers/Channels > Active HTLCs. There are active HTLCs in `RTL`, this confirms our loop out request has been sent.

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
