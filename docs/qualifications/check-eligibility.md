# Voucherify API — Check Eligibility

> 📄 Source: https://docs.voucherify.io/api-reference/qualifications/check-eligibility

---

## 📌 คืออะไร

**POST /v1/qualifications** — เช็คว่า customer + order นี้มีสิทธิ์ใช้ voucher / promotion ไหนบ้าง  
ใช้ก่อน checkout เพื่อแสดง available discounts ให้ user เห็น **โดยไม่หัก quota**

### เมื่อไหร่ควรใช้

| Use case | Endpoint |
|---|---|
| แสดง promotions ที่ใช้ได้ในหน้า cart | ✅ Check Eligibility |
| ยืนยัน code ก่อน apply | Validation |
| ใช้จริงตอน checkout | Redemption |

---

## 🔧 Endpoint

```
POST https://{cluster}.voucherify.io/v1/qualifications
```

---

## 📋 Request Body Fields

| Field | Type | Required | คำอธิบาย |
|---|---|---|---|
| `customer` | `object` | ✅ | Customer ที่ต้องการเช็ค |
| `order` | `object` | ✅ | Order / cart ที่ต้องการเช็ค |
| `redeemables` | `array` | ❌ | ระบุ redeemables ที่ต้องการเช็คเฉพาะ (ถ้าไม่ส่ง = เช็คทั้งหมดที่ qualify) |
| `scenario` | `string` | ❌ | `ALL`, `CUSTOMER_WALLET`, `AUDIENCE_ONLY`, `PRODUCTS_ONLY` |
| `mode` | `string` | ❌ | `BASIC` หรือ `ADVANCED` |
| `tracking_id` | `string` | ❌ | tracking สำหรับ analytics |
| `metadata` | `object` | ❌ | custom data เพิ่มเติม |

### `scenario` Values

| Value | คำอธิบาย |
|---|---|
| `ALL` | เช็คทุก redeemables ที่ qualify (default) |
| `CUSTOMER_WALLET` | เช็คเฉพาะ vouchers ใน wallet ของ customer |
| `AUDIENCE_ONLY` | เช็คเฉพาะ promotions ที่ customer เข้าเงื่อนไข |
| `PRODUCTS_ONLY` | เช็คเฉพาะ promotions ที่เกี่ยวกับ products ใน order |

---

## 📦 Request Body ตัวอย่าง

### เช็ค promotions ทั้งหมดที่ใช้ได้

```json
{
  "customer": {
    "source_id": "user-001",
    "email": "john@example.com"
  },
  "order": {
    "source_id": "order-from-my-system-001",
    "amount": 15000,
    "items": [
      {
        "source_id": "SKU-001",
        "quantity": 2,
        "price": 5000,
        "amount": 10000
      }
    ]
  },
  "scenario": "ALL",
  "mode": "ADVANCED"
}
```

### เช็ค redeemables เฉพาะที่ระบุ

```json
{
  "customer": { "source_id": "user-001" },
  "order": { "amount": 15000 },
  "redeemables": [
    { "object": "voucher", "id": "PROMO10" },
    { "object": "voucher", "id": "FREESHIP" },
    { "object": "promotion_tier", "id": "promo_tier_abc" }
  ]
}
```

---

## 📦 Response Structure

```json
{
  "redeemables": {
    "object": "list",
    "total": 2,
    "data": [
      {
        "id": "PROMO10",
        "object": "voucher",
        "result": {
          "discount": {
            "type": "PERCENT",
            "percent_off": 10
          }
        },
        "order": {
          "amount": 15000,
          "discount_amount": 1500,
          "total_amount": 13500
        },
        "applicable_to": { "data": [], "total": 0, "object": "list" },
        "inapplicable_to": { "data": [], "total": 0, "object": "list" }
      }
    ]
  },
  "order": {
    "amount": 15000,
    "total_discount_amount": 0,
    "total_amount": 15000
  },
  "tracking_id": "track_abc123"
}
```

---

## ⚡ cURL ตัวอย่าง

```bash
curl --request POST \
  --url https://{cluster}.voucherify.io/v1/qualifications \
  --header 'X-App-Id: YOUR_APP_ID' \
  --header 'X-App-Token: YOUR_APP_TOKEN' \
  --header 'Content-Type: application/json' \
  --data '{
    "customer": { "source_id": "user-001" },
    "order": {
      "amount": 15000,
      "items": [
        { "source_id": "SKU-001", "quantity": 2, "price": 5000, "amount": 10000 }
      ]
    },
    "scenario": "ALL",
    "mode": "ADVANCED"
  }'
```

---

## 🚀 Implementation Notes

### 1) TypeScript Interfaces

```typescript
type QualificationScenario =
  | 'ALL'
  | 'CUSTOMER_WALLET'
  | 'AUDIENCE_ONLY'
  | 'PRODUCTS_ONLY';

interface CheckEligibilityParams {
  customer: { id?: string; source_id?: string; email?: string };
  order: {
    source_id?: string;
    amount: number;
    items?: OrderItem[];
  };
  redeemables?: Array<{ object: 'voucher' | 'promotion_tier'; id: string }>;
  scenario?: QualificationScenario;
  mode?: 'BASIC' | 'ADVANCED';
  tracking_id?: string;
  metadata?: Record<string, unknown>;
}

interface QualificationRedeemableItem {
  id: string;
  object: 'voucher' | 'promotion_tier';
  created_at: string;
  result: {
    discount?: {
      type: string;
      percent_off?: number;
      amount_off?: number;
    };
  };
  order: {
    amount: number;
    discount_amount: number;
    total_amount: number;
  };
  applicable_to: { data: unknown[]; total: number; object: 'list' };
  inapplicable_to: { data: unknown[]; total: number; object: 'list' };
}

interface CheckEligibilityResponse {
  redeemables: {
    object: 'list';
    total: number;
    data: QualificationRedeemableItem[];
  };
  order: Record<string, unknown>;
  tracking_id: string | null;
}
```

### 2) Check Eligibility Function

```typescript
async function checkEligibility(
  params: CheckEligibilityParams
): Promise<CheckEligibilityResponse> {
  return voucherifyRequest('/v1/qualifications', {
    method: 'POST',
    body: JSON.stringify(params)
  });
}
```

### 3) แสดง Available Promotions ใน Cart

```typescript
async function getAvailablePromotions(params: {
  customerId: string;
  cartAmount: number;
  items: OrderItem[];
}): Promise<QualificationRedeemableItem[]> {
  const res = await checkEligibility({
    customer: { source_id: params.customerId },
    order: {
      amount: toVoucherifyAmount(params.cartAmount),
      items: params.items.map(item => ({
        source_id: item.sku,
        quantity: item.quantity,
        price: toVoucherifyAmount(item.price),
        amount: toVoucherifyAmount(item.price * item.quantity)
      }))
    },
    scenario: 'ALL',
    mode: 'ADVANCED'
  });

  return res.redeemables.data;
}
```

### 4) เช็คว่า Code ที่ User กรอกใช้ได้ไหม

```typescript
async function isCodeEligible(
  code: string,
  customerId: string,
  cartAmount: number
): Promise<boolean> {
  const res = await checkEligibility({
    customer: { source_id: customerId },
    order: { amount: toVoucherifyAmount(cartAmount) },
    redeemables: [{ object: 'voucher', id: code }]
  });

  return res.redeemables.total > 0;
}
```

---

## ⚠️ ข้อควรระวัง

| เรื่อง | รายละเอียด |
|---|---|
| **ไม่หัก quota** | Check Eligibility ไม่ทำให้ voucher ถูกใช้ — ต้อง redeem จริงเมื่อ checkout |
| **ใช้ `mode: ADVANCED`** | ได้ข้อมูลละเอียดกว่า BASIC |
| **`scenario: ALL` อาจช้า** | ถ้า campaign เยอะ ควรระบุ redeemables ที่ต้องการเช็คเฉพาะ |
| **result ≠ ผล redeem จริง** | Qualification ไม่ได้ lock — ผลอาจเปลี่ยนได้ก่อน redeem |

---

## ✅ Checklist

```
[ ] ใช้ Check Eligibility เพื่อแสดง promotions ให้ user เห็น
[ ] ใช้ mode: ADVANCED เพื่อได้ข้อมูลครบ
[ ] ระบุ scenario ที่เหมาะสมเพื่อลด latency
[ ] อย่าใช้ผล Qualification แทน Validation
[ ] ต้อง Validate + Redeem จริงตอน checkout
```

---

## 🔗 Links

- [Check Eligibility - Voucherify API Reference](https://docs.voucherify.io/api-reference/qualifications/check-eligibility)
- [Qualification Object](https://docs.voucherify.io/api-reference/qualifications/qualification-object)
- [Validation Object](https://docs.voucherify.io/api-reference/validations/validation-object)
- [Stackable Redemptions](https://docs.voucherify.io/api-reference/redemptions/stackable-redemptions-object)
