
## What is CC Enabled Token
We have core assets support to CC enabled coins/tokens aka colored coins. An assetchain could now be used as a tokenizing platform. The tokens inherited the utxos. They are utxos, just special ones (colored coins actually).

These are assets, like NXT assets, or you can refer them as tokens. Tokenizing chain can be done within an assetchain. They can be generated from any chain with CC enabled.
There are rpc calls like: tokencreate, tokentransfer, tokenaddress <pubkey> which returns a dedicated address where all tokens for a pubkey end up. Also there exchange rpc calls like: tokenbid, tokenask, tokenswap, tokenfillbid, tokenfillask and tokenfillswap.
Using an explorer (addressindex) then the assets balance for an address can be tallied.

It requires locking X amount of native coins to create the supply for a token. So, if you want a unique token, make it 1 satoshi. Using 1 coin gives you 100 million tokens. Or you can interpret it with different decimal precision.
Each satoshi can be 1 token. So from 1.00000000 coins it makes 100 million tokens, but without any fractions.

## How would those tokens move?
It is still a ways to go from being done. `tokentransfer` rpc that works like `sendmany`. Well, you need to specify the source txid/vout as it is critical to match outputs with inputs. If you send to burn address, it is burnt.

## Available RPC Calls
```
== Tokens ==  
tokenaddress [pubkey]  
tokenask numtokens tokenid price  
tokenbalance tokenid [pubkey]  
tokenbid numtokens tokenid price  
tokencancelask tokenid asktxid  
tokencancelbid tokenid bidtxid  
tokencreate name supply description  
tokenfillask tokenid asktxid fillunits  
tokenfillbid tokenid bidtxid fillamount  
tokeninfo tokenid  
tokenlist  
tokenorders [tokenid]  
tokentransfer tokenid destpubkey amount  
```
