# Dice Info
Displays informationa about the specific Dice Contract using `fundingtxid`. Use [`dicelist`](./dicelist.md) to get a list of tx hashes that you can use here.

Usage: `diceinfo fundingtxid`
### Example
Command:
```shell
./komodo-cli -ac_name=ATEST diceinfo 0d6e82af9959caec14d7af42fd67db68a45bcd23c755457ebf192a52d62c599c
```
Output:
```JSON
{
  "result": "success",
  "fundingtxid": "0d6e82af9959caec14d7af42fd67db68a45bcd23c755457ebf192a52d62c599c",
  "name": "dice",
  "sbits": 1701013860,
  "minbet": "1.00000000",
  "maxbet": "100.00000000",
  "maxodds": 10,
  "timeoutblocks": 5,
  "funding": "1000.00000000"
}
```
