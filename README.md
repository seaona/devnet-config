# Configurate your own Tenderly Devnet state

**Goal**: in this repo we are going to modify CryptoPunks contract to get a Punk NFT into our desired address. We will use [Tenderly devnets](https://docs.tenderly.co/devnets/yaml-template) for that purpose.

1. Get the contract address
    - The punks contract address is `0xb47e3cd837ddf8e4c57f05d70ab865de6e193bbb`

2. Get the contract code from [Etherscan](https://etherscan.io/address/0xb47e3cd837ddf8e4c57f05d70ab865de6e193bbb#code)
    - Identify which variable you want to change. In our case, we want to change the mapping where the token owners are set, so it is the `mapping(uint => address) punkIndexToAddress` and the `mapping(address=>uint256) balanceOf`
3. Get the storage layout

![storage](/CryptoPunksMarket.svg)

Now we can observe which slots we'll like to modify. In this case, I'll modify the `punkIndexToAddress` mapping and the `balanceOf` mapping, corresponding to slots 10 and 11.

### punkIndexToAddress
1. Get the slot address
    - The key is the crypto punk id. We choose to own the Crypto punk with id 1, so we need to left pad this to a 32 bytes value `0x0000000000000000000000000000000000000000000000000000000000000001`
    - The mapping is declared at storage slot position 10, padded to 32 bytes would be
    `0x000000000000000000000000000000000000000000000000000000000000000A`
    - Concatenating the 2 values we get
    `0x0000000000000000000000000000000000000000000000000000000000000001000000000000000000000000000000000000000000000000000000000000000A`
    - Calculate the keccak256 (notice we should pass the value in lower case and input type hex). You can calculate it [here](https://emn178.github.io/online-tools/keccak_256.html)
    - The resulting slot is `0xbbc70db1b6c7afd11e79c0fb0051300458f1a3acb8ee9789d9b6b26c61ad9bc7`
2. Set the value you want
    - In this case we pass our address where we want to store the punk, again padded to 32 bytes `0x00000000000000000000000007be9763a718c0539017e2ab6fc42853b4aeeb6b`

### balanceOf
1. Get the slot address
    - The key is the address, we need to left pad this to a 32 bytes value `0x0000000000000000000000007Be9763a718C0539017E2Ab6fC42853b4aEeb6B`
    - The mapping is declared at storage slot position 11, padded to 32 bytes would be
    `0x000000000000000000000000000000000000000000000000000000000000000B`
    - Concatenating the 2 values we get
    `0x0000000000000000000000007Be9763a718C0539017E2Ab6fC42853b4aEeb6B000000000000000000000000000000000000000000000000000000000000000B`
    - Calculate the keccak256 (notice we should pass the value in lower case and input type hex). You can calculate it [here](https://emn178.github.io/online-tools/keccak_256.html)
    - The resulting slot is `0x65ca9b454c6468268ab14e3b8f391b420b731f3aa9ac0f4a1eb0a190aaebdcbe`
2. Set the value you want
    - In this case we pass the punk id we want to own (1), again padded to 32 bytes `0x0000000000000000000000000000000000000000000000000000000000000001`


## Create the Config File
Now we can create the Tenderly config YAML file. We can copy the default template and add our changes regarding the punk contract.
[Here](./devnet.yml) is the result. I've also included some ERC20 and ETH balance to my account too.

## Validate results with eth_getStorageAt request
Now we are going to validate that the sotrage has been correctly modified by using the `eth_getStorageAt` method on the Tenderly RPC we just setup
- POST request with the following JSON body. Notice we cannot point directly to the slot,  as it's a `mapping` variable

```
{
    "jsonrpc":"2.0",
    "method":"eth_getStorageAt",
    "params":[
        "0xb47e3cd837ddf8e4c57f05d70ab865de6e193bbb", // contract address
        "0xbbc70db1b6c7afd11e79c0fb0051300458f1a3acb8ee9789d9b6b26c61ad9bc7", // storage address
        "latest"
    ],
    "id":1
}

- We get the following response, with our address as the owner of the NFT with id 1
```
{
    "id": 1,
    "jsonrpc": "2.0",
    "result": "0x00000000000000000000000007be9763a718c0539017e2ab6fc42853b4aeeb6b"
}


- Now we can also make a call for checking the reverse. We pass our address to get our tokens. In this case the request is:

```
{
    "jsonrpc":"2.0",
    "method":"eth_getStorageAt",
    "params":[
        "0xb47e3cd837ddf8e4c57f05d70ab865de6e193bbb", // 436
        "0x65ca9b454c6468268ab14e3b8f391b420b731f3aa9ac0f4a1eb0a190aaebdcbe",
        "latest"
    ],
    "id":1
}
```

- And we get the following response

```
{
    "id": 1,
    "jsonrpc": "2.0",
    "result": "0x0000000000000000000000000000000000000000000000000000000000000001"
}
```

You can check that if you change the endpoint to Infura, the results are displaying the real owner of the asset.