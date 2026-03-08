# Understanding Midnight Network for EVM Developers




---

## Introduction: A Different Kind of Blockchain

If you've been building on Ethereum or any EVM-compatible chain, you're used to a world where everything is transparent. Every transaction, every balance, every contract interaction it's all visible on the blockchain for everyone to see. That's the design philosophy of EVM chains: transparency equals security.

**Midnight Network flips this on its head.** It's a blockchain built with privacy as the default, not transparency. This isn't just about hiding transaction amounts it's a fundamentally different architecture that uses zero-knowledge proofs to let you prove things are correct without revealing what those things are.

This guide will walk you through the major mental shifts you need to make when coming from EVM development.

---

## The Big Picture: What Makes Midnight Different?

### 1. Privacy-First Architecture

**In EVM chains:** When you deploy a token contract, everyone can see every balance, every transfer, every approval. Tools like Etherscan show your complete transaction history. This transparency is a feature it's how we audit contracts and verify behavior.

**In Midnight:** You can have contracts where balances are private, transfers are private, but the contract logic is still verified and correct. The blockchain knows the rules are being followed (because of zero-knowledge proofs), but it doesn't know the specific amounts or participants.

**Why this matters:** This model targets applications that need verifiability plus confidentiality, such as privacy-sensitive financial, identity, and compliance flows.

### 2. Zero-Knowledge Proofs Instead of Transparent Execution

**In EVM chains:** Your Solidity code compiles to bytecode. Every validator runs that bytecode and sees the exact same state changes. If you transfer 100 tokens, every node sees you had 200, now you have 100, and Bob has 100 more. Everyone re-executes everything to verify it.

**In Midnight:** Your contract functions compile to "circuits" mathematical representations that can be proven with zero-knowledge proofs. When you transfer tokens, you generate a proof that says "I followed the rules" without revealing what those inputs and outputs were. Validators check the proof (which is fast) rather than re-executing everything (which is slow and exposes data).

**Why this matters:** This is a completely different execution model. You're not writing functions that run on a virtual machine you're writing functions that get turned into mathematical circuits. The tooling, debugging, and mental model are all different.

### 3. Two Kinds of State: Public and Private

**In EVM chains:** All contract state is public. If you store a mapping of balances, anyone can query any balance. Private variables in Solidity just mean "not automatically exposed via a getter" they're still visible on-chain if you know where to look.

**In Midnight:** Contracts have public ledger state on-chain, while private state remains local to each DApp/user context. Zero-knowledge proofs let you prove correctness involving private inputs without revealing those inputs.

**Why this matters:** You need to explicitly decide what goes into public ledger state and what stays in local private state.

---

## Token Models: EVM vs Midnight

### EVM Token Model

In EVM chains, tokens are primarily smart contract-based. When you create a token, you deploy a contract (like ERC-20 or ERC-721) that maintains an account-based ledger of balances. Every token is:

- **Contract-managed:** All logic lives in smart contract code
- **Fully transparent:** All balances and transfers are publicly visible
- **Account-based:** State is stored as mappings of addresses to balances
- **Uniform approach:** Whether it's ETH transfers or ERC-20 tokens, everything uses the same transparent model

### Midnight Token Model

Midnight offers **two fundamental token categories**, each with privacy options:

**Ledger Tokens** live directly on Midnight's blockchain, managed by the core protocol itself. These exist as UTXOs and benefit from maximum efficiency and security. They can be:
- **Shielded Ledger Tokens:** Private, UTXO-based, maximum efficiency (e.g., private payments, confidential transfers)
- **Unshielded Ledger Tokens:** Transparent, UTXO-based, high performance (e.g., NIGHT tokens, public treasuries)

**Contract Tokens** are created and managed by Compact smart contracts, similar to ERC-20 but with privacy options. They use account-based patterns within contracts and can be:
- **Shielded Contract Tokens:** Programmable privacy, custom logic, account-based (e.g., private securities, confidential rewards)
- **Unshielded Contract Tokens:** Full programmability, ERC-20 style (e.g., governance tokens, public DeFi protocols)

**Why this matters:** In EVM, you're limited to transparent contract tokens. In Midnight, you choose between ledger tokens (for performance and native privacy) or contract tokens (for complex programmability), and whether to make them shielded or unshielded. This flexibility lets you optimize for your specific use case.

---

## The Language: From Solidity to Compact

### Solidity: What You Know

Solidity is a custom language inspired by JavaScript/Java. It has special keywords like `payable`, `view`, `pure`, and concepts like `msg.sender`, `block.timestamp`, and modifier chains. It's designed to compile to EVM bytecode.

### Compact: The New World

Compact is Midnight's language, and it looks like TypeScript at first glance. But there are critical differences:

**Circuits Are Entry Points:** In Solidity, you write `function transfer(...)`. In Compact, you write `circuit transfer(...)` for entry points that users can call directly in transactions. Circuits are compiled into zero-knowledge proof circuits. This has implications circuits need to be mathematically representable, so there are constraints on what you can do inside them.

**Ledger State and Witnesses Are Separate Concepts:** In Solidity, you modify state directly. In Compact, circuits can update ledger fields, while witness functions are callbacks implemented in your DApp (outside Compact) for local/private inputs.

**Types are Stricter:** Solidity has one address type: `address`. Midnight has multiple address-like types because different kinds of entities exist on the network. You have `ContractAddress` (for smart contracts), `ZswapCoinPublicKey` (for user wallets), and `Either<ZswapCoinPublicKey, ContractAddress>` (for "this could be either a wallet or a contract"). You need to handle these explicitly.

---

## Wallets: From One Key Pair to Three Components

### EVM Wallets: Simple

On EVM chains, your wallet is just a key pair. You have a private key, it derives a public key, which hashes to an address. That's it. Your ETH balance and all token balances are associated with that one address. MetaMask manages this one key pair for you.

### Midnight Wallets: Complex but Powerful

Midnight wallets are actually **three separate wallets** derived from one seed phrase:

**1. Shielded Wallet:** This is your private wallet. When you hold assets here, nobody knows how much you have. When you send from here, nobody knows how much you sent (unless you tell them). This is your main wallet for privacy.

**2. Unshielded Wallet:** This is like a traditional EVM address it's public and transparent. Sometimes you want transparency (like receiving payment for goods), so this wallet exists for those cases. You can move funds between shielded and unshielded.

**3. Dust Wallet:** This is your "gas tank." In EVM chains, you hold ETH for gas. In Midnight, DUST is handled in a separate wallet component dedicated to fee/resource operations.

**Why this matters:** When you write a contract function that receives an address, you need to handle the possibility of both shielded addresses and unshielded addresses. When you interact with contracts, you might pay from your shielded wallet while gas comes from your dust wallet. This three-wallet system enables privacy while maintaining flexibility.

---

## Gas: DUST is Not Like ETH

### In EVM: ETH for Gas

Gas is simple on EVM chains: you hold ETH in your wallet. When you send a transaction, you specify a gas price and limit. The network deducts ETH to pay for execution. It's all automatic and seamless.

### In Midnight: DUST Network Resource

DUST works differently:

**It Comes From a Setup Flow:** Midnight docs describe generating/setting up DUST from eligible NIGHT/unshielded UTXOs before sending transactions.

**Separate Wallet Component:** DUST is managed through its own wallet component within your overall wallet. It handles fee payment during transaction balancing separately from shielded/unshielded operations.

**Why this matters:** DUST management is operationally different from ETH gas. You need to complete the DUST setup/generation step before submitting transactions.

---

## Deployment: Proof Servers and Providers

### EVM Deployment: RPC Node

Deploying on EVM chains is straightforward: you connect to an RPC node (Infura, Alchemy, your own node), compile your contract to bytecode, sign a transaction that creates the contract, and broadcast it. The network executes the bytecode in the constructor and stores the resulting contract.

### Midnight Deployment: More Infrastructure

Deploying on Midnight requires more setup:

**1. Proof Server:** You need a proof server running (typically via Docker) to generate zero-knowledge proofs for transactions. For privacy, docs recommend using a local proof server, or a remote one you control over an encrypted channel, because proof requests can include sensitive private-state data.

**2. Multiple Providers:** Instead of just an RPC provider, you set up several providers:
- **Private State Provider:** Manages your private state
- **Public Data Provider:** Queries public blockchain data
- **ZK Config Provider:** Provides zero-knowledge circuit configurations
- **Proof Provider:** Connects to your local proof server

**3. Wallet Initialization:** You initialize your three-wallet system with all these providers. This is more complex than just creating an ethers.js wallet.

**4. DUST Registration:** After wallet initialization, you must register DUST before you can deploy anything.

**5. Contract Deployment:** Finally, you deploy the contract with constructor parameters through the same proving-enabled workflow.

**Why this matters:** Deployment has more prerequisites. You can't just run `npx hardhat deploy`. You need Docker running a proof server, multiple provider configurations, and explicit DUST registration. It's more setup, but it enables the privacy features.

---

## Transactions: Sign and Broadcast vs Generate and Prove

### EVM Transactions: Sign and Broadcast

On EVM chains, transactions are simple:
1. Construct transaction data (function call, parameters)
2. Sign it with your private key
3. Broadcast to the network
4. Every validator re-executes your transaction
5. State changes are applied if execution succeeds

This takes a few seconds. The network does the work.

### Midnight Transactions: Generate Proof First

On Midnight, transactions are different:
1. Construct transaction data (circuit call, parameters)
2. **Your local machine generates a zero-knowledge proof** 
3. Sign the proof with your private key
4. Broadcast the proof to the network
5. Validators verify the proof (fast) instead of re-executing (slow)
6. State changes are applied if the proof is valid

**Step 2 is the game-changer.** Proof generation happens on your machine (or your proof server). The deployment guide notes this step may take around 30-60 seconds for that flow.

**Why this matters:** Transactions feel slower because proof generation happens before broadcasting. You need to wait for proof generation locally before the transaction even hits the network. This is the trade-off for privacy proof generation is the computational cost of keeping data private.

---

## Contract Design: New Mental Models

### State Management: Ledger Operations and Witness Inputs

In Solidity, you're used to mutating state:

You write things like `balance -= amount` and the state changes in place.

In Compact, circuits can update ledger state, and witness functions provide local/private inputs. Instead of blending everything together:

You keep on-chain verifiable logic in circuits and keep sensitive/local data access in witness implementations outside Compact.

**Why this matters:** You can't just port Solidity logic directly. You need to separate public ledger operations from local private-data access.

### Address Handling: Multiple Types

In Solidity, every recipient is an `address`. You don't care if it's a user or a contract it's all the same 20-byte value.

In Compact, recipients can be:
- A shielded wallet address (ZswapCoinPublicKey)
- An unshielded wallet address (different type)
- A contract address (ContractAddress)

You use `Either<ZswapCoinPublicKey, ContractAddress>` to say "this could be either type." You need to handle both cases in your logic.

**Why this matters:** Type checking is stricter. You can't just pass any address anywhere. You need to think about what kinds of addresses your function accepts and handle each type appropriately.

### Privacy Boundaries: What to Reveal

This is entirely new: you decide what's public and what's private.

In an EVM token, balances are always public. In a Midnight token, you could make balances private (only the owner knows their balance) while keeping total supply public (everyone can verify total supply without seeing individual balances).

You might have a voting contract where votes are private but the final tally is public.

**Why this matters:** Privacy is a design decision you make consciously. You need to think about information flow: what needs to be public for verification, what can stay private for confidentiality, and how zero-knowledge proofs bridge the gap.

---
