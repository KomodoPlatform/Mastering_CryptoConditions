## Create new Rewards Funding
Usage: `rewardscreatefunding name amount APR mindays maxdays mindeposit`

Note: If you create a plan with `mindeposit: 10000`, make sure you have added 10000 + tx fees using the `rewardsaddfunding` call after creating the plan. The Rewards contract is set to require deposit amount of funding in rewards plan as assurance it will have the funds needed to pay.

### Step 1: Create raw transaction HEX using your own parameter
For the example command we used
```
name = FREE # Name of your Rewards plan
amount = 1000 # Amount to start the Rewards plan
APR = 5% # Annual percentage of rewards
mindays = 1 # Minimum number of days the funds will be locked
maxdays = 10 # Maximum number of days the funds will be locked
mindeposit = 10 # Minimum deposit amount
```
Example Command:
```shell
./komodo-cli -ac_name=ATEST rewardscreatefunding FREE 1000 5 1 10 10
```
Output:
```JSON
{
  "result": "success",
  "hex": "01000000015022a9ead688a68f018bf18f44ad5a04689edacfcd68c62283d6113d24e647b50000000048473044022021776c2a758d6e9f8dda9e97ca088a777656759b0cf4426ce2e23a04bfa915880220180b674b90ea812b7f20b159b6a03e454fce6fe87bd6261b9cb65f21b142006401ffffffff0400e8764817000000302ea22c802065686d47a4049c2c845a71895a915eb84c04445896eec5dc0be40df0b31372da8103120c008203000401cc1027000000000000232103da60379d924c2c30ac290d2a86c2ead128cb7bd571f69211cb95356e2dcc5eb9ace069fb0501090000232103fe754763c176e1339a3f62ee6b9484720e17ee4646b65a119e9f6370c7004abcac00000000000000002c6a2ae54646524545000000000065cd1d000000008051010000000000002f0d000000000000ca9a3b0000000000000000"
}
```
### Step 2: Broadcast/send the raw hex/transaction

This will output you the txid which is the `fundingtxid` or the Rewards plan id.

Example Command:
```shell
./komodo-cli -ac_name=ATEST sendrawtransaction 01000000015022a9ead688a68f018bf18f44ad5a04689edacfcd68c62283d6113d24e647b50000000048473044022021776c2a758d6e9f8dda9e97ca088a777656759b0cf4426ce2e23a04bfa915880220180b674b90ea812b7f20b159b6a03e454fce6fe87bd6261b9cb65f21b142006401ffffffff0400e8764817000000302ea22c802065686d47a4049c2c845a71895a915eb84c04445896eec5dc0be40df0b31372da8103120c008203000401cc1027000000000000232103da60379d924c2c30ac290d2a86c2ead128cb7bd571f69211cb95356e2dcc5eb9ace069fb0501090000232103fe754763c176e1339a3f62ee6b9484720e17ee4646b65a119e9f6370c7004abcac00000000000000002c6a2ae54646524545000000000065cd1d000000008051010000000000002f0d000000000000ca9a3b0000000000000000
```
Output:
```JSON
c0c5165902fd21728aed707888db082a72a013def8902c21626c1e5214f00fae
```
### Step 3: Decode the raw transaction (optional to check if the values are sane)
Example Command:
```shell
./komodo-cli -ac_name=ATEST decoderawtransaction 01000000015022a9ead688a68f018bf18f44ad5a04689edacfcd68c62283d6113d24e647b50000000048473044022021776c2a758d6e9f8dda9e97ca088a777656759b0cf4426ce2e23a04bfa915880220180b674b90ea812b7f20b159b6a03e454fce6fe87bd6261b9cb65f21b142006401ffffffff0400e8764817000000302ea22c802065686d47a4049c2c845a71895a915eb84c04445896eec5dc0be40df0b31372da8103120c008203000401cc1027000000000000232103da60379d924c2c30ac290d2a86c2ead128cb7bd571f69211cb95356e2dcc5eb9ace069fb0501090000232103fe754763c176e1339a3f62ee6b9484720e17ee4646b65a119e9f6370c7004abcac00000000000000002c6a2ae54646524545000000000065cd1d000000008051010000000000002f0d000000000000ca9a3b0000000000000000
```
Output:
```
{
  "txid": "c0c5165902fd21728aed707888db082a72a013def8902c21626c1e5214f00fae",
  "size": 321,
  "version": 1,
  "locktime": 0,
  "vin": [
    {
      "txid": "b547e6243d11d68322c668cdcfda9e68045aad448ff18b018fa688d6eaa92250",
      "vout": 0,
      "scriptSig": {
        "asm": "3044022021776c2a758d6e9f8dda9e97ca088a777656759b0cf4426ce2e23a04bfa915880220180b674b90ea812b7f20b159b6a03e454fce6fe87bd6261b9cb65f21b142006401",
        "hex": "473044022021776c2a758d6e9f8dda9e97ca088a777656759b0cf4426ce2e23a04bfa915880220180b674b90ea812b7f20b159b6a03e454fce6fe87bd6261b9cb65f21b142006401"
      },
      "sequence": 4294967295
    }
  ],
  "vout": [
    {
      "value": 1000.00000000,
      "valueSat": 100000000000,
      "n": 0,
      "scriptPubKey": {
        "asm": "a22c802065686d47a4049c2c845a71895a915eb84c04445896eec5dc0be40df0b31372da8103120c008203000401 OP_CHECKCRYPTOCONDITION",
        "hex": "2ea22c802065686d47a4049c2c845a71895a915eb84c04445896eec5dc0be40df0b31372da8103120c008203000401cc",
        "reqSigs": 1,
        "type": "cryptocondition",
        "addresses": [
          "RTsRBYL1HSvMoE3qtBJkyiswdVaWkm8YTK"
        ]
      }
    },
    {
      "value": 0.00010000,
      "valueSat": 10000,
      "n": 1,
      "scriptPubKey": {
        "asm": "03da60379d924c2c30ac290d2a86c2ead128cb7bd571f69211cb95356e2dcc5eb9 OP_CHECKSIG",
        "hex": "2103da60379d924c2c30ac290d2a86c2ead128cb7bd571f69211cb95356e2dcc5eb9ac",
        "reqSigs": 1,
        "type": "pubkey",
        "addresses": [
          "RMgye9jeczNjQx9Uzq8no8pTLiCSwuHwkz"
        ]
      }
    },
    {
      "value": 98999.99980000,
      "valueSat": 9899999980000,
      "n": 2,
      "scriptPubKey": {
        "asm": "03fe754763c176e1339a3f62ee6b9484720e17ee4646b65a119e9f6370c7004abc OP_CHECKSIG",
        "hex": "2103fe754763c176e1339a3f62ee6b9484720e17ee4646b65a119e9f6370c7004abcac",
        "reqSigs": 1,
        "type": "pubkey",
        "addresses": [
          "RANyPgfZZLhSjQB9jrzztSw66zMMYDZuxQ"
        ]
      }
    },
    {
      "value": 0.00000000,
      "valueSat": 0,
      "n": 3,
      "scriptPubKey": {
        "asm": "OP_RETURN e54646524545000000000065cd1d000000008051010000000000002f0d000000000000ca9a3b00000000",
        "hex": "6a2ae54646524545000000000065cd1d000000008051010000000000002f0d000000000000ca9a3b00000000",
        "type": "nulldata"
      }
    }
  ]
}
```
