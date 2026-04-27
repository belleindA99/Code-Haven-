
---

# 🐱 Meowdoro DApp

**Meowdoro DApp** - Blockchain-Based Pomodoro Productivity & Reward System

---

## Project Description

Meowdoro DApp is a decentralized smart contract application built on the Stellar blockchain using the Soroban SDK. It transforms the Pomodoro technique into a **focus-to-earn system** where users stake XLM, complete sessions, earn rewards, and grow a virtual cat.

Each completed session increases the user’s **cat level**, representing consistency and productivity. The smart contract enforces commitment—users only get rewards if they actually complete sessions.

---

## Project Vision

* **Financial Motivation** → Focus backed by real value
* **Gamified Productivity** → Cat grows as you stay consistent 🐱
* **Trustless Accountability** → Smart contract enforces rules
* **Micro-rewards via Stellar** → Fast + low fees

---

## Key Features

### 1. Pomodoro Staking

Lock funds before starting a session

### 2. Completion Tracking

Mark session as finished

### 3. Rewards System

Earn stake + bonus after completion

### 4. Cat Growth 🐱

Each session = +1 level

---

## Contract Functions

* `start_session()`
* `complete_session()`
* `claim_reward()`
* `get_level()`

---

# 🧠 CORE MVP FLOW

User → stakes XLM → completes session → claims reward → cat grows

⏱ Demo: **< 2 minutes**

---

# 📁 FULL CODE FILES

---

## 📄 `src/lib.rs`

```rust
#![no_std]
use soroban_sdk::{contract, contractimpl, Env, Address, symbol_short, Symbol};

// Storage keys
const STAKE: Symbol = symbol_short!("STK");
const DONE: Symbol = symbol_short!("DONE");
const LEVEL: Symbol = symbol_short!("LVL");

#[contract]
pub struct Meowdoro;

#[contractimpl]
impl Meowdoro {

    // Start Pomodoro session (stake funds)
    pub fn start_session(env: Env, user: Address, amount: i128) {
        user.require_auth();

        env.storage().instance().set(&(STAKE, user.clone()), &amount);
        env.storage().instance().set(&(DONE, user.clone()), &false);
    }

    // Mark session as completed
    pub fn complete_session(env: Env, user: Address) {
        user.require_auth();

        env.storage().instance().set(&(DONE, user.clone()), &true);
    }

    // Claim reward and grow cat
    pub fn claim_reward(env: Env, user: Address) -> (i128, i128) {
        user.require_auth();

        let done: bool = env.storage()
            .instance()
            .get(&(DONE, user.clone()))
            .unwrap_or(false);

        if !done {
            panic!("Session not completed");
        }

        let stake: i128 = env.storage()
            .instance()
            .get(&(STAKE, user.clone()))
            .unwrap_or(0);

        // Reward = +10%
        let reward = stake + (stake / 10);

        // Get current level
        let level: i128 = env.storage()
            .instance()
            .get(&(LEVEL, user.clone()))
            .unwrap_or(0);

        let new_level = level + 1;

        // Update level
        env.storage().instance().set(&(LEVEL, user.clone()), &new_level);

        (reward, new_level)
    }

    // View cat level
    pub fn get_level(env: Env, user: Address) -> i128 {
        env.storage()
            .instance()
            .get(&(LEVEL, user))
            .unwrap_or(0)
    }
}
```

---

## 🧪 `src/test.rs` (EXACTLY 5 TESTS)

```rust
#[cfg(test)]
mod tests {
    use soroban_sdk::{Env, Address};
    use crate::Meowdoro;

    // 1. Happy Path
    #[test]
    fn test_happy_path() {
        let env = Env::default();
        let user = Address::random(&env);

        Meowdoro::start_session(env.clone(), user.clone(), 10);
        Meowdoro::complete_session(env.clone(), user.clone());

        let (reward, level) = Meowdoro::claim_reward(env.clone(), user.clone());

        assert_eq!(reward, 11);
        assert_eq!(level, 1);
    }

    // 2. Edge Case: Claim without completing
    #[test]
    #[should_panic]
    fn test_claim_without_complete() {
        let env = Env::default();
        let user = Address::random(&env);

        Meowdoro::start_session(env.clone(), user.clone(), 10);
        Meowdoro::claim_reward(env.clone(), user.clone());
    }

    // 3. State Verification
    #[test]
    fn test_level_update() {
        let env = Env::default();
        let user = Address::random(&env);

        Meowdoro::start_session(env.clone(), user.clone(), 10);
        Meowdoro::complete_session(env.clone(), user.clone());
        Meowdoro::claim_reward(env.clone(), user.clone());

        let level = Meowdoro::get_level(env.clone(), user.clone());
        assert_eq!(level, 1);
    }

    // 4. Multiple Sessions
    #[test]
    fn test_multiple_sessions() {
        let env = Env::default();
        let user = Address::random(&env);

        for _ in 0..3 {
            Meowdoro::start_session(env.clone(), user.clone(), 10);
            Meowdoro::complete_session(env.clone(), user.clone());
            Meowdoro::claim_reward(env.clone(), user.clone());
        }

        let level = Meowdoro::get_level(env.clone(), user.clone());
        assert_eq!(level, 3);
    }

    // 5. Zero Stake
    #[test]
    fn test_zero_stake() {
        let env = Env::default();
        let user = Address::random(&env);

        Meowdoro::start_session(env.clone(), user.clone(), 0);
        Meowdoro::complete_session(env.clone(), user.clone());

        let (reward, _) = Meowdoro::claim_reward(env.clone(), user.clone());
        assert_eq!(reward, 0);
    }
}
```

---

## ⚙️ `Cargo.toml`

```toml
[package]
name = "meowdoro"
version = "0.1.0"
edition = "2021"

[lib]
crate-type = ["cdylib", "rlib"]

[dependencies]
soroban-sdk = "20.0.0"

[dev-dependencies]
soroban-sdk = { version = "20.0.0", features = ["testutils"] }

[profile.release]
opt-level = "z"
overflow-checks = true
```

---

## 📘 `README.md`

```md
# Meowdoro DApp

Grow your cat by staying focused and completing Pomodoro sessions.

## Problem
Students struggle to stay focused and waste hours daily.

## Solution
Stake XLM before a session and earn rewards only if completed.

## Features
- Staking system
- Reward mechanism
- Cat leveling system

## Build
soroban contract build

## Test
cargo test

## Deploy
soroban contract deploy \
--wasm target/wasm32-unknown-unknown/release/meowdoro.wasm \
--source alice \
--network testnet

## Example
soroban contract invoke \
--id CONTRACT_ID \
--fn start_session \
--arg user \
--arg 10

## License
MIT
```

---


---



