## Token Balance

### Check your own token balance

Usage: `tokenbalance tokenid`

Example command:
```shell
./komodo-cli -ac_name=ATEST tokenbalance c5bbc34e6517c483afc910a3b0585c40da5c09b7c5d2d9757c5c5075e2d41b59
```
output:
```JSON
{
  "result": "success",
  "CCaddress": "RRPpWbVdxcxmhx4xnWnVZFDfGc9p1177ti",
  "tokenid": "c5bbc34e6517c483afc910a3b0585c40da5c09b7c5d2d9757c5c5075e2d41b59",
  "balance": 99989
}
```

### Check token address for a specific pubkey

Usage: `tokenaddress [pubkey]`

Example command:
```shell
./komodo-cli -ac_name=ATEST tokenaddress 028702e30d8465d6aa85f35d2f58c06a6ee17f23f376b56044dadf7b793f2c12b9
```
output:
```JSON
{
  "result": "success",
  "AssetsCCaddress": "RGKRjeTBw4LYFotSDLT6RWzMHbhXri6BG6",
  "Assetsmarker": "RFYE2yL3KknWdHK6uNhvWacYsCUtwzjY3u",
  "CCaddress": "RG6mr23tQ9nUhmi5GEnYqjfkqZt9x2MRXz",
  "myCCaddress": "RG6mr23tQ9nUhmi5GEnYqjfkqZt9x2MRXz",
  "myaddress": "RDjG4sM1y4udiJSszF6BLotqUnZX79Rom9"
}

```
