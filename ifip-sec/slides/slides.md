# Introduction

## History of Bitcoin?

## Bitcoin and scalability

- 1 block/10 min.
- Max. block size 1 MB
- Avg. Tx size 500 bytes
- Avg. 2000 Txs per 10 min.
- $\approx$ 3 Tx/s

## Capacity and security

Naively increasing capacity reduces security [@Sompolinsky2015]:

- Larger blocks
- Faster block creation

::: notes

Non-malicious forks happen when mining nodes are mining a block when they haven't yet heard of the latest block mined.
So they happen during the propagation of the latest block through the network.
Mining the wrong block is a waste of resources, and it reduces security, because it diverts computing power away from securing the actual blockchain to "securing" an orphaned block that will be rejected by the network.

Larger blocks take longer to propagate through the network.
At any time a larger proportion of mining nodes will not have heard of the latest block.
Those mining nodes will keep working on appending to what they think is the latest block.
Hence more forks of the blockchain will occur resulting in a reduction of the security threshold of the system

Similarly, faster block creation means more blocks are being created while the latest block propagates through the network. Leading to more orphaned blocks being created.

So there's a delicate balance to be struck between propagation time, block size and block creation rate.
:::

## Off-chain transactions

Not broadcasting Txs to the blockchain, to increase capacity.

- High Frequency Trading [@Hearn2013]
- Decker-Wattenhofer duplex payment channels [@Decker2015]
- Poon-Dryja payment channels [@Poon2016]
- Decker-Russell-Osuntokun eltoo Channels [@Decker2018]

::: notes

The idea of reducing the amount of transactions to be broadcasted, without sacrificing the core properties of Bitcoin, namely it being trustless and distributed is nothing new.

Satoshi Nakamoto himself proposed High Frequency Trading: Updating transactions that are not yet committed to the blockchain. By keeping track of the updates with a sequence number and only allowing for the highest/last sequence number to be mined. Only the final state of the transaction would be committed to the blockchain. All prior states would be kept off-chain. But this proposal couldn't operate in a trustless environment. An maleficent party could collude with a miner to broadcast a prior state.

Payment channels are a technique for exchanging transactions between two participants. A typical payment channel will only commit two transactions to the blockchain. A funding or channel opening transaction, and a close transaction. During the lifetime of a channel, the participants update the current state of the channel by exchanging transactions that aren't broadcasted to the blockchain.

Several different techniques have been proposed. Poon-Dryja payment channels form the basis of Lightning Network. Lightning Network is currently the only Payment Channel Network in production.

:::

## Lightning Network

Drawing of inner workings

## Multi-hop Payments

Drawing of multi-hop payments HTLC

# Background

## LN privacy Threat Model [@Malavolta2017]

- Balance security
- Serializability
- (Off-path) value privacy
- (On-path) relationship anonymity

::: notes

– Balance security: participants don’t run the risk of losing coins to a malevolent adversary.
– Serializability: executions of a PCN are serializable as understood in concurrency control of transaction processing. All transactions that are concurrently processed are assumed to be correct by themselves (no dependencies exist between them), so *any* serial execution of these transactions is legitimate. Serializability states that if you can convert a schedule of execution of transactions into a any serial schedule of execution of those same transactions, resulting in an equivalent system state, is correct.
– (Off-path) value privacy: malicious participants in the network cannot learn information about payments they aren’t part of.
– (On-path) relationship anonymity: given at least on honest inter- mediary, corrupted intermediaries cannot determine the sender and the receiver of a transaction better than just by guessing.

:::

## Naïve Balance Discovery Attack

Drawing of simple BDA

## Enhancements of Naïve BDA

- $M$ creates fake invoice h(x)
- Use a binary search algorithm
- Parameterize the accuracy threshold
- Attack all open channels of $A$

::: notes

M creates a fake invoice, as if created by B.  The fake invoice cannot be detected by A, only by B. This greatly reduces the economic costs for the adversary.

A binary search is far more effective in honing in on the final result.

Using a lower threshold for accuracy improves the speed of the algorithm.

Balances of all public channels that A has with peer nodes can be easily discovered, once a channels with sufficient capacity has been opened between M and A.

:::

## Limits of Basic BDA

- Size of payment is limited by $MAX_PAYMENT_ALLOWED$
- $MAX_PAYMENT_ALLOWED = 2^{32} - 1$
- Channels with $balance_{ab} \geq 2^{32}$ remain undisclosed

::: notes

The maximum size of the payment is bounded, as specified in the Basis of Lightning Technology (BOLT) protocol.

A Payment cannot be bigger than a unsigned 32-bit integer, so the maximum size of a payment is $2^{32} - 1$

Channels with a $balance_{ab} \geq 2^{32}$ remain undisclosed, because Malory is unable to create a payment that is big enough for Alice to reply with the *insufficient funds* error. This doesn't mean all channels with a capacity $C_{ab} \geq 2^{32}$ remain undisclosed, because the balance distribution may be such that $balance_{ab} < 2^{32}$. But the amount of channels in the entire network with a capacity lower than $2^{32}$ do give a lower bound to the amount of channels of which the balance can be disclosed.

# Method

## BDA Improvement: Two-way channel probing

- Channels with a capacity $C_{ab} \geq 2^{32}$ and  $C_{ab} < 2^{33}$
- Open a channel with $A$
- If the Basic BDA algorithm fails, open up a channel with B
- Continue the attack $M \rightarrow B \rightarrow A$

::: notes

Channels with a capacity between $2^{32}$ and $2^{33}$ have by definition one side of the channel with a balance lower than $2^{32}$. So when the basic BDA is exhausted and doesn't return a value, you can open a second channel with $B$ and continue the attack. This two-way channel probing raises the lower bound of disclosable channels to $2^{33}$. Again, channels with a capacity larger than or equal to $2^{33}$ might still be disclosable, depending on the distribution of the balance, but the lower bound of disclosable channels is channels with a capacity less than $2^{33}$.

:::

## Leverage Client Type

There are three main Lightning Clients

- Use 1ML to find client types
- Select 3 most common clients
- Test algorithms against different clients

::: notes

We used 1ML Lightning Network Search and Analysis Engine to estimate respective proportions of different client in Lightning Network.
1ML is a website that publishes the current state of the LN graph and allows for node owners to self-report on a voluntary basis the type of client they use.

Using the 3 clients with the largest network share, we set up a local cluster of Lightning Nodes with all 3 clients represented. All LN nodes used Bitcoin Core’s Bitcoind implementation as the Bitcoin backend. Bitcoind ran in regression testing mode, known as regtest mode. This is a local test mode, making it possible to almost instantly create blocks. Using regtest mode, the different implementations could be tested without incurring transaction fees for the on-chain transactions and without having to wait for blocks to be mined.

In this local cluster we tested the Basic BDA algorithm and the improved algorithm against each possible permutation of 3 clients, M, A and B.

:::

# Results

## Payment of Death

- MAX_PAYMENT_ALLOWED

## Channels affected

# Discussion

## Impact of POD

## Countermeasures

# Conclusion



## References {.allowframebreaks}

:::{#refs}
:::

# Thank you!
