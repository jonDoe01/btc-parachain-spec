BTC-Relay Architecture
====================

BTC-Relay is a component of the BTC Parachain. Below, we provide an overview of it's components, as well as relevant actors - offering references to the full specification contained in the rest of this document. 

.. figure:: ../figures/architecture.png
    :alt: BTC Parachain architecture diagram

    Overview of the BTC Parachain architecture. Bitcoin block headers are submitted to the Verification Component, which interacts with the Utils, Parser and Failure Handling components, as well as the Parachain Storage. 


BTC Relay is a component / module of the BTC Parachain. 
It's main functionality is verification and storage of Bitcoin block headers, as well as verification of Bitcoin transaction inclusion proofs. 

Actors
~~~~~~~

BTC-Relay differentiates between the following actors:

* **Users** - Polkadot users which interact with the BTC Parachain, as well as other Parachains / modules which make calls to the verification functions of BTC-Relay.

* **Staked Relayers** - staked relayers lock up collateral in the BTC Parachain and are resposible for running Bitcoin `full nodes <https://bitcoin.org/en/full-node>`_ and verify that blocks submitted to BTC Relay are valid (and hence that transactional data for these blocks is available). Staked relayers can halt BTC-Relay in case a failure is detected. See `Failure Handling </spec/failure-handling.html#failure-handling>`_ for more details. 

.. note:: While any user can submit block headers to BTC-Relay, this role can be assigned to staked relayers, given these participants already run Bitcoin full nodes and check validity of stored blocks.

* **(Parachain) Governance Mechanism** - the correct operation of the BTC-Parachain, and hence of BTC-Relay, is overseen by the the Parachain's governance mechanism (e.g. committee of consensus partcipants and other key actors of the BTC-Parachain). 

.. note:: At the time of writing, the exact composition of the governance mechanism / commitee is not fully defined in Polkadot. This specification assumes the existence and correct operation of this merchanism - although modifications may be necessary to accomodate complexities and additional functional requirements, once the governance mechanism is fully specified.



Components
~~~~~~~~~~~


Storage
-------
 
Here the Bitcoin block headers and additional data structures, necessary for operating BTC-Relay, are persisted. See `Data Model </spec/data-model.html#data-model>`_ for more details. 

Verification
------------

The Verification component offers functionality to verify Bitcoin block headers and transaction inclusion proofs. See   `Storage and Verification </spec/functions.html#storage-and-verification>`_ for the function specification.

In more details, the verification component performs the operations of a `Bitcoin SPV client <https://bitcoin.org/en/operating-modes-guide#simplified-payment-verification-spv>`. See `this paper (Appendix D) <https://eprint.iacr.org/2018/643.pdf>`_ for a more detailed and formal discussion of the necessary functionality. 

* *Difficulty Adjustment* - check and keep track of Bitcoin's difficulty adjustment mechanism, so as to be able to determine when the PoW difficulty target needs recomputed.

* *PoW Verification* - check that, given a 80 byte Bitcoin block header and it's block hash, (i) the block header is indeed the pre-image to the hash (or compute it directly) and (ii) the PoW hash matches the difficulty target specified in the block header.

* *Chain Verification* - check that the block header references and existing block, already stored in BTC Relay. 

* *Main Chain Detection / Fork Handling* - when given two conflicting Bitcoin chains, determine the *main chain*, i.e. the chain with the most accumulated PoW (longest chain in Bitcoin, though under consideration of the difficulty adjustment mechanism). 

* *Transaction Inclusion Verification* - given a transaction, a reference to a block header, the transactions's index in that block and a Merkle Tree path, determine whether the transaction is indeed included in the specified block header (which in turn must be already verified and stored in the Bitcoin main chain tracked by BTC-Relay). 
 


An overview and explanation of different types of blockchain state verification in the context of cross-chain communication, specifically the difference between full validation of transaction and mere verification of their inclusion in the underlying blockchain, can be found `in this paper (Section 5) <https://eprint.iacr.org/2019/1128.pdf>`_.


Utils
-----

The Utils component provides "helper" functions used in by the Storage and Verification component, such as calculation of Bitcoin's double SHA256 hash, or re-construction of Merkle Trees. See `Utils </spec/helpers.html#utils>`_ for the full function specification.

Parser
------

The Parser component offers functions to parse Bitcoin's block and transaction data structures, e.g. extracting the Merkle Tree Root from a block header or the OP_RETURN field from a transaction. See `Parser </spec/parser.html#parser>`_ for the full function specification.

Failure Handling
-----------------

The Failure Handling component exposes the control mechanism for BTC-Relay's operation. Specifically, *staked relayers* and the *governance mechanism* can halt BTC-Relay if failures are detected, and recover from previous failures. See `Failure Handling </spec/failure-handling.html#failure-handling>`_ for and overview of possible failure modes and the full function specification.