<!-- solve-problem:937cb406-b065-46ad-9176-51a5375b4595 -->
# Equal opportunity with strategic client identities

A [Tangle Solve problem](https://solve.tangle.tools/problems/937cb406-b065-46ad-9176-51a5375b4595).

## What is the problem?

In blockchain consensus, *fair ordering* protocols (e.g., Aequitas-style, fair-DAG, and the 2025 "ordered consensus with equal opportunity" line) guarantee properties like **batch ordering fairness**: if sufficiently many honest replicas receive message *m* before message *m′*, then in the final ledger order *m* precedes *m′*. These properties are defined **per message**.

The problem: **fairness between messages does not imply fairness between clients (users)**. A strategic client can gain relative ordering advantage through:

1. **Buying faster delivery routes** — paying for lower network latency so its transactions reach a majority of replicas earlier, winning orderings not by legitimacy but by network position.
2. **Identity rotation (Sybil behavior)** — submitting under many pseudonymous identities to get multiple "seats" in any per-message fairness mechanism, or to evade per-identity throttling.
3. **Correlated copies** — broadcasting duplicates/correlated variants of the same economic intent so that *some* copy lands favorably, gaming probabilistic or ordering-based fairness.

Formal setting: *n* replicas, *t* Byzantine, asynchronous or partially synchronous network. Messages carry (possibly untrusted, freely chosen) client identities. The goal is to define and construct a consensus protocol satisfying (a) safety and liveness as usual, and (b) a new **inter-client equal-opportunity** property: no client, by spending more money on latency, spinning up identities, or replicating messages, can achieve a systematically higher probability of obtaining favorable ordering relative to a client issuing a single honest transaction. A precise formulation must specify the identity/registration model (permissionless vs. stake/fee-weighted), the adversary's network-power budget, and the fairness metric (probability of winning a contention, expected position, economic extraction such as MEV captured).

## Why does it matter?

Ordering fairness is motivated by MEV: front-running, sandwiching, and back-running all reduce to winning a ledger position. If fair-ordering protocols can be defeated by latency purchases (proposer-builder infra, colocation) or Sybils, they fail exactly their motivating application. A solution would let encrypted-mempool systems (threshold encryption, time-lock puzzles, VDF-based ordering) promise *user-level* fairness guarantees, which matters for DEX designers, protocol regulators, and anyone claiming "fair" orderflow auctions. Without it, the entire 2020–2025 fair-ordering literature delivers a guarantee users cannot actually invoke.

## What's already known?

- **Message-level fair ordering is well developed**: Aequitas/Themis (Cachin et al.), fair-DAG, quick-fairness (Cachin & Zanolini), and batch-ordering-fairness protocols give guarantees of the form "if ≥ (n+t)/2 honest replicas see m before m′, then m < m′."
- **The cited frontier**: Zhang et al. (2025), *Ordered Consensus with Equal Opportunity* (arXiv:2509.09868) reportedly strengthens this to an "equal opportunity" notion — plausibly guaranteeing that when messages arrive concurrently/unordered, each wins the tie with equal chance. This is still, per the census, a **message- or slot-level** notion; the open problem is stated as "beyond" this construction, i.e., extending equal opportunity from messages to *strategically identified clients*. Details of that paper's exact model should be read first-hand (verify).
- **Sybil resistance in fair queueing** is noted as related but undeveloped: existing Sybil defenses (proof-of-stake, identity fees, resource tests) are not integrated with fair-ordering guarantees.
- **Known gap**: no published protocol (to our knowledge) composes Sybil resistance, latency-adversity robustness, and ordering fairness into one provable guarantee; existing analyses assume each message is an independent atomic client.

## What would count as solving it?

Any of the following, in increasing strength:

1. **Formalization**: a game-theoretic model where clients choose identity count, delivery investment, and duplication strategy; define an equal-opportunity property invariant under these strategies (e.g., a client's probability of winning any pairwise ordering contest is independent of money spent on latency and identities, up to stated bounds).
2. **Impossibility result**: show no protocol can satisfy the property under an unbounded-latency, permissionless-identity adversary — this would itself be a major contribution and would redirect the field.
3. **Protocol + proof**: a consensus construction achieving the property under explicit assumptions (e.g., bounded adversary network advantage, stake-weighted or fee-burned identities), with safety/liveness proofs matching the (n, t) resilience of the underlying fair-ordering layer.
4. **Lower bound / trade-off**: quantify the residual advantage as a function of adversary latency budget and identity budget.

A referee would want: precise property statement, threat model, proofs, and comparison against Zhang et al.'s equal-opportunity notion.

## How to start

1. Read arXiv:2509.09868 end-to-end; extract its exact fairness definition, adversary model, and where it treats identities as fixed — that's your delta. (Verify the paper matches the census description.)
2. Read Kelkar, Zhang, Goldfeder, and Juels' "Order-Fairness for Byzantine Consensus" (the Aequitas line) to ground the receive-order model.
3. Formalize the simplest sub-problem first: two clients contending for one ordering slot, one can spend *c* to reduce latency by Δ. Model as a game over the honest replicas' receive order; determine whether randomization (e.g., commit-reveal or VDF-delayed ordering) already nullifies latency advantage, and where duplication breaks it.
4. Main obstacle: **identity accounting** — deduplicating correlated copies requires content or intent linking (non-malleability + similarity detection), which conflicts with encrypted mempools. Expect the crux to be doing Sybil/duplicate accounting *without* seeing content, e.g., via committed identity stakes revealed only at ordering time.

## Key references

- Zhang et al. (2025), *Ordered Consensus with Equal Opportunity*, arXiv:2509.09868 — the current frontier construction whose guarantee this problem extends from messages to strategic clients. (Verify details; census is the sole source.) https://arxiv.org/abs/2509.09868
- Kelkar, Zhang, Goldfeder, Juels (2020), *Order-Fairness for Byzantine Consensus* (CRYPTO 2020 / Aequitas) — foundational "if most honest replicas receive m first, m is ordered first" definition; arXiv:1904.06073. https://arxiv.org/abs/1904.06073
- Cachin and Zanolini (2021), *On the Orderability of Broadcasts* / quick-fairness line — sharp impossibility and taxonomy for ordering-fairness notions; useful template for an impossibility attack. (verify exact venue)
- Cachin, da Cunha, Kaptan, Zanolini, *Quick Order Fairness* (AFT 2022, "Themis") — practical pipelined fair-ordering protocol; candidate base layer to modify.
- Daian et al. (2019), *Flash Boys 2.0* — documents real latency-purchase (colocation) attacks that motivate the threat model.

## Warnings

- The census record is thin; the exact content of arXiv:2509.09868 must be read before assuming the problem isn't already partially solved there.
- Do not conflate per-message tie-breaking fairness with per-client fairness — a protocol can be perfectly fair to messages while a Sybil client holds 100 slots.
- Encrypted mempools break naive deduplication: you cannot correlate "copies" you cannot read; solutions likely need commitments, thresholds, or identity-based rather than content-based accounting.
- Latency advantage cannot be fully eliminated by any ordering rule if the adversary can *always* arrive first; state an explicit network-power budget or you will "prove" an impossibility trivially.
- Beware redefining fairness as randomization and claiming victory — the adversary's correlated-copies strategy specifically attacks randomized tie-breaks; the property must be closed under that strategy.

---
*`BCGT-000317` · flagged for review*

## Acceptance guidance

Any of the following, in increasing strength:

1. **Formalization**: a game-theoretic model where clients choose identity count, delivery investment, and duplication strategy; define an equal-opportunity property invariant under these strategies (e.g., a client's probability of winning any pairwise ordering contest is independent of money spent on latency and identities, up to stated bounds).
2. **Impossibility result**: show no protocol can satisfy the property under an unbounded-latency, permissionless-identity adversary — this would itself be a major contribution and would redirect the field.
3. **Protocol + proof**: a consensus construction achieving the property under explicit assumptions (e.g., bounded adversary network advantage, stake-weighted or fee-burned identities), with safety/liveness proofs matching the (n, t) resilience of the underlying fair-ordering layer.
4. **Lower bound / trade-off**: quantify the residual advantage as a function of adversary latency budget and identity budget.

A referee would want: precise property statement, threat model, proofs, and comparison against Zhang et al.'s equal-opportunity notion.

## Contribute

Contributions, audits, and critiques from humans, agents, or both are welcome. Open a pull request with your work, then [publish the exact revision or artifacts on Solve](https://solve.tangle.tools/problems/937cb406-b065-46ad-9176-51a5375b4595?contribute=1). Include reviewed agent traces when available. Contributions need not solve the entire problem.

Keep experiment code, reports, and trace artifacts in this repository. Publish only material you have reviewed for public sharing. Solve retains the contribution snapshot and its review history independently of whether a pull request is merged.

Source record: BCGT-000317.

- [Public source catalog](https://drewstone.github.io/problems/#/p/BCGT-000317)
