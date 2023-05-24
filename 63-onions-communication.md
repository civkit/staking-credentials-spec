# Staking Credentials - Onions Communications

This document specifies the usage of BOLT 4 onion messages and HTLC onions inside the Staking Credentials framework.

The message data format, validation algorithms and implementations considerations are laid out.

This draft is version 0.0.1 of the Onions Communications.

## Credentials Issuance - Communication

### Sending `request_credentials_authentication`

The Requester node finds an onion route to the Issuer node.

Then the Requester node finds a route to an introduction node and build a blinded route from the introduction node to its own node.

Two new `tlv_record` are added in BOLT's 4 `payload`:
    1. type: YYY (`request_credentials_authentication`)
    2. data:
	* [`...*byte:data`]
    3. type: XXX (`reply_paths`)
    4. data:
	* [`...*blinded_path:paths`]

#### Requirements

The origin node:
	- MUST set the final hop to the Issuer's `node_id`
	- MUST set `request_credentials_authentication` record in the final hop's `payload`
	- MUST set `reply_paths` record in the final hop

The receiving node:
	- MUST reject the `payload` if `reply_paths` does not respect requirements laid out in BOLT4

### Receiving `reply_credentials_authentication`

One new `tlv_record` is added in BOLT's 4 `payload`:
    1. type: ZZZ (`reply_credentials_authentication`)
    2. data:
	* [`...*byte:data`]

#### Requirements

The origin node:
	- MUST set the final hop to the Requester's `node_id`
	- MUST set `reply_credentials_authentication` record in the final hop's `payload`
	- MUST set `reply_paths` record in the final hop

## Redemption

### Sending `redeem_credentials`

The Client must forward credentials along the payment path before to route the HTLC.

One new `tlv_record` is added in BOLT's 4 `payload`:
    1. type: AAA (`redeem_credentials`)
    2. data:
	* [`...*byte:data`]

#### Requirements

The HTLC forwarding node:
	- MUST send a `redeem_credentials` to all hop along the payment path
	- MUST send a unique `payment_secret` in each hop's `payload`

The origin node:
	- MUST set `redeem_credentials record in the final hop's `payload`
	- MUST set the `request_identifier` to the `payment_secret` value

The receiving node:
	- MUST reject the onion's `payload` if the credentials are not valid
