# Brave Ad Interactions

The users BBA can be stored a merkle tree. 

The tree is of depth 2^10 (which gives enough space to store interactions with 1,024 advertisers).

The BBA tree uses an account model, the leaves represent the number of interactions with specific advertisers. The leaves of the tree are a Pedersen hash of (interaction_count, advertiser_id, user_secret_key). This is required to hide the add interaction count from brave.

When the user is shown an advert, they create a proof of add interaction which is a PLONK proof, proving:

1. Brave signed the old merkle root nullifier and the current merkle root (or the current merkle root is that of an empty tree)
2. An existing leaf in the tree is a Pedersen hash of (interaction_count, advertiser_id, user_secret_key)
3. The existing leaf is inserted at the position of the old leaf, whose value is pedersen(interaction_count + 1, advertiser_id, user_secret_key)


This proof can be communicated to Brave via a secure channel.
The circuit validates two signatures - the signature that describes the current interaction, and the signature that describes the previous ad interaction (the second signature is not required if the current tree is empty). This inductively ensures that there exists a chain of Brave signatures that leads back to when the user's BBA tree was empty.

Double spending is prevented if Brave only ever signs against a given nullifier once.

The protocol architecture consists of 3 circuits: 

1. **Ad Interaction Circuit**: generated when a user interacts with an advert
2. **Rewards Circuit**: generated when a user wishes to convert accrued payments in their BBA into a token balance
3. **Batching Circuit**: a rollup circuit that aggregates together reward circuit proofs, amortizing the cost of rewards payments across multiple users

## Estimated Costs


|  Circuit | WASM prover times | Native prover times |
| -------- | -------- | -------- |
| Ad Interaction Circuit     | <5s     | <1s     |
| Reward Circuit | 40-80s | 5-10s |
| Batching Circuit | n/a | 1-2 minutes (if parallelized)

To process a size-1024 batched rollup, we can target a per-user cost of approximately either **6,300 gas OR 21,300 gas**  (depending on whether their previous token balance is zero or nonzero respectively)

If an L1 smart contract is used to only validate the correctness of proofs, this cost drops to ~1,100 gas per user.

# Ad Interaction Circuit

### ◈ Index of Functions

In the Pseudocode to follow, we use the following function names:

+ `ComputeNF` **Nullifier Function**, which we assume can be modeled as a random oracle, and only depends on $\text{nk } mod \text{ } r$
+ `SchnorrVerify` Verifies a Schnorr signature
+ `SigHash` Uses a hash function that can be modeled as a random oracle, to hash a message into a pseudorandom number. We compress the input message using a Pedersen hash, then compute a Blake3 hash of the compressed message.
+  `Enc`: Encrypts a leaf of the data tree via a Pedersen hash

### ◈ Public Inputs

* `current_tree_nullifier`
* `new_tree_root`
* `advertiser_index`
* `brave_signature`

### ◈ Private Inputs

* `previous_advertiser_commitment`
* `current_tree_root`
* `user_secret`
* `previous_brave_signature`
* `previous_tree_nullifier`
* `previous_advertiser_index`
* `click_counter`

### ◈ Circuit Constants

* `BRAVE_PUB_KEY`: Brave's public key, used to verify signatures
* `EMPTY_TREE_ROOT`: Precomputed root of an empty BBA tree

### ◈ Circuit Logic

1. Compute `current_advertiser_commitment = Enc(click_counter, advertiser_index, user_secret)`
2. Compute `new_advertiser_commitment = Enc(click_counter + 1, advertiser_index, user_secret)`
3. Compute `sig_msg = SigHash(new_advertiser_comm.x, current_tree_nullifier, new_tree_root, advertiser_index)`
4. Validate `SchnorrVerify(sig_msg, BRAVE_PUB_KEY, brave_signature)`
5. Validate `Membership(current_advertiser_commitment, advertiser_index, current_tree_root)`
6. Validate `Membership(new_advertiser_commitment, advertiser_index, new_tree_root)`
7. Validate `current_tree_nullifier = ComputeNF(current_tree_root, user_secret)`
8. Validate `previous_sig_msg = SigHash(previous_advertiser_commitment, previous_tree_nullifier, current_tree_root, previous_advertiser_index)`
9. Validate `SchnorrVerify(previous_sig_msg, BRAVE_PUB_KEY, previous_brave_signature) OR current_tree_root == EMPTY_TREE_ROOT`

### ◈ Circuit Costs

* `Enc`: click_counter = 32-bits, advertiser_index = 10-bits, user_secret = 254-bits => 296-bit Pedersen hash ~ 70 constraints  
* `SigHash`: message length = 778-bits. Message compression = 778-bit Pedersen hash ~ 180 constraints. Blake3 hash of compressed message = 2,000 constraints. Total = 2,180 constraints.
* `SchnorrVerify`: 1 variable-base scalar mul (256) + 1 fixed-base scalar mul (64) = 320 constraints
* `Membership`: 10 512-bit Pedersen hashes = 1,280 constraints
* `ComputeNF:`: 1 Blake3 hash = 2,000 constraints

Total constraint cost ~ 9,300 constraints

The ad interactions circuit’s expected gate gount is < 2^14 (under 5 seconds prover time for most devices in WASM and under 1 second natively).

# Reward Circuit

An L2 rollup system like Aztec can be used to record the last payment merkel root for each user in a data tree and scale reward payments.

To claim rewards, a user will open their BBA Merkle tree inside a zk-SNARK circuit. Every non-zero leaf is a private witness in the circuit. The user performs the following:

2^10 leaves are hashed together to produce a Merkle root
The computed Merkle root must be equal to a root signed by Brave
The total value of all opened leaves is accumulated
The output of the circuit is the following:
The total sum of tokens to be paid to the user

### ◈ Circuit Constants

1. `BRAVE_PUB_KEY`

### ◈ Public Inputs

1. `tree_nullifier`
2. `user_reward`
3. `user_address`
4. `fee_schedule`: a lookup table that describes the fees paid by each advertiser per interaction


### ◈ Private Inputs

1. `tree_root`
2. `brave_signature
3. `previous_tree_nullifier`
4. `previous_advertiser_commitment`
5. `previous_advertiser_index`

### ◈ Circuit Logic

* let `accrued_rewards = 0`
* For `i in [0, ..., 1023]:`
    * let `advertiser_commitment_i = Enc(click_counter_i, i, user_secret)`
    * `accured_rewards = accrued_rewards + fee_schedule(i) * click_counter_i`
* Validate `tree_root == BuildTree(advertiser_commitment_0, ..., advertiser_commitment_1023)`
* Validate `user_reward == accrued_rewards`
* Validate `tree_nullifier = ComputeNF(tree_root, user_secret)`
8. Validate `previous_sig_msg = SigHash(previous_advertiser_commitment, previous_tree_nullifier, tree_root, previous_advertiser_index)`
9. Validate `SchnorrVerify(previous_sig_msg, BRAVE_PUB_KEY, previous_brave_signature)

### ◈ Circuit Costs

* `n` calls to `Enc`, each costing ~70 constraints
* `BuildTree`: requires `2n` 512-bit Pedersen hashes, each costing ~90 constraints for a total cost of 180n constraints

Estimated total cost ~ `250n` constraints

| Tree size     | Pedersen hash gate cost | Estimated prover time (wasm) | Estimated prover time (native) |
|---------------|-------------------------|------------------------------|--------------------------------|
| 1,024  (2^10) | 256,000 (2^19)          | 40-80                        | 5-10                            |
| 2,048  (2^11) | 368,640 (2^19)          | 80-160                        | 10-20                           |
| 4,096  (2^12) | 737,280 (2^20)          | 160-320                       | 20-40                          |
| 8,192  (2^13) | 1,474,560 (2^21)        | 320-640                      | 40-80                          |
| 16,384 (2^14) | 2,949,120 (2^22)        | 640-1280                      | 80-160                          |

# Batching Circuit (size n)

The batching circuit validates a size-n block of reward proofs and correctly updates the set of spent tree nullifiers.

The gas-efficiency of processing rewards is improved by compressing all user rewards into a single SHA256 hash output. In a smart contract, it costs the verifier algorithm 200 gas to process a 32-byte public input. However it only costs 6 gas per 32-bytes to perform a SHA256 hash.

(i.e. the verifier contract hashes together the user reward outputs and feeds a single public input into the circuit)

Each reward output is a tuple of the destination user address and the token value. Both are represented in 32-bytes to reduce gas costs, by using a 12-byte integer to represent the token value (with 20 bytes for the Eth address).

### ◈ Index of Functions

+ `Extract` **Extraction Function** extracts 14 public inputs from a proof, validates the result matches the rollup’s inner public inputs
+ `Aggregate` **Proof Aggregation Function** for ultimate batch verification outside the circuit, given a verification key and (optional, defined by 4th input parameter) a previous output of Aggregate. Returns a BN254 point pair
+ `NonMembershipUpdate` **Nullifier Update Function** checks a nullifier is not in a nullifier set given its root, then inserts the nullifier and validates the correctness of the associated merkle root update

### ◈ Public Inputs

1. `user_rewards_hash`
2. `old_tree_nullifier_root`
3. `new_tree_nullifier_root`
4. `total_reward`
5. Recursive proof output `Q_n` (16 public inputs that represent 2 BN254 elliptic curve points)

### ◈ Private Inputs

1. `user_reward_0, ..., user_reward_n`
2. `user_address_0, ..., user_address_n`
3. `reward_output_0, ..., reward_output_n`
4. `tree_nullifier_0, ..., tree_nullifier_n`
5. `tree_nullifier_root_0, ..., old_tree_nullifier_root_{2n-1}`

### ◈ Circuit Logic (pseudocode)


* Let `Q0 = [0, 0]`
* Let `reward_count = 0`
* For `i in [0, ..., n-1]:`
    * Let `pub_inputs = Extract(PI_i)`
    * Let `Q_{i+1} = Aggregate(PI_i, pub_inputs, REWARD_VK, Q_{i}, (i > 1))`
    * Validate `NonMembershipUpdate(tree_nullifier_root_2i, tree_nullifier_root_2i+1, tree_nullifier_i)`
    * Validate `user_reward_i < 2^{92}`
    * Validate `user_address_i <  2^{160}`
    * Validate `reward_output_i == user_address_i + user_reward_i * 2^{160}`
    * `reward_count = reward_count + user_reward_i`
* Validate `user_rewards_hash = SHA256(reward_output_0, ..., reward_output_n)`
* Validate `old_tree_nullifier_root == tree_nullifier_root_0`
* Validate `new_tree_nullifier_root == tree_nullifier_root_{2n - 1}`
* Validate `total_reward == reward_count`

### ◈ Circuit Costs

The batch circuit will have a similar complexity to our current rollup circuit . It currently takes us 4 minutes to aggregate together 28 transactions into a rollup. These 28-tx rollups proofs are constructed in parallel. Once these proofs are made, we aggregate them together into a size-896 rollup. This takes 4 minutes. This is using TurboPlonk, however. Once we transition to UltraPlonk these proving times will be 2.5x-3x faster.

The batching circuit will also have approximately 10% fewer constraints than our existing rollup circuit, due having fewer merkle tree updates to perform

# Smart Contract Architecture

The following spec assumes it is possible to re-deploy a BAT token smart contract to improve the gas-efficiency of withdrawing rewards.

The TurboPlonk verifier smart contract can verify a proof in 430,000 gas. This includes the calldata cost for the 1.4kb proof size (excluding public inputs).

We expect the gas cost for verifying UltraPlonk proofs to increase to 450,000 gas.

The token smart contract would share an identical interface to the existing BAT token, with an additional method `verifyBatchedRewards`, that processes a batching circuit proof. The following logic must be executed:

### Transaction Inputs

1. `reward_output_0, ..., reward_output_n` (n * 32 bytes)
2. `total_reward`
3. `new_tree_nullifier_root`
4. Recursive proof output `Qn`
5. Proof `PI` (1.4kb)

### Contract Logic

1. Compute `user_rewards_hash = SHA256(reward_output_0, ..., reward_output_n)`
2. Load `old_tree_nullifier_root = sload(nullifier_root)`
3. Validate `Verify(PI, Qn, old_tree_nullifier_root, new_tree_nullifier_root, total_reward, user_rewards_hash)`
4. Update `balance[brave] -= total_reward`
5. Update `nullifier_root = sstore(new_tree_nullifier_root)`
6. For `i in [0, ..., n-1]`:
    * Let `user_address = reward_output_i.slice(12, 32)`
    * Let `user_reward = reward_output_i.slice(0, 12)`
    * Update `balance[user_address] += user_reward`


### Gas efficiency analysis

Each user reward consumes 32-bytes of `calldata`, currently priced at 16 bytes per nonzero byte.

**balance update**: updating a user balance requires 2 steps

1. compute the storage slot of the `mapping(address => uint256) balances` storage variable
2. call `sstore(add(sload(balances.slot), user_reward))` to update the user balance

Computing the storage slot per user requires 42 gas. `sload` consumes 200 gas and `sstore` consumes either 5,000 gas (if the existing balance is non-zero) or 20,000 gas (if the existing balance is zero)

Subtotal = 5,242 or 20,242 gas

**proessing public inputs**: to reduce verifier compute gas costs, we compute a SHA256 hash of the user reward data. This costs 12 gas per reward (6 gas for the hash, 6 gas to load the user reward from calldata into memory), plus ~1,000 gas.

**proof verification**: approx 450,000 gas (including proof calldata)

**user reward calldata**: 512 * n gas

**base tx cost**: it costs 21,000 gas to send an Eth transaction

**updating other storage variables**: (nullifier root && brave balance) 10,400 gas

#### Total Cost Estimate

Fixed costs: ~480,000 gas

User costs: either ~5,800 gas or 20,800 gas depending on their existing balance.

Cost per user: $5,800/20,800 + \frac{480,000}{n}$

If `n = 1024`, cost = ~6,270 gas or ~21,270 gas.



| Rollup size | Estimated cost per user |
| -------- | -------- |
| 256     | 7,500 or 22,500     | 
| 512 | 6,750 or 21,750 |
| 1024 | 6,270 or 21,270



# Q&A

---
title: Notes/Questions AZTEC submission 
tags: rfcc, themis
---

#### Is the nullifier related to the user secret key? Or instead related to the hash root? My concern is whether a nullifier is supplied for each updated merkle root?

A: The nullifier is related to the hash root. Each user will posess a 'nullifier secret key' (this can be the same as their secret key), and the nullifier will be a hash of [merkle tree root, user secret key].
For the reward circuit, only the nullifier of the complete tree being opened will need to be added into the reward circuit's 'nullifier tree'
There was an error in the spec we sent you regarding the update proof. For the update proof, Brave would need to sign against the old *nullifier* and not the old tree root. This maximally preserves privacy; if Brave receives multiple requests to sign BBA nullifiers they cannot determine which users the requests come from by examining the public inputs.
Assuming Brave caches a list of all nullifiers they have signed over, double spending is prevented by never signing over a nullifier more than once.

#### 896 txs per rollup --- Does this consider performing the payment to each user?
Yes it does

#### What is the cost associated with computing the batch proof?
The batch circuit will have a similar complexity to our current rollup circuit. It currently takes us 4 minutes to aggregate together 28 transactions into a rollup. These 28-tx rollups proofs are constructed in parallel. Once these proofs are made, we aggregate them together into a size-896 rollup. This takes 4 minutes. This is using TurboPlonk, however. Once we transition to UltraPlonk these proving times will be 2.5x-3x faster.

(Still possible to optimize it)

#### What about having in each merkle tree, instead of a pedersen commitment of ad_interaction, the pedersen commitment of the number of interactions . This way we would be happy with a merkle tree of depth 2^10, and the proof update would be “I’ve updated leaf at position n by one unit”. I don’t know if we can exploit the additive homomorphic property.

Can I clarify what the desired outcome of this change would be? If I understand correctly, the BBA tree changes from an append-only UTXO style tree (1 leaf per ad interaction) to an account-based tree, where each position maps to a given ad provider and the leaf value is a count of the number of ads clicked on.
The outcome is that the tree size is bounded by the number of advertisers on the platform and not the maximum number of supported user interactions (the former being smaller than the latter), is this accurate?
If the circuit validates that a leaf value only increases by 1 then it should be possible to prevent the circuit from leaking information to Brave.
However, the position of the leaf being updated can't be exposed to Brave without leaking information about which advertiser owns the ad the user clicked on. Is this information already sent to Brave as part of authorizing a BBA update?
We might be able to change the circuit to avoid broadcasting position `n` though. If the proof merely says "I’ve updated a leaf at a hidden position one unit”, would that be sufficient? It depends on whether Brave needs to link a BBA update to a given advertiser.

### Errata
I wanted to highlight that I think I made some errors when estimating how long computing a reward proof will take the user. I underestimated the likely proving times for a user using WebAssembly to compute a proof. A short analysis is below:
The cost is largely determined by the size of the BBA tree. The user will need to perform 2 512-bit pedersen hashes per leaf to completely open the tree. Each 512-bit hash will cost ~90 constraints (the cost is lower than the interaction circuit because we can use larger lookup table for larger circuits).
Rough costs for various tree sizes are below:

| Tree size     | Pedersen hash gate cost | Estimated prover time (wasm) | Estimated prover time (native) |
|---------------|-------------------------|------------------------------|--------------------------------|
| 1,024  (2^10) | 184,320 (2^18)          | 20-40                        | 3-5                            |
| 2,048  (2^11) | 368,640 (2^19)          | 40-80                        | 5-10                           |
| 4,096  (2^12) | 737,280 (2^20)          | 80-160                       | 10-20                          |
| 8,192  (2^13) | 1,474,560 (2^21)        | 160-320                      | 20-40                          |
| 16,384 (2^14) | 2,949,120 (2^22)        | 320-640                      | 40-80                          |

Ideally the reward proof uses a native prover built into the Brave browser, the WASM performance times are quite long. The tree size is shrunk to 2^10 this
N.B. The reason the user has to open the complete tree is so that they can compute the number of tokens they have earned, by combining each ad interaction with a fee read from the fee schedule. When does this linking to the fee schedule need to occur? If we moved the fee schedule into the interaction circuit, the user could keep track of a running total of tokens they have earned, and there would be no need to open the BBA tree when computing the reward proof.
In fact, if a running total of tokens earned is tracked in the interaction circuit, it might be possible to remove the BBA tree entirely. I'm happy to discuss this further on the call.
