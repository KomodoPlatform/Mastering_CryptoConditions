## Token List

This call will list all tokens that are available. Please report if this call is not displaying any tokenid.

To find detailed info about any specific token, use [`tokeninfo`](./tokeninfo.md) command using the `tokenid` from the list.

Example Command:  
```shell
$ ./komodo-cli -ac_name=ATEST tokenlist
```

Output:  
```JSON
[
  "307c094bce80205ec56abd43041530b0cd6faf449ea84cd2ae49339cfc3c222c",
  "e7d034fb7dbad561c9a86dcbcc64aa89e1d311891b4e7c744280b7de13b1186f",
  "21020a609c162fa2d0bc223acfff14bb0b886743303f5e4a661dade7a69b24a5",
  "c5bbc34e6517c483afc910a3b0585c40da5c09b7c5d2d9757c5c5075e2d41b59",
  "e4895451cae47f8f10303c3594888b739f044f7c778623318d877e8df365cc66",
  "045a31b7e38b1538d111ea87ad9ec53952a70e9a5e8d076f7ed7923d8723f02d",
  "f4131ee56a47273195a899f60a187862aa8e39a974b5a19d860e2fe69f60242f",
  "9217014eae0a83a0b64632f379c1b474859794f9eaf1cf1eecf5804ed6124a5e",
  "9eec77a3e02dec0ca60ead7e8cfb6cb6809c40fe54b804e51d5c6c2a445ffbf3",
  "43850dfce744581ef44775086625745adecd628993c5ff4c1c786cfd21009add"
]
```
