# OTC Routes

## Standard Response Envelope

All OTC methods return the same top-level response shape.

| Key | Type | Required | Description |
| --- | --- | --- | --- |
| `success` | `boolean` | Yes | `true` for successful responses, `false` for failures. |
| `data` | `T` | Yes | Method-specific response payload. For error cases this is `{}`. |
| `message` | `string` | No | Success or error message. |
| `timestamp` | `string` | Yes | ISO 8601 timestamp. |

### Error Response Example

```json
{
  "success": false,
  "data": {},
  "message": "API-key does not exist.",
  "timestamp": "2026-04-02T10:24:50.911Z"
}
```

## Shared Types

### Enums

| Type | Allowed Values |
| --- | --- |
| `OtcOrderSide` | `'BUY' \| 'SELL'` |
| `OtcOrderType` | `'MARKET' \| 'LIMIT'` |
| `OtcTransferKind` | `'Fiat' \| 'Crypto'` |
| `OtcOperationStatus` | `'Success' \| 'Pending' \| 'Failed'` |

### EntityCreds

Used when a partner supports entity-specific credentials.

| Key | Type | Required |
| --- | --- | --- |
| `entityName` | `string` | No |
| `trafficType` | `string` | No |

## Routes

### 1. `getAccountDetails`

Returns supported fiat and crypto accounts for a partner.

#### Request

| Key | Type | Required | Description |
| --- | --- | --- | --- |
| `partnerName` | `string` | Yes | OTC partner name. |
| `asset` | `string` | No | Filter by asset/currency. |
| `beneId` | `string` | No | Partner-specific beneficiary identifier. Currently used by Caliza. |
| `entityCreds` | `EntityCreds` | No | Entity-specific credential selector. |

#### Response `data`

| Key | Type | Required |
| --- | --- | --- |
| `fiat` | `FiatAccountDetails[]` | Yes |
| `crypto` | `CryptoAccountDetails[]` | Yes |

#### `FiatAccountDetails`

| Key | Type | Required |
| --- | --- | --- |
| `id` | `string` | Yes |
| `name` | `string` | No |
| `asset` | `string` | Yes |
| `accountNumber` | `string` | No |
| `accountType` | `string` | No |
| `creationDate` | `string` | Yes |
| `deleted` | `boolean` | Yes |
| `network` | `string` | No |

#### `CryptoAccountDetails`

| Key | Type | Required |
| --- | --- | --- |
| `id` | `string` | Yes |
| `address` | `string` | Yes |
| `asset` | `string` | Yes |
| `tag` | `string` | No |
| `memo` | `string` | No |
| `network` | `string` | No |
| `deleted` | `boolean` | Yes |
| `creationDate` | `string` | Yes |

### 2. `getDepositDetails`

Returns a deposit transaction by partner-specific identifier.

#### Request

| Key           | Type | Required | Description                          |
|---------------| --- | --- |--------------------------------------|
| `partnerName` | `string` | Yes | OTC partner name.                    |
| `id`          | `string` | Yes | Deposit identifier.                  |
| `asset`       | `string` | Yes | Deposit amount.                      |
| `amount`      | `string` | Yes | Deposit asset.                       |
| `entityCreds` | `EntityCreds` | No | Entity-specific credential selector. |

#### Response `data`

| Key | Type | Required |
| --- | --- | --- |
| `id` | `string` | Yes |
| `asset` | `string` | No |
| `amount` | `number` | No |
| `timestamp` | `string` | Yes |
| `type` | `OtcTransferKind` | Yes |
| `status` | `OtcOperationStatus` | Yes |

### 3. `trade`

Places a buy or sell order.

#### Request

| Key            | Type | Required | Description                          |
|----------------| --- |----------|--------------------------------------|
| `partnerName`  | `string` | Yes      | OTC partner name.                    |
| `fromCurrency` | `string` | Yes      | Source currency.                     |
| `toCurrency`   | `string` | Yes      | Destination currency.                |
| `side`         | `OtcOrderSide` | Yes      | `BUY` or `SELL`.                     |
| `size`         | `number \| string` | Yes      | Order size.                          |
| `type`         | `OtcOrderType` | Yes      | `MARKET` or `LIMIT`.                 |
| `price`        | `number \| string` | No       | If type is `LIMIT`,then required.      |
| `clientOid`    | `string` | No       | Client order identifier.             |
| `entityCreds`  | `EntityCreds` | No       | Entity-specific credential selector. |

#### Response `data`

| Key | Type | Required |
| --- | --- | --- |
| `orderId` | `string` | Yes |
| `clientOid` | `string` | No |

### 4. `tradeDetails`

Returns details for a trade/order.

#### Request

| Key | Type | Required | Description |
| --- | --- | --- | --- |
| `partnerName` | `string` | Yes | OTC partner name. |
| `id` | `string` | Yes | Partner order identifier. |
| `fromCurrency` | `string` | Yes | Source currency. |
| `toCurrency` | `string` | Yes | Destination currency. |
| `entityCreds` | `EntityCreds` | No | Entity-specific credential selector. |

#### Response `data`

| Key         | Type                 | Required |
|-------------|----------------------| --- |
| `orderId`   | `string`             | Yes |
| `clientOid` | `string`             | No |
| `symbol`    | `string`             | No |
| `status`    | `OtcOperationStatus` | Yes |
| `price`     | `number`             | Yes |
| `size`      | `number`             | No |
| `error`     | `string`             | No |
| `timestamp` | `string`             | Yes |

### 5. `payout`

Creates a payout/withdrawal request.

#### Request

| Key | Type | Required | Description |
| --- | --- |----------| --- |
| `partnerName` | `string` | Yes      | OTC partner name. |
| `fromId` | `string` | No       | Source account/wallet identifier. |
| `toId` | `string` | Yes      | Destination identifier. Some adapters resolve this from DB-backed account records. |
| `fromCurrency` | `string` | Yes      | Source currency. |
| `toCurrency` | `string` | Yes      | Destination currency. |
| `amount` | `number \| string` | Yes      | Payout amount. |
| `clientOid` | `string` | No       | Client payout identifier. |
| `entityCreds` | `EntityCreds` | No       | Entity-specific credential selector. |

#### Response `data`

| Key | Type | Required |
| --- | --- | --- |
| `orderId` | `string` | Yes |
| `clientOid` | `string` | No |

### 6. `payoutDetails`

Returns the current status of a payout.

#### Request

| Key | Type | Required | Description |
| --- | --- | --- | --- |
| `partnerName` | `string` | Yes | OTC partner name. |
| `id` | `string` | Yes | Payout identifier. |
| `entityCreds` | `EntityCreds` | No | Entity-specific credential selector. |

#### Response `data`

| Key | Type | Required |
| --- | --- | --- |
| `orderId` | `string` | Yes |
| `clientOid` | `string` | No |
| `status` | `OtcOperationStatus` | Yes |
| `amount` | `number` | No |
| `fiatAmount` | `number` | No |
| `error` | `string` | No |
| `timestamp` | `string` | Yes |

### 7. `transferRate`

Returns fee/rate data for a transfer flow.

#### Request

| Key            | Type | Required | Description |
|----------------| --- | --- | --- |
| `partnerName`  | `string` | Yes | OTC partner name. |
| `fromCurrency` | `string` | Yes | fromCurrency for fee lookup. |
| `toCurrency`   | `string` | Yes | toCurrency for fee lookup. |
| `amount`       | `number \| string` | Yes | amount for fee lookup. |
| `entityCreds`  | `EntityCreds` | No | Entity-specific credential selector. |

#### Response `data`

| Key | Type | Required |
| --- | --- | --- |
| `fee` | `number` | No |

### 8. `tradeRate`

Returns quote/pricing details before trade execution.

#### Request

| Key | Type | Required | Description |
| --- | --- | --- | --- |
| `partnerName` | `string` | Yes | OTC partner name. |
| `fromCurrency` | `string` | Yes | Source currency. |
| `toCurrency` | `string` | Yes | Destination currency. |
| `side` | `OtcOrderSide` | Yes | `BUY` or `SELL`. |
| `size` | `number \| string` | Yes | Requested size. |
| `type` | `OtcOrderType` | Yes | `MARKET` or `LIMIT`. |
| `entityCreds` | `EntityCreds` | No | Entity-specific credential selector. |

#### Response `data`

| Key | Type | Required |
| --- | --- | --- |
| `fromCurrency` | `string` | Yes |
| `toCurrency` | `string` | Yes |
| `toCurrencyPrice` | `number` | No |
| `fromCurrencyPrice` | `number` | No |
| `quoteId` | `string` | No |

## Partner-Specific Notes

### Caliza

- `getAccountDetails` may use `beneId`.

### Hercle

- `tradeDetails` is matched against the partner `orderId`.

### B2C2

- `clientOid` is required in the `trade`.
