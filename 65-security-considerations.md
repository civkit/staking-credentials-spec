# Staking Credentials - Security Considerations

This document details security considerations around the Staking Credentials design.

The major security and privacy attacks are laid out.

This draft is version 0.0.1 of the Security Considerations.

## Credentials Replay

Authenticated credentials can be re-played by a malicious Client to benefit from a service multiple times with a single scarce asset pledged during Issuance.

During the Redemption phase, the Provider should log the credentials usage in a compact accumulator with fast membership test operations.

## Credentials Authentication DoS

During the Credential Issuance phase, a malicious Requester can forward an unbounded number of `request_credentials_authentication` messages. The processing
of those messages, and notably the validation of the scarce asset can lead to a number of computational operations with an asymmetry in favor of the malicious
Requester (e.g fetching txid from a transaction index), therefore giving leverage to halt the Issuer host.

## Deanonymization Attacks

A Requester can be targeted by an eclipse attacks at the peer-to-peer gossip-network level, therefore being announced an issuance public key different from the
honest network. The gossip networking stack should be robust against eclipse attacks.

## Onion Bandwidth Jamming

The incoming onion communication channels of an Issuer or Provider can be overflowed with dummy onions forward until the previous hop dried up the connections
on those links.
