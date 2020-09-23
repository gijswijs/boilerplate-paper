\newcommand{\mpa}{{\footnotesize\textsf{MAX\_PAYMENT\_ALLOWED}}}
\newcommand{\mpam}{{\textsc{max\_payment\_allowed}}}

# Introduction

## Bitcoin and scalability

- 1 block/10 min.
- Max. block size 1 MB
- Avg. Tx size 500 bytes
- Avg. 2000 Txs per 10 min.
- $\approx$ 3 Tx/s

::: notes

Good afternoon, Good evening, Good night or Good afternoon, depending on the time zone you currently occupy.

I will present our paper "Improvements of the Balance Discovery Attack on Lightning Network Payment Channels"

Bitcoin's scalability issues are well known.

With 1 block broadcasted to the chain every 10 minutes and a block size of 1 MB, and an average transaction size of 500 bytes, we see that Bitcoin roughly allows for 3 transactions per second.

If Bitcoin wants to be a viable alternative to current centralized payment networks it needs to achieve comparable throughput which is in the order of magnitude of several thousand transactions per second.

:::

## Off-chain transactions

Not broadcasting Txs to the blockchain, to increase capacity.

- High Frequency Trading [@Hearn2013]
- Decker-Wattenhofer duplex payment channels [@Decker2015]
- Poon-Dryja payment channels [@Poon2016]
- Decker-Russell-Osuntokun eltoo Channels [@Decker2018]

::: notes

By keeping transaction off-chain, you could greatly increase the throughput of the network.

The idea of reducing the amount of transactions broadcasted is nothing new.

Satoshi Nakamoto himself proposed High Frequency Trading: Updating transactions that are not yet committed to the blockchain. But his was not secure and could not proposal couldn't operate in a trustless environment.

Payment channels are a technique for exchanging transactions between two participants. A typical payment channel will only commit two transactions to the blockchain. A funding transaction, and a close transaction. During the lifetime of a channel, the participants update the current state of the channel by exchanging commitment transactions that aren't broadcasted to the blockchain. The funding transaction determines the total capacity of the channel, whereas the commitment transactions keep track of the current balance of a channel. So at the any point in time the balance of the channel always adds up to the total capacity.

Several different payment channel techniques have been proposed. Poon-Dryja payment channels form the basis of Lightning Network. Lightning Network is currently the only Payment Channel Network in production.

:::

## Lightning Network

![Lightning Network](./slides/images/lightning-1.svg)

::: notes

Here we see a simplified view of the Lightning Network.

It is a Peer-to-peer network where nodes create payment channels between each other.

Note that the network graph and the channel balances are publicly broadcasted, the current state of the channels are not. So each node only knows the balance of its own channels.

:::
## A Pays 20 mBTC to F

![Lightning Network](./slides/images/lightning-2.svg)

::: notes

If node A wants to pay node F 20 mBTC, it has to find a route that potentially can route the payment.

in this case A -> C -> F.

:::

## A Pays 20 mBTC to F

![Lightning Network](./slides/images/lightning-3.svg)

::: notes

If the balances along the route allow for the payment to be routed, then the balances are updated accordingly.

:::

## A Pays 20 mBTC to F

![Lightning Network](./slides/images/lightning-4.svg)

## A Pays 30 mBTC to F

![Lightning Network](./slides/images/lightning-5.svg)

::: notes

If A would want route a second payment to F, this time of 30 mBTC it might try to use the same route: A -> C -> F
The capacities of those channels are big enough to allow for that payment, and since the balance between C and F are unknown to A, it is worth a try.
C will reply with an error though, stating insufficient balance.

:::

## A Pays 30 mBTC to F

![Lightning Network](./slides/images/lightning-6.svg)

::: notes

A will try to find an alternative route for the payment, in this case: A -> B -> D -> F
The capacities of those channels *are* big enough to allow for that payment.

:::

## A Pays 30 mBTC to F

![Lightning Network](./slides/images/lightning-7.svg)

::: notes

This time the payment does succeed, since the balances are sufficient.
This routing by trial and error is an integral part of the Lightning Network.

:::

## Payment Hash

- Receiver will only accept payments with a known payment hash
- Payment hash is the hash of a random number, created by the *receiver*
- Receiver shares the payment hash with the *sender*
- Sender uses payment hash to identify the transaction

::: notes

What is important to note is that F will only accept a payment if F has an open invoice for that payment. An invoice is a hash of a random number, created by F. This hash is called the payment hash. F shares the payment hash with A, and A will use that hash to identify the payment.

:::

# Background


## Threat Model [@Malavolta2017]

- Balance security
- Serializability
- (Off-path) value privacy
- (On-path) relationship anonymity

::: notes

With this very basic introduction out of the way, let us proceed to the research background of our paper.

A formal analysis of privacy in the context of PCNs has been hindered by a lack of a rigorous threat model. Malavolta [-@Malavolta2017] proposed a threat model in 2017 that by now is the de facto standard for Lightning Network privacy research. It consists of 4 points.

– Balance security: participants shouldn't run the risk of losing their coins.
– Serializability: Transactions that are executed concurrently shouldn't leave the system in a conflicting state. Another way of looking at this is to say that the Lightning Network should not reintroduce the double spending problem that the Bitcoin Network so elegantly solved.
– (Off-path) value privacy: Nodes should not learn information about payments they are not part of.
– (On-path) relationship anonymity: Nodes in a multi-hop payment path cannot determine the sender and receiver of a transaction.

Our paper focusses on the 3rd point: Value privacy.

:::

## Naïve Balance Discovery Attack

![BDA](./slides/images/sequence-diagram-simple.svg)

::: notes

It was already known that the system of trial and error for making payments can be used to probe balances in the network.

:::

## Enhancements of Naïve BDA [@Herrera-Joancomarti2019]

- Mallory creates fake invoice h(x)
- Use a binary search algorithm
- Parameterize the accuracy threshold
- Attack all open channels of $Alice$ with her peers

::: notes

Herrera-Joancomarti [-@Herrera-Joancomarti2019] was the first to propose a workable Balance Disclosure Attack.

His main contributions were:

M creates a fake invoice, as if created by B.  The fake invoice cannot be detected by A, only by B. This greatly reduces the economic costs for the adversary.

A binary search is used for effectiveness

A accuracy threshold is used to end the algorithm faster once the result is within a certain range of the actual balance.

Balances of all public channels that Alice has with her peer nodes can be easily discovered, once a channels with sufficient capacity has been opened between Mallory and Alice.

Since monitoring balances over time makes it possible to detect payments, it can be used to learn information about payments an adversary isn’t part of [@Malavolta2017].

As such the balance disclosure attack it is an attack on value privacy.

:::

## Basic Balance Discovery Attack

![BDA](./slides/images/sequence-diagram.svg)

::: notes

Here you can see what it means for the algorithm to use fake payment hashes. Instead of payments succeeding, payments no either don't succeed because of an unknown payment hash, or because of insufficient balance. The distinction between those two failures makes it possible to disclose the balance.

:::

## Limits of Basic BDA

- Size of payment is limited by $\mpa{}$
- $\mpa{} = 2^{32} - 1$ mSat
- Channels with $balance_{ab} \geq 2^{32}$ remain undisclosed

::: notes

The maximum size of the payment is bounded, as specified in the Lightning protocol.

A Payment cannot be bigger than $2^{32} - 1$ milli-satoshi

Channels with a $balance_{ab} \geq 2^{32}$ remain undisclosed, because Malory is unable to create a payment that is big enough for Alice to reply with the *insufficient funds* error. This doesn't mean all channels with a capacity $C_{ab} \geq 2^{32}$ remain undisclosed, because the balance distribution may be such that $balance_{ab} < 2^{32}$. But the amount of channels in the entire network with a capacity lower than $2^{32}$ do give a lower bound to the amount of channels of which the balance can be disclosed.

:::

# Method

## BDA Improvement: Two-way channel probing

- Channels with a capacity $C_{ab} \geq 2^{32}$ and  $C_{ab} < 2^{33}$
- Open a channel with Alice
- If the Basic BDA algorithm fails, open up a channel with Bob
- Continue the attack Mallory $\rightarrow$ Bob $\rightarrow$ Alice

::: notes

This brings us to the method section of our research.

We propose two-way channel probing.

Channels with a capacity between $2^{32}$ and $2^{33}$ have by definition one side of the channel with a balance lower than $2^{32}$. So when the basic BDA is exhausted and doesn't return a value, Mallory can open a second channel with Bob and continue the attack. This two-way channel probing raises the lower bound of disclosable channels to $2^{33}$. Again, channels with a capacity larger than or equal to $2^{33}$ might still be disclosable, depending on the distribution of the balance, but the lower bound of disclosable channels is channels with a capacity less than $2^{33}$.

:::

## Leverage Client Type

- Use 1ML to find client types
- Select 3 most common clients
- Test algorithms against different clients

::: notes

Secondly, we propose using the client type as a way to improve the Balance Discovery Attack. All clients need to adhere to the Lightning Network protocol, but the protocol isn't implemented uniformly, as we will show.

We used 1ML Lightning Network Search and Analysis Engine to estimate respective proportions of different client in Lightning Network.
1ML is a website that publishes the current state of the LN graph and allows for node owners to self-report on a voluntary basis the type of client they use.

Using the 3 clients with the largest network share, we set up a local cluster of Lightning Nodes with all 3 clients represented. All LN nodes used Bitcoin Core’s Bitcoind implementation as the Bitcoin backend. Bitcoind ran in regression testing mode, known as regtest mode. This is a local test mode, making it possible to almost instantly create blocks. Using regtest mode, the different implementations could be tested without incurring transaction fees for the on-chain transactions and without having to wait for blocks to be mined.

In this local cluster we tested the Basic BDA algorithm and the improved algorithm against each possible permutation of 3 clients.

:::

# Results

## Channels Affected by Two-way BDA

<div id="fig:capacity">
\includestandalone[width=0.7\textwidth]{./slides/images/channels-affected}
Cumulative percentage graph of payment channels ordered by increasing capacity.
</div>

::: notes

based on a snapshot of the network we show that our new algorithm shows a 6 percentage point increase in vulnerable channels.
The graph shows the original limit of the BDA, and the improved limit at twice that value.

:::

## Client Differences and BDA

- 2 clients allow for payment requests above the protocol limit
- Allows for probing balances $> \mpa{}$
- Only channels with LND and/or Eclaire nodes
- The number of susceptible channels can be estimated.

::: notes

Our research showed that 2 out of 3 of the main clients (namely LND and Eclaire) allow for payment requests above the limit set by the protocol.

The most obvious consequence of this is that it allows for probing channel balances above that limit. Using a LND node it is, at least in theory, possible to run the Basic BDA algorithm up until the limit of maximum channel capacity.

We tried this in a test setting, and this was indeed the case. But in special cases this led to unexpected behavior, that we will explain later.

For now it is suffice to say that channels that have LND and or Eclair clients on either end are vulnerable to limitless balance probing.

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

Since client type isn't broadcasted on the LN network, we can only estimate how many channels are vulnerable for limitless probing.

Shown here is the table with the network shares, based on the information retrieved from the 1ML website.

Using this information we estimate that additional 4 percentage point of channels can are vulnerable to this limitless probing.

:::

## Payment of Death

- IF: C-lightning node receives or routes a payment
- IF: Payment is $> \mpa{}$
- IF: Balances allow for payment $> \mpa{}$
- THEN: C-lighting node closes channel

::: notes

As mentioned before, the channels with the remaining most common client, c-lightning, showed behavior that made it impossible to  disclose the balance, but did reveal a vulnerability of the network.

If a C-lightning node is being requested to route a payment to another node, or is the receiver of a payment,
and if that payment is bigger than $\mpa{}$,
then not only will the c-lightning node fail the payment, it will also close the channel with the previous node.

In a multi-hop payment, the previous node isn't necessarily the node from which the payment originated.

Consider the basic scenario, where Mallory and Alice run LND, and Bob runs c-lightning. Both channels between Mallory and Alice and between Alice and Bob have balances that allow for payments bigger than the $\mpa{}$ limit. If Mallory would create a fake payment with an amount above that limit, Bob would close down it’s channel with Alice, without Alice being able to mitigate this in any way.

We coined the term Payment of Death for this attack, after the infamous Ping of Death.

Using the same estimation techniques we estimated that 2.7% of all channels were susceptible for the Payment of Death.

Obviously we have notified the developers of the LN implementations by means of a responsible disclosure. And clients should by now have been patched.

:::

# Discussion

## Comparison to basic BDA

|       Disclosable channels       | basic BDA [@Herrera-Joancomarti2019] (%) | improved BDA (%) |
| :------------------------------- | ---------------------------------------: | ----------------------: |
| $C < 2^{32}$                     |                                    89.10 |                   88.49 |
| $C \geq 2^{32} \land C < 2^{33}$ |                                        0 |                    5.79 |
| $C \geq 2^{33}$                  |                                        0 |                    4.09 |
| TOTAL                            |                                    89.10 |                   98.37 |

Table: Basic BDA and Two-way probing BDA compared

::: notes

Let's compare our results with the results of Herrera-Joancomarti.

Herrera-Joancomarti [-@Herrera-Joancomarti2019] reported that 89.10% of all channels could have their balances exactly disclosed. In this table you see the contribution to the total percentage of channels being disclosed, split out by capacity size, compared between the basic BDA and our improved BDA.

Our research showed that we can improve the total share of channels being disclosed to 98.37%, a 9.27 percentage point increase.

The decrease of the contribution of the "small" channels, is caused by the slight increase of nodes and channels with more capital allocated.

:::

## Impact of POD

- Highly capitalized nodes are vulnerable
- Average capacity of channels with $C \geq 2^{32}$: 10,196,116 msat
- POD vulnerable channels represent 17.5% of network capacity

::: notes

The properties of the vulnerability make it so that the highly capitalized nodes are more vulnerable, since it are these nodes that have channels with a balance above \mpa{} limit.

The average capacity of those channels is roughly ten thousand satoshi. Using that average combined with the estimated proportions of affected channels, 17.5% of the total capacity of the network could have been taken down with an organized attack.

It’s reasonable to assume that these channels are responsible for routing an even bigger share of the payments on the network. So such an attack could have substantial impact on the ability to route payments of the network as a whole.

:::

## Countermeasures

- Randomly or selectively deny payment requests
- Do not provide the failure reason in the failure message
- Add random noise
- Enforce Lightning Network specs in all clients
- Don't consider malformed payments a reason for closing down channels.

::: notes

Countermeasures to Balance Disclosure Attacks were already known, but our research showed that if anything, the need for them has grown. Those countermeasures are:

- Randomly or selectively deny payment requests. The problem with denying payments requests is that it deteriorates the performance of the payment network as a whole. 
- Conveying less information in failure messages. If the failure message wouldn't let us make the distinction between an unknown payment hash and insufficient balance, this attack wouldn't work. But then again, in scenario's not involving benign actors, this information is really useful for the functioning of the network.
- Add random noise to the actual balance. Accept or deny payments based on random noise being added or substracted to the balance. This is promising, but does leave us with the issue that we should be able to route the payment even if the random noise *added* satoshi's to the balance that are not actually there. Some recent developments, namely Just-In-Time-routing with rebalancing look promising.
- Specifically for the Payment of Death vulnerability it goes without saying that clients should enforce the BOLT specifications.
- Lastly, clients should not consider malformed payments a reason for closing down channels, since this can be abused by adversaries. It is easy to create malformed payment requests to close down channels between benign actors.

:::

## Conclusion

- Improved the basic BDA with 9.3 perc. point
- Showed channels capacity above $2^{32}$ and even $2^{33}$ are vulnerable for BDA
- Implementation differences can be leveraged
- New attack POD has large impact on total network capacity

::: notes

This paper presented an improvement to the algorithm of the original BDA. We showed that by approaching a payment channel from both sides instead of from one side, payment channels with a higher capacity than in the original BDA are now also susceptible to this attack. We exposed differences in the implementation of the Lightning Network specification by the main three clients that can be leveraged by an adversary. These differences led us to develop new attack that closes down payment channels where the attacker isn’t part of. This attack has the power to shutdown a large part of the total network capacity.

:::

## References {.allowframebreaks}

:::{#refs}
:::

# Thank you!