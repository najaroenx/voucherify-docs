# Voucherify API — Validate Stackable Discounts

> 📄 Source: https://docs.voucherify.io/api-reference/validations/validate-stackable-discounts

---

## 📌 คืออะไร

**POST /v1/validations** — ตรวจสอบว่า redeemables (vouchers, promotions) ที่ต้องการใช้พร้อมกัน **valid ไหม**  
ใช้ก่อน checkout จริง เพื่อยืนยันส่วนลดและ lock session ชั่วคราว

### Flow การใช้งาน

```
1. User กรอก code หรือเลือก promotion
         ↓
2. POST /v1/validations  ← ตรงนี้
   ├─ invalid → แสดง error ให้ user
   └─ valid → ได้ session.key กลับมา
         ↓
3. POST /v1/redemptions  (ส่ง session.key ไปด้วย)
         ↓
4. ถ้า user ยกเลิก → POST /v1/sessions/{key}/release
```

---

## 🔧 Endpoint

```
POST https://{cluster}.voucherify.io/v1/validations
```

---

## 📋 Request Body Fields

| Field | Type | Required | คำอธิบาย |
|---|---|---|---|
| `redeemables` | `array` | ✅ | รายการ vouchers/promotions ที่ต้องการใช้ |
| `customer` | `object` | ✅ | Customer ที่ใช้ |
| `order` | `object` | ✅ | Order / cart |
| `session` | `object` | ❌ | ตั้งค่า session lock |
| `metadata` | `object` | ❌ | custom data เพิ่มเติม |

### `redeemables` Item Fields

| Field | Type | Required | คำอธิบาย |
|---|---|---|---|
| `object` | `string` | ✅ | `"voucher"` หรือ `"promotion_tier"` |
| `id` | `string` | ✅ | Voucher code หรือ promotion tier ID |
| `gift` | `object` | ❌ | สำหรับ GIFT_VOUCHER — ระบุจำนวนที่ใช้ |

### `session` Object

| Field | Type | คำอธิบาย |
|---|---|---|
| `type` | `string` | `"LOCK"` — lock quota ระหว่าง checkout |
| `key` | `string` | กำหนด session key เอง (optional) |
| `ttl` | `integer` | อายุ session (default: 7) |
| `ttl_unit` | `string` | `HOURS`, `DAYS`, `MINUTES` |

---

## 📦 Request Body ตัวอย่าง

### Validate voucher code เดียว

```json
{
  "redeemables": [
    { "object": "voucher", "id": "PROMO10" }
  ],
  "customer": {
    "source_id": "user-001"
  },
  "order": {
    "source_id": "order-001",
    "amount": 15000,
    "items": [
      {
        "source_id": "SKU-001",
        "quantity": 2,
        "price": 7500,
        "amount": 15000
      }
    ]
  },
  "session": {
    "type": "LOCK",
    "ttl": 30,
    "ttl_unit": "MINUTES"
  }
}
```

### Validate หลาย redeemables (stackable)

```json
{
  "redeemables": [
    { "object": "voucher", "id": "PROMO10" },
    { "object": "voucher", "id": "FREESHIP" },
    { "object": "promotion_tier", "id": "promo_tier_abc" }
  ],
  "customer": { "source_id": "user-001" },
  "order": {
    "amount": 15000,
    "items": [
      { "source_id": "SKU-001", "quantity": 2, "price": 7500, "amount": 15000 }
    ]
  },
  "session": { "type": "LOCK", "ttl": 30, "ttl_unit": "MINUTES" }
}
```

### Validate Gift Card (ระบุจำนวนที่ใช้)

```json
{
  "redeemables": [
    {
      "object": "voucher",
      "id": "GIFT50",
      "gift": { "credits": 5000 }
    }
  ],
  "customer": { "source_id": "user-001" },
  "order": { "amount": 15000 },
  "session": { "type": "LOCK", "ttl": 30, "ttl_unit": "MINUTES" }
}
```

---

## ⚡ cURL ตัวอย่าง

```bash
curl --request POST \
  --url https://{cluster}.voucherify.io/v1/validations \
  --header 'X-App-Id: YOUR_APP_ID' \
  --header 'X-App-Token: YOUR_APP_TOKEN' \
  --header 'Content-Type: application/json' \
  --data '{
    "redeemables": [
      { "object": "voucher", "id": "PROMO10" }
    ],
    "customer": { "source_id": "user-001" },
    "order": {
      "amount": 15000,
      "items": [
        { "source_id": "SKU-001", "quantity": 2, "price": 7500, "amount": 15000 }
      ]
    },
    "session": { "type": "LOCK", "ttl": 30, "ttl_unit": "MINUTES" }
  }'
```

---

## 🚀 Implementation Notes

### 1) TypeScript Interfaces

```typescript
interface ValidationRedeemableInput {
  object: 'voucher' | 'promotion_tier';
  id: string;
  gift?: { credits: number };
}

interface ValidationSession {
  type: 'LOCK';
  key?: string;
  ttl?: number;
  ttl_unit?: 'MINUTES' | 'HOURS' | 'DAYS';
}

interface ValidateStackableParams {
  redeemables: ValidationRedeemableInput[];
  customer: { id?: string; source_id?: string; email?: string };
  order: {
    source_id?: string;
    amount: number;
    items?: OrderItem[];
    metadata?: Record<string, unknown>;
  };
  session?: ValidationSession;
  metadata?: Record<string, unknown>;
}

interface ValidationResult {
  valid: boolean;
  redeemables: ValidationRedeemableResult[];
  order: Record<string, unknown>;
  tracking_id: string | null;
  session?: {
    key: string;
    type: string;
    ttl: number;
    ttl_unit: string;
  };
}

interface ValidationRedeemableResult {
  status: 'APPLICABLE' | 'INAPPLICABLE' | 'SKIPPED';
  id: string;
  object: 'voucher' | 'promotion_tier';
  order: Record<string, unknown>;
  result: Record<string, unknown>;
  applicable_to: { data: unknown[]; total: number; object: 'list' };
  inapplicable_to: { data: unknown[]; total: number; object: 'list' };
}
```

### 2) Validate Function

```typescript
async function validateStackableDiscounts(
  params: ValidateStackableParams
): Promise<ValidationResult> {
  return voucherifyRequest('/v1/validations', {
    method: 'POST',
    body: JSON.stringify(params)
  });
}
```

### 3) Validate + เก็บ session key

```typescript
async function validateAndGetSession(
  codes: string[],
  customerId: string,
  order: { amount: number; items: OrderItem[] }
): Promise<{ valid: boolean; sessionKey?: string; discountAmount?: number }> {
  const res = await validateStackableDiscounts({
    redeemables: codes.map(id => ({ object: 'voucher', id })),
    customer: { source_id: customerId },
    order: {
      amount: toVoucherifyAmount(order.amount),
      items: order.items
    },
    session: { type: 'LOCK', ttl: 30, ttl_unit: 'MINUTES' }
  });

  if (!res.valid) {
    return { valid: false };
  }

  const orderAfter = res.order as { total_discount_amount?: number };
  return {
    valid: true,
    sessionKey: res.session?.key,
    discountAmount: fromVoucherifyAmount(orderAfter.total_discount_amount ?? 0)
  };
}
```

### 4) Release Session ถ้า user ยกเลิก

```typescript
async function releaseValidationSession(sessionKey: string): Promise<void> {
  await voucherifyRequest(`/v1/sessions/${sessionKey}`, {
    method: 'DELETE'
  });
}
```

---

## ⚠️ ข้อควรระวัง

| เรื่อง | รายละเอียด |
|---|---|
| **Session Lock** | quota ถูก lock ชั่วคราว — ต้อง release ถ้า user ยกเลิก |
| **`valid: true` ≠ ทุก redeemable ใช้ได้** | เช็ค `status` ของแต่ละ redeemable ด้วย |
| **Session TTL** | ตั้งให้เหมาะสม — สั้นเกินทำให้ expire ก่อน user กด checkout |
| **amount หน่วย cent** | `order.amount: 15000` = 150 บาท |
| **ต้องส่ง session.key ตอน redeem** | เก็บ `session.key` ไว้ใช้ตอน POST /v1/redemptions |

---

## ✅ Checklist

```
[ ] ส่ง session: { type: "LOCK" } ทุกครั้ง
[ ] เก็บ session.key ไว้ใช้ตอน redeem
[ ] เช็ค valid: true ก่อนไป checkout
[ ] เช็ค status ของแต่ละ redeemable ด้วย
[ ] Release session ถ้า user ยกเลิก checkout
[ ] ตั้ง ttl ให้นานพอ (แนะนำ 30 นาที)
[ ] amount ทุก field หน่วยเป็น cent/satang
```

---

## 🔗 Links

- [Validate Stackable Discounts - Voucherify API Reference](https://docs.voucherify.io/api-reference/validations/validate-stackable-discounts)
- [Validation Object](https://docs.voucherify.io/api-reference/validations/validation-object)
- [Redeem Stackable Discounts](https://docs.voucherify.io/api-reference/redemptions/redeem-stackable-discounts)
- [Release Validation Session](https://docs.voucherify.io/api-reference/vouchers/release-validation-session)
- [Errors](https://docs.voucherify.io/api-reference/errors)
