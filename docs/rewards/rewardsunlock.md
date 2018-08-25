## Rewards Unlock
This will unlock your funds from specific Rewards Plan after the minimum lock time is met. Otherwise, you will get the followin error:
```JSON
APR 5.00000000 minseconds.86400 maxseconds.864000 mindeposit 10.00000000
minseconds 86400 maxseconds 864000
duration 72347 (1533050138 - 1532977791)
duration 72347 < minseconds 86400
amount 500.00000000 -> reward 0.00000000
error code: -1
error message:
basic_string::_M_construct null not valid
```

Usage: `rewardsunlock name fundingtxid [txid]`

### Step 1: Create raw transaction
Example Command:
```shell
./komodo-cli -ac_name=ATEST rewardsunlock FREE c0c5165902fd21728aed707888db082a72a013def8902c21626c1e5214f00fae 86185406f5836b60a381f254d743983b3719bf61cee5ca0c07dbd0806aee1094
```
Output:
```JSON

```
### Step 2: Broadcast raw transaction
Example Command:

```
Output:
```JSON

```
### Step 3: Decode raw transaction (optional to check if the values are sane)
Example Command:
```shell

```
Output:
```JSON

```
