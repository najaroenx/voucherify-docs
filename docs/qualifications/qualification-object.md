# Voucherify API — Qualification Object

> 📄 Source: https://docs.voucherify.io/api-reference/qualifications/qualification-object

---

## 📌 คืออะไร

**Qualification Object** คือ response ที่ได้กลับมาจาก Check Eligibility endpoint  
บอกว่า customer คนนี้ กับ order นี้ **มีสิทธิ์ใช้ redeemable ไหนบ้าง** พร้อมรายละเอียด discount ที่จะได้รับ

### Qualification vs Validation vs Redemption

| | Qualification | Validation | Redemption |
|---|---|---|---|
| **ทำอะไร** | เช็คว่า qualify ไหม | เช็ค + lock session | ใช้จริง |
| **หักจาก quota** | ❌ ไม่หัก | ❌ ไม่หัก | ✅ หัก |
| **Use case** | แสดง available promotions | ยืนยันก่อน checkout | checkout จริง |

---

## 📦 Qualification Object Structure

```json
{
  "redeemables": {
    "object": "list",
    "total": 3,
    "data": [
      {
        "id": "PROMO10",
        "object": "voucher",
        "created_at": "2024-05-01T00:00:00Z",
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
        "applicable_to": {
          "data": [],
          "total": 0,
          "object": "list"
        },
        "inapplicable_to": {
          "data": [],
          "total": 0,
          "object": "list"
        }
      }
    ]
  },
  "order": {
    "amount": 15000,
    "items_discount_amount": 0,
    "total_discount_amount": 0,
    "total_amount": 15000,
    "items": [
      {
        "object": "order_item",
        "product_id": "prod_001",
        "quantity": 2,
        "amount": 15000
      }
    ]
  },
  "tracking_id": "track_abc123"
}
```

---

## 🔑 Fields Reference

### Top-level Fields

| Field | Type | คำอธิบาย |
|---|---|---|
| `redeemables` | `object` | list ของ redeemables ที่ qualify |
| `redeemables.object` | `string` | `"list"` |
| `redeemables.total` | `integer` | จำนวน redeemables ที่ qualify |
| `redeemables.data` | `QualificationItem[]` | รายละเอียดแต่ละ redeemable |
| `order` | `object` | Order หลังคำนวณส่วนลด |
| `tracking_id` | `string` | Tracking ID ที่ส่งมาจาก request |

### Per-Redeemable Fields (ใน `redeemables.data`)

| Field | Type | คำอธิบาย |
|---|---|---|
| `id` | `string` | Voucher code หรือ promotion tier ID |
| `object` | `string` | `"voucher"` หรือ `"promotion_tier"` |
| `created_at` | `string` | เวลาสร้าง |
| `result` | `object` | ผลลัพธ์ที่จะได้ถ้า redeem — มี `discount` |
| `order` | `object` | Order หลังหักส่วนลดของ redeemable นี้ |
| `applicable_to` | `object` | items ที่ส่วนลดนี้ใช้ได้ |
| `inapplicable_to` | `object` | items ที่ส่วนลดนี้ใช้ไม่ได้ |

---

## 🔗 Links

- [Qualification Object - Voucherify API Reference](https://docs.voucherify.io/api-reference/qualifications/qualification-object)
- [Check Eligibility](https://docs.voucherify.io/api-reference/qualifications/check-eligibility)
- [Validation Object](https://docs.voucherify.io/api-reference/validations/validation-object)
