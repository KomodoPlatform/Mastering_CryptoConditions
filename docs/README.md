# Komodo Platform's CryptoConditions Contract development
Komodo CC Token testing description, setup &amp; how-to

## Available Contracts
Select the contract to learn more about the contract from below.  
[Tokens](./token)  
[Rewards](./rewards)  
[Faucet](./faucet)  
[Dice](./dice)  

## WIP Contracts
Lotto  
Ponzi  
Auction  
FSM  

## How to start testing?
Compile Komodo, navigate to `src` dir, start the test chain with your pubkey and issue the SmartContract RPC commands to test. All the instructions to get you started are below.
```shell
cd ~
sudo apt-get update
sudo apt-get upgrade -y
sudo apt-get install build-essential pkg-config libc6-dev m4 g++-multilib autoconf libtool ncurses-dev unzip git python zlib1g-dev wget bsdmainutils automake libboost-all-dev libssl-dev libprotobuf-dev protobuf-compiler libgtest-dev libqt4-dev libqrencode-dev libdb++-dev ntp ntpdate nano software-properties-common curl libcurl4-gnutls-dev cmake clang

git clone https://github.com/nanomsg/nanomsg
cd nanomsg
cmake . -DNN_TESTS=OFF -DNN_ENABLE_DOC=OFF
make -j2
sudo make install
sudo ldconfig

cd ~
git clone https://github.com/jl777/komodo
cd komodo
git checkout jl777
./zcutil/fetch-params.sh
./zcutil/build.sh -j$(nproc)

cd ~/komodo/src
./komodod -ac_cc=1 -ac_name=<CHAIN_NAME> -addressindex=1 -spentindex=1 -ac_supply=1000 -ac_reward=10000000000000 -pubkey=<your_pub_key> -addnode=195.201.20.230 -addnode=195.201.137.5 -addnode=195.201.20.230 -addnode=94.130.224.11 &
```

## For Developers
Source repo: https://github.com/jl777/komodo  
Source directory: https://github.com/jl777/komodo/tree/dev/src/cc  
Useful Links:
- https://raw.githubusercontent.com/jl777/komodo/dev/src/cc/assets.cpp
- https://raw.githubusercontent.com/jl777/komodo/dev/src/cc/dice.cpp
- https://raw.githubusercontent.com/jl777/komodo/dev/src/cc/rewards.cpp
- https://raw.githubusercontent.com/jl777/komodo/dev/src/cc/token.cpp

1. Add EVAL_CODE to eval.h
2. Initialize the variables in the CCinit function below
3. Write a Validate function to reject any unsanctioned usage of vin/vout
4. Make helper functions to create rawtx for RPC functions
5. Add rpc calls to rpcserver.cpp and rpcserver.h and in one of the rpc.cpp files
6. Add the new .cpp files to src/Makefile.am

1, 2 and 6 are not even coding tasks. 4 and 5 are non-consensus time, mostly dealing with JSON. 3 is the main work needed, which makes sense as a different 3 is what makes it a different contract. A lot of a contracts can use slightly modified functions from the other CC contracts. So the best way to do a new one would be to pick the one that is closest to what you want and start morphing it.

## General guidance on reporting issues on discord (Regarding CryptoConditions and SmartContract development):

* the specific chain parameters so anyone can connect to it
* the **EXACT** rpc call and parameters you used
*  **the most important!** the raw tx generated
*  clear description of why you think it is a bug. for now you need to look at the raw tx details to make sure all vins are valid, signed and all vouts are sane.
* Please don't post things like "i tried X and it didnt work" as that does not help at all at this stage. These are raw transaction level things and until everything works, things won't work at the higher level.

## Suggested RPC Commands
```
WIP
```
---
### [Frequently Aksed Questions (faq)](./faq.md)  
---
