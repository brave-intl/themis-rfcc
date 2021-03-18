# THEMIS RFC&C

The Request For Comments and Code (RFC&C) community event is a 2 month event
where the Brave Research team, companies and projects in the cryptography space,
and researchers will come together to engage in technical discussions and
collaborations to push THEMISv2 forward towards production. The RFC&C is a great
opportunity for crypto projects and researchers to actively shape the future of
the BAT ecosystem. As we go through the various phases, we will regularly share
updates with the community and hold sessions to answer the questions that the
participating teams might have.

This repo contains resources for teams to participate in the RFC&C event:

- [RFC&C and THEMIS technical report](./rfcc-themis-tech-report-v1.0.pdf) current
  revision: v1.0
- [Constant size Black-Box Accumulators technical report](./rfcc-themis-bbas-v1.0.pdf) current revision: v1.0
- [Submission template](https://hackmd.io/QX4zgIeaQ6OSGJEwFYHtEw)
- FAQ (below)

## FAQ

### THEMIS protocol

- **Is the reward calculation proof of THEMIS constrained to a specific curve and ZKP scheme?**

THEMIS is not constrained to any particular curve or ZKP scheme yet. However, the BBA scheme we designed requires bilinear friendly curves (refer to the BBA report). We are working on an implementation of the signature scheme necessary to implement the BBA which is curve-agnostic ([Structure Preserving Signatures over Equivalence Classes (github)](https://github.com/brave-experiments/sps-eq/)). One of the goals of the RFC&C is to figure out which schemes and params are best suited to implement the THEMIS protocol. When selecting the curve and ZKP scheme, we will focus on optimizing computation performance, with special emphasis on the computation required by the user to generate the proof of correct reward calculation. In addition, those choices should take into consideration the communication overhead and the scalability and costs of verifying the proofs off and on-chain.

- **Would you consider a BBA that doesn't require a pairing friendly curve (but rather relies on ZKPs to do some extra work)?**

We could consider a different BBA that does not require pairings, and instead uses zero knowledge proofs. The main concern of such solutions is the computation and communication complexity of the ’show protocol’ - when the user requests an update by proving it owns a valid token. Note that the update requests may happen up to 10 days a day per user (see question below).
 
In the pairing based construction THEMIS uses, the user simply needs to randomise the token and signature (5 exponentiations and two inversions modulo de order of the group), and send it (5 EC points, 4 from G1 and 1 from G2). In a ZKP based construction, the user needs to generate the proof and send it (the computation and bandwidth required here is the concern). 
 
However, if some solution which leverages a ZKP scheme instead of pairing-based would achieve better performance and bandwidth, the protocol may change to accommodate those improvements. 

- **What’s the frequency of the ad update request? How about the frequency of the reward request, which requires the reward calculation and proof from the user?**

We assume that each user may request 10 BBA updates to Brave over one day. Those requests update maps to an ad interaction. On the other hand, we assume that a user may calculate the reward and request a reward from Brave once every month.

- **How you will be able to decentralise other aspects of Brave, beside the reward calculation and the ad attribution metrics for advertisers? For example, how does THEMIS decentralize the payment of the rewards or the logic behind the ad mechanics?**

THEMIS is part of progressively decentralizing Brave Rewards and Ads. Currently, the protocol focuses on decentralizing and bringing transparency to the rewards calculation and to the ad attribution metrics for advertisers, which is non-trivial. The efforts to decentralize other components of the Brave Rewards and Ads ecosystem are out of scope of this RFC&C, although they can be considered by the teams when submitting their RFC proposals.

- **What the dimension of the ad vectors (catalog size) are and also an upper bound on how big each entries of the vectors are (how many times can a user interact with a single ad per epoch)?**

For the purpose of testing and estimates, we can assume that each catalog has up to 128 ads per campaign (feel free to estimate for 256 ad, although we're not expecting this amount of ads per catalog in the medium term). As for the upper bound of interactions per ad per user, we can do a back of the envelop calculation: each user can interact with up to 10 ads/day. The max time per ad catalog epoch is one month. If we consider that a user has interacted with the same ad every day, as many times as possible, we get an upper limit of 310 (10 * 31).

### RFC&C Practicalities

- **How to participate in the RFC&C event?**

Feel free to reach out anyone at the Brave Research team on our  [RFC&C and
THEMIS technical report](./rfcc-themis-tech-report-v1.0.pdf).

- **Should the participant teams work on all phases of the protocol?**

Not necessarily. Although we welcome submissions which tackle many aspects of
the protocol as possible, we will consider submissions that focus on specific
parts of the protocol (e.g. anonymous ad attribution, reward verification
scheme, etc). We prioritize the impact and correctness of the submissions over
completeness.

- **Should the submissions be based on the current design of the THEMISv2 protocol?**

No, the current protocol design described in this report is a starting point
and an example of a system the Brave Research Team sees as a good trade-off
between privacy, security, scalability and production-readiness. Submissions
may not use any of the components of THEMIS. However, the submissions should
respect the goals and requirements outlined in the Section Goals and Requirements
of the [RFC&C and THEMIS technical report](./rfcc-themis-tech-report-v1.0.pdf).
