How to write utxo based CryptoConditions contracts for KMD chains
by jl777

This is not the only smart contracts methodology that is possible to build on top of OP_CHECKCRYPTOCONDITION, just the first one. All the credit for getting OP_CHECKCRYPTOCONDITION working in the Komodo codebase goes to @libscott. I am just hooking into the code that he made and tried to make it just a little easier to make new contracts.

There is probably some fancy marketing name to use, but for now, I will just call it "CC contract" for short, knowing that it is not 100% technically accurate as the CryptoConditions aspect is not really the main attribute. However, the KMD contracts were built to make the CryptoConditions codebase that was integrated into it to be more accessible.
Since CC contracts run native C/C++ code, it is turing complete and that means that any contract that is possible to do on any other platform will be possible to create via CC contract.

utxo based contracts are a bit harder to start writing than for balance based contracts. However, they are much more secure as they leverage the existing bitcoin utxo system. That makes it much harder to have bugs that issue a zillion new coins from a bug, since all the CC contract operations needs to also obey the existing bitcoin utxo protocol.

This document will be heavily example based so it will utilize many of the existing reference CC contracts. After understanding this document, you should be in a good position to start creating either a new CC contract to be integrated into komodod or to make rpc based dapps directly.

Chapter 0 - Bitcoin Protocol Basics
Chapter 1 - OP_CHECKCRYPTOCONDITION
Chapter 2 - CC contract basics
Chapter 3 - CC vouts
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