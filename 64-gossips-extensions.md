# Staking Credentials -- Gossips Extensions

This document specifies the BOLT 7 Gossips Extensions inside the the Staking Credentials framework.

The message data format, validation algorithms and implementations considerations are laid out.

This draft is version 0.0.1 of the Gossips Extensions.

### The `credential_policy` message

This gossip message contains informations regarding an Issuer's scarce assets acceptance and
credential policy. It ties a list of asset proofs and credentials to the associated Issuer node key.

1. type: 37563 (`credential_policy`)
2. data:
    * [`signature`:`signature`]
    * [`u32`:`timestamp`]
    * [`u16`:`flen`]
    * [`point`: `issuance_pubkey`]
    * [`flen*byte`: `assetproof_features`]
    * [`u16`:`flen`]
    * [`flen*byte`:`credentials_features`]
    * [`u16:`flen`]
    * [`u32`:`asset_to_credential`]
    * [`u32`:`expiration_height`]

#### Requirements

The origin node:
  - MUST set `signature` to the signature of the double-SHA256 of the entire remaining packet after `signature` (using the key given by `node_id`)
  - MUST set `timestamp` to be greater than that of any previous `credential_policy` it has previously created.
    - MAY base it on a UNIX timestamp.
  - MUST set `issuance_pubkey` to the public key used to authenticated credentials during Issuance.
  - MUST set `assetproof_features` to the list of supported scarce assets.
  - MUST set `credentials_features` to the list of supported credentials cryptosystems.
  - MUST set `asset_to_credential` to the ratio between scarce assets accepted and quantity of authenticated credentials released.
  - MUST set `expiration_height` to the block height until which the credentials are valid.

The receiving node:
  - if the signature is not valid for the entire remaining packet after `signature`:
    - MUST reject the policy message
  - if the `expiration_height` is already past for the Bitcoin chain tip seen by the client:
    - MUST reject the policy message

#### Rationale

The asset to credentials ratio precises for an amount of scarce assets provided an equivalent quantity of credentials that is authenticated by the Issuer.

The expiration height must be indicated to enable rotation of the credential policy.

## The `service_policy` message

This gossip message contains informations regarding a Contract's liquidity risk-management policy.
It ties a list of credential issuers and the covered services to the associated Provider node key.

1. type: 37564 (`service_policy`)
2. data:
    * [`signature`:`provider_signature`]
    * [`u32`:`timestamp`]
    * [`u16`:`pubkeylen`]
    * [`pubkeylen*byte`:`pubkeys`]
    * [`u16`:`servicelen`]
    * [`servicelen*byte`:`serviceids`]
    * [`u32`: `credentials_to_service`]
    * [`u32`:`expiration_height`]

#### Requirements

The origin node:
  - MUST set `signature` to the signatuee of the double-SHA256 of the entire remaining packet after `signature` (using the key given by `node_id`)
  - MUST set `timestamp` to be greater than that of any previous `service_policy` it has previously created.
  - MUST set `pubkeys` to the list of all Issuer pubkeys supported as announced by `credentials_policy`.
  - MUST set `service_ids` to the list of all Providers services supported.
  - MUST set `expiration_height` to the block height until which the credentials are valid.

The receiving node:
  - if the signature is not valid for the entire remaining packet after `signature`:
    - MUST reject the policy message
  - if the `expiration_height` is already past for the Bitcoin chain tip seen by the client:
    - MUST reject the policy message

#### Rationale

The credentials to service ratio precises for an amount of scarce assets provided the service units offered by the provider.
