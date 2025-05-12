# On‑Chain ECDSA Public Key Recovery

#### This project enables any EVM-compatible dApp to designate an on-chain address whose ECDSA public key is used for encrypting sensitive data off-chain. By recovering the user’s uncompressed public key on-chain, you can encrypt payloads so that **only** the holder of the matching private key can decrypt them. Ideal for secure storage of private notes, confidential credentials, or setting up peer-to-peer encrypted messaging, this pattern lets you control exactly which address is authorized to read the encrypted data.
---

## Repository Structure

```
├── README.md               ← this file
├── index.html              ← static page (HTML + JS)
└── contracts/
    └── PubKeyRecovery.sol  ← flattened contract with SECP256K1 library
```

---

## Prerequisites

- A local HTTP server (e.g. `npx serve .`) to serve `index.html` at `http://localhost:PORT`
- A Solidity compiler (0.8.17+) or Remix IDE for contract deployment
- **ethers.js** is loaded via CDN in `index.html`

---

## 1. Deploy the Solidity Contract

1. Open `contracts/PubKeyRecovery.sol` in Remix or your IDE.
2. Compile with Solidity version `0.8.17` or newer.
3. Deploy to **any** EVM network (Mainnet, Testnet, private chain).
4. Note the deployed contract address (e.g. `0xYourDeployedAddress`).

---

## 2. Configure the Front‑End Page

1. Open `index.html`.
2. Replace the placeholder contract address:
   ```js
   const CONTRACT_ADDRESS = '0xYourDeployedAddress';
   ```
3. Customize **chain parameters** if you wish to suggest a specific network:
   ```js
   // Optional: edit to reflect your preferred network
   const NETWORK_PARAMS = {
     chainId: '0x1',                     // e.g. Ethereum Mainnet = '0x1'
     chainName: 'Ethereum Mainnet',      // Friendly name
     rpcUrls: ['https://mainnet.infura.io/v3/YOUR_KEY'],
     nativeCurrency: { name: 'ETH', symbol: 'ETH', decimals: 18 },
     blockExplorerUrls: ['https://etherscan.io']
   };
   ```
4. Save your changes.

---

## 3. Serve & Use the Page

1. Start a local HTTP server:
   ```bash
   npx serve .
   ```
2. Open `http://localhost:5000/index.html` in your browser.
3. **Connect Wallet**: Click **Connect Wallet** and approve in your injected wallet.
4. **Recover PubKey**: Click **Recover PubKey On‑Chain**, sign the challenge message, and view the raw uncompressed public key (64‑byte X‖Y hex) from the contract.

> **Note:** Browsers treat `http://localhost` as a secure context, so Web3 APIs work without HTTPS.

---

## How It Works

1. **Digest Generation**: `ethers.hashMessage(message)` applies EIP‑191 prefix to create a 32‑byte digest.
2. **Signature**: Wallet signs via `signer.signMessage(message)`.
3. **Split**: `ethers.Signature.from(signature)` extracts `(r, s, v)`.
4. **On‑Chain Recovery**: `recoverPubKey(digest, v, r, s)` calls into `SECP256K1.recover(...)` to perform EC math and return `(x, y)`.
5. **Display**: Front end concatenates X||Y into hex and displays it.

---

## Use Cases

### Encrypted Data Storage

- **Off‑Chain Encryption**: Use the recovered public key to encrypt sensitive data before storing on IPFS or other decentralized storage.
- **Access Control**: Only the matching private key holder can decrypt, ensuring privacy for public storage.

### P2P Encrypted Messaging

- **Direct Messaging**: Exchange public keys on‑chain or off‑chain; use them for end‑to‑end encryption over any transport (IPFS PubSub, WebSockets).
- **Identity Verification**: Requiring a signed challenge binds the public key to the signer’s address, preventing spoofing.

---

## Customization

- **Contract Address**: Edit `CONTRACT_ADDRESS` in `index.html` to point at your deployed contract.
- **Network Parameters**: Adjust `NETWORK_PARAMS` to suggest any EVM chain for network switching in the UI (optional).
- **Challenge Message**: Modify the `message` string in `index.html` to suit your application context.

---

## Security & Notes

- **Gas**: On‑chain EC recovery is gas‑heavy; best used for key retrieval demos or one‑time operations.
- **Alternative**: To recover only the signer’s address, use Solidity’s built‑in `ecrecover` for lower gas costs.
- **Localhost**: `http://localhost` is treated as secure—no HTTPS setup needed locally.

---

## License

MIT © Open Source
