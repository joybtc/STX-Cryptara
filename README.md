

#  STX-Cryptara - Decentralized KYC Verification Platform

**STX-Cryptara** is a decentralized KYC (Know Your Customer) verification platform built on the Stacks blockchain using the Clarity smart contract language. It enables users to request verification, and allows contract owners to approve, reject, revoke, and manage verification statuses in a transparent and tamper-proof way.

---

## 📜 Overview

STX-Cryptara manages and tracks KYC status for blockchain addresses. It ensures:

* Self-service KYC requests.
* Controlled verification and rejection by a designated contract owner.
* Full traceability of KYC actions with timestamped records.
* The ability to transfer contract ownership.
* Status and metadata query capabilities via read-only functions.

---

## 🚀 Features

* ✅ **Request Verification**: Users can submit their KYC data for review.
* 🔐 **Verify or Reject**: Only the contract owner can approve or reject KYC requests.
* 🔁 **Revoke Status**: The owner can revoke an address's KYC status.
* 🔄 **Ownership Transfer**: Ownership of the contract can be securely transferred.
* 🔎 **Read-Only Status Query**: Anyone can check the verification status of an address.

---

## 🛠️ Smart Contract Details

### ✅ Verification Status Codes

| Code | Status       | Description                         |
| ---- | ------------ | ----------------------------------- |
| `0`  | Not Verified | No KYC request submitted or revoked |
| `1`  | Pending      | KYC request submitted, under review |
| `2`  | Verified     | KYC request approved                |
| `3`  | Rejected     | KYC request rejected                |

---

## 📂 Contract Structure

### 🔐 Data Variables

* `contract-owner` - The principal that owns and controls the contract.

### 🗃️ Maps

* `verified-addresses` - Maps principals to their KYC data:

  ```clojure
  {
    status: uint,        ;; Current verification status
    timestamp: uint,     ;; Last status update block height
    kyc-data: string,    ;; Submitted KYC information (max 500 chars)
    verifier: principal  ;; Who performed the last status update
  }
  ```

---

## 📤 Public Functions

### `request-verification(kyc-data)`

Request KYC verification. KYC data must be non-empty and ≤ 500 characters.
**Access:** Anyone
**Errors:** `ERR_ALREADY_VERIFIED`, `ERR_INVALID_INPUT`

---

### `verify-address(address)`

Approve a pending verification request.
**Access:** Contract Owner only
**Errors:** `ERR_UNAUTHORIZED`, `ERR_INVALID_STATUS`

---

### `reject-verification(address)`

Reject a pending verification request.
**Access:** Contract Owner only
**Errors:** `ERR_UNAUTHORIZED`, `ERR_INVALID_STATUS`

---

### `revoke-verification(address)`

Reset a verified or pending user back to “not verified.”
**Access:** Contract Owner only
**Errors:** `ERR_UNAUTHORIZED`, `ERR_INVALID_STATUS`

---

### `transfer-ownership(new-owner)`

Transfer contract ownership to a new principal.
**Access:** Contract Owner only
**Errors:** `ERR_UNAUTHORIZED`, `ERR_INVALID_NEW_OWNER`

---

## 🔍 Read-Only Functions

### `get-verification-status(address)`

Returns the current status, timestamp, verifier, and KYC data of an address.
**Returns default values if not found.**

---

### `is-contract-owner(address)`

Returns `true` if the given address is the contract owner.

---

## 🔒 Private Helpers

* `is-valid-principal(address)`
  Prevents contract self-calls and sender manipulating others.

* `is-valid-kyc-data(data)`
  Ensures KYC data is not empty and ≤ 500 characters.

* `validate-status-change(current-status, allowed-statuses)`
  Ensures that the current status is within a list of permitted statuses.

* `validate-input-address(address)`
  Validates a principal before use.

* `validate-owner-only()`
  Ensures that only the contract owner can call restricted functions.

* `is-valid-new-owner(new-owner)`
  Validates that a new owner address is not equal to the current owner or the contract.

---

## ⚠️ Error Codes

| Code   | Name                    | Description                                   |
| ------ | ----------------------- | --------------------------------------------- |
| `u100` | `ERR_UNAUTHORIZED`      | Caller is not the contract owner              |
| `u101` | `ERR_ALREADY_VERIFIED`  | Address already submitted KYC                 |
| `u102` | `ERR_NOT_VERIFIED`      | Address not verified                          |
| `u103` | `ERR_INVALID_STATUS`    | Status not eligible for transition            |
| `u104` | `ERR_INVALID_INPUT`     | Provided input is malformed or disallowed     |
| `u105` | `ERR_INVALID_NEW_OWNER` | New owner is invalid or same as current owner |

---

## 🧪 Deployment Notes

* The contract initializes the deployer (`tx-sender`) as the owner and sets a default verification record.
* You may extend the logic to include multiple verifiers, KYC expiration, or off-chain integration via oracles.

---

## 📘 Example Usage

```clojure
;; User requests KYC
(request-verification u"Name: Alice; ID: 12345")

;; Contract owner approves
(verify-address 'SP...ALICE)

;; Owner checks status
(get-verification-status 'SP...ALICE)
```

---
