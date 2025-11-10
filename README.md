# 🏗️ Stablecoin Bridge with LP Integration

*Clarity Smart Contract Documentation*
*Author: Senior Stacks Developer*
*Language: [Clarity](https://docs.stacks.co/write-smart-contracts/clarity-language)*

---

## 📘 Overview

The **Stablecoin Bridge with LP Integration** is a decentralized financial primitive designed for the **Stacks blockchain**. It enables users to **lock BTC**, **mint a USD-pegged stablecoin**, and **participate in liquidity pools** that facilitate stablecoin swaps and collateral management.

This contract integrates **stablecoin minting**, **collateralized debt positions (CDPs)**, and **liquidity provision (LP)** under a unified system, enabling a secure, collateral-backed stable asset ecosystem built directly on Stacks.

---

## 🧩 System Overview

### Core Components

| Component                        | Description                                                                                                  |
| -------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| **Collateral Vaults**            | Users lock BTC and mint stablecoins against collateral, maintaining a minimum collateralization ratio.       |
| **Stablecoin Module**            | Handles minting, burning, and supply tracking of the USD-pegged stablecoin.                                  |
| **Liquidity Pool (LP)**          | Allows users to add BTC and stablecoins into a shared liquidity pool, earning pool tokens as proof of share. |
| **Oracle Integration**           | The contract owner updates the BTC/USD oracle price, ensuring collateral valuations remain accurate.         |
| **Collateral Ratio Enforcement** | Dynamic validation ensures users remain above liquidation thresholds.                                        |

---

## ⚙️ Contract Architecture

### **1. Initialization Layer**

* **`initialize(initial-price)`**
  Sets the initial BTC/USD price and activates the contract.
  Only the **contract owner** (deployer) can call this function.
  Prevents re-initialization through `ERR-ALREADY-INITIALIZED`.

* **`update-price(new-price)`**
  Updates the oracle price manually by the contract owner.
  Includes validation to prevent invalid or extreme values.

---

### **2. Collateral Management**

* **`deposit-collateral(btc-amount)`**
  Allows users to deposit BTC into their collateral vault.
  Enforces a **minimum deposit requirement** and securely transfers BTC to the contract vault.

* **`check-collateral-requirement` (private)**
  Ensures collateral ratio stays above the **150% minimum**.
  Used during minting to protect system stability.

* **`calculate-collateral-ratio` (private)**
  Returns the percentage ratio of BTC value (in USD) to stablecoin debt.

---

### **3. Stablecoin Logic**

* **`mint-stablecoin(amount)`**
  Allows users to mint new stablecoins against locked BTC.
  Validates collateralization ratio, maximum supply, and mint limits.

* **`burn-stablecoin(amount)`**
  Burns stablecoins to reduce minted debt and free collateral.
  Ensures user has sufficient balance and updates vault accordingly.

---

### **4. Liquidity Provision**

* **`add-liquidity(btc-amount, stable-amount)`**
  Adds liquidity to the BTC/stablecoin pool.
  Mints **LP tokens** proportional to the provided assets.
  Updates pool balances and tracks provider contributions.

* **`remove-liquidity(lp-tokens)`**
  Removes liquidity proportionally based on LP share.
  Returns BTC and stablecoins to the provider.
  Adjusts pool and user records accordingly.

---

### **5. Utility & Read-Only Functions**

* **`get-vault-details(owner)`**
  Returns a user’s BTC collateral, minted stablecoin amount, and last update height.

* **`get-collateral-ratio(owner)`**
  Returns the collateralization ratio for a given vault owner.

* **`get-pool-details()`**
  Returns pool balances, total stablecoin supply, and oracle price.

* **`get-lp-details(provider)`**
  Provides LP token balance and contributions for a liquidity provider.

---

## 🧮 Key Constants

| Constant                   | Description                                | Example                       |
| -------------------------- | ------------------------------------------ | ----------------------------- |
| `MINIMUM-COLLATERAL-RATIO` | Minimum safe collateral ratio (in %)       | `150%`                        |
| `LIQUIDATION-RATIO`        | Collateral ratio for liquidation threshold | `130%`                        |
| `MINIMUM-DEPOSIT`          | Minimum BTC deposit (in sats)              | `1,000,000` (0.01 BTC)        |
| `POOL-FEE-RATE`            | Fee applied to swaps/liquidity             | `0.3%`                        |
| `MAX-MINT-AMOUNT`          | Global cap for minting stablecoins         | `1,000,000,000,000` (10K USD) |
| `MAX-PRICE`                | Maximum valid BTC/USD price                | `100,000,000,000` (1M USD)    |

---

## 🧠 Error Handling

| Code    | Identifier                    | Description                        |
| ------- | ----------------------------- | ---------------------------------- |
| `u1000` | `ERR-NOT-AUTHORIZED`          | Caller lacks required permissions  |
| `u1001` | `ERR-INSUFFICIENT-BALANCE`    | Insufficient balance for operation |
| `u1002` | `ERR-INVALID-AMOUNT`          | Invalid or out-of-range amount     |
| `u1003` | `ERR-INSUFFICIENT-COLLATERAL` | Collateral ratio below requirement |
| `u1004` | `ERR-POOL-EMPTY`              | Liquidity pool is empty            |
| `u1005` | `ERR-SLIPPAGE-TOO-HIGH`       | Trade exceeds slippage tolerance   |
| `u1006` | `ERR-BELOW-MINIMUM`           | Deposit amount below threshold     |
| `u1007` | `ERR-ABOVE-MAXIMUM`           | Exceeds configured maximum         |
| `u1008` | `ERR-ALREADY-INITIALIZED`     | Contract already initialized       |
| `u1009` | `ERR-NOT-INITIALIZED`         | Contract not yet initialized       |
| `u1010` | `ERR-INVALID-PRICE`           | Price value out of valid range     |

---

## 🔄 Data Flow (Simplified)

```plaintext
   ┌──────────────────────────────────────────┐
   │               User Wallet                │
   └──────────────────────────────────────────┘
                │ deposit BTC
                ▼
   ┌──────────────────────────────────────────┐
   │          Collateral Vault (CDP)          │
   └──────────────────────────────────────────┘
                │ mint stablecoin
                ▼
   ┌──────────────────────────────────────────┐
   │          Stablecoin Module               │
   └──────────────────────────────────────────┘
                │ add liquidity
                ▼
   ┌──────────────────────────────────────────┐
   │           Liquidity Pool (LP)            │
   └──────────────────────────────────────────┘
                │ update price (admin)
                ▼
   ┌──────────────────────────────────────────┐
   │          Oracle / Contract Owner         │
   └──────────────────────────────────────────┘
```

---

## 🔐 Security Considerations

* **Authorization Control:** Only the contract owner may initialize or update oracle prices.
* **Collateral Safety:** Minting is always checked against real-time collateral ratios.
* **No Over-Minting:** Hard limits on mint amounts and total supply prevent inflation.
* **Liquidity Integrity:** LP token issuance is proportional and non-dilutive.
* **Price Validation:** Prevents oracle manipulation through range constraints.

---

## 🧭 Future Extensions

* Integration with on-chain BTC price oracles (e.g., **ALEX**, **Stacks Bridge**)
* Automated liquidation module for undercollateralized positions
* Dynamic fee structure for LP rewards
* Multi-asset collateral support (e.g., STX, sBTC)

---

## 📄 License

This smart contract is released under the **MIT License**.
Use and modify freely with attribution to the original author.
