# BitFlow: Bitcoin Payment Channels on Stacks

## Overview

**BitFlow** is a Bitcoin Layer 2 payment channel framework built on the **Stacks** blockchain. Inspired by the **Lightning Network**, it brings scalable, low-latency, and low-cost micropayments to Bitcoin using smart contract programmability, trustless channel management, and Lightning-compatible message structures.

By leveraging Stacks’ smart contracts and Bitcoin's native cryptographic primitives (ECDSA, nLockTime analogs), BitFlow enables high-throughput off-chain payments that settle on-chain with strong security guarantees.

---

## Key Features

* ⚡ **Lightning Network Compatibility**
  Seamless integration with Lightning-style tools and wallets using compatible message and signature formats.

* 🔒 **Bitcoin Security Model**
  Uses ECDSA signature validation and time-locked dispute resolution similar to Bitcoin Script primitives.

* 🕒 **Sub-Second Settlement Finality**
  Enables instant bidirectional payments without waiting for on-chain confirmations.

* 🧾 **Trustless Channel Management**
  Channels operate without intermediaries, backed by mathematically enforced conditions.

* ⚖️ **Flexible Closure Options**
  Supports cooperative (dual-signature) and unilateral (dispute-based) channel termination flows.

* 🛠️ **Programmable Settlement Logic**
  Built on Clarity, allowing composability with other DeFi and dApp logic on Stacks.

---

## System Architecture

```
+-------------------+
|  Bitcoin Layer 1  | ← BTC Security Layer
+-------------------+
        ▲
        |
        | BTC Signatures / UTXO Concepts
        ▼
+-------------------+
|     Stacks L1     | ← Smart Contract Layer
|  (Clarity-based)  |
+-------------------+
        ▲
        | STX Escrow / State Channels
        ▼
+--------------------------+
|     BitFlow Channels     |
|                          |
| - Channel Registry       |
| - Dispute Mechanism      |
| - Signature Verification |
+--------------------------+
        ▲
        |
        ▼
+--------------------------+
|   Lightning-Compatible   |
|       Clients / UI       |
+--------------------------+
```

---

## Contract Architecture

The BitFlow system is implemented as a single smart contract written in **Clarity**, operating entirely on the Stacks blockchain. The key modules include:

### 1. **Channel Lifecycle Management**

* `create-channel` – Opens a new payment channel with locked STX
* `fund-channel` – Adds additional funds to an existing open channel
* `close-channel-cooperative` – Trustless cooperative settlement using dual signatures
* `initiate-unilateral-close` – Starts a dispute-based closure with a time-locked challenge window
* `resolve-unilateral-close` – Finalizes unilateral closure if the dispute period has passed

### 2. **State Tracking**

A `payment-channels` map maintains all relevant information about each channel:

* Unique `channel-id` (32-byte identifier)
* `participant-a` and `participant-b` principals
* Total funds and per-participant balances
* Channel `is-open` status
* `dispute-deadline` for arbitration windows
* `nonce` for state transition ordering (Lightning-style updates)

### 3. **Validation Layer**

Ensures strict type and logic safety:

* Signature length and structure
* Channel ID formatting
* Deposit constraints
* Authorization checks

### 4. **Cryptographic Verification**

Implements Bitcoin-style signature validation (mocked for devnet, pluggable with `secp256k1` verification libraries in production).

### 5. **Emergency Governance**

* `emergency-withdraw` – Admin function to evacuate funds in critical situations

---

## Data Flow: Cooperative vs. Unilateral Closure

### ⚡ Cooperative Closure

```
Participants A & B → Sign final state
         |
         ▼
close-channel-cooperative()
         |
         ▼
Funds distributed → Channel marked closed
```

### ⚠️ Unilateral Closure

```
Participant A → Signs proposed state
         |
         ▼
initiate-unilateral-close()
         |
         ▼
[ Dispute Window Opens ] (144 blocks)
         |
         ▼
If Unchallenged → resolve-unilateral-close()
         |
         ▼
Funds distributed → Channel marked closed
```

---

## Installation & Deployment

### Requirements

* [Clarity Language](https://docs.stacks.co/docs/write-smart-contracts/clarity-overview)
* Stacks blockchain devnet/testnet setup (e.g., `stacks-blockchain`, `clarinet`)

### Deploy with Clarinet

```bash
clarinet check      # Typecheck Clarity code
clarinet test       # Run unit tests (if implemented)
clarinet deploy     # Deploy contract to devnet/testnet
```

> ⚠️ **Note**: The current `verify-signature` function is a placeholder. Replace with a production-ready ECDSA verification module for mainnet usage.

---

## Usage

### Create Channel

```clojure
(create-channel channel-id participant-b initial-deposit)
```

### Fund Existing Channel

```clojure
(fund-channel channel-id participant-b additional-funds)
```

### Cooperative Close

```clojure
(close-channel-cooperative channel-id participant-b balance-a balance-b sig-a sig-b)
```

### Unilateral Close

```clojure
(initiate-unilateral-close channel-id participant-b balance-a balance-b signature)
(resolve-unilateral-close channel-id participant-b)
```

### Retrieve Channel Info

```clojure
(get-channel-info channel-id participant-a participant-b)
```

---

## Security Considerations

* **Replay Protection**: Nonce tracking can be extended to ensure state progression and prevent stale updates.
* **Dispute Handling**: The 144-block (\~24 hours) lock ensures enough time for challenge submission.
* **Signature Integrity**: Full integration of `secp256k1` verification is recommended for production-level signature validation.

---

## License

MIT License © 2025 BitFlow Contributors

---

## Authors

Developed by a senior Clarity engineer as part of the **BitFlow** project, focused on bringing scalable Bitcoin-native payments to Web3.
