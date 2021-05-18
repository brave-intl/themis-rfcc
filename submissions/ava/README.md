## Ava Labs submission

**Team:** [Ava Labs](https://avalabs.org)

**Submission date:** 31st March 2021

## Proposal Description

We propose a system that maintains the core off-chain THEMISv2 protocol, and uses the Avalanche network's EVM-compatible Contract chain ("C-Chain") to commit to reward proofs and handling reward payouts. A STARK-based proof batching system is recommended that allow us to greatly beat scalability and cost requirements.

We focus on properties 1, 2, 3, and 5 while leaving room open for satisfying 4 and 6 in future iterations.

## Stage Descriptions

Stages outlined in section 3.1 of "Brave Ads x THEMIS RFC&C".

### Stages 1-3

Protocol stages 1-3  should use the SQS-EQ technique outlined by "Black Box Accumulators in the Context of THEMIS". Importantly, this provides succinct inductively proven commitments to interaction vectors that we may later verify inside a ZK circuit. We refer to this as a "proof-carrying reward calculation".

### Stage 4

After a user has calculated their reward and sends it Brave, Brave MAY exchange the SQS-EQ commitment for cryptographic token signed with a ZK circuit friendly signature scheme for increased performance.

The resulting token should then be fed into a STARK or SNARK prover that batches together all redemptions over a time period, `T`. This allows the THEMIS system to use an approximately constant-sized amount of on-chain resources, and the actual amount of resources consumed can be traded off for increased latency.

An ideal implementation, from a performance and cost standpoint, would be the [StarkEx VM](https://starkware.co/product/starkex/) from [StarkWare](https://starkware.co). Its prover is benchmarked at [3,000 transactions per second](https://docs.google.com/presentation/d/1Ahgt3_a3td7eHulSU7lFF_M14F_ajQfeSXpdyO5Tohs/edit?usp=sharing), which should be the same as verifying signatures from Brave's key on reward commitments (assuming the exchange to a circuit-friendly signature). This is significantly more than the 20 per second required to achieve the desired 50,000,000 rewards proven per month and all 5,000,000 concurrent users could have a reward processed within half an hour.

### Stage 5

Avalanche can handle at least [4,500 transactions per second](https://www.avalabs.org/why-avalanche), meaning issuing reward payments will not be prohibitive even if every transaction were issued individually. Using the existing STARK infrastructure would enable payouts to be even more cheaply. It may be viable to bake the payout directly into the verification batching circuit, making proof publication and reward payouts atomic.

## Costs and Scalability Summary

> Costs: Price of on-chain reward verification of below $0.1 (per reward);

Yes, approximately **$0.005** is possible using StarkEx on the Avalanche C-Chain at current gas and $AVAX prices. Future optimizations are planned to reduce this, and we could support custom pre-compiles and optimizations to reduce these costs even further.

> Scalability 1: The target protocol is able to support up to 5M concurrent
users

Yes, **every user could process a reward once per 28 minutes** (5,000,000 users / 3,000 TPS / 60 seconds).

> Scalability 2: The target protocol is able to verify 50M rewards per month
and to prove their validity on-chain;

Yes, easily. With 3,000 TPS a StarkEx batch proof could support much more than 50M rewards per month and Avalanche can cheaply and quickly finalize these on-chain.


> Client-side performance: Many clients will run on mobile devices, hence the
computation and communication requirements must be kept low.

The client-side interaction performance should be bound by SQS-EQ performance which should not be prohibitive.

## Open Challenges

Responses to proposed open challenges (section 4.2)

> What is the best proof system for THEMISv2. This question has many
dimensions to it:
– What are the requirement for a trusted setup?
– How does the proof artefacts affect the costs and scalability of the
on-chain proof verification?
– Which tools and frameworks can we use to implement and test the
proof circuits?

STARK-based systems have no trusted setup. The proofs are slightly larger than SNARK based systems, which slightly increases costs if the data is to be stored on chain. The [Cairo](https://www.cairo-lang.org) toolchain can be used to write a batch proof circuit like we need.

> Are there services that could perform the proof verification and batching
for Brave?

Yes, StarkWare is one such company that could provide proofs already and other companies are able to form using the available toolchain.

> How to prove that the reward was calculated based on a BBA, state which
was the result of an opened BBA that has been issued and updated by
the expected issuer? We see different paths to answering this question,
namely:
> 1. Build a circuit that verifies the validity of the BBA and its signature
and that performs the open within the circuit.
> 2. Prove the validity of the signature outside independently of the circuit. This would require a separate batching for the signature verification step.

Both would work. The latter removes a degree of trust from Brave but reduces the overall throughput by requiring slower operation in the circuit. Given the potentially performance already beating the requirements, exploring option 1 seems worthwhile but falling back to option 2 isn't terrible.


> Is Ethereum the best blockchain for verifying the batches of proofs from
the reward calculation? How would it compare with other L1 blockchains
in terms of scalability, costs, and degree of decentralization?

Avalanche has a higher overall throughput with lower costs. It has a much higher upperbound on decentralization, potentially supporting millions of staking consensus participants. We will also have solutions in place soon for dealing with state bloat, including pruning and syncing to state commitments. These changes are in development right now.


> Instead of offloading the reward calculation to the user , could a L1
blockchain calculate the rewards of the users based on the BBA state,
while guaranteeing the privacy and trust requirements required by
THEMISv2 2.2? If so,
> 1. what are the trade-offs between the L1 and L2 approaches, considering privacy, scalability, costs and degree of decentralization?

An L1 solution would provide a higher degree of decentralization, by not relying a smaller set of sequencers (e.g. only Brave). It would decrease scalability and make privacy more difficult. An ideal solution is a high capacity L1 like Avalanche in addition to batched proofs that could be generated, at least in the future, by multiple parties.

## Development Roadmap Estimates

The tools required for implementing the Anonymous Tunnel, SQS-EQ protocol, StarkEx circuit, and Avalanche C-Chain contracts are available today. Proof of concepts could be started immediately.

## References

Avalanche Contract Chain (C-Chain): https://docs.avax.network/build/avalanchego-apis/contract-chain-c-chain-api

StarkEx Performance: https://docs.google.com/presentation/d/1Ahgt3_a3td7eHulSU7lFF_M14F_ajQfeSXpdyO5Tohs/edit?usp=sharing

STARK Circuit Development Toolchain: https://www.cairo-lang.org/docs/