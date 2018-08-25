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

### Check token balance of specific pubkey

Usage: `tokenbalance tokenid [pubkey]`

Example command:
```shell
./komodo-cli -ac_name=ATEST tokenbalance c5bbc34e6517c483afc910a3b0585c40da5c09b7c5d2d9757c5c5075e2d41b59 028bb4ae66aa4f1960a4aa822907e800eb688d9ab2605c8067a34b421748c67e27
```
output:
```JSON
{
  "result": "success",
  "CCaddress": "RQymbXA8FfWw2AaHv7oC8JRKo9W5HkFVMm",
  "tokenid": "c5bbc34e6517c483afc910a3b0585c40da5c09b7c5d2d9757c5c5075e2d41b59",
  "balance": 999900011
}
```
