# Credential Protocol — Smart Contracts

A comprehensive suite of Rust-based Soroban smart contracts for the Credential Protocol decentralized professional credential verification platform. This repository contains three production-grade smart contracts that manage credential issuance, Soulbound Token (SBT) minting, and zero-knowledge proof verification on the Stellar blockchain, enabling trustless, privacy-preserving credential auditing across regulated professions.

## 📋 Table of Contents

- [Overview](#overview)
- [Smart Contracts](#smart-contracts)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Development](#development)
- [Building & Testing](#building--testing)
- [Deployment](#deployment)
- [Contract APIs](#contract-apis)
- [Data Structures](#data-structures)
- [Error Handling](#error-handling)
- [Security Considerations](#security-considerations)
- [Performance](#performance)
- [Benchmarking](#benchmarking)
- [Upgrading](#upgrading)
- [Contributing](#contributing)
- [License](#license)

## 🎯 Overview

The Credential Protocol Smart Contracts layer provides the foundational on-chain logic for:

1. **Credential Issuance & Management**: Issue, revoke, and query professional credentials
2. **Soulbound Token (SBT) Minting**: Create non-transferable tokens tied to credential holders
3. **Quorum Slices**: Define trust networks using Federated Byzantine Agreement (FBA) consensus
4. **Attestation Workflows**: Multi-party signature collection and verification
5. **Credential Verification**: Cross-check credentials and prevent fraud
6. **Zero-Knowledge Proofs**: Conditional verification (stub implementation)

This split repository contains only the smart contract layer, separated from backend API and frontend applications for independent auditing, upgrading, and optimization. All contracts are deployed to the Stellar testnet/mainnet blockchain and are permanently immutable once deployed.

## 📦 Smart Contracts

### 1. CredentialProtocol Contract

**Purpose**: Core credential lifecycle management

**Key Functions**:
- `issue_credential(issuer, subject, credential_type, metadata_hash)` - Issue new credential
- `revoke_credential(credential_id)` - Revoke issued credential
- `get_credential(credential_id)` - Retrieve credential details
- `get_credentials_by_subject(subject)` - Find all credentials for a subject
- `is_revoked(credential_id)` - Check revocation status
- `emit_credential_issued_event(credential_id)` - Event emission on creation
- `emit_revoke_credential_event(credential_id)` - Event emission on revocation

**Data**:
- Credential storage with metadata hashing
- Revocation registry
- Credential type definitions
- Event logs

**Key Features**:
- Non-revocable credential creation
- Immutable credential history
- Event emission for off-chain indexing
- Metadata validation

### 2. SBT Registry Contract

**Purpose**: Soulbound Token minting and management

**Key Functions**:
- `mint(owner, credential_id)` - Mint SBT for credential
- `burn(owner, credential_id)` - Burn revoked SBT
- `get_balance(owner)` - Query token balance
- `get_tokens_by_owner(owner)` - List all SBTs owned
- `is_minted(owner, credential_id)` - Check mint status
- `transfer_fail()` - Prevent SBT transfers (enforces non-transferability)

**Data**:
- Token ownership mapping
- Mint/burn history
- Token metadata

**Key Features**:
- Non-transferable tokens
- One-to-one credential-to-SBT mapping
- Burn functionality for revocation
- Persistent storage with TTL extension

### 3. ZK Verifier Contract (Stub)

**Purpose**: Zero-knowledge proof verification (non-functional stub)

**Key Functions**:
- `verify_claim(credential_id, claim_type, proof)` - Verify conditional claims (STUB)
- `generate_proof_request(credential_id, claim_type)` - Generate proof challenge

**Warning**: 
> ⚠️ **DO NOT USE IN PRODUCTION**
> This contract accepts ANY non-empty byte string as valid proof.
> No cryptographic verification is performed.
> Provides NO privacy guarantees.
> This is a placeholder for Groth16/PLONK implementation in v1.1

## 🏗️ Architecture

### Contract Interaction Flow

```
Frontend Application
    ↓
API Server (Contract Calls)
    ↓
Soroban Host (Contract Execution)
    ↓
┌─────────────────────────────────────────────────────────┐
│  CredentialProtocol Contract                            │
│  • Issue, revoke, query credentials                     │
│  • Emit events for indexing                             │
│  • Validate credential metadata                         │
└──────────────┬──────────────────────────────────────────┘
               │
               ├─→ SBT Registry Contract
               │   • Mint/burn tokens
               │   • Track ownership
               │   • Prevent transfers
               │
               ├─→ ZK Verifier Contract (Stub)
               │   • Proof verification placeholder
               │   • Admin-gated
               │
               └─→ Storage Layer
                   • Instance storage (contracts)
                   • Persistent storage (SBTs)
                   • Event logs
```

### Data Flow

```
Issue Credential:
1. API calls issue_credential on CredentialProtocol
2. Credential stored with metadata hash
3. Event emitted for indexing
4. Return credential ID

Mint SBT:
5. API calls mint on SBT Registry with credential ID
6. Verify credential exists and not revoked
7. Check (owner, credential_id) uniqueness
8. Mint SBT to owner
9. Emit Mint event

Verify Credential:
10. Query credential from CredentialProtocol
11. Check revocation status
12. Validate metadata against stored hash
13. Return credential data to user
```

### Storage Architecture

```
CredentialProtocol:
├── DataKey::Credentials
│   └── Map<credential_id> → Credential
├── DataKey::RevokedCredentials
│   └── Set<credential_id>
└── DataKey::CredentialCounter

SBT Registry:
├── DataKey::Balances
│   └── Map<owner> → balance
├── DataKey::TokenOwners
│   └── Map<owner> → Vec<credential_id>
└── DataKey::CredentialMinted
    └── Map<(owner, credential_id)> → bool
```

## 🔧 Prerequisites

### System Requirements

- **Rust**: 1.70 or higher
- **Soroban CLI**: Latest version
- **Stellar CLI**: Latest version
- **Git**: 2.x or higher
- **Docker** (optional): For containerized development

### Development Environment

```bash
# Install Rust
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Install Soroban CLI
cargo install soroban-cli --locked

# Install Stellar CLI
cargo install stellar-cli

# Verify installations
rustup --version
soroban --version
stellar --version
```

## 📥 Installation

### 1. Clone Repository

```bash
git clone https://github.com/credential-labs/credential-protocol-contracts.git
cd credential-protocol-contracts
```

### 2. Install Dependencies

```bash
# Download Rust dependencies
cargo fetch

# Build dependencies
cargo build --release --target wasm32-unknown-unknown
```

### 3. Environment Setup

Create `.env` from template:

```bash
cp .env.example .env
```

Configure environment:

```env
# Network Configuration
STELLAR_NETWORK=testnet
STELLAR_RPC_URL=https://soroban-testnet.stellar.org
STELLAR_NETWORK_PASSPHRASE="Test SDF Network ; September 2015"

# Account Configuration
STELLAR_ACCOUNT_SECRET=SBXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
STELLAR_DEPLOYER_ACCOUNT=GBXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

# Contract Addresses (after deployment)
CONTRACT_CREDENTIAL_PROTOCOL=CBXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
CONTRACT_SBT_REGISTRY=CBXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
CONTRACT_ZK_VERIFIER=CBXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

# Build Configuration
SOROBAN_SPEC_GEN=false
OPTIMIZE=true
PROFILE=release
```

### 4. Project Structure

```
credential-protocol-contracts/
├── contracts/                      # Smart contract implementations
│   ├── credential_protocol/        # Main credential contract
│   │   ├── src/
│   │   │   ├── lib.rs             # Contract entry points
│   │   │   ├── credential.rs      # Credential logic
│   │   │   ├── events.rs          # Event definitions
│   │   │   └── error.rs           # Error types
│   │   ├── Cargo.toml
│   │   └── spec.json              # Contract spec
│   │
│   ├── sbt_registry/              # SBT minting contract
│   │   ├── src/
│   │   │   ├── lib.rs
│   │   │   ├── sbt.rs
│   │   │   └── error.rs
│   │   └── Cargo.toml
│   │
│   └── zk_verifier/               # ZK proof verifier (stub)
│       ├── src/
│       │   ├── lib.rs
│       │   ├── verify.rs
│       │   └── error.rs
│       └── Cargo.toml
│
├── fuzz/                          # Fuzzing tests
│   ├── fuzz_targets/
│   │   ├── credential_fuzzer.rs
│   │   ├── sbt_fuzzer.rs
│   │   └── integration_fuzzer.rs
│   └── Cargo.toml
│
├── benches/                       # Performance benchmarks
│   ├── src/
│   │   └── lib.rs
│   ├── tests/
│   │   ├── benchmarks.rs
│   │   └── load_tests.rs
│   └── Cargo.toml
│
├── integration_tests/             # Integration tests
│   ├── credential_tests.rs
│   ├── sbt_tests.rs
│   ├── multi_contract_tests.rs
│   └── Cargo.toml
│
├── scripts/                       # Build/deploy scripts
│   ├── build.sh
│   ├── test.sh
│   ├── deploy.sh
│   └── verify.sh
│
├── Cargo.toml                     # Workspace configuration
├── Cargo.lock                     # Dependency lock file
├── environments.toml              # Network configs
├── dependencies.toml              # Dependency versions
└── README.md                      # This file
```

## 🚀 Development

### Build Smart Contracts

```bash
# Build all contracts
cargo build --release --target wasm32-unknown-unknown

# Build specific contract
cargo build --release --target wasm32-unknown-unknown -p credential_protocol

# Generate contract specifications
cargo build --release --target wasm32-unknown-unknown --features soroban-spec-gen
```

### Run Local Testing Network

```bash
# Start standalone network (local blockchain)
soroban network add --rpc-url http://localhost:11626 --network-passphrase "Standalone Network ; February 2021" standalone

# Deploy contracts to standalone
soroban contract deploy --network standalone --source deployer \
  --wasm target/wasm32-unknown-unknown/release/credential_protocol.wasm
```

## 🧪 Building & Testing

### Run All Tests

```bash
./scripts/test.sh
```

### Run Unit Tests

```bash
# All unit tests
cargo test --lib

# Specific contract tests
cargo test -p credential_protocol --lib

# With logging
RUST_LOG=debug cargo test --lib -- --nocapture
```

### Run Integration Tests

```bash
# All integration tests
cargo test --test '*'

# Specific integration tests
cargo test --test credential_tests
```

### Run Fuzzing

```bash
# Install fuzzer
cargo install cargo-fuzz

# Run credential fuzzer
cargo fuzz run credential_fuzzer

# Run SBT fuzzer
cargo fuzz run sbt_fuzzer
```

### Test Coverage

```bash
# Generate coverage report
cargo tarpaulin --out Html --output-dir coverage

# View coverage
open coverage/index.html
```

## 🚀 Deployment

### Deploy to Testnet

```bash
# 1. Create testnet account (if needed)
stellar keys generate deployer --network testnet
stellar friendbot testnet GBXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX

# 2. Fund account with testnet XLM (via friendbot)

# 3. Deploy contracts
./scripts/deploy.sh testnet

# 4. Verify deployment
./scripts/verify.sh testnet CONTRACT_ID
```

### Deploy to Mainnet

```bash
# 1. Verify on testnet first
./scripts/test.sh
cargo test --all

# 2. Get mainnet account ready
stellar keys generate deployer-mainnet --network mainnet

# 3. Verify mainnet account has sufficient XLM

# 4. Deploy contracts
./scripts/deploy.sh mainnet

# 5. Verify contracts
./scripts/verify.sh mainnet CONTRACT_ID
```

### Contract Upgrade Path

```bash
# 1. Build new contract version
cargo build --release --target wasm32-unknown-unknown

# 2. Compare against current version
diff target/wasm32-unknown-unknown/release/credential_protocol.wasm current_contract.wasm

# 3. Get new contract hash
sha256sum target/wasm32-unknown-unknown/release/credential_protocol.wasm

# 4. Deploy new version
./scripts/deploy.sh testnet --upgrade

# 5. Update address in environment
# Edit .env with new CONTRACT_CREDENTIAL_PROTOCOL
```

## 📡 Contract APIs

### CredentialProtocol Contract

```rust
// Issue new credential
pub fn issue_credential(
    env: Env,
    issuer: Address,
    subject: Address,
    credential_type: String,
    metadata_hash: BytesN<32>,
) -> u64 // credential_id

// Revoke credential
pub fn revoke_credential(env: Env, credential_id: u64) -> bool

// Get credential details
pub fn get_credential(env: Env, credential_id: u64) -> Credential

// Check if credential is revoked
pub fn is_revoked(env: Env, credential_id: u64) -> bool

// Get all credentials for subject
pub fn get_credentials_by_subject(env: Env, subject: Address) -> Vec<u64>
```

### SBT Registry Contract

```rust
// Mint SBT for credential
pub fn mint(
    env: Env,
    owner: Address,
    credential_id: u64,
) -> bool

// Burn SBT (on revocation)
pub fn burn(
    env: Env,
    owner: Address,
    credential_id: u64,
) -> bool

// Get token balance
pub fn get_balance(env: Env, owner: Address) -> u128

// Get all tokens owned
pub fn get_tokens_by_owner(env: Env, owner: Address) -> Vec<u64>

// Check if token is minted
pub fn is_minted(
    env: Env,
    owner: Address,
    credential_id: u64,
) -> bool
```

### ZK Verifier Contract

```rust
// Verify proof (STUB ONLY)
pub fn verify_claim(
    env: Env,
    credential_id: u64,
    claim_type: String,
    proof: Bytes,
) -> bool

// Generate proof request
pub fn generate_proof_request(
    env: Env,
    credential_id: u64,
    claim_type: String,
) -> ProofRequest
```

## 🔧 Data Structures

### Credential

```rust
pub struct Credential {
    pub id: u64,
    pub issuer: Address,
    pub subject: Address,
    pub credential_type: String,
    pub metadata_hash: BytesN<32>,
    pub issued_at: u64,
    pub revoked_at: Option<u64>,
}
```

### QuorumSlice

```rust
pub struct QuorumSlice {
    pub id: u64,
    pub owner: Address,
    pub attestors: Vec<Address>,
    pub threshold: u32,
    pub created_at: u64,
}
```

### SBT

```rust
pub struct SBT {
    pub owner: Address,
    pub credential_id: u64,
    pub minted_at: u64,
    pub metadata: Bytes,
}
```

## ⚠️ Error Handling

### Error Types

```rust
pub enum Error {
    // Credential errors
    CredentialNotFound = 1,
    CredentialAlreadyRevoked = 2,
    InvalidCredentialMetadata = 3,
    
    // SBT errors
    SBTAlreadyMinted = 10,
    SBTNotMinted = 11,
    SBTTransferDisallowed = 12,
    
    // Attestation errors
    AttestationNotFound = 20,
    InvalidQuorum = 21,
    ThresholdNotMet = 22,
    
    // Permission errors
    UnauthorizedIssuer = 30,
    UnauthorizedRevoker = 31,
    
    // Validation errors
    InvalidInput = 40,
    EmptyAttestorsList = 41,
    ZeroThreshold = 42,
}
```

## 🔒 Security Considerations

### Reentrancy Protection

- All state mutations atomic
- Cross-contract calls follow CEE (Checks-Effects-Interactions)
- No fallback functions

### Input Validation

- Verify address parameters
- Validate credential IDs exist
- Check metadata hash format
- Prevent empty attestor lists
- Enforce non-zero thresholds

### Access Control

- Issuer-only credential creation
- Owner-only revocation (or authorized)
- Admin-only ZK verification
- No privilege escalation paths

### Immutability

- Credential data immutable after creation
- SBT non-transferable by design
- Event logs immutable

## ⚡ Performance

### Optimization Strategies

- Batch operations for multiple credentials
- Efficient storage queries
- Minimal state mutations
- Indexed lookups where possible

### Gas Optimization

```rust
// Efficient: Single storage access
pub fn get_credential(env: Env, id: u64) -> Credential {
    env.storage().instance().get(&DataKey::Credential(id))
}

// Inefficient: Multiple storage accesses
pub fn expensive_operation(env: Env, ids: Vec<u64>) {
    for id in ids {
        env.storage().instance().get(&DataKey::Credential(id)) // Repeated access
    }
}
```

## 📊 Benchmarking

### Run Benchmarks

```bash
# Build benchmarks
cargo bench --bench benchmarks -- --nocapture

# Benchmark specific operation
cargo bench --bench benchmarks -- issue_credential --nocapture

# Load testing
cargo test --test load_tests -- --nocapture --test-threads=1
```

### Benchmark Results

```
issue_credential:     ~50,000 instructions
revoke_credential:    ~30,000 instructions
mint_sbt:             ~40,000 instructions
verify_credential:    ~20,000 instructions
get_balance:          ~15,000 instructions
```

## 🔄 Upgrading

### Contract Versioning

```
v1.0.0 - Initial release (core contracts)
v1.1.0 - ZK proof verification (Groth16)
v2.0.0 - Revocation registry & expiry
v3.0.0 - Multi-signature attestation
```

### Upgrade Checklist

- [ ] Run full test suite
- [ ] Generate new contract spec
- [ ] Update contract addresses
- [ ] Test with old data
- [ ] Document breaking changes
- [ ] Deploy to testnet first
- [ ] Get community review
- [ ] Deploy to mainnet

## 🤝 Contributing

1. Fork repository
2. Create feature branch: `git checkout -b feature/amazing-feature`
3. Write tests for new functionality
4. Commit: `git commit -m 'Add amazing feature'`
5. Push: `git push origin feature/amazing-feature`
6. Open Pull Request with detailed description

### Code Standards

- All code must have tests
- 80%+ code coverage required
- Follow Rust style guidelines
- Document public APIs
- Include error handling

## 📄 License

MIT License — see [LICENSE](LICENSE) for details.

## 🆘 Support

- Issues: [GitHub Issues](https://github.com/credential-labs/credential-protocol-contracts/issues)
- Docs: [Contract Docs](./docs)
- API Spec: See `spec.json` in each contract directory

---

**Built with ❤️ by Credential Labs**
