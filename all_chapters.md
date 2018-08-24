
## How to write utxo based CryptoConditions contracts for KMD chains - by jl777

This is not the only smart contracts methodology that is possible to build on top of OP_CHECKCRYPTOCONDITION, just the first one. All the credit for getting OP_CHECKCRYPTOCONDITION working in the Komodo codebase goes to @libscott. I am just hooking into the code that he made and tried to make it just a little easier to make new contracts.

There is probably some fancy marketing name to use, but for now, I will just call it "CC contract" for short, knowing that it is not 100% technically accurate as the CryptoConditions aspect is not really the main attribute. However, the KMD contracts were built to make the CryptoConditions codebase that was integrated into it to be more accessible.
Since CC contracts run native C/C++ code, it is turing complete and that means that any contract that is possible to do on any other platform will be possible to create via CC contract.

utxo based contracts are a bit harder to start writing than for balance based contracts. However, they are much more secure as they leverage the existing bitcoin utxo system. That makes it much harder to have bugs that issue a zillion new coins from a bug, since all the CC contract operations needs to also obey the existing bitcoin utxo protocol.

This document will be heavily example based so it will utilize many of the existing reference CC contracts. After understanding this document, you should be in a good position to start creating either a new CC contract to be integrated into komodod or to make rpc based dapps directly.

- Chapter 0 - Bitcoin Protocol Basics
- Chapter 1 - OP_CHECKCRYPTOCONDITION
- Chapter 2 - CC contract basics
- Chapter 3 - CC vouts
Chapter 4 - CC rpc extensions
Chapter 5 - CC validation
Chapter 6 - faucet example
Chapter 7 - rewards example
Chapter 8 - assets example
Chapter 9 - dice example
Chapter 10 - lotto example
Chapter 11 - channels example
Chapter 12 - limitless possibilities
Chapter 13 - different languages
Chapter 14 - runtime bindings
Chapter 15 - rpc based dapps



Chapter 0 - Bitcoin Protocol Basics
There are many aspects of the bitcoin protocol that isnt needed to understand the CC contracts dependence on it. Such details will not be discussed. The primary aspect is the utxo, unspent transaction output. Just a fancy name for txid/vout, so when you sendtoaddress some coins, it creates a txid and the first output is vout.0, combine it and txid/0 is a specific utxo.

Of course, to understand even this level of detail requires that you understand what a txid is, but there are plenty of reference materials on that. It is basically the 64 char long set of letters and numbers that you get when you send funds.

Implicit with the utxo is that it prevents double spends. Once you spend a utxo, you cant spend it again. This is quite an important characteristic and while advanced readers will point out chain reorgs can allow a double spend, we will not confuse the issue with such details. The important thing is that given a blockchain at a specific height's blockhash, you can know if a txid/vout has been spent or not.

There are also the transactions that are in memory waiting to be mined, the mempool. And it is possible for the utxo to be spent by a tx in the mempool. However since it isnt confirmed yet, it is still unspent at the current height, even if we are pretty sure it will be spent in the next block.

A useful example is to think about a queue of people lined up to get into an event. They need to have a valid ticket and also to get into the queue. After some time passes, they get their ticket stamped and allowed into the event.

In the utxo case, the ticket is the spending transaction and the event is the confirmed blockchain. The queue is the mempool.





Chapter 1 - OP_CHECKCRYPTOCONDITION
In the prior chapter the utxo was explained. However, the specific mechanism used to send a payment was not explained. Contrary to what most people might think, on the blockchain there are not entries that say "pay X amount to address". Instead what exists is a bitcoin script that must be satisfied in order for the funds to be able to be spent.

Originally, there was the pay to pubkey script:
<pubkey> <checksig>

About as simple of a payment script that you can get. Basically the pubkey's signature is checked and if it is valid, you get to spend it. Once problem satoshi realized was that with Quantum Computers such payment scripts are vulnerable! So, he made a way to have a cold address, ie. an address whose pubkey isnt known. At least it isnt known until it is spent, so it is only Quantum resistant prior to the first spend. This line of reasoning is why we have one time use addresses and a new change address for each transaction. Maybe in some ways, this is too forward thinking as it makes things a lot more confusing to use and easier to lose track of all the required private keys.

However, it is here to stay and its script is:
<hash the pubkey> <pubkey> <verify hash matches> <checksig>
With this, the blockchain has what maps to "pay to address", just that the address is actually a base58 encoded (prefix + pubkeyhash). Hey, if it wasnt complicated, it would be easy!

In order to spend a p2pkh (pay to pubkey hash) utxo, you need to divulge the pubkey in addition to having a valid signature. After the first spend from an address, its security is degraded to p2pk (pay to pubkey) as its pubkey is now known. The net result is that each reused address takes 25 extra bytes on the blockchain, and that is why for addresses that are expected to be reused, I just use the p2pk script.

Originally, bitcoin allowed any type of script opcodes to be used directly. The problem was some of them caused problems and satoshi decided to disable them and only allow standard forms of payments. Thus the p2pk and p2pkh became 99%+ of bitcoin transactions. However, going from having a fully scriptable language that can create countless payment scripts (and bugs!), to having just 2... well it was a "short term" limitation. It did last for some years but eventually a compromise p2sh script was allowed to be standard. This is a pay to script hash, so it can have a standard format as the normal p2pkh, but have infinitely more flexibility.
<hash the script> <script> <verify hash matches>

Wait, something is wrong! If it was just that, then anybody that found out what the required script (called redeemscript) was, they could just spend it. I forgot to say that the redeemscript is then used to determine if the payment can be spent or not. So you can have a normal p2pk or p2pkh redeemscript inside a p2sh script.

OK, I know that just got really confusing. Let us have a more clear example:

redeemscript <- pay to pubkey 
p2sh becomes the hash of the redeem script + the compares

So to spend it, you need to divulge the redeemscript, which in turn requires you to divulge the pubkey. Put it all together and the p2sh mechanism verifies you not only had the correct redeemscript by comparing its hash, but that when the redeemscript is run, it is satisfied. In this case, that the pubkey's signature was valid.
If you are still following, there is some good news! OP_CHECKCRYPTOCONDITION scripts are actually simpler than p2sh scripts in some sense as there isnt this extra level of script inside a scripthash. @libscott implemented the addition of OP_CHECKCRYPTOCONDITION to the set of bitcoin opcodes and what it does is makes sure that a CryptoConditions script is properly signed.

Which gets us to the CryptoConditions specification, which is a monster of a IETF (Internet standards) draft and has hundred(s) of pages of specification. I am sure you are happy to know that you dont really need to know about it much at all! Just know that you can create all sorts of cryptoconditions and its binary encoding can be used in a bitcoin utxo. If the standard CC contracts dont have the power you need, it is always possible to expand on it. So far, most all the CC contracts only need the power of a 1of1 CC script, which is 1 signature combined with custom constraints. The realtime payment channels CC is the only one of the reference CC contracts so far that didnt fit into this model, it needed a 1of2 CC script.

The best part is that all these opcode level things are not needed at all. I just wanted to explain it for those that need to know all the details of everything.





Chapter 1 - OP_CHECKCRYPTOCONDITION
In the prior chapter the utxo was explained. However, the specific mechanism used to send a payment was not explained. Contrary to what most people might think, on the blockchain there are not entries that say "pay X amount to address". Instead what exists is a bitcoin script that must be satisfied in order for the funds to be able to be spent.

Originally, there was the pay to pubkey script:
<pubkey> <checksig>

About as simple of a payment script that you can get. Basically the pubkey's signature is checked and if it is valid, you get to spend it. Once problem satoshi realized was that with Quantum Computers such payment scripts are vulnerable! So, he made a way to have a cold address, ie. an address whose pubkey isnt known. At least it isnt known until it is spent, so it is only Quantum resistant prior to the first spend. This line of reasoning is why we have one time use addresses and a new change address for each transaction. Maybe in some ways, this is too forward thinking as it makes things a lot more confusing to use and easier to lose track of all the required private keys.

However, it is here to stay and its script is:
<hash the pubkey> <pubkey> <verify hash matches> <checksig>
With this, the blockchain has what maps to "pay to address", just that the address is actually a base58 encoded (prefix + pubkeyhash). Hey, if it wasnt complicated, it would be easy!

In order to spend a p2pkh (pay to pubkey hash) utxo, you need to divulge the pubkey in addition to having a valid signature. After the first spend from an address, its security is degraded to p2pk (pay to pubkey) as its pubkey is now known. The net result is that each reused address takes 25 extra bytes on the blockchain, and that is why for addresses that are expected to be reused, I just use the p2pk script.

Originally, bitcoin allowed any type of script opcodes to be used directly. The problem was some of them caused problems and satoshi decided to disable them and only allow standard forms of payments. Thus the p2pk and p2pkh became 99%+ of bitcoin transactions. However, going from having a fully scriptable language that can create countless payment scripts (and bugs!), to having just 2... well it was a "short term" limitation. It did last for some years but eventually a compromise p2sh script was allowed to be standard. This is a pay to script hash, so it can have a standard format as the normal p2pkh, but have infinitely more flexibility.
<hash the script> <script> <verify hash matches>

Wait, something is wrong! If it was just that, then anybody that found out what the required script (called redeemscript) was, they could just spend it. I forgot to say that the redeemscript is then used to determine if the payment can be spent or not. So you can have a normal p2pk or p2pkh redeemscript inside a p2sh script.

OK, I know that just got really confusing. Let us have a more clear example:

redeemscript <- pay to pubkey 
p2sh becomes the hash of the redeem script + the compares

So to spend it, you need to divulge the redeemscript, which in turn requires you to divulge the pubkey. Put it all together and the p2sh mechanism verifies you not only had the correct redeemscript by comparing its hash, but that when the redeemscript is run, it is satisfied. In this case, that the pubkey's signature was valid.
If you are still following, there is some good news! OP_CHECKCRYPTOCONDITION scripts are actually simpler than p2sh scripts in some sense as there isnt this extra level of script inside a scripthash. @libscott implemented the addition of OP_CHECKCRYPTOCONDITION to the set of bitcoin opcodes and what it does is makes sure that a CryptoConditions script is properly signed.

Which gets us to the CryptoConditions specification, which is a monster of a IETF (Internet standards) draft and has hundred(s) of pages of specification. I am sure you are happy to know that you dont really need to know about it much at all! Just know that you can create all sorts of cryptoconditions and its binary encoding can be used in a bitcoin utxo. If the standard CC contracts dont have the power you need, it is always possible to expand on it. So far, most all the CC contracts only need the power of a 1of1 CC script, which is 1 signature combined with custom constraints. The realtime payment channels CC is the only one of the reference CC contracts so far that didnt fit into this model, it needed a 1of2 CC script.

The best part is that all these opcode level things are not needed at all. I just wanted to explain it for those that need to know all the details of everything.






Chapter 2 - CC contract basics
Each CC contract has an eval code, this is just an arbitrary number that is associated with a specific CC contract. The details about a specific CC contract are all determined by the validation logic, that is ultimately what implements a CC contract.
However, unlike the normal bitcoin payments, where it is validated with only information in the transaction, a CC contract has the power to do pretty much anything. It has full access to the blockchain and even the mempool, though using mempool information is inherently more risky and needs to be done carefully or for exclusions, rather than inclusions.

However, this is the CC contract basics chapter, so let us ignore mempool issues and deal with just the basics. Fundamentally there is no structure for OP_CHECKCRYPTOCONDITION serialized scripts, but if you are like me, you want to avoid having to read and understand a 1000 page IETF standard. What we really want to do is have a logical way to make a new contract and have it be able to be coded and debugged in an efficient way.

That means to just follow a known working template and only changing the things where the existing templates are not sufficient, ie. the core differentiator of your CC contract.

In the ~/komodo/src/cc/eval.h file all the eval codes are defined, currently:

#define FOREACH_EVAL(EVAL)             \
        EVAL(EVAL_IMPORTPAYOUT, 0xe1)  \
        EVAL(EVAL_IMPORTCOIN,   0xe2)  \
        EVAL(EVAL_ASSETS,   0xe3)  \
        EVAL(EVAL_FAUCET, 0xe4) \
        EVAL(EVAL_REWARDS, 0xe5) \
        EVAL(EVAL_DICE, 0xe6) \
        EVAL(EVAL_FSM, 0xe7) \
        EVAL(EVAL_AUCTION, 0xe8) \
        EVAL(EVAL_LOTTO, 0xe9) \
        EVAL(EVAL_MOFN, 0xea) \
        EVAL(EVAL_CHANNELS, 0xeb) \
        EVAL(EVAL_ORACLES, 0xec) \
        EVAL(EVAL_PRICES, 0xed) \
        EVAL(EVAL_PEGS, 0xee) \
        EVAL(EVAL_TRIGGERS, 0xef) \
        EVAL(EVAL_PAYMENTS, 0xf0) \
        EVAL(EVAL_GATEWAYS, 0xf1)
Ultimately, we will probably end up with all 256 eval codes used, for now there is plenty of room. I imagined that similar to my coins repo, we can end up with a much larger than 256 number of CC contracts and you select the 256 that you want active for your blockchain. That does mean any specific chain will be limited to "only" having 256 contracts. Since there seems to be so few actually useful contracts so far, this limit seems to be sufficient. I am told that the evalcode can be of any length, but the current CC contracts assumes it is one byte. 

The simplest CC script would be one that requires a signature from a pubkey along with a CC validation. This is the equivalent of the pay to pubkey bitcoin script and is what most of the initial CC contracts use. Only the channels one needed more than this and it will be explained in its chapter.
We end up with CC scripts of the form (evalcode) + (pubkey) + (other stuff), dont worry about the other stuff, it is automatically handled with some handy internal functions. The important thing to note is that each CC contract of this form needs a single pubkey and eval code and from that we get the CC script. Using the standard bitcoin's "hash and make an address from it" method, this means that the same pubkey will generate a different address for each different CC contract!

This is an important point, so I will say it in a different way. In bitcoin there used to be uncompressed pubkeys which had both the right and left half combined, into a giant 64 byte pubkey. But since you can derive one from the other, compressed pubkeys became the standard, that is why you have bitcoin pubkeys of 33 bytes instead of 65 bytes. There is a 02, 03 or 04 prefix, to mean odd or even or big pubkey. This means there are two different pubkeys for each privkey, the compressed and uncompressed. And in fact you can have two different bitcoin protocol addresses that are spendable by the same privkey. If you use some paper wallet generators, you might have noticed this.

CC contracts are like that, where each pubkey gets a different address for each evalcode. It is the same pubkey, just different address due to the actual script having a different evalcode, it ends up with a different hash and thus a different address. Now funds send to a specific CC address is only accessible by that CC contract and must follow the rules of that contract.
I also added another very useful feature where the convention is for each CC contract to have a special address that is known to all, including its private key. Before you panic about publishing the private key, remember that to spend a CC output, you need to properly sign it AND satisfy all the rules. By everyone having the privkey for the CC contract, everybody can do the "properly sign" part, but they still need to follow the rest of the rules.

From a user's perspective, there is the global CC address for a CC contract and some contracts also use the user pubkey's CC address. Having a pair of new addresses for each contract can get a bit confusing at first, but eventually we will get easy to use GUI that will make it all easy to use.







Chapter 3 - CC vins and vouts
You might want to review the bitcoin basics and other materials to refresh about how bitcoin outputs become inputs. It is a bit complicated, but ultimately it is about one specific amount of coins that are spent, once spent it is combined with the other coins that are also spent in that transaction and then various outputs are created.

vin0 + vin1 + vin2 -> vout0 + vout1

That is a 3 input, 2 output transaction. The value from the three inputs are combined and then split into vout0 and vout1, each of the vouts gets a spend script that must be satisfied to be able to be spent. Which means for all three of out vins, all the requirements (as specified in the output that created them) are satisfied.

Yes, I know this is a bit too complicated without a nice chart, so we will hope that a nice chart is added here:

[nice chart goes here]

Out of all the aspects of the CC contracts, the flexibility that different vins and vouts created was the biggest surprise. When I started writing the first of these a month ago, I had no idea the power inherent in the smart utxo contracts. I was just happy to have a way to lock funds and release them upon some specific conditions.
After the assets/tokens CC contract, I realized that it was just a tip of the iceberg. I knew it was Turing complete, but after all these years of restricted bitcoin script, to have the full power of any arbitrary algorithm, it was eye opening. Years of writing blockchain code and having really bad consequences with every bug naturally makes you gun shy about doing aggressive things at the consensus level. And that is the way it should be, if not very careful, some really bad things can and do happen. The foundation of building on top of the existing (well tested and reliable) utxo system is what makes the CC contracts less likely for the monster bugs. That being said, lack of validation can easily allow an improperly coded CC contract to have its funds drained.

The CC contract breaks out of the standard limitations of a bitcoin transaction. Already, what I wrote explains the reason, but it was not obvious even to me at first, so likely you might have missed it too. If you are wondering what on earth I am talking about, THAT is what I am talking about!

To recap, we have now a new standard bitcoin output type called a CC output. Further, there can be up to 256 different types of CC outputs active on any given blockchain. We also know that to spend any output, you need to satisfy its spending script, which in our case is the signature and whatever constraints the CC validation imposes. We also have the convention of a globally shared keypair, which gives us a general CC address that can have funds sent to it, along with a user pubkey specific CC address.
Let us go back to the 3+2 transaction example:

vin0 + vin1 + vin2 -> vout0 + vout1

Given the prior paragraph, try to imagine the possibilities the simple 3+2 transaction can be. Each vin could be a normal vin, from the global contract address, the user's CC address and the vouts can also have this range. Theoretically, there can be 257 * 257 * 257 * 257 * 257 forms of a 3+2 transaction!

In reality, we really dont want that much degrees of freedom as it will ensure a large degree of bugs! So we need to reduce things to a more manageable level where there are at most 3 types for each, and preferably just 1 type. That will make the job of validating it much simpler and simple is better as long as we dont sacrifice the power. We dont.

Ultimately the CC contract is all about how it constrains its inputs, but before it can constrain them, they need to be created as outputs. More about this in the CC validation chapter.



