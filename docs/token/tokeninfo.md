## Token Info
This call shows all information about any token. Currently, it returns the following info:
- Token id string  
- Public key of the owner  
- Name of the token  
- Total token supply  
- Description of the token

Example Command:
```shell
./komodo-cli -ac_name=ATEST tokeninfo 43850dfce744581ef44775086625745adecd628993c5ff4c1c786cfd21009add
```
Output:  
```JSON
{
  "result": "success",
  "tokenid": "43850dfce744581ef44775086625745adecd628993c5ff4c1c786cfd21009add",
  "owner": "03fe754763c176e1339a3f62ee6b9484720e17ee4646b65a119e9f6370c7004abc",
  "name": "TAKA",
  "supply": "100000.00000000",
  "description": "Testing phase 3."
}
```
