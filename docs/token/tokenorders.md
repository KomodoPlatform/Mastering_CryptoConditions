## Displaying the orderbook

Usage: `tokenorders [tokenid]`

Good to know:
```
Fill: 'b' is the offer, 'B' is the fill
Ask: 's' is the offer, 'S' is the ask
fillbid for bid
fillask for ask
```

If you place an offer, until someone fills the order, it just sits there. 

**buy offer, bid**  
tokenbid -> 'b'  
tokenbidfill -> 'B'  
You place a buy order with 'b'  
Someone else fills it with 'B'  

**sell offer, ask**
tokenask -> 's'  
tokenaskfill -> 'S'  
You place a sell order with 's'  
Someone else fills it with 'S'  


### Show all available orders

Example Command:
```shell
./komodo-cli -ac_name=ATEST tokenorders
```
Output:
```JSON
[
  {
    "funcid": "B",
    "txid": "b9d305e9b6a82e715efce9b6244cc15fef131baf1893a7eb45b199c23b3fb806",
    "vout": 0,
    "amount": 0,
    "bidamount": 0,
    "origaddress": "RQymbXA8FfWw2AaHv7oC8JRKo9W5HkFVMm",
    "tokenid": "c5bbc34e6517c483afc910a3b0585c40da5c09b7c5d2d9757c5c5075e2d41b59"
  },
  {
    "funcid": "b",
    "txid": "45b3f7874fc4a2699729a9792bc7679f6b5f11035a29ad9f661425b19534dd1d",
    "vout": 0,
    "amount": 1000,
    "bidamount": 1000,
    "origaddress": "RQymbXA8FfWw2AaHv7oC8JRKo9W5HkFVMm",
    "tokenid": "c5bbc34e6517c483afc910a3b0585c40da5c09b7c5d2d9757c5c5075e2d41b59",
    "totalrequired": 1000,
    "price": 1
  },
  {
    "funcid": "B",
    "txid": "d4643ce47e9799681a4549468d47c85337367f0ef2733afe1d79c50175e6ae32",
    "vout": 0,
    "amount": 0,
    "bidamount": 0,
    "origaddress": "R9sDyKt2kW5uJaoZT6GF9e3WRbGioBuhoZ",
    "tokenid": "e7d034fb7dbad561c9a86dcbcc64aa89e1d311891b4e7c744280b7de13b1186f"
  },
  {
    "funcid": "B",
    "txid": "0909df82ade3193c9a630dd80947141f34489732e9a2f8346790304ebbdcc251",
    "vout": 0,
    "amount": 0,
    "bidamount": 0,
    "origaddress": "R9sDyKt2kW5uJaoZT6GF9e3WRbGioBuhoZ",
    "tokenid": "e7d034fb7dbad561c9a86dcbcc64aa89e1d311891b4e7c744280b7de13b1186f"
  },
  {
    "funcid": "b",
    "txid": "a8d60a3ce429ccb885ad445e7a4534130a35d2424d1883c6513d0f4da2fe9a92",
    "vout": 0,
    "amount": 150,
    "bidamount": 150,
    "origaddress": "RQymbXA8FfWw2AaHv7oC8JRKo9W5HkFVMm",
    "tokenid": "c5bbc34e6517c483afc910a3b0585c40da5c09b7c5d2d9757c5c5075e2d41b59",
    "totalrequired": 100,
    "price": 1.5
  },
  {
    "funcid": "B",
    "txid": "03e118fc442a223df4dd87add64f142e1bfd99baee94c8be26bc77ed809d50a4",
    "vout": 0,
    "amount": 0,
    "bidamount": 0,
    "origaddress": "R9sDyKt2kW5uJaoZT6GF9e3WRbGioBuhoZ",
    "tokenid": "e7d034fb7dbad561c9a86dcbcc64aa89e1d311891b4e7c744280b7de13b1186f"
  }
]
```

### Show orders for specific token
Usage: `tokenorders [tokenid]`

Example Command:
```shell
./komodo-cli -ac_name=ATEST tokenorders c5bbc34e6517c483afc910a3b0585c40da5c09b7c5d2d9757c5c5075e2d41b59
```
Output:
```JSON
[
  {
    "funcid": "B",
    "txid": "b9d305e9b6a82e715efce9b6244cc15fef131baf1893a7eb45b199c23b3fb806",
    "vout": 0,
    "amount": 0,
    "bidamount": 0,
    "origaddress": "RQymbXA8FfWw2AaHv7oC8JRKo9W5HkFVMm",
    "tokenid": "c5bbc34e6517c483afc910a3b0585c40da5c09b7c5d2d9757c5c5075e2d41b59"
  },
  {
    "funcid": "b",
    "txid": "9dabd8c01bb7d59455b64fe100617149c20cb4520d266183686aa4986fd3021d",
    "vout": 0,
    "amount": 100,
    "bidamount": 100,
    "origaddress": "RQymbXA8FfWw2AaHv7oC8JRKo9W5HkFVMm",
    "tokenid": "c5bbc34e6517c483afc910a3b0585c40da5c09b7c5d2d9757c5c5075e2d41b59",
    "totalrequired": 100,
    "price": 1
  },
  {
    "funcid": "b",
    "txid": "45b3f7874fc4a2699729a9792bc7679f6b5f11035a29ad9f661425b19534dd1d",
    "vout": 0,
    "amount": 1000,
    "bidamount": 1000,
    "origaddress": "RQymbXA8FfWw2AaHv7oC8JRKo9W5HkFVMm",
    "tokenid": "c5bbc34e6517c483afc910a3b0585c40da5c09b7c5d2d9757c5c5075e2d41b59",
    "totalrequired": 1000,
    "price": 1
  },
  {
    "funcid": "b",
    "txid": "a8d60a3ce429ccb885ad445e7a4534130a35d2424d1883c6513d0f4da2fe9a92",
    "vout": 0,
    "amount": 150,
    "bidamount": 150,
    "origaddress": "RQymbXA8FfWw2AaHv7oC8JRKo9W5HkFVMm",
    "tokenid": "c5bbc34e6517c483afc910a3b0585c40da5c09b7c5d2d9757c5c5075e2d41b59",
    "totalrequired": 100,
    "price": 1.5
  }
]
```
