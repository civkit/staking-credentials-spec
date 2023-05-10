# Staking Credentials

This document specifies the Staking Credentials architecture and requirements for its constituents
protocols used for constructing anonymous credentials mechanisms in the context of Bitcoin financial
contracts and Civkit functionary services.

This draft is version 0.0.1 of the Staking Credentials architecture.

## Introduction

Staking Credentials is an architecture designed for Bitcoin financial contracts, which is based on 
anonymous credentials. These credentials not only unlink their issuance from their consumption, 
but also allow users to stake credentials, reducing the costs associated with entering future Bitcoin 
financial contracts or accessing CivKit functionary services. Contract providers can select the quality 
and thresholds of assets with precision to achieve a satisfactory level of risk, combining monetary 
and reputation-based strategies.

The Staking Credentials approach involves clients presenting a basket of scarce assets 
(Lightning preimage, stake certificates) and blinded credentials to the issuance server. The credentials 
are then authenticated and returned to the client. Once unblinded, the credentials can be used to fulfill 
a contract or service request with a service provider server. These credentials are anonymous, as a given 
credential cannot be linked to the protocol instance in which it was initially issued.

The Staking Credentials architecture comprises two protocols: credential issuance and redemption. The issuance 
protocol operates between two endpoints, known as the Requester and Issuer, and includes one function: Credential 
Authentication. The entity implementing Credential Authentication, referred to as the Issuer, is responsible for 
countersigning the credentials in response to requests from the Requester. Issuers may accept different base 
assets depending on their risk strategy and the overhead costs they wish to transfer to the Committer for 
fulfilling a contract or service request. Requesters and Issuers can agree on an authentication method 
based on the desired cryptographic properties.

The redemption protocol runs between two endpoints, referred to as the Client and Provider, and involves 
one function: Credential Consumption. The entity implementing Credential Consumption, known as the Client, 
is responsible for providing an authenticated credential that covers the contract risk or service request 
risk, as defined by the Provider. A Client can also be a Provider, but these entities can be separate as well. 
Similarly, an Issuer can be a Provider, but they can also be distinct entities. A Client can aggregate 
authenticated credentials from multiple Requesters.

The credentials issuance and redemption protocols operate in concert as shown in the figure below:

```
		 ___________						        ____________
		|	    |      1. Scarce Asset + Blinded Credentials       |	    |
		|	    |------------------------------------------------->|  	    |
		|	    |						       |	    |
		| Requester |						       |   Issuer   |
          ------|	    |	   2. Authenticated Unblinded Credentials      |	    |
          |	|	    |<-------------------------------------------------|	    |
	  |	|___________|						       |____________|
	  |										 ^
	  |										 |
	  |  3. Authenticated Unblinded Credentials					 |
	  |										 |
	  |				       5. Credentials Authentication Validation	 |
	  |										 |
	  |										 |
	 _V__________								    _____|________
	|	     |  4. Authenticated Unblinded Credentials + Service Request   |		  |
	|	     |------------------------------------------------------------>|		  |
	|	     |								   |		  |
	|   Client   |								   |   Proviedr   |
	|	     |         6. Contract Acceptance OR Contract Reject	   |		  |
	|	     |<------------------------------------------------------------|		  |
	|____________|								   |______________|

```

This documents describes requirements for both issuance and redemption protocols. It also provides
recommendations on how the architecture should be deployed to ensure the economic effectiveness of
assets, the compatibility of incentives, the conservation of the client UX, the privacy and security
and the decentralization of the ecosystem.

## Terminology

The following terms are used throughout this document.

- Client: An entity that pledges a Credential as assets to redeem a Contract or Service Request.
- Requester: An entity that commit a set of scarce assets to redeem an authenticated Credential from an Issuer.
- Issuer: An entity that accept a scarce asset in exchange of authenticating a Credential.
- Provider: An entity that accept a set of Credentials as payment in case of Contract risk or Service providance to the client.

## Architecture

The Staking Credentials architecture consists of four logical entities -- Client, Issuer, Requester, Provider --
that work in concert for credentials issuance and redemption.

## Credentials Issuance

The credential issuance is an authorization protocol in which the Requester presents scarce assets and blinded credentials to the Issuer 
for approval. Typically, the exchange rate of scarce assets to credentials, as determined by the Issuer, should have been discovered by the 
Committer during a non-specific protocol phase.

There are a number of scarce assets that can serve, including (non-limited):
- proof-of-payment (e.g Lightning preimage or on-chain txid)
- ecash token
- stakes certificates
- "faithful usage" reward

Issuer should consider which base assets to accept in function of the risk-management strategy they adopt. If
they aim for a 0-risk coverage, where all the contract or service execution risk is transferred on the counterparty,
they should pick up the default of on-chain/off-chain payment, where each contract cost unit should be covered by
a satoshi unit.

The credential issuance can be deployed as "native" or "reward".

In the "native" flow, a scarce asset for credential authentication should be exchanged a priori of a redemption phase:

```
		 ___________					    ____________
		|	    |	  1. Scarce asset + Credentials    |	        |
		|           |------------------------------------->|	        |
		|           |					   |	        |
		| Committer |					   |   Issuer   |
		|	    |	  2. Authenticated Credentials	   |		|
		|	    |<-------------------------------------|		|
		|___________|					   |____________|

```

In the "reward" context, one type of scarce asset, i.e contract fees are paid by the Client to the Provider,
a posteriori of a successful redemption phase. A quantity of authenticated credentials should be given
from the linked Issuer, back to the Committer:

```
	         ____________										
		|	     |						  ______________
		|	     |					         |		|
		|	     |						 |		|
		|   Client   |      1. Contract or Service fees		 |		|
		|	     |------------------------------------------>|   Provider   |
		|	     |						 |		|
		|____________|						 |		|
									 |______________|
										|
							 ____________		|
		 _____________				|	     |		|
		|	      |				|	     |		|
		|	      |				|	     |		|
		|	      |	   3. Authenticated	|   Issuer   |<----------
		|  Requester  |        Credentials	|	     |
		|	      |<------------------------|	     |  2. Unauthenticated Credential Callback
		|	      |				|____________|
		|_____________|

```

During issuance, the credentials should be blinded to ensure future unlinkability during the redemption phase.

Discovery of assets-to-credentials announced by the Issuer and consumed by the Client should be defined in
its own document.

A privacy-preserving communication protocol between Client and Issuer for credentials issuance should
be defined in its own document.

## Redemption

The redemption protocol is an identification process in which the Client presents authenticated credentials to the 
Provider in order to redeem acceptance of a contract. The credentials must be unblinded before being presented to the 
Provider. The number of credentials attached to the contract request should meet the contract liquidity units required 
by the Contract Provider. Generally, the exchange rate of credentials to contract/service units, as announced by the 
Provider, should have been discovered by the Committer during an unspecified protocol phase.

The protocol works as in the figure below:

```
					1. Authenticated Unblinded Credential
	 ____________			      		    +		      		      ______________
	|	     |				Contract/Service Request    		     |		    |
	|	     |---------------------------------------------------------------------->|		    |
	|	     |									     |		    |
	|   Client   |									     |   Provider   |
	|	     |        2. Contract/Service Acceptance OR Contract/Service Reject	     |		    |
	|	     |<----------------------------------------------------------------------|		    |
	|____________|									     |______________|
												    |
												    | 3. Contract/Service
												    | 	    Execution
												    |------------------------>
```

Depending on the contract execution outcome (success or failure), an additional fee can be paid by the Client
to the Provider. This fee should be determined in function of market forces constraining the type of contract/service
executed.

### Redemption Protocol Extensibility

The Staking Credentials and redemption protocols are both designed to be adaptable to extensions that enhance the current set of functionalities by 
introducing new types or modes of Bitcoin financial contracts coverage. Among these are long-term contracts based on a timelock that exceeds the 
typical requirements of Lightning payments. Another potential application is the adjustment of opening asymmetries in multi-party Lightning channel 
funding transactions.

## Deployment Considerations

### Lightning HTLC routing

In this model, a HTLC forwarder (Committer) send an off-chain payment and a list of blinded credentials to a Routing hop (Issuer).
The Routing hop counter-sign the blinded credentials and send them back to the HTLC forwarder (Credentials Authentication phase).
The HTLC forwarder (Client) unblinds the credential, attach a HTLC and send the request for acceptance to the Routing hop (Contract
Provider). If the quantity of credentials attached satisfies the Routing hop, the HTLC is accepted and forward to the next hop
in the payment path.

This model is shown below:

```
		 _____________	   1. Off-chain payment + Blinded Credentials	      ____________
		|	      |----------------------------------------------------->|		  |
		|	      |							     |		  |
		|     HTLC    |	      2. Counter-signed Blinded Credentials 	     |  Routing   |  4. HTLC relay
		|  forwarder  |<-----------------------------------------------------|    hop     |--------------->
		|	      |							     |		  |
		|	      |	     3. Unblinded Credentials + HTLC forward 	     |		  |
		|_____________|----------------------------------------------------->|____________|

```


## Security Considerations

A significant security risk from the Issuer's perspective is the potential double-spending of credentials by a Client. Within the context of the 
Staking Credential architecture, double-spending refers to the reuse of a credential for multiple contract requests, which can result in a direct 
loss of time value if contract fees are not paid by the Client for each individual contract request.

To mitigate this risk, the Issuer should maintain a private database to log every credential associated with a contract request. This private database 
should be accessible to the Contract Provider, allowing them to validate all credentials submitted for a contract request.

From the Committer's perspective, a major security risk involves the tampering of credentials during transmission between the Committer and Issuer 
hosts. Credentials could be compromised based on monitoring of credential volume traffic. To address this issue, credential transport should be 
authenticated, and packets should be obfuscated to protect against network-level inspection.

## References

- [Privacy Pass Architecture](https://www.ietf.org/archive/id/draft-ietf-privacypass-architecture-09.html)
- [Mitigating Channel Jamming with Stakes Certificates](https://lists.linuxfoundation.org/pipermail/lightning-dev/2020-November/002884.html)
- [Solving channel jamming issue of the lightning network](https://jamming-dev.github.io/book/about.html)
- [Discreet Log Contracts specification](https://github.com/discreetlogcontracts/dlcspecs/)
- [Lightning interactive transaction construction protocol](https://github.com/lightning/bolts/pull/851)

## Copyright

This document is placed in the public domain.
