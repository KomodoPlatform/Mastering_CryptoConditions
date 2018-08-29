# Dice Finish
Dice Finish rebroadcasts a bet that has already been broadcasted.

Usage: `dicefinish name fundingtxid bettxid`
### Example
Command:
```shell
./komodo-cli -ac_name=AT1 dicefinish DICE3 4132ca8e8d46df9f8a8cbe83c99794497e06bbd190bd71f4abcdedf84e90952e d54335073e549cd75a050fd4d6ba5939307cda7096ba0f3da779fb7d07e46343
```
Output:
```JSON
{
  "result": "success",
  "hex": "0"
}

```
If the bet has not finished or is stuck, `hex` will have a value that then needs to be broadcasted.
