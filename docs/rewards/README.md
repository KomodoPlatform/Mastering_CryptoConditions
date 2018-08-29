## What is rewards contract?
Rewards CC allows to create a master-node like rewards program. Which gives the user the ability to earn rewards by locking coins.  
APR and mindeposit are configurable per rewards plan (there can be many active at any given time), even the required holding period.

Note: If you create a plan with `mindeposit: 10000`, make sure you have added 10000 + tx fees using the `rewardsaddfunding` call after creating the plan. The Rewards contract is set to require deposit amount of funding in rewards plan as assurance it will have the funds needed to pay.

## Available RPC Calls

== Rewards ==  
[rewardsaddfunding name fundingtxid amount](./rewardsaddfunding.md)  
[rewardsaddress [pubkey]](./rewardsaddress.md)  
[rewardscreatefunding name amount APR mindays maxdays mindeposit](./rewardscreatefunding.md)  
[rewardsinfo fundingtxid](./rewardsinfo.md)  
[rewardslist](./rewardslist.md)  
[rewardslock name fundingtxid amount](./rewardslock.md)  
[rewardsunlock name fundingtxid [txid]](./rewardsunlock.md)  

