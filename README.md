# Valkyrie v1.0.0 Documentation
Valkyrie is a decentralized marketplace protocol on Vechain. It provides the infrastructure to build markets and auctions for non-fungible tokens following the VIP181 standard.

### Relayer Model
Valkyrie uses a relayer model where bids and offers are off-chain VIP190-compliant signed messages. Only bid/offer cancellation and auction completion are on-chain transactions. This allows for a more efficient platform while maintaining the security benefits of on-chain auctions.

To use the protocol, a user must `approve` Valkyrie's contract for the tokens and NFT they wish to swap. This is part of the VIP180/181 standards and allows Valkyrie to settle auctions on behalf of the users involved.

In this model the relayer collects and stores orders from bidders and sellers, and displays them given its own choice in logic. The protocol provides the flexibility to design the auctions per your relayers goals, while ensuring the flexibility of the settlement.

![image](https://user-images.githubusercontent.com/747165/57430576-006b6b00-71e5-11e9-9657-817eb5e90c4e.png)

### Order Model
Both bids and offers are packaged into an abstract `Order` object which represents the message to be signed and relayed.
```
interface Order {
  orderType: OrderType,
  originatorAddress: string,
  collectibleAddress: string,
  collectibleId: BigNumber,
  tokenAddress: string,
  tokenAmount: BigNumber,
  feeAddress: string,
  feeAmount: BigNumber,
  expiration: BigNumber,
  salt: BigNumber,
}

enum OrderType {
  BID = 0,
  OFFER = 1
}
```

##### `orderType`
`BID` - A bidder is placing a bid on a token up for auction.

`OFFER` - A seller is placing an executable offer on their own NFT (buy it now price).

##### `originatorAddress`
The originator address is bidder's address for `BID` orders, and the seller's address for `OFFER` orders.

##### `collectibleAddress` and `collectibleId`
Designates the NFT being bid on or sold.

##### `tokenAddress` and `tokenAmount`
The VIP180 token and amount of that token that is willing to be swapped for the specified NFT. For `BID` this designates the bidder's bid price, while for `OFFER` it designates the sellers buy-it-now price.

**Note** Valkyrie only works with VIP180 tokens, not VET itself. For auctions denominated in VET, please use [VET+](#vet) as a wrapper token.
##### `feeAddress`
The address that should collect the fee, usually the relayer's wallet address.

##### `feeAmount`
The amount of the fee which is deducted from the `tokenAmount` and distributed to `feeAddress` upon auction completion. Fee must be in the same token as the order itself. 

**Note** This is an amount and not a percentage. This value must be smaller than `tokenAmount`. This final value distributed to the seller upon completion is `tokenAmount` - `feeAmount`.

##### `expiration`
Expiration time of the order in unix time (seconds). It is recommended to set this to the auction close time for `OFFER` orders, and auction close time plus some completion time buffer (1-3 days) for `BID` orders.

##### `salt`
Salt can be any 32-byte number. Recommended as a random 32-byte value or timestamp.

### Signature Scheme
The signature is derived using VIP190 on the blake2b hash of the solidity-packed order.

A javascript example of computing the order hash to be signed:
```
    import { cry } from "thor-devkit";
    import * as abi from 'ethereumjs-abi';
    import * as _ from 'lodash';

    const orderAbi = [
      {value: valkyrieAddress, type: "address"}, // Address of the Valkyrie contract
      {value: order.orderType, type: "uint8"},
      {value: order.originatorAddress, type: "address"},
      {value: order.collectibleAddress, type: "address"},
      {value: order.collectibleId, type: "uint256"},
      {value: order.tokenAddress, type: "address"},
      {value: order.tokenAmount, type: "uint256"},
      {value: order.feeAddress, type: "address"},
      {value: order.feeAmount, type: "uint256"},
      {value: order.expiration, type: "uint256"},
      {value: order.salt, type: "uint256"},
    ];

    const types = _.map(orderAbi, o => o.type);
    const values = _.map(orderAbi, o => o.value);
    const hashBuff = cry.blake2b256(abi.solidityPack(types, values));
    return '0x' + hashBuff.toString('hex');
```

The signature may be obtained via web3's personal sign:
```
web3.eth.personal.sign(orderHash, signerAddress)
```

### Valkyrie Contract Addresses
Valkyrie's contract is deployed at:

`Mainnet` - `0x9c3f984b8567548e9de74174a5109992aa61b5ad`

`Testnet` - `0x5b246b6ed49fed9503f2ebbf5824a111aac38f6d`

### ABI specification

View full ABI of the Valkyrie contract [here](https://github.com/totient-labs/valkyrie-docs/blob/master/abi/valkyrie.abi).

### VET+
VET+ is a wrapper token to add VIP180 functionality to VET. For security reasons, VET+ is identical to [WETH](https://weth.io/) in all but name, redeployed on Vechain.

The VET+ contract is deployed at:

`Mainnet` - `0xB69DEd9F0Da15D240ee6803dacd7FCF68744e8ff`

`Testnet` - `0x4086B2D7eb716CA7C81B680792B8ecd2cfa2a7c8`