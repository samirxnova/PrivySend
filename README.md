# PrivySend

**PrivySend** is a decentralized, privacy-first file sharing application built on **Filecoin** with the **Synapse SDK**. It empowers users to securely upload, encrypt, and share files using client-side encryption, ensuring that only intended recipients can access them. PrivySend redefines file sharing by combining the ease of web-based transfers with the security of end-to-end encryption and the permanence of decentralized storage.

---

## ✨ Features

* **Client-Side Encryption** – Files are encrypted before leaving the user’s device using AES-256.
* **Decentralized Storage** – Files are uploaded and pinned to Filecoin via the **Synapse SDK**.
* **Zero-Knowledge Sharing** – Encrypted file links/tokens are generated that only recipients can use.
* **Ticket-Based Access** – Granular control using Synapse’s allowance and ticket system.
* **Privacy by Default** – Metadata is minimized; file contents never pass through PrivySend servers unencrypted.
* **Streamlined UX** – Simple web-based flow: upload → encrypt → share → redeem.

---

## 🔗 Architecture Overview

1. **Upload Flow**

   * User selects a file in the web app.
   * File is encrypted locally using AES-256.
   * Encrypted file is chunked and uploaded to Filecoin through Synapse SDK.
   * A Filecoin content ID (CID) is returned.
   * PrivySend generates a secure share token (CID + decryption key).

2. **Download Flow**

   * Recipient receives the tokenized link (e.g. `privysend.xyz/redeem/{ticket}`).
   * PrivySend fetches the encrypted file via Synapse SDK using the CID.
   * File is decrypted locally using the embedded key.
   * Recipient can download or view the decrypted file.

3. **Access Control**

   * Synapse SDK’s allowance + ticket system is used to manage bandwidth and permissions.
   * Links can be one-time-use, multi-use, or time-limited.

---

## ⚙️ Tech Stack

* **Frontend**: Next.js (React, TypeScript, TailwindCSS)
* **Backend API**: Node.js (Express or Next.js API routes)
* **Storage**: Filecoin via Synapse SDK
* **Encryption**: AES-256 (Web Crypto API)
* **Authentication (optional)**: Ethereum wallet login for advanced users

---

## 🚀 Setup Instructions

### 1. Clone & Install

```bash
git clone https://github.com/your-org/privysend.git
cd privysend
npm install
```

### 2. Configure Environment

Create `.env.local` with the following values:

```env
# Filecoin / Synapse API credentials
SYNAPSE_API_KEY=your_synapse_api_key
SYNAPSE_SECRET=your_synapse_secret
```

### 3. Run Locally

```bash
npm run dev
```

Visit [http://localhost:3000](http://localhost:3000).

---

## 🛠️ Synapse SDK Integration

PrivySend integrates Filecoin’s **Synapse SDK** to handle decentralized file uploads, bandwidth management, and ticket redemption.

### Upload Example

```ts
import { Synapse } from "@filecoin/synapse";
import crypto from "crypto";

const synapse = new Synapse({ apiKey: process.env.SYNAPSE_API_KEY });

// Encrypt file
async function encryptFile(fileBuffer: Buffer, key: Buffer) {
  const iv = crypto.randomBytes(16);
  const cipher = crypto.createCipheriv("aes-256-gcm", key, iv);
  const encrypted = Buffer.concat([cipher.update(fileBuffer), cipher.final()]);
  return { encrypted, iv, authTag: cipher.getAuthTag() };
}

// Upload file
async function uploadFile(file: File) {
  const buffer = Buffer.from(await file.arrayBuffer());
  const key = crypto.randomBytes(32); // AES-256 key
  const { encrypted, iv, authTag } = await encryptFile(buffer, key);

  const result = await synapse.upload(encrypted, { metadata: { iv, authTag } });
  return { cid: result.cid, key: key.toString("base64") };
}
```

### Generate Share Link

```ts
function generateShareLink(cid: string, key: string) {
  return `https://privysend.xyz/redeem?cid=${cid}&key=${key}`;
}
```

### Redeem & Decrypt

```ts
async function downloadAndDecrypt(cid: string, key: string, iv: Buffer, authTag: Buffer) {
  const encrypted = await synapse.download(cid);
  const decipher = crypto.createDecipheriv("aes-256-gcm", Buffer.from(key, "base64"), iv);
  decipher.setAuthTag(authTag);
  const decrypted = Buffer.concat([decipher.update(encrypted), decipher.final()]);
  return decrypted;
}
```

---

## 🔐 Security Considerations

* **Encryption First**: Files are encrypted before upload; Filecoin never stores plaintext.
* **Key Separation**: Decryption keys are never stored with CIDs; shared links bundle them securely.
* **Ephemeral Tickets**: Synapse ticketing ensures file links expire or are one-time use.
* **Transport Security**: HTTPS enforced for all PrivySend APIs.
* **Optional Wallet Login**: Users can sign uploads with Ethereum wallets to prove file ownership.

---

## 🧩 Roadmap

* [ ] Drag & Drop multi-file uploads
* [ ] Advanced ticketing (time-based, usage count)
* [ ] Mobile-friendly PWA version
* [ ] ZK-proofs for access verification
* [ ] DAO-managed storage pools for subsidized uploads
