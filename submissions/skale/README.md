## SKALE submission

Submission date: 30th March

### Proposal description

The SKALE Network provides infrastructure for projects to use any type of curve or ZKP scheme within a fast, containerized, EVM-compatible SKALE Chain. Since SKALE primarily focuses on infrastructure, we would like to comment on the requirements that impact scalability, speed, and cost to run the THEMIS protocol within a web3 cloud network.

With the SKALE Network, the THEMIS protocol can run zero cost transactions on a Proof-of-Stake chain distributed across 16 nodes, each running containerized EVM. The 16 nodes operating the SKALE Chain are selected out of 150 decentralized nodes, and randomly rotated to prevent collusion. SKALE chains can exchange transactions or messages with other SKALE Chains and Ethereum networks, allowing the Brave Browser to use the SKALE Network for powering the THEMIS protocol in addition to enabling BAT transfers to and from Ethereum or other chains.

### Changes to the THEMIS protocol
Three of the biggest challenges resulting from generating the proof of correct reward calculation, is storing the data in a fast, secure, and cost effective environment. The constraints added by the cost of gas on Ethereum mainnet, will make it costly to operate the THEMIS protocol without extreme optimizations required to lower gas consumption.

The proposal is to store data resulting from the THEMIS protocol on the SKALE Network, which would remove the per transaction gas cost, remove the requirement for batching data prior to storing on chain, and provide fast finality in less than 3 seconds per transaction. 

SKALE Chains are available in three sizes, and the size determines the amount of Transactions Per Second (TPS) & available block and EVM storage. SKALE Chains must first be created by staking SKALE Tokens, which makes the cost of running the THEMIS protocol known up front, without having to guess at a per transaction cost each time a transaction is sent to the SKALE Chain.


#### SKALE Chains Sizes and Cost
| | Small Chain | Medium Chain | Large Chain |
| -------- |-------- | -------- | -------- |
| Transactions Per Second | 16 TPS     | 125 TPS     | 2000 TPS     |
| Cost | 600 SKL per year    | 4,625 SKL per year    | 74,000 SKL per year     |



> Pricing is based off of the price of the SKL token. Currently the SKL token is estimated to be 0.71 USD USD per SKL token.

---
### Impact to THEMIS Protocol Goals and Property Values 

#### Goals
| Goal | Using SKALE Network |
| -------- | -------- |
| Support reward computation based on user ad interactions without leaking information about user behavior    | THE SKALE Network technology is based on BLS threshold signatures, Distributed Key Generation, SGX, and SKALE Consensus (Asynchronous Binary Byzantine Agreement consensus protocol), which all work together to provide a secure network for storing data on a blockchain. The SKALE Network would be able to store the data in EVM, and using any curve or ZKP scheme of choice, you can obfuscate the user's identity which will prevent user interactions from being leaked.  |
| Allow all participants to verify that the rewards are being correctly computed    | All data is publicly available on the SKALE Chain and can be accessed over JSON RPC using Web3, Ether JS, etc., which will allow anyone to verify the data at all times.  |
| Allow advertisers to verify whether budget metrics (how the campaign budget has been spent) provided by Brave are correct   | All data/events/transactions can be stored on SKALE Chains at zero-gas-cost and available for all to verify. All blocks signed by threshold supermajority of validator nodes.     |




|  | Property Requirement | Using SKALE Network |
| -------- | -------- | -------- |
| Property 1  | Privacy-preserving ad interaction: An ad interaction can not be linked to a user    | EVM compatible SKALE Chains can support EVM compatible curves/ZKP schemes.     |
| Property 2  | Ad interaction unlinkability: Two ad interactions cannot be linked together by Brave or an advertiser      | SKALE is compatible with any curve or ZKP scheme.     |
| Property 3  | Advertiser campaign analytics privacy: The campaign analytics of an advertiser must be only available to Brave and the advertiser     | EVM compatible SKALE chains can support EVM compatible curves/ZKP schemes.     |
| Property 4  | Interaction state update verifiability: Users can verify that the interaction state is correctly updated during the protocol, according to their ad interactions     | All reward data is stored under zero-cost gas transactions on a Brave SKALE Chain, and all blocks are signed by a threshold of validator nodes using BLS threshold cryptography.    |
| Property 5  | Decentralized reward request verifiability: Any participant can verify that rewards requests from users are valid with respect to the state updated by Brave; The result of the reward verification must be committed to a public blockchain for visibility purposes     | The [SKALE Netowrk validator community represents over 45 independent operators](https://skale.network/blog/validator-list-for-skale/) running over 150 nodes, and the SKALE Mainnet has been live for over 120 days, and have been fully decentralized from the initial launch. All data stored within the SKALE Network is completely public, and available for any participate to verify reward requests from users. |
| Property 6  | Advertiser verifiability: Advertisers can verify that the budget expenses corresponding to their ad campaigns are being spent based on the confirmation events received by Brave     | Since all data is public available, Advertisers will also be able to verify that the budget expenses are correct directly from the SKALE Netowrk. |


### Demo of SKALE Chain Speed

Our CEO, Jack O'Holleran, and CTO, Stan Kladko, released a demo highlighting the speed of SKALE Chains.

[Watch Demo Here](https://youtu.be/WJzT3Fb9bkQ)

#### Key facts for Medium Chain Demo

* SKALE Network consists of many blockchains with fast performance at low cost.
* Demo 1: SKALE Medium Chain
* 16 nodes distributed in the cloud running a 1/32 size chain
* More than 2 blocks/second
* More than 100 transactions/second
* 100 transactions/second is about 8.5 Million transactions per day
* Medium chains offer a good balance between price, decentralization, and performance

![](https://i.imgur.com/9tneLkk.jpg)


#### Key facts for Large Chain Demo

* Demo 2: SKALE Large Chain (for power hungry applications)
* SKALE consensys running on 16 very powerful machines, pushing performance limits (Showing what a typical credit card processor needs)
* Over 1600 transactions/second in a stable state
* Blocktimes only increased to around 4 seconds even with 16X the transactions of the medium chain

![](https://i.imgur.com/KA9XOjr.jpg)

### Additional Information

The SKALE Network is already fully decentralized, and is supported by a validator community represented by over 45 independent operators running over 150 nodes. Since the network is EVM compatible, all existing tools built for Ethereum will work directly within the SKALE Network as well. For example, the SKALE Network is fully compatible with the Brave Wallet, and you can test out connecting a SKALE Chain to your brave wallet here:

https://codesandbox.io/s/brave-wallet-integration-skale-dev-docs-bave-83rvp

**All SKALE Chains contain the following features:**
* Full EVM Support for Solidity Smart Contracts
* Interchain Messaging for managing Tokens (ETH, ERC20, ERC721, etc.)
* Decentralized Storage
* Integration Support for All Ethereum Tools
* Wallet Support for API and HSM Wallets

Lastly, the SKALE Network uses a [unique combination of several technologies](https://skale.network/blog/technical-highlights) to achieve scalability, security, interoperability, and progressive decentralization:


| Technology | Impact |
|--|--|
| Pooled Validation Proof-of-Stake | [Scalable security model across validators and delegators](https://skale.network/blog/the-skale-network-why-randomness-rotation-and-incentives-are-critical-for-secure-scaling/) |
|Hybrid container architecture | [Agile allocation of on-demand composable compute resources across the network](https://skale.network/blog/containerization-the-future-of-decentralized-infrastructure/) |
| Threshold Cryptography | Supermajority signature signing with ABBA consensus supports Byzantine Fault Tolerance and resolves  [data-availability](https://skale.network/blog/the-data-availability-problem/)  issues |
| Trusted-Execution Environment | [Fast block signing and multiple chain support using threshold cryptography](https://github.com/skalenetwork/SGXWallet) |
| Asynchronous Binary Byzantine Agreement (ABBA) Consensus | [Mathematically provable, fast-finality, leaderless, and Byzantine fault tolerant](https://skale.network/blog/skale-consensus/) |
| Ethereum Network | Public, open-source, and decentralized operation of the SKALE Network via SKALE Manager contracts |

### Open challenges & Development Roadmap Estimates

The SKALE Network is already live and completely decentralized on mainnet. SKALE Chains will be available for use on mainnet as early as April 26th, 2021, which will allow Brave to implement the THEMIS Protocol directly on mainnet next month. In the meantime, to validate that the SKALE Network will meet the heavy computation needs for the THEMIS Protocol, a POC can be launch on the SKALE testnet prior to mainnet launch.