# Formal Proof of Correctness

This section provides formal reasoning that the Relaxed Mempool satisfies safety, liveness, and data availability. The analysis follows the standard Byzantine fault-tolerant model with n = 3f + 1 validators, where at most f may behave arbitrarily and at least 2f + 1 behave honestly. The network is partially synchronous, meaning that after some unknown global stabilization time (GST), message delays are bounded by a finite value Δ. Two quorum thresholds are used in the protocol:

- Strong quorum: 2f + 1 signatures, required for block finalization.  
- Weak quorum: f + 1 signatures, sufficient for speculative progress and data availability, since such a quorum always includes at least one honest validator.

Each validator continuously produces DAG nodes, where each node represents a batch of transactions. A block is formed as a causally closed subset of DAG nodes (a DAG cut). The consensus layer finalizes blocks by certifying such DAG cuts through quorum signatures.

---

## 1. Safety

**Property:** Two conflicting blocks cannot both be finalized by honest validators. A conflict refers to two blocks representing distinct DAG cuts that cannot coexist in one consistent history.

**Proof:**  
(1) Each finalized block must contain a valid strong certificate with at least 2f + 1 signatures.  
(2) Any two strong quorums overlap in at least f + 1 validators, and at least one in this intersection is honest.  
(3) An honest validator never signs two conflicting blocks.  
(4) Therefore, if one block obtains a strong certificate, a conflicting block cannot collect an independent quorum because the shared honest validators would refuse to sign it.  
(5) Consequently, at most one of any pair of conflicting blocks can be finalized.

The Relaxed Mempool enforces causal dependency rules across DAG nodes. A block cannot be finalized unless all DAG nodes it references, directly or indirectly, are already certified or finalized. Thus, any finalized block has a complete causal history, and every dependency appears in the final order. This prevents finalizing a block while its dependencies remain unresolved.

Equivocation by Byzantine validators is detectable. A validator that issues two conflicting DAG nodes at the same local height exposes its signature in both. Honest validators verify signatures and reject any branch that includes conflicting certificates. Since a strong certificate requires 2f + 1 signatures, equivocation cannot produce two finalized conflicting blocks.

By quorum intersection, dependency enforcement, and equivocation detection, the protocol guarantees that only a single consistent history can be finalized. Safety therefore holds.

---

## 2. Liveness

**Property:** The system eventually finalizes new blocks once the network becomes synchronous after GST, provided that at least f + 1 honest validators remain active.

**Proof:**  
(1) Each validator can create DAG nodes representing local batches, and each node is weakly certified by obtaining f + 1 acknowledgments.  
(2) Even if the consensus layer is temporarily blocked during asynchronous periods, honest validators continue generating and exchanging DAG nodes. The DAG therefore grows continuously, storing pending transactions for later commitment.  
(3) After GST, message delays become bounded by Δ, allowing validators to exchange missing DAG nodes and certificates reliably.  
(4) A subset of DAG nodes that are all weakly certified and causally complete forms a DAG cut eligible for commitment.  
(5) When 2f + 1 validators confirm this cut, the corresponding block obtains a strong certificate, indicating agreement by a supermajority of honest nodes.  
(6) The consensus module then commits this block and all its referenced nodes.

Since validators continue producing DAG nodes during asynchrony and strong certificates eventually form after GST, the system cannot enter a permanent deadlock. Any unfinalized data are later included in a committed block once sufficient signatures accumulate. The Relaxed Mempool therefore satisfies liveness.

---

## 3. Data Availability

**Property:** All batches included in finalized DAG nodes remain accessible to honest validators. No committed block can depend on unavailable data.

**Proof:**  
(1) A validator signs a batch only after fully receiving and verifying its contents. Therefore, each weak certificate with f + 1 signatures ensures that at least one honest validator holds the complete batch data.  
(2) Even if up to f validators crash or act maliciously, at least one honest validator in the quorum retains the data.  
(3) When new DAG nodes are produced, they reference recent nodes by hash and certificate, which propagates both the existence and accessibility of earlier data throughout the network.  
(4) Once a batch obtains a strong certificate, its existence and location are known to all honest validators through subsequent DAG references.  
(5) Before finalization, validators verify that every node within the selected DAG cut is certified and retrievable. Nodes that are unavailable or uncertified cannot appear in a finalizable block.  
(6) As a result, any finalized block consists entirely of DAG nodes whose data are confirmed to be accessible to honest validators.

Because every certified batch is acknowledged by at least one honest node and all finalized DAG nodes are certified, all committed data remain accessible. Data availability is therefore guaranteed.

---

## 4. Conclusion

Under the standard Byzantine fault-tolerant assumptions of quorum intersection, partial synchrony, and authenticated communication, the Relaxed Mempool satisfies the following three correctness properties:

1. Safety: Two conflicting blocks cannot both be finalized.  
2. Liveness: The system continues to make progress once the network stabilizes.  
3. Data Availability: All finalized data remain accessible through honest validators.
