## Configurate your own Tenderly Devnet state

**Goal**: in this repo we are going to modify CryptoPunks contract to get a Punk NFT into our desired address. We will use [Tenderly devnets](https://docs.tenderly.co/devnets/yaml-template) for that purpose.

1. Get the contract address
    - The punks contract address is `0xb47e3cd837ddf8e4c57f05d70ab865de6e193bbb`

2. Get the contract code from [Etherscan](https://etherscan.io/address/0xb47e3cd837ddf8e4c57f05d70ab865de6e193bbb#code)
    - Identify which variable you want to change. In our case, we want to change the mapping where the token owners are set, so it is the `mapping(uint => address) punkIndexToAddress`
2. Get the storage layout

![storage](/CryptoPunksMarket.svg)

3. Get the slot address
    - The key is the crypto punk id. We choose to own the Crypto punk with id 1, so we need to left pad this to a 32 bytes value `0x0000000000000000000000000000000000000000000000000000000000000001`
    - The mapping is declared at storage slot position 10, padded to 32 bytes would be
    `0x000000000000000000000000000000000000000000000000000000000000000A`
    - Concatenating the 2 values we get
    `0x0000000000000000000000000000000000000000000000000000000000000001000000000000000000000000000000000000000000000000000000000000000A`
    - Calculate the keccak256 (notice we should pass the value in lower case and input type hex). You can calculate it [here](https://emn178.github.io/online-tools/keccak_256.html)
    - The resulting slot is `0xbbc70db1b6c7afd11e79c0fb0051300458f1a3acb8ee9789d9b6b26c61ad9bc7`
4. Set the value you want
    - In this case we pass our address where we want to store the punk, again padded to 32 bytes `0x0000000000000000000000007Be9763a718C0539017E2Ab6fC42853b4aEeb6B`
5. We are now ready to set the slot key value pair in our [devnet config file](./devnet-config.yml)

