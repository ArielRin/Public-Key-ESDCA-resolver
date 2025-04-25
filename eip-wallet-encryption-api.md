---
eip: <to be assigned>
title: Wallet-Based Asymmetric Encryption API Standard
description: A standard JSON-RPC interface for Ethereum wallets to expose public encryption keys and decrypt data using account keypairs.
author: Cheyne Hayworth (@cheynespc)
discussions-to: https://ethereum-magicians.org/t/eip-wallet-encryption-api-standard-for-eth-getencryptionpublickey-and-eth-decrypt/0000
status: Draft
type: Interface
created: 2025-04-25
requires: 191, 712
---

## Simple Summary

This proposal defines a standard JSON-RPC API for Ethereum-compatible wallets to support asymmetric encryption. It introduces `eth_getEncryptionPublicKey` and `eth_decrypt` methods that allow applications to encrypt messages to an Ethereum account and have them decrypted by the wallet, using the same keypair used for transaction signing.

## Abstract

Ethereum wallets today provide APIs for transaction signing and authentication. However, there is no standard mechanism for supporting public-key encryption using the existing SECP256k1 keypair associated with an account. MetaMask implements a non-standard approach to this, but broader adoption is blocked by the lack of a unified interface.

This proposal defines a wallet API standard that allows clients to:
- Retrieve the encryption-compatible public key of an account
- Request the wallet to decrypt data encrypted to that key

This enables end-to-end encrypted communication, decentralized identity systems, and secure off-chain storage integrated directly with the user's Ethereum identity.

## Motivation

There is a growing demand for decentralized privacy-preserving features in Web3:
- Secure messaging between Ethereum accounts
- Confidential note or credential sharing
- Encrypted identity metadata or KYC payloads

The lack of a standard encryption interface has forced developers to create app-side key management systems, breaking compatibility with wallets and increasing complexity. This EIP provides a universal, secure, and wallet-native encryption interface.

By extending existing signing keypairs to support encryption via standard ECIES techniques, wallets can offer consistent privacy primitives without exposing the private key or requiring external infrastructure.

## Specification

### `eth_getEncryptionPublicKey`

Returns the public key used for encryption, derived from the SECP256k1 keypair of the given account.

#### Parameters

- `address`: The Ethereum address (hex string) whose encryption key is being requested.

#### Returns

- A 66-byte hex-encoded compressed public key in SECP256k1 format (prefix 0x02 or 0x03).

#### Example

```json
{
  "method": "eth_getEncryptionPublicKey",
  "params": ["0x18ff7f454b6a3233113f51030384f49054dd27bf"]
}
```

```json
{
  "result": "0x03b28c...f2c"
}
```

### `eth_decrypt`

Decrypts an encrypted payload using the account's private key.

#### Parameters

- `encryptedData`: A hex-encoded ECIES-encrypted message, including:
  - Ephemeral public key
  - Nonce
  - Ciphertext
- `address`: The address of the wallet account to decrypt with.

#### Returns

- A UTF-8 decoded plaintext string.

#### Example

```json
{
  "method": "eth_decrypt",
  "params": ["0x04abcdef...", "0x18ff7f454b6a3233113f51030384f49054dd27bf"]
}
```

```json
{
  "result": "This is a confidential message."
}
```

## Rationale

This specification uses well-established cryptographic primitives:

- ECIES over SECP256k1 (same curve used for Ethereum signing)
- AES-GCM or XSalsa20-Poly1305 for symmetric encryption
- The encrypted payload includes nonce, ephemeral public key, and ciphertext

This allows for safe, forward-secure encrypted communication with ephemeral key exchange, all without exposing or reusing the wallet’s signing keys in an unsafe way.

## Security Considerations

### Consent

Wallets must prompt users before performing a decryption operation. Decryption should not occur silently or in the background.

### Key Usage

The same key used for signing transactions is reused for decryption. While mathematically safe under ECIES, wallets must isolate these operations logically and avoid overexposing key usage to prevent potential side-channel attacks.

### Misuse and Abuse

Wallets should:
- Rate-limit decryption requests
- Block malformed payloads
- Log suspicious activity (e.g., frequent decryption requests without user interaction)

### Signature-Based Verification

To bind a public key to a user’s identity off-chain, apps may request a signed message including the public key. This is out of scope of the encryption API itself but can be used for identity proofing.

## Backwards Compatibility

There is no existing standard for encryption in Ethereum wallets. This EIP defines new RPC methods which can be adopted independently by each wallet.

Wallets that do not implement this EIP should return standard JSON-RPC error code -32601 (`Method not found`).

## Reference Implementation

MetaMask:
- https://docs.metamask.io/guide/rpc-api.html#eth_getencryptionpublickey
- https://github.com/MetaMask/eth-sig-util

Open-source ECIES libraries:
- https://github.com/MetaMask/eth-sig-util
- https://github.com/paulmillr/noble-secp256k1

## Copyright

Copyright and related rights waived via CC0.
