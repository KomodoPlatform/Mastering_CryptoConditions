## Dice Smart Contract


### Available RPC calls

==Dice==  
[diceaddfunds name fundingtxid amount](./diceaddfunds.md)  
[diceaddress [pubkey]](./diceaddress.md)  
[dicebet name fundingtxid amount odds](./dicebet.md)  
[dicefinish name fundingtxid bettxid](./dicefinish.md)  
[dicefund name funds minbet maxbet maxodds timeoutblocks](./dicefund.md)  
[diceinfo fundingtxid](./diceinfo.md)  
[dicelist](./dicelist.md)  
[dicestatus name fundingtxid bettxid](./dicestatus.md)  

The diceplan creator needs to be running dicestatus in a crontask or some regular frequency. This finishes any unfinished bets and also creates entropy tx.

Additionally, you need to create txids with hashed entropy, basically any dice tx other than a dicebet will add hashed entropy, but you need to create a few at first via diceaddfunds.

The diceinfo, dicelist, diceaddress work just like the rewards counterparts.

Once there is a dice plan with funds, you can make dicebets. For now to resolve it the creator of the diceplan needs to do a dicewinner or diceloser. 

Last I will add a dicerefund that allows anybody to undo a dicebet, this would happen only if the diceplan node is offline. It could be that it refunds or it becomes an automatic win.

In order to save a step, I dont hash the entropy of the dicebet. But I guess I need to, if we want it to refund after timeout instead of automatic win, as the way it is now would allow the house account to just not complete a large losing bet.

The dicefund creator can actually finalize any dicebet transactions and it should properly deal with paying winners and not paying losing tx.

In the event a house account tries to cheat by not dicefinish for winning bets, ie. trying to not payout. When the dice plan's expiration happens, it is treated as a win and anybody can complete the dicefinish.

When the dicefinish completes a bet for either win or loss, it attaches the original entropy value so the hash of it can be verified. In other words it is provably fair and random for each and every bet.The dicebet is the one that chooses what house entropy to use, so that alone gives the power to determine the outcome to the dicebettoer. And as long as the bettor's entropy is a high entropy value, the outcome is totally random.

Technically I generate 2 256 bit numbers from the two entropy values. I just SHA256(house entropy + bettor entropy) for the house and SHA256(bettor entropy + house entropy) for the bettor. Then for odds of > 1, the entropy value is divided by the odds and the two numbers compared. The bettor value adjusted by odds needs to be bigger than the house value.

I think the payout matches the risk. for a 1:1, the two values are directly compared and a win is 2x what was bet.
