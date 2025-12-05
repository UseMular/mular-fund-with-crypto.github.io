# Mular Wallet Funding Integration Guide

This documentation provides a comprehensive guide for integrating the Mular Wallet Funding service into your mobile application. This service enables businesses to offer crypto wallet funding to their users, instantly converting crypto deposits into Naira in the user's wallet.

## Overview

The Mular Wallet Funding API allows you to:
1.  **Create Sub Accounts**: Register your users on the Mular system.
2.  **Generate Wallet Addresses**: specific crypto deposit addresses for your users.
3.  **List Markets/Wallets**: View supported cryptocurrencies and networks.
4.  **Manage Sub Users**: Retrieve lists of registered sub-users.
5.  **Hosted Checkout**: Use a pre-built web view for funding.

## Base URL

All API requests should be made to:

```
https://api.mular.co
```

## Authentication

Authentication guidelines are specific to your integration type. Typically, you will identify your business using your `mular_id` (e.g., `mular-business`) in the URL path for certain endpoints.

## Integration Flow

1.  **Onboard User**: When a user wants to use the crypto funding feature, first create a "Sub Account" for them on Mular using their email and bank details.
2.  **Get Assets**: Fetch the list of supported cryptocurrencies and networks using "Get Sub User Wallets".
3.  **Generate Address**: When the user selects an asset, generate a specific deposit address for them.
4.  **Fund**: The user sends crypto to this address. Mular processes the conversion and settles the Naira equivalent.

---

## Endpoints

### 1. Create Sub User

Register a user from your platform as a sub-account on Mular. This is required before generating addresses for them.

**Endpoint:**
`POST /api/mular/partners`

**Headers:**
- `Content-Type`: application/json
- `Accept`: application/json

**Request Body:**

| Field | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `email` | String | Yes | User's email address |
| `nibss_bank_code` | String | Yes | Bank code (NIBSS) |
| `account_number` | String | Yes | User's bank account number |
| `external_ref` | String | Yes | A unique reference ID from your system for this user |

**Example Request:**

```json
{
    "email": "john@example.com",
    "nibss_bank_code": "00012",
    "account_number": "00000000000",
    "external_ref": "1234-1324-1324-1342"
}
```

**Success Response (201 Created):**

```json
{
    "id": "e33...",
    "email": "john@example.com",
    "account_name": "John Doe",
    "account_number": "00000000000",
    "nibss_bank_code": "00012",
    "bank_name": "Providus Bank",
    "external_ref": "1234-1324-1324-1342"
}
```

---

### 2. Get Sub User Wallets (Market List)

Retrieve the list of supported cryptocurrencies and their network details. This is useful for displaying available funding options to your users.

**Endpoint:**
`GET /api/mular/partners/:mular_id/wallets`

**Path Variables:**
- `mular_id`: Your business identifier on Mular (e.g., `mular-business`).

**Query Parameters:**
- `sub_user`: (Optional) The `external_ref` of the sub-user.
- `currency`: (Optional) Filter by currency code (e.g., `usdt`).
- `network`: (Optional) Filter by network (e.g., `bep20`).

**Example Request:**
`GET https://api.mular.co/api/mular/partners/your-business-id/wallets?sub_user=1234-1324-1324-1342`

**Success Response (200 OK):**

```json
[
    {
        "id": "...",
        "name": "USDT Tether",
        "currency": "usdt",
        "deposit_address": "0x438...",
        "default_network": "bep20",
        "image": "https://...",
        "buy": "1501.83",
        "sell": "1475.77",
        "networks": [
            {
                "id": "bep20",
                "name": "Binance Smart Chain",
                "deposits_enabled": true,
                "withdraws_enabled": true
            },
            ...
        ]
    },
    ...
]
```

---

### 3. Get Sub User Wallet Address

Generate or retrieve a specific deposit address for a sub-user for a chosen currency and network.

**Endpoint:**
`GET /api/mular/partners/:mular_id/address`

**Path Variables:**
- `mular_id`: Your business identifier.

**Query Parameters:**
- `sub_user`: (Required) The `external_ref` of the sub-user.
- `currency`: (Required) Currency code (e.g., `usdt`).
- `network`: (Required) Network code (e.g., `bep20`).

**Example Request:**
`GET https://api.mular.co/api/mular/partners/your-business-id/address?currency=usdt&network=bep20&sub_user=1234-1324-1324-1342`

**Success Response (200 OK):**

```json
{
    "status": true,
    "address": "0x438B48753B0d6CE93ac0BFfac4F009bf69bf8268",
    "network": "bep20"
}
```

---

### 4. List Sub Users

Retrieve a paginated list of all sub-users registered under your partner account.

**Endpoint:**
`GET /api/mular/partners/sub_accounts`

**Success Response (200 OK):**

```json
{
    "current_page": 1,
    "data": [
        {
            "id": "...",
            "email": "john@example.com",
            "account_name": "John Doe",
            "account_number": "00000000000",
            ...
        }
    ],
    "first_page_url": "...",
    "total": 1
}
```

---

### 5. Sub User PWM Link (Hosted Flow)

If you prefer not to build the UI yourself, you can redirect the user to a hosted payment page.

**Endpoint:**
`GET pay.mular.co/business/:mularid`

**Path Variables:**
- `mularid`: Your business identifier.

**Query Parameters:**
- `sub_user`: The `external_ref` of the sub-user.

**Usage:**
Redirect the user's browser/webview to:
`https://pay.mular.co/business/your-business-id?sub_user=1234-1324-1324-1342`
