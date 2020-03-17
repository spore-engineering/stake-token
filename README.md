# STAKE Token Distribution
A repository for STAKE token initialization and distribution used by POSDAO implementation

## Related links:
- Security audit: 
  - [in xDai's GitHub](https://github.com/xdaichain/stake-token/blob/master/audit/Quantstamp/DPOS%20token-Audit%20Final%20Report.pdf)
  - [in Quantstamp's GitHub](https://github.com/quantstamp/DPOS-token-review)
- Documentation: https://poanetwork.github.io/dpos-token/
- Distribution model: https://forum.poa.network/t/dpos-staking-token-rewards-and-emission-model/2469
- POSDAO contracts repository: https://github.com/poanetwork/posdao-contracts
- POSDAO White Paper: https://forum.poa.network/t/posdao-white-paper/2208

## Usage
### Install Dependencies
```
npm install
```
### Testing
```
npm run test
```

### Deployment and initialization

1. For both Private Offering rounds deploy a separate `MultipleDistribution` contract, pass the `_privateOfferingParticipants`, `_privateOfferingParticipantsStakes` arrays to its `addParticipants` function, and then call `finalizeParticipants` function. \
\
The `finalizeParticipants` function will add `address(0)` to the participant set if the share of the `address(0)` is not zero.\
\
The `addParticipants` and `finalizeParticipants` functions can also be called right before the `Distribution` contract initialization.

2. Deploy the `Distribution` contract. Pass the addresses of all participants (including the `PrivateOffering` addresses) to its constructor.

3. For both `PrivateOffering` contracts call the `MultipleDistribution.setDistributionAddress` to set the address of the `Distribution` contract inside the `PrivateOffering` contract.

4. Deploy the `ERC677BridgeToken` contract (and pass `Distribution` and `PrivateOffering` contracts addresses to the constructor).

5. Call `preInitialize` function of the `Distribution` contract with `ERC677BridgeToken` address as a parameter. It releases `Public Offering` and `Liquidity Fund` tokens.

6. Call `initialize` function of the `Distribution` contract. It releases Private Offering tokens, and the countdown for cliff periods and installments starts from this moment.

### Test deployment and initialization (Kovan)
Run your local node.
Uncomment the lines with `preInitialize` and `initialize` calls in `2_deploy_contracts.js` and run:
```
 ECOSYSTEM_FUND_ADDRESS=0xb28a3211ca4f9bf8058a4199acd95c999c4cdf3b PUBLIC_OFFERING_ADDRESS=0x975fe74ec9cc82afdcd8393ce96abe039c6dba84 FOUNDATION_REWARD_ADDRESS=0xb68d0a5c0566c39e8c2f8e15d8494032fd420da1 LIQUIDITY_FUND_ADDRESS=0x7f29ce8e46d01118888b1692f626d990318018ea PRIVATE_OFFERING_DATA_1=./example.csv PRIVATE_OFFERING_DATA_2=./example.csv ./node_modules/.bin/truffle migrate --reset --network kovan
```
Note: don't forget to change the input data.

## User roles

### Owner

The owner is supposed to be a MultiSig Wallet contract. The owner can only call the following functions:

- `ERC677BridgeToken.transferOwnership` to transfer ownership of the `ERC677BridgeToken` contract to another address;
- `ERC677BridgeToken.setBridgeContract` to set the address of bridge contract;
- `ERC677BridgeToken.claimTokens` to transfer coins or specified tokens to the specified address if someone sent coins/tokens to the contract mistakenly;
- `Distribution.transferOwnership` to transfer ownership of the `Distribution` contract to another address;
- `Distribution.preInitialize` to pre-initialize the `Distribution` contract (unlock tokens for `Public Offering` and `Liquidity Fund`);
- `Distribution.initialize` to initialize the `Distribution` and `PrivateOffering` contracts;
- `MultipleDistribution.transferOwnership` to transfer ownership of the `MultipleDistribution` contract to another address;
- `MultipleDistribution.addParticipants` to add MultipleDistribution participants before initializing;
- `MultipleDistribution.finalizeParticipants` to finalize the list of MultipleDistribution participants before initializing;
- `MultipleDistribution.setDistributionAddress` to set the `Distribution` contract address;
- `MultipleDistribution.burn` to burn unallocated tokens (send them to `address(0)`).

### Any address

The following methods can be called by anyone:

- `ERC677BridgeToken` public methods (`transferAndCall`, `transfer`, `transferFrom`, `approve`, `increaseAllowance`, `decreaseAllowance`);
- `Distribution.makeInstallment` to transfer daily installment to specified pool;
- `Distribution.initialize` (if 90 days after pre-initialization are expired) to initialize the `Distribution` and `PrivateOffering` contracts.

### Private Offering participant

- `MultipleDistribution.withdraw` to withdraw participant share.
