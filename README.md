# **sBTC-FlexiMint - Smart Contract**

## **Overview**

The **sBTC-FlexiMint** smart contract is a **semi-fungible token (SFT) vault** built on the Stacks blockchain.
It allows an **admin** to create and manage **SFT-based assets**, track token balances, set pricing, manage allowances, and record price history.

This contract is suitable for **tokenized assets**, **fractionalized ownership**, **digital collectibles**, **membership passes**, and any scenario where semi-fungible tokens are required.

---

## **Features**

* **Token Creation (Minting):**
  Admins can mint new SFT tokens with a unique name, category, maximum supply, and initial price.
* **Balance Management:**
  Automatically allocates the entire max supply of a newly minted token to the contract admin.
* **Price Tracking:**
  Maintains a historical log of token price updates for transparency and auditing.
* **Allowances & Authorizations:**
  Supports an allowance system where holders can authorize others to spend tokens on their behalf.
* **Access Control:**
  Only the **contract admin** can mint new tokens or perform restricted operations.
* **Error Handling:**
  Implements strict input validation and descriptive error codes.

---

## **Data Structures**

### **1. Tokens**

Stores metadata for each token.

| Field               | Type               | Description                        |
| ------------------- | ------------------ | ---------------------------------- |
| `token-id`          | `uint`             | Unique identifier for each token   |
| `token-name`        | `string-ascii(64)` | Name of the token                  |
| `token-category`    | `string-ascii(32)` | Category/type of token             |
| `max-supply`        | `uint`             | Maximum number of tokens allowed   |
| `token-price`       | `uint`             | Current price of the token         |
| `last-price-update` | `uint`             | Timestamp of the last price update |

---

### **2. Balances**

Tracks how many tokens each principal holds.

| Field      | Type        | Description                |
| ---------- | ----------- | -------------------------- |
| `holder`   | `principal` | Address of the token owner |
| `token-id` | `uint`      | ID of the token            |
| `amount`   | `uint`      | Number of tokens owned     |

---

### **3. Allowances**

Manages approved token spending by third parties.

| Field            | Type        | Description                     |
| ---------------- | ----------- | ------------------------------- |
| `holder`         | `principal` | Owner of the tokens             |
| `authorized`     | `principal` | Approved spender                |
| `token-id`       | `uint`      | ID of the token                 |
| `allowed-amount` | `uint`      | Maximum tokens allowed to spend |

---

### **4. Price History**

Keeps historical records of token price changes.

| Field       | Type   | Description                  |
| ----------- | ------ | ---------------------------- |
| `token-id`  | `uint` | Token identifier             |
| `timestamp` | `uint` | Block timestamp              |
| `price`     | `uint` | Token price at the timestamp |

---

## **Error Codes**

| Code | Error Name                    | Description                           |
| ---- | ----------------------------- | ------------------------------------- |
| 100  | `err-not-authorized`          | Sender is not authorized              |
| 101  | `err-token-exists`            | Token with this ID already exists     |
| 102  | `err-token-not-found`         | Token does not exist                  |
| 103  | `err-insufficient-funds`      | Not enough balance                    |
| 104  | `err-invalid-token-name`      | Token name is invalid                 |
| 105  | `err-invalid-category`        | Token category is invalid             |
| 106  | `err-invalid-max-supply`      | Max supply must be greater than zero  |
| 107  | `err-invalid-token-price`     | Token price must be greater than zero |
| 108  | `err-invalid-recipient`       | Recipient address is invalid          |
| 109  | `err-invalid-transfer-amount` | Transfer amount is invalid            |
| 110  | `err-insufficient-allowance`  | Authorized allowance too low          |
| 111  | `err-invalid-authorized-addr` | Authorized address is invalid         |
| 112  | `err-invalid-price-update`    | Invalid price update attempt          |

---

## **Public Functions**

### **1. `mint-token`**

Creates a new semi-fungible token.

#### **Definition**

```clojure
(define-public (mint-token
    (token-name (string-ascii 64))
    (token-category (string-ascii 32))
    (max-supply uint)
    (token-price uint)
)
```

#### **Parameters**

| Name             | Type               | Description                      |
| ---------------- | ------------------ | -------------------------------- |
| `token-name`     | `string-ascii(64)` | Name of the token                |
| `token-category` | `string-ascii(32)` | Token's category/type            |
| `max-supply`     | `uint`             | Maximum number of tokens allowed |
| `token-price`    | `uint`             | Initial price of the token       |

#### **Returns**

* `(ok token-id)` → If the token is successfully created.
* `(err ...)` → If validation fails.

#### **Behavior**

* Only the **contract admin** can mint tokens.
* Generates a unique `token-id` automatically.
* Initializes balance for the **contract admin**.
* Records the initial price in **price-history**.

#### **Example Usage**

```clojure
(mint-token "Golden Pass" "Membership" u1000 u500)
;; => (ok u1)
```

---

### **2. `is-valid-token`** *(read-only)*

Checks if a token exists.

#### **Definition**

```clojure
(define-read-only (is-valid-token (token-id uint)))
```

#### **Parameters**

| Name       | Type   | Description     |
| ---------- | ------ | --------------- |
| `token-id` | `uint` | ID of the token |

#### **Returns**

* `true` → If the token exists.
* `false` → If the token does not exist.

#### **Example Usage**

```clojure
(is-valid-token u1)
;; => true
```

---

## **Admin Management**

* The **contract admin** is set at deployment (`tx-sender`).
* Admin privileges:

  * Minting new tokens.
  * Future functions like updating prices, burning tokens, or changing metadata.

---

## **Future Enhancements**

* **Token Transfers:** Allow users to transfer tokens between principals.
* **Allowance Spending:** Implement `transfer-from` functionality.
* **Dynamic Price Updates:** Let the admin update prices and record them automatically.
* **Burning Mechanism:** Enable token removal to manage circulating supply.
* **Events & Notifications:** Emit events for better off-chain tracking.

---

## **Deployment Notes**

* Ensure that the deploying account is the intended **contract admin**.
* Initial supply of each token is automatically allocated to the admin.
* Pricing logic assumes token prices are in microSTX or another consistent unit.

---

## **Example Workflow**

### **Step 1 — Mint a Token**

```clojure
(mint-token "VIP Ticket" "Event" u500 u250)
;; => (ok u1)
```

### **Step 2 — Check If Token Exists**

```clojure
(is-valid-token u1)
;; => true
```

### **Step 3 — Query Balances**

```clojure
(map-get? balances { holder: 'ST3...', token-id: u1 })
;; => (some { amount: u500 })
```

---
