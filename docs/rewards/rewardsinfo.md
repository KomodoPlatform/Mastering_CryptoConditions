## Information about specific reward contract
Usage: `rewardsinfo`

### Terminology
```
name = Name of the Rewards plan
sbits = The first 8 chars of the name as a 64 bit int
APR = Annual percentage of Rewards
minseconds = Minimum time the funds will be locked in seconds
maxseconds = Maximum time the funds will be locked in seconds
mindeposit = Minimum deposit amount
funding = The amount of funds the rewards plan contains
```

### Example
Command:
```shell
./komodo-cli -ac_name=ATEST rewardsinfo c0c5165902fd21728aed707888db082a72a013def8902c21626c1e5214f00fae
```

Output:
```JSON
{
  "result": "success",
  "fundingtxid": "c0c5165902fd21728aed707888db082a72a013def8902c21626c1e5214f00fae",
  "name": "FREE",
  "sbits": 1162170950,
  "APR": "5.00000000",
  "minseconds": 86400,
  "maxseconds": 864000,
  "mindeposit": "10.00000000",
  "funding": "1000.00000000"
}
```
