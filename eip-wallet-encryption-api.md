---
layout: default
title: Wallet-Based Asymmetric Encryption API Standard
description: A standard JSON-RPC interface for Ethereum wallets to expose public encryption keys and decrypt data using account keypairs.
---

# Wallet-Based Asymmetric Encryption API Standard

**EIP**: <to be assigned>  
**Title**: Wallet-Based Asymmetric Encryption API Standard  
**Author**: Cheyne Hayworth ([@cheynespc](https://github.com/CheyneWeb3))  
**Discussions-To**: [https://ethereum-magicians.org/t/eip-wallet-encryption-api-standard-for-eth-getencryptionpublickey-and-eth-decrypt/0000](https://ethereum-magicians.org/t/eip-wallet-encryption-api-standard-for-eth-getencryptionpublickey-and-eth-decrypt/0000)  
**Status**: Draft  
**Type**: Interface  
**Created**: 2025-05-12  
**Requires**: 191, 712  

---

## Simple Summary

This proposal defines a standard JSON-RPC API for Ethereum-compatible wallets to support asymmetric encryption. It introduces `eth_getEncryptionPublicKey` and `eth_decrypt` methods that allow applications to encrypt messages to an Ethereum account and have them decrypted by the wallet, using the same keypair used for transaction signing.

---

## Abstract

Ethereum wallets today provide APIs for transaction signing and authentication. However, there is no standard mechanism for supporting public-key encryption using the existing SECP256k1 keypair associated with an account. MetaMask implements a non-standard approach to this, but broader adoption is blocked by the lack of a unified interface.

This proposal defines a wallet API standard that allows clients to:

- Retrieve the encryption-compatible public key of an account  
- Request the wallet to decrypt data encrypted to that key

This enables end-to-end encrypted communication, decentralized identity systems, and secure off-chain storage integrated directly with the user's Ethereum identity.

---

## Motivation

There is a growing demand for decentralized privacy-preserving features in Web3:

- Secure messaging between Ethereum accounts  
- Confidential note or credential sharing  
- Encrypted identity metadata or KYC payloads

The lack of a standard encryption interface has forced developers to create app-side key management systems, breaking compatibility with wallets and increasing complexity. This EIP provides a universal, secure, and wallet-native encryption interface.

By extending existing signing keypairs to support encryption via standard ECIES techniques, wallets can offer consistent privacy primitives without exposing the private key or requiring external infrastructure.

---

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
