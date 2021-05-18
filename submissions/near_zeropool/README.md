## ZeroPool & NEAR submission

**PoC Code**: https://github.com/zeropoolnetwork/libzeropool


### Proposal description

This proposal is to use ZCash-like private transactions for Brave to pay to the user. Given the expected number of ~100M transactions per day three options are suggested to reduce on-chain costs.

1. Using the anonymous connection between User and Brave: User sends advertising event data to Brave, Brave sends signed transaction receipt to User.

2. There are three options for data availability:
    a. transaction data kept only by Brave, which still publishes the merkle root on-chain (**PoA case**). Brave can make this data available via centralized repository.
    b. transactions are validated by independent validators who publish approval on-chain (**optimistic validium case**).
    c. all transactions are published on-chain (**optimistic rollup case**).

3. User can join all rewards to one output and withdraw it to NEAR, nobody can link any two advertising, joining, or withdrawing events.

4. Everybody can verify ZeroPool rollup history for the correctness of all state transitions.

5. If Brave has not included the signed transaction given to the User in the rollup, User can present it either to contract or via social channels as a fraud proof.

Between three options in (2) it really depends on the properties that aimed to achieve. Option (2.a.) is the cheapest one and matches THEMIS the most. Other options are progressively more decentralized and censorship / fault resistant and rely less on Brave operation, which means open to other parties to participate in this economy.

We suggest that it can be progressively implemented from (2.a) to (2.c), allowing to ramp up and test all the rest of infrastructure.

#### Why optimstic ZCash approach?

**The core benefit of this approach over using BBA, is that this is a generic private micro-payment framework.**

Among other things, this allows User to pay to creators from the received rewards without exiting. For example, tips in Brave can be implemented using this same transaction and rollup approach. All Users, Creators would have an account on NEAR blockchain and can receive rewards there on the rollup.

Exits can also be made private to allow user to maintain their privacy while using other apps in the ecosystem.

This proposal also addresses two potential issues discussed in [Discord](https://discord.com/channels/796818447350497410/796818447891038210/819254685268967447) thread:
 - Brave misbehavior: after transaction was signed by Brave and sent to User, User has a evidence that can be used to report misbehavior to smart contract or post to social media. If User can stop interacting with Ads if Brave doesn't provide such receipt.
 - Brave exit censorship: User is able to submit their own exit in case Brave has censored.

#### Why NEAR?

One of the core value props of NEAR is the fact that it scales with demand due to sharding. Network will add more capacity (more shards) if there is more usage coming. This means that fees are maintained at the same level even as network gains more adoption. NEAR Governance also have committed to maintain or reduce fees over time in USD equivalent to make sure that network stays affordable for developers and users even as $NEAR token value changes.

Using NEAR as a cheap data availability (~1N per 1Gb of data availaiblity) and smart contract chain, allows to maintain low cost for all operations. Also via NEAR, Users who exited can cheaply transact with plehora of other applications (creator economy, open finance, social and more).

### Contributions summary and performance results/estimates

#### 1. Transaction creation for Brave, 1 input and 64 outputs


|Proof generation|Proof size|Tx size|Proof verification|
|-|-|-|-|
|1907ms (on i9-9880H)|128 bytes|6150 bytes including proof|65ms (on i9-9880H)|

#### 2. Transaction creation for user, 8 inputs, 1 output

|Proof generation|Proof size|Tx size|Proof verification|
|-|-|-|-|
|6915ms (on i9-9880H)|128 bytes|486 bytes including proof|5ms (on i9-9880H)|


#### 3. Proof verification (batch of N proofs)

In optimistic rollup or optimistic validium data should be just published, but the validity of the data could be proven via fraud proofs on the contract if the rollup operator cripples the state of rollup/validium.

Usually one fraud proof is about 20 times more expensive then native coin transfer, but publishing fraud proofs is very rare event.

Multiple proofs could be batched for 4 times faster verification.

#### 4. E2E costs (1M users)

On 1M users and 100 transactions per users daily we should reach 1200 tps. So, it's enough to use 20 servers for computing proofs.

Generally smart contract transaction on NEAR is 0.0001 - 0.0005 $N. Also NEAR team is working on reducing price by optimizing smart contact runners and other infra.

Total amount of data to store for data availability is ~2TB annually.

Depending on the data availability choice:
 a. PoA case: Brave stores all the transactions. Periodic check-in merkle root of the latest transactions on-chain. Doing check-in once a minute will mean ~500k tx annually (~300N per year).
 
 b. Optimistic validium case: set of validators store ~2TB, sign the merkle root and that gets check in on-chain. This costs similarly on-chain as just adds marginally more data and verification.
 
 c. Optimistic rollup case: all transactions are included on-chain. Aggregating 1024 tx per block will require ~40M block publish transactions per year (100m tx per day * 365 / 1024). Costing up to 20k NEAR per year.
 
For exits: each batch of exits will be up to 0.0005 N, which means that for 1M exits it will take 500N.

### Changes to the THEMIS protocol

Core change is switching away from using BBA to using ZCash like transactions between Brave and User.

For each ad interaction User sends to Brave interaction information and new public key. Brave responds back with signed transaction receipt (including merkle path in the rollup block).

The multi-broadcast that allows advertisters to track the usage is maintained the same as in THEMIS protocol.

User locally maintains list of all the transaction receipts received from Brave.

When exiting, User combines these outputs into one exit transaction. User can request Brave to perform an exit on their behalf. Or if Brave declined for either reason, User will be able to exit themself as well.

Brave can batch the exit requests as well balancing time to exit with costs.

### PoC and Measurements

> Description of the PoC implementation and measurements obtained
 
**Code repository**: https://github.com/zeropoolnetwork/libzeropool
**Documentation**: https://github.com/zeropoolnetwork/libzeropool/blob/master/README.md

**Quick start guide**
- How to compile/run the PoC code?
```bash
cargo build --release
```
- How to reproduce the measurements?
```bash
# for 8 to 1 join case
cargo test --release -- --nocapture test_circuit_tx_setup_and_prove
```

### Open challenges

> Challenge: Brave censoring exits due to various reasons.

In the future, to reduce ability for Brave to track and censor exits, NEAR's unique account model can be leveraged to approve a pre-paid single use key for user to submit on-chain transaction with exit proof (e.g. they can sign one and only one transaction with it and key becomes invalid). This can allow users to exit themself without going through Brave after initial setup.

Brave can issue such key to User, when user can prove they accumulated some amount of rewards.

### Development Roadmap Estimates

> Description of the development and deployment roadmap. Which frameworks/libraries and/or cryptographic schemes to use? Are those currently available? Open source? Have the cryptographic schemes and other software been submited to security audits?

We use Groth16 proving system with ZeroPool private transaction circut, developing at https://github.com/zeropoolnetwork. All our software is open source. Low level things (like SRS generation or Groth16 implementation) passed security audit. Our current circuits are still WIP. We are planning to submit it to security audit at Q2 2021.

> Timeline estimates for scheme/s in the proposal to land in production? 

We can reach the production for 3 months for development (limited performance) + 3 month (high performance) + time for a security audit. The unaudited beta version could be published earlier.


### Future work

1. Prover optimizations
In case of proving multiple circuits with the same SRS, there is some optimizations space. Also, some heavy operations could be moved to GPU

2. Migrating to novel proving systems with better performance
