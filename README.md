# Blueshift Vault

A lightweight, secure Solana vault program built with [Pinocchio](https://github.com/anza-xyz/pinocchio) - demonstrating native Solana development without frameworks.

[![Build](https://img.shields.io/badge/build-passing-brightgreen)]()
[![Tests](https://img.shields.io/badge/tests-7%2F7-success)]()
[![Pinocchio](https://img.shields.io/badge/pinocchio-0.9.2-blue)]()
[![Solana](https://img.shields.io/badge/solana-compatible-purple)]()

---

## 🎯 Overview

Blueshift Vault is a minimal Solana program that allows users to deposit and withdraw SOL using Program Derived Addresses (PDAs). Built with Pinocchio for maximum performance and minimal compute unit consumption.

### Key Features

- 🔒 **Secure PDA-based vaults** - Each user gets their own vault derived from their pubkey
- ⚡ **Zero-copy operations** - Leverages Pinocchio's zero-copy design for optimal performance
- 🛡️ **Comprehensive security** - Prevents double deposits and unauthorized withdrawals
- ✅ **Fully tested** - 7 comprehensive integration tests using LiteSVM
- 📦 **Minimal dependencies** - No heavy frameworks, just pure native Solana

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    Blueshift Vault                      │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌──────────────┐              ┌──────────────┐       │
│  │   Deposit    │              │   Withdraw   │       │
│  │              │              │              │       │
│  │ • Validate   │              │ • Validate   │       │
│  │ • Check PDA  │              │ • Check PDA  │       │
│  │ • Transfer → │              │ • Transfer ← │       │
│  └──────────────┘              └──────────────┘       │
│         │                              │               │
│         └──────────────┬───────────────┘               │
│                        │                               │
│                  ┌─────▼─────┐                        │
│                  │   Vault   │                        │
│                  │    PDA    │                        │
│                  └───────────┘                        │
└─────────────────────────────────────────────────────────┘
```

---

## 📁 Project Structure

```
blueshift_vault/
├── src/
│   ├── lib.rs                    # Program entrypoint and instruction routing
│   └── instructions/
│       ├── mod.rs                # Module exports
│       ├── deposit.rs            # Deposit instruction implementation
│       └── withdraw.rs           # Withdraw instruction implementation
├── tests/
│   └── vault_tests.rs            # LiteSVM integration tests
├── Cargo.toml
└── README.md
```

---

## 🚀 Getting Started

### Prerequisites

- [Rust](https://rustup.rs/) (latest stable)
- [Solana CLI](https://docs.solana.com/cli/install-solana-cli-tools) (v1.18+)
- Solana BPF toolchain

### Installation

1. **Clone the repository**
   ```bash
   git clone <your-repo-url>
   cd blueshift_vault
   ```

2. **Install Solana BPF toolchain**
   ```bash
   solana-install init
   cargo install --git https://github.com/solana-labs/cargo-build-sbf
   ```

3. **Build the program**
   ```bash
   cargo build-sbf
   ```

4. **Run tests**
   ```bash
   cargo test --tests
   ```

---

## 💡 How It Works

### Deposit Flow

1. User calls `deposit(amount)` with their vault PDA
2. Program validates:
   - User is a signer
   - Vault is owned by System Program
   - Vault is empty (0 lamports)
   - Vault PDA matches expected derivation
   - Amount is non-zero
3. Transfers `amount` lamports from user to vault via System Program CPI

### Withdraw Flow

1. User calls `withdraw()` with their vault PDA
2. Program validates:
   - User is a signer
   - Vault is owned by System Program
   - Vault is not empty (> 0 lamports)
   - Vault PDA matches expected derivation
3. Vault PDA signs the transfer using seeds
4. Transfers all lamports from vault back to user

### PDA Derivation

```rust
let (vault_pda, bump) = Pubkey::find_program_address(
    &[b"vault", owner.as_ref()],
    &PROGRAM_ID
);
```

---

## 🧪 Testing

The project includes 7 comprehensive tests using [LiteSVM](https://github.com/LiteSVM/litesvm):

| Test | Description |
|------|-------------|
| `test_deposit_success` | ✅ Validates successful deposit |
| `test_deposit_with_zero_amount_fails` | ✅ Rejects zero-amount deposits |
| `test_deposit_non_empty_vault_fails` | ✅ Prevents double deposits |
| `test_withdraw_success` | ✅ Validates successful withdrawal |
| `test_withdraw_empty_vault_fails` | ✅ Prevents withdrawing from empty vault |
| `test_withdraw_unauthorized_fails` | ✅ Ensures only owner can withdraw |
| `test_deposit_and_withdraw_full_cycle` | ✅ Tests complete deposit-withdraw flow |

Run all tests:
```bash
cargo test --tests -- --nocapture
```

---

## 🔐 Security

### Security Features

- **PDA-based access control** - Only the original depositor can withdraw from their vault
- **Empty vault validation** - Prevents double deposits by ensuring vault starts at 0 lamports
- **Signer verification** - All operations require the owner's signature
- **Owner checks** - Validates vault is owned by System Program
- **Amount validation** - Rejects zero or invalid deposit amounts

## 📚 Learn More

### Resources

- **Pinocchio Documentation**: [Blueshift Learn](https://learn.blueshift.gg/en/courses/pinocchio-for-dummies/pinocchio-101)
- **Pinocchio GitHub**: [anza-xyz/pinocchio](https://github.com/anza-xyz/pinocchio)
- **LiteSVM**: [LiteSVM/litesvm](https://github.com/LiteSVM/litesvm)
- **Solana Docs**: [docs.solana.com](https://docs.solana.com)

### Why Pinocchio?

Compared to Anchor and native `solana-program`:

| Feature | Anchor | Native | Pinocchio |
|---------|--------|--------|-----------|
| Zero-copy operations | ❌ | ❌ | ✅ |
| Small binary size | ⚠️ | ✅ | ✅ |
| Low CU consumption | ⚠️ | ✅ | ✅ |
| Macro-free | ❌ | ✅ | ✅ |
| Type safety | ✅ | ⚠️ | ✅ |

---

## 🛠️ Development

### Building

```bash
# Build the BPF program
cargo build-sbf

# Build with release optimizations
cargo build-sbf --release
```

### Testing

```bash
# Run all tests
cargo test --tests

# Run specific test
cargo test test_deposit_success

# Run with output
cargo test --tests -- --nocapture
```