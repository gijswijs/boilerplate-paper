\newcommand{\mpa}{{\footnotesize\textsf{MAX\_PAYMENT\_ALLOWED}}}
\newcommand{\mpam}{{\textsc{max\_payment\_allowed}}}

# Introduction

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

![Lightning Network](./slides/images/lightning.svg)

## Multi-hop Payments

![Multi-hop Payments](./slides/images/multi-hop.svg)

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

![BDA](./slides/images/sequence-diagram-simple.svg)

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

Since monitoring balances over time makes it possible to detect payments, it can be used to learn information about payments an adversary isn’t part of [@Malavolta2017].

As such it is an attack on value privacy.

:::

## Basic Balance Discovery Attack

![BDA](./slides/images/sequence-diagram.svg)

## Limits of Basic BDA

- Size of payment is limited by $\mpa{}$
- $\mpa{} = 2^{32} - 1$
- Channels with $balance_{ab} \geq 2^{32}$ remain undisclosed

::: notes

The maximum size of the payment is bounded, as specified in the Basis of Lightning Technology (BOLT) protocol.

A Payment cannot be bigger than a unsigned 32-bit integer, so the maximum size of a payment is $2^{32} - 1$

Channels with a $balance_{ab} \geq 2^{32}$ remain undisclosed, because Malory is unable to create a payment that is big enough for Alice to reply with the *insufficient funds* error. This doesn't mean all channels with a capacity $C_{ab} \geq 2^{32}$ remain undisclosed, because the balance distribution may be such that $balance_{ab} < 2^{32}$. But the amount of channels in the entire network with a capacity lower than $2^{32}$ do give a lower bound to the amount of channels of which the balance can be disclosed.

:::

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

## Number of channels

- LN is a graph $G$, # vertices $n = \left | G(V) \right |$ ; # edges $m = \left | G(E) \right |$
- $n = 3608$ ; $m = 9438$
- $m_{C > 2^{32}} = 1086$
- $m_{C > 2^{33}} = 540$

::: notes

By using a snapshot of the network we can establish the lower bound of the amount of channels that can have their balance disclosed.
In this paper we used a snapshot made on the 3rd of October, 2019.
The the two-way channel probing algorithm allows for the probing of channels with a capacity between $2^{32}$ and $2^{33}$. Based on this snapshot that would be $1086 - 540 = 546$ channels extra.

We can use the network shares of the different clients to estimate which part of the 540 channels with a $capacity \geq 2^{33}$

The LN is a graph $G$, with the number of vertices $n = \left | G(V) \right |$ and the number of edges $m = \left | G(E) \right |$.
Our analysis yielded the following values for n and m:
$n = 3608$
$m = 9438$ with $1086$ channels having a capacity greater than $2^{32}$ and $540$ channels having a capacity greater than $2^{33}$.

:::

## Channels Affected by Two-way BDA

<div id="fig:capacity">
\includestandalone[width=0.7\textwidth]{./slides/images/channels-affected}
Cumulative percentage graph of payment channels ordered by increasing capacity.
</div>


## BOLT implementation differences

- BOLT specs changed $\textsc{amount\_msat}$ 32-bit to 64 bit (unsigned)
- Theoretical limit of payment changed from $2^{32} - 1$ to $\textsc{MAX\_FUNDING\_ALLOWED}$
- BOLT protocol: 4 most significant bytes set to 0
- Clients implemented this differently

::: notes

The BOLT specs were changed on the 23rd of May, 2017 for the variable containing the payment amount, $\textsc{amount\_msat}$
Additional specifications required the sending node to set the four most significant bytes of amount_msat to 0, keeping the $\mpa{}$ at the original limit.

The theoretical limit changed to $2^{64} - 1$ but is now bounded by the maximum channel size set by $\textsc{MAX\_FUNDING\_ALLOWED}$ of 16,777,215 satoshi. ($2^{24}$)

C-lighting was the only client that fully adhered to the BOLT spec.
Eclair seems to have a arbitrary limit at 5 billion msat ($5 \cdot 10^9$).
LND has some unverified RPC's that don't take the limit into account at all.

So, using LND, it's possible to create fake payments up to the maximum channel capacity.

:::

## Implementation Differences and BDA

- Allows for probing balances $> \mpa{}$
- Only channels with LND and/or Eclaire nodes
- The number of susceptible channels can be estimated.

::: notes

The most obvious consequence of these differences in implementation is that it allows for probing channel balances above the limit of $\mpa{}$. Using a LND node it is, at least in theory, possible to run the BDA algorithm up until the limit of the $\textsc{MAX\_FUNDING\_ALLOWED}$.

Only channels that have LND and or Eclair clients on either end are susceptible. The reason that c-lightning is not susceptible is special, and leads to behavior that we will explain later on in this presentation.

:::

## Client Network Shares

|   Client    |  n   | Proportion (%) |  CI[^CI] (%)  |
| :---------- | ---: | -------------: | :-----------: |
| LND         |  220 |          80.59 | (79.35-81.83) |
| c-lightning |   40 |          14.65 | (13.54-15.76) |
| Eclair      |   11 |           4.03 |  (3.41-4.65)  |
| Other       |    2 |           0.73 |  (0.47-1.00)  |

Table: Proportion of nodes running different Lightning clients

[^CI]: 95% Confidence interval

::: notes

The self reporting of client type resulted in this table.

:::

## Channel Types

- Vertex type = client type (LND: $type_l$, c-lightning: $type_c$, Eclair: $type_e$)
- Edge type is defined by the pair of vertices it connects (e.g.: $type_{(l, c)}$ connects LND to c-lightning)
- $type_{(l, c)} \equiv type_{(c, l)}$

::: notes

The client software defines the type of the vertex. $type_l$ for LND nodes, $type_c$ for c-lightning nodes and $type_e$ for Eclair nodes.
An edge is said to be of $type_{(l, c)}$ if it connects a $type_l$ vertex and a $type_c$ vertex. The graph is without self-loops and undirected, so edge $type_{(l, c)} \equiv type_{(c, l)}$.

We can now combine the estimated network shares with the number of channels found in our snapshot, to estimate the different edge types.

:::


## Estimation of # Edge Types

- $P(type_{(l, l)}) = 0.8059^2$
- $P(type_{(c, c)}) = 0.1465^2$
- $P(type_{(e, e)}) = 0.0403^2$
- $P(type_{(l, c)}) = 2 \times 0.8059 \times 0.1465$
- $P(type_{(l, e)}) = 2 \times 0.8059 \times 0.0403$
- $P(type_{(c, e)}) = 2 \times 0.1465 \times 0.0403$

## Disclosable channels with $C \geq 2^{33}$

- $P(type_{([c, e, l], [c, e, l])}) \times 540$
- $P(type_{(l, l)} \cup type_{(l, e)}) \times 540 = 386$
- Total disclosable channels: $9438 - 540 + 386 = 9284$

::: notes

As mentioned earlier, only channels that have LND and/or Eclair clients on both ends are susceptible.

Now, assuming vertex type and channel capacity have a covariance of zero, the number of edges of each edge type, having a capacity greater than $2^{33}$ is calculated as follows: $P(type_{([c, e, l], [c, e, l])}) \times 540$

We are interested in the $type_{(l, l)}$ and $type_{(l, e)}$. Based on the network shares we found, we estimate that 386 channels can have their balance disclosed.

So a total of 9438 - 540 + 386 = 9284 channels have balances that can be disclosed. This is 98.4% of all channels.

:::

## Payment of Death

- IF: C-lightning node receives or routes a payment
- IF: Payment is $> \mpa{}$
- IF: Balances allow for payment $> \mpa{}$
- THEN: C-lighting node closes channel

::: notes

As mentioned before, the channels with c-lightning on one end showed behavior that made it impossible to  disclose the balance, but did reveal a vulnerability.

If a C-lightning node is being requested to route a payment to another node, or is the receiver of a payment,
and if that payment is bigger than $\mpa{}$,
then not only will the c-lightning node fail the payment, it will also close the channel with the previous node.

In a multi-hop payment, the previous node isn't necessarily the node from which the payment originated.

Consider the basic scenario, where Mallory and Alice run LND, and Bob runs c-lightning. Both channels between Mallory and Alice and between Alice and Bob have balances that allow for payments bigger than the $\mpa{}$ limit. If Mallory would create a fake payment with an amount above that limit, Bob would close down it’s channel with Alice, without Alice being able to mitigate this in any way.

We coined the term Payment of Death for this attack, after the infamous Ping of Death.

:::

## Channels affected by POD

- $type_{(l, c)}$ channels, with a balance above \mpa{}
- $m_{C > 2^{32}} * \times P(type_{(l, c)}) = 256$
- 2.7% of all channels is susceptible to POD

::: notes

We have notified the developers of the LN implementations by means of a responsible disclosure.

:::

# Discussion

## Comparison to basic BDA

|       Disclosable channels       | basic BDA [@Herrera-Joancomarti2019] (%) | two-way probing BDA (%) |
| :------------------------------- | ---------------------------------------: | ----------------------: |
| $C < 2^{32}$                     |                                    89.10 |                   88.49 |
| $C \geq 2^{32} \land C < 2^{33}$ |                                        0 |                    5.79 |
| $C \geq 2^{33}$                  |                                        0 |                    4.09 |
| TOTAL                            |                                    89.10 |                   98.37 |

Table: Basic BDA and Two-way probing BDA compared

::: notes

Herrera-Joancomarti [-@Herrera-Joancomarti2019] reported that 89.10% of all channels could have their balances exactly disclosed. In this table you see the contribution to the total percentage of channels being disclosed, split out by capacity size, compared between the basic and the two-way probing BDA.

Our research showed that we can improve the total share of channels being disclosed to 98.37%, a 9.27 percentage point increase.

The decrease of the contribution of the "small" channels, is caused by the slight increase of nodes and channels with more capital allocated.

:::

## Impact of POD

- Highly capitalized nodes are vulnerable
- Average capacity of channels with $C \geq 2^{32}$: 10,196,116 msat
- POD vulnerable channels represent 17.5% of network capacity

::: notes

The properties of the vulnerability make it so that the highly capitalized nodes are more vulnerable, since it are these nodes that have channels with a balance above \mpa{} limit.

The average capacity of those 1086 channels is roughly 10 thousand satoshi. Using that average combined with the estimated proportions of affected channels, 17.5% of the total capacity of the network could be taken down with an organized attack.

It’s reasonable to assume that these channels are responsible for routing a disproportionate amount of the payments on the network. So such an attack could have substantial impact on the ability to route payments of the network as a whole.

:::

## Countermeasures

- Randomly or selectively deny payment requests
- Do not provide the failure reason in the failure message
- Add random noise
- Enforce BOLT specs in all clients
- Don't consider malformed payments a reason for closing down channels.

::: notes

Countermeasures to Balance Disclosure Attacks were already known, but our research showed that if anything, the need for them has grown. Those countermeasures are:

- Randomly or selectively deny payment requests. The problem with randomly denying payments requests is that it deteriorates the performance of the payment network as a whole. Selectively denying payment requests based on simple heuristics, come with problems of its own. Because Lightning Network uses Onion Routing, a node cannot know the source of a payment request. This fact can be used to circumvent most of the heuristics that can be put in place to detect Balance Disclosure Attacks. The attacks would be considerably more difficult to mount, but not impossible, and more importantly economically not more costly. On the other hand those heuristics can also be abused by an adversary to make nodes deny payments on high-traffic channels. In other words, creating a denial of service attacks of sorts.
- Conveying less information in failure messages. If the failure message wouldn't let us make the distinction between an unknown payment hash and insufficient balance, this attack wouldn't work. But then again, in common scenario's not involving a BDA, this information is really useful for the functioning of the network.
- Add random noise to the actual balance. Accept or deny payments based on random noise being added or substracted to the balance. This is promising, but does leave us with the issue that we should be able to route the payment even if the random noise added satoshi's to the balance that aren't there. Some recent developments in this direction (JIT-routing with rebalancing) are promising.
- Specifically for the Payment of Death vulnerability it goes without saying that clients should enforce the BOLT specifications.
- Lastly, clients should not consider malformed payments a reason for closing down channels, since this can be abused by adversaries. It is easy to use Onion Routing to close down channels between benign actors. 

:::

## Conclusion

- Improved the basic BDA with 9.3 perc. point
- Showed channels capacity above $2^{32}$ and even $2^{33}$ are vulnerable for BDA
- Implementation differences can be leveraged
- Developed a new attack: POD
- Network wide POD attack has a large impact on network capacity

::: notes

This paper presented an improvement to the algorithm of the original BDA. We showed that by approaching a payment channel from both sides instead of from one side, payment channels with a higher capacity than in the original BDA are now also susceptible to this attack. We exposed differences in the implementation of the BOLT specification by the main three clients that can be leveraged by an adversary. These differences led us to develop new attack that closes down payment channels where the attacker isn’t part of. This attack has the power to shutdown a large part of the network capacity.

:::

## References {.allowframebreaks}

:::{#refs}
:::

# Thank you!

# Bitcoin Socratic FREE! bonus slides

## Academia is always behind

The reality has changed:

- AMP
- WUMBO channels
- POD vulnerability is fixed by now (right?)

::: notes

Are balances private? If you would do more BDA's over a longer time you can track payments.
Does AMP make that more difficult?

WUMBO channels are bigger, and therefor more channels will stay out of reach of a BDA?

:::

## Differential Privacy

![](slides/images/differantial-privacy.png)

::: notes

Suppose you have a process that takes some database as input, and returns some output.
This process can be anything. For example, it can be:

- computing some statistic
- a machine learning training process

To make a process differentially private, you usually have to modify it a little bit. Typically, you add some randomness, or noise.

Now, remove somebody from your database, and run your new process on it. If the new process is differentially private, then the two outputs are basically the same. This must be true no matter who you remove, and what database you had in the first place.

Now we are not trying to make a database hide the fact that somebody has been removed from the database. We are trying to hide transactions.
So the noise should be big enough to "hide" an average transaction.

:::

## Problems with adding noise to your balance

- You still need to be able to route the transaction
- Use a folded distribution? Breaks differential privacy.
- JIT routing with rebalancing?

# The End
