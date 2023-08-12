
# Solana Compressed NFT (cNFT) Setup
An unnofficial guide and library of scripts for setting up compressed NFT collections on Solana. 

[![Discord](https://img.shields.io/badge/Discord-%235865F2.svg?style=for-the-badge&logo=discord&logoColor=white)
](https://discord.gg/qYtg533qze)

#### Start a New Project
Click the big green "Use Template" button to start a new project on your account.

#### System Requirements
- [Node.js](https://nodejs.org/en)
- [Solana CLI](https://docs.solana.com/cli/install-solana-cli-tools)

## Getting Started
To mint compressed NFTs we need to setup a few things.
1. Generate keypairs.
2. Prepare/upload collection metadata.
3. Create a collection NFT.
4. Create merkle tree.

### 1. Generate Keypairs
The following command will generate two new keypairs in the current directory that we will use in future steps.

```bash
npm run generate-keypairs
```

This should result in in two new files added to the current directory. 

- `./payer-keypair.json`
- `./tree-keypair.json`

### 2. Fund Payer Keypair
The `payer-keypair` will be the primary key we use as the authority for everything and the fee payer. In order to use it, you'll need to drop some SOL and obtain $SHDW token.

#### View in Phantom
If you can use your own scripts to prepare funds, that's preferred. But if you want UI for managing your SOL and swapping for $SHDW, you can import the value of `payer-keypair.json` into something like Phantom.
_Don't connect that wallet to any websites._ 

#### Recommended Funding
The exact amounts will vary depending on your tree size and how much data you want to store, but this should be a good starting point.
1. Add `0.5 SOL` to the payer wallet.
2. Swap `0.1 SOL` for $SHDW token.

### 3. Create Storage Account
We will use [Shadow Drive](docs.shadow.cloud) to store our files on-chain. First, we need to define a place to keep these files. Run the following script to define a storage account with `1GB` of space. Adjust the script in `package.json` as needed.

`npm run create:storage-account`

If successful, a `storageAccount` property will be added to your `package.json`.

### 4. Upload Metadata
Now that we have a storage account we can upload our collection and NFT metadata. In the case of this example we will have static metadata for every NFT but you can check the [OPOS Outliers](https://github.com/TipLink/opos-outliers) GitHub repo for examples on how you could do dynamic minting.

#### Upload Collection Metadata
The following will upload the file located at `./metadata/collection-metadata.json` and name the file `collection.json`.

```bash
npm run upload:collection-metadata
```

If successful, a `collectionMetadataUrl` property will be added to the `package.json`.

#### Upload Collection Metadata
The following will upload the file located at `./metadata/nft-metadata.json` and give it a random name.

```bash
npm run upload:nft-metadata
```

If successful, a `nftMetadataUrl` property will be added to the `package.json`.

### 5. Create Collection
To mint all of our NFTs to a collection, we need to mint a collection NFT that has an ID and metadata that represents our collection.

#### Mint Collection
Run the following to use `payer-keypair.json`, `collection-metadata.json`, and `collectionMetadataUrl` from `package.json` to mint your collection NFT.

```bash
npm run create:collection
```

The details of the new collection will be added to `package.json` under `collectionNft`.

### 6. Create Merkle Tree
The merkle tree holds the state of our collection and who owns what.

#### Calculate Tree Size
The size and qualities of the tree depends on your projects needs. For this example I picked some defaults for us to work off of located in `package.json` under `treeConfig`. These defaults will create a tree with roughly `65k` available leaves (supply), cost about `0.35` SOL to make and have a max proof size of 8 making it compatible with Tensor.

##### Alternate Calculations
If you want to calculate your own values, I wrote a CLI to determine `maxBufferSize` and `maxDepth` values based on a desired max proof size. Run the following to calculate results for max proof size 8.

```bash
npx cnfts calculate 8
```

##### Deep Dive
If you want to learn more, I learned a lot from watching the following video by [Solandy](https://twitter.com/HeyAndyS).

-> [Compressed NFTs Deep Dive: What you need to know before you mint!](https://www.youtube.com/watch?v=nM3trQX2_5o)

#### Create Tree
Once you have calculated your tree values or you have decided to use the defaults, it's time to generate the merkle tree.

Run the following command to create the tree.
The `./tree-keypair.json` file will represent the merkle tree, and `./payer-keypair.json` will be used as the `treeCreator` and `payer`. The `maxBufferSize`, `maxDepth`, and `canopyDepth` will be reach from `package.json`.

```bash
npm run create:merkle-tree
```

If successful, a URL to preview the tree will be returned. Load that in your browser and make sure everything looks right. 

### 7. Mint Compressed NFT
Run the following command to mint an NFT to the `payer-keypair.json` wallet.

```bash
npm run create:nft
```