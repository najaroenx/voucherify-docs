# Voucherify API — Validation Object

> 📄 Source: https://docs.voucherify.io/api-reference/validations/validation-object

---

## 📌 คืออะไร

**Validation Object** คือ response ที่ได้กลับมาจาก Validate Stackable Discounts endpoint  
บอกว่า redeemables ที่ส่งไป **valid ไหม** พร้อม session key สำหรับ lock quota ชั่วคราว

### Validation vs Qualification vs Redemption

| | Qualification | Validation | Redemption |
|---|---|---|---|
| **หัก quota** | ❌ | ❌ (แต่ lock) | ✅ |
| **session** | ❌ | ✅ lock session | ✅ consume |
| **Use case** | แสดง promotions | ยืนยันก่อน checkout | checkout จริง |

---

## 📦 Validation Object Structure

```json
{
  "valid": true,
  "redeemables": [
    {
      "status": "APPLICABLE",
      "id": "PROMO10",
      "object": "voucher",
      "order": {
        "amount": 15000,
        "discount_amount": 1500,
        "total_discount_amount": 1500,
        "total_amount": 13500,
        "applied_discount_amount": 1500
      },
      "applicable_to": {
        "data": [],
        "total": 0,
        "object": "list"
      },
      "inapplicable_to": {
        "data": [],
        "total": 0,
        "object": "list"
      },
      "result": {
        "discount": {
          "type": "PERCENT",
          "percent_off": 10,
          "effect": "APPLY_TO_ORDER"
        }
      }
    }
  ],
  "order": {
    "amount": 15000,
    "items_discount_amount": 0,
    "total_discount_amount": 1500,
    "total_amount": 13500,
    "items": [
      {
        "object": "order_item",
        "source_id": "SKU-001",
        "quantity": 2,
        "amount": 15000,
        "price": 7500
      }
    ]
  },
  "tracking_id": "track_abc123",
  "session": {
    "key": "session-checkout-001",
    "type": "LOCK",
    "ttl": 7,
    "ttl_unit": "DAYS"
  }
}
```

---

## 🔑 Fields Reference

### Top-level Fields

| Field | Type | คำอธิบาย |
|---|---|---|
| `valid` | `boolean` | `true` = redeemables ทั้งหมด valid |
| `redeemables` | `ValidationRedeemable[]` | ผลลัพธ์ต่อ redeemable |
| `order` | `object` | Order หลังคำนวณส่วนลดทั้งหมด |
| `tracking_id` | `string` | Tracking ID |
| `session` | `object` | Session lock info |

### Per-Redeemable Fields

| Field | Type | คำอธิบาย |
|---|---|---|
| `status` | `string` | `APPLICABLE`, `INAPPLICABLE`, `SKIPPED` |
| `id` | `string` | Voucher code หรือ promotion tier ID |
| `object` | `string` | `"voucher"` หรือ `"promotion_tier"` |
| `order` | `object` | Order หลังหัก redeemable นี้ |
| `result` | `object` | รายละเอียด discount ที่ได้ |
| `applicable_to` | `object` | items ที่ส่วนลดใช้ได้ |
| `inapplicable_to` | `object` | items ที่ส่วนลดใช้ไม่ได้ |

### Session Object

| Field | Type | คำอธิบาย |
|---|---|---|
| `key` | `string` | Session key (ใช้ตอน redeem) |
| `type` | `string` | `"LOCK"` — lock quota ชั่วคราว |
| `ttl` | `integer` | อายุ session |
| `ttl_unit` | `string` | `HOURS`, `DAYS`, `MINUTES` |

> ⚠️ **Session Lock** = quota ถูก lock ชั่วคราว ถ้าไม่ redeem ภายใน TTL จะ release อัตโนมัติ

---

## 🔗 Links

- [Validation Object - Voucherify API Reference](https://docs.voucherify.io/api-reference/validations/validation-object)
- [Validate Stackable Discounts](https://docs.voucherify.io/api-reference/validations/validate-stackable-discounts)
- [Redeem Stackable Discounts](https://docs.voucherify.io/api-reference/redemptions/redeem-stackable-discounts)
- [Release Validation Session](https://docs.voucherify.io/api-reference/vouchers/release-validation-session)
