## Starkware submission

** Note: StarkWare did not submit a formal proposal, and the information provided here is the output of back-of-envelope calculations and discussions between the teams.**


In this document we present our understanding of the best solution that could run using STARKs. These notes are the output of conversations with some of the StarkWare team. We begin by giving an overview of the solution proposed, some estimates, and open questions that should be resolved. 
STARK based solution

The main idea is to replace the BBAs with proofs generated using cairo. The commitment of the state is encoded as a single merkle tree root. Each of the leaves of this tree encodes the number of interactions each ad had, i.e. the interaction vector is represented by the leaves of the tree (there would be some randomisation added here to avoid brute forcing the state of a given merkle tree root). This root is initialised at zero, and signed by Brave. When a user interacts with an ad, it updates the leaf of the merkle tree corresponding to the ad viewed. Next, it sends the event viewed to Brave, adi, with a proof of the following statement: 

{(T): Signature.Verify(T) = true  Tree.Update(T, adi) = T'}


where T,T', are the old and new merkle tree roots respectively. Note that if the old root is valid, there should exist a valid signature for it. In the proof, the value of the hold commitment (and its corresponding signature) is hidden. Function Signature.Verify()simply verifies if there exists a valid signature for the given input. Function Tree.Update(, ) computes the update of the tree whose root is given as input, with the event (also given as input). 
After the proof validates, Brave would sign the updated merkle root, T', and return it to the user. Using the above scheme would result in important improvements at the time of batching proofs. 

Estimates of this proof: 
- 2-10 seconds on a smartphone
- 40KB
- 120K updates could be batched in a single STARK proof
- The batch proof would take a few minutes in an amazon machine
- Estimate of gas cost per transaction is 50gas/update

Questions: 
- Are the estimates considering the proof that the signature (or each old commitment) is not double spent?

Yes. Adding DS protection shouldn’t affect proving time significantly, but will necessitate Brave to maintain extra state.

- Do we have battery consumption estimates of how much the proof generation takes?

No, one may estimate those based on the computation time mentioned above. 

- How does the proof of "no double spend" grow with respect to the number of requests? 

Negligible addition, that is independent of the number of requests. Here’s a protocol: 

Each user i picks a secret key si and submits H(si) to Brave, and Brave signs this with Brave’s secret key, let sigi be the resulting signature, and let kpub be Brave’s public key.

Now, for each epoch j, each user i sends a message (mi,j, pi,j) where mi,j = H(si || j) and pi,j is the STARK proof of the statement 

(*)   “I know si. sigi ,such that sigi  is the signature of H(si) with kpub and mi,j = H(si || j). “

Brave will verify such statements and keep for each epoch j the set of verified mi,j. Notice this is a decent scheme because mi,j  doesn’t leak i, yet is deterministically computed from sigi and the epoch number j. Thus, one user may only submit one message mi,j per epoch.
Batching user updates in lesser proofs
The batching numbers look great - however, the problem comes when considering the complexity on the user side (communication and computationally speaking). 10 seconds and 40KB may be too much for a user to compute five times per hour. To this end, one option is to have users batch several updates in a single proof. In this section we draft our understanding of such a solution.

When users interact with an ad, adi, instead of submitting a proof, they simply submit a commitment to the event together with the event itself. Particularly, they send the following tuple, [adi, Comm(adi)]. We assume that users report honestly, and therefore we don't need this value to be signed. You want these signed, or else, Brave should maintain some data structure with this data. But simpler to simply sign, as you intended in the previous section. 

Now, the proof will come once per day, where the user will update it's current merkle root with several ad events, S=[adi]iV, where Vcontains the indices of all ads viewed during the day. Then, the statement being proved is the following:

{(T): Signature.Verify(T) = true  Tree.SeveralUpdate(T, S) = T'  eS,  Comm(e)}

where Tree.SeveralUpdate(,) is simply an update of a merkle root corresponding to a set of events. The last check ensures that for every event in S there exists a commitment which has not been double spent. 

Questions:
- Does this proof grow linearly with respect to the number of events contained in S?

No, it grows logarithmically in |S|

- How is the size of this proof affected?

Single update has a proof of size 40KB. A 1000-size update will have a proof of size < 60KB and a 10,000 update proof will be < 80KB
How is the size of the batch proof affected?

See answer to previous question (I don’t understand how this question differs from previous)

- How does the proof that a commitment is not double spent affect the running times?

Negligibly, the prover has to prove 4 invocations of a crypto primitive (which is a negligible amount of computation)

Q&As

- Re: batching of the proofs - is it correct to say that it should take about 2-10s to generate a prove of 10 updates on the client side? And a size of <60KB per proof? In production, this proof would be generated by the user daily.

- we quoted 2-10 sec on a smartphone for a single update. For more updates it'll scale sublinearly because the biggest metric is the number of invocations of crypto primitives that are involved in the update. If you have 10 updates in a tree of depth 8, then if all of them touch the same leaf, or adjacent leaves, you pay only for one merkle path. Even if the paths are random there will be some joint parts in the upper parts. So for 10 updates my guesstimate is that it will be 20-100 seconds on a smartphone (but around 1 second at most, on a desktop, say, a quadcore) 


- As far as we understood, to verify a batch proof would take a few minutes in an amazon ec2 instance. How big would that batch proof be? i.e. are we talking about a few minutes to verify a batched proof of 10000 proofs, or a few minutes to verify a batch with 10 proofs? We'd expect each user to send a batch proof of 10 proofs per day. Can we estimante how much compute time would take for Brave to verify the one batch proof of 10 proofs for 1M users/day on an Amazon instance? (and how beefy would the instance need to be?)

- verification is very fast. I assume you're referring to proving: proving even large batches, say, of 120K updates (among multiple different users) should take a few minutes on a standard large amazon machine. and the proof for this large batch would be around 100KB, need to check the numbers (I'm extrapolating from the size of a proof for 4000 trades on StarkEx, which corresponds to 120K updates in your wolrd), will get back

- If you want batching by users, the reasonable thing is for the users to submit their data to a service and a single proof would be generated for all of them together.

The scheme that we have been discussing refers to the update of the merkelized state of ad interactions that each user keeps locally and commits to Brave every day. This is the first part of THEMIS. The second part consists of the user calculating the BAT rewards based on the latest merkelized state and proof its correctness to Brave. After the reward is calculated and the proof has been generated, the user can request X BAT and Brave can prove its correctness. Eventually we'd like to batch thousands of proofs of reward calculation from multiple users and verify it on L1. How could STARKs and Cairo help here? Do you have any estimates with regards to the proof generation/batching/verification and L1 costs of such a scheme?(edited)

- Computing the inner product (the Brave reward) is roughly the same complexity as proving an update. Reason is that the logic of doing inner product on the leaves is negligble compared to the proof that the cryptography (hash invocations) was done correctly. So I assume a single STARK proof can attest to the correct computation of thousands of such inner products, in one single proof.
Concretely: if you have a commitment to 256 leaves, computing the inner product (the epoch Brave reward) is roughly like a single Deversifi trade (which costs ~300 hashes, you're looking at roughly same number of hashes). So, e.g., a single STARK proof can cover 4000 reward calculations, verifying that proof onchain is ~ 5M gas, amortized over all users, so roughly 1K gas per user.
