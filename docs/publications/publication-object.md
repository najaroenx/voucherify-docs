# Voucherify API — Publication Object

> 📄 Source: https://docs.voucherify.io/api-reference/publications/publication-object

---

## 📌 คืออะไร

**Publication** คือ event ที่เกิดขึ้นเมื่อ **assign voucher ให้กับ customer** คนใดคนหนึ่ง  
พูดง่าย ๆ คือ "การแจก voucher code ให้ลูกค้า" — 1 publication = 1 voucher ที่ถูก assign ให้ 1 customer

### Publication ต่างจาก Redemption ยังไง?

| | Publication | Redemption |
|---|---|---|
| **คืออะไร** | แจก voucher ให้ customer | customer ใช้ voucher นั้น |
| **เกิดเมื่อ** | ก่อนใช้งาน | ตอนใช้งานจริง (checkout) |
| **ผลลัพธ์** | customer ได้รับ code | ได้รับส่วนลด |

---

## 📦 Publication Object Structure

```json
{
  "id": "pub_abc123",
  "object": "publication",
  "created_at": "2024-06-10T09:59:00Z",
  "updated_at": null,
  "channel": "email",
  "result": "SUCCESS",
  "customer_id": "cust_xyz789",
  "customer": {
    "id": "cust_xyz789",
    "source_id": "user-001",
    "name": "John Doe",
    "email": "john@example.com",
    "object": "customer"
  },
  "voucher": {
    "id": "v_abc456",
    "code": "PROMO10",
    "discount": {
      "type": "PERCENT",
      "percent_off": 10
    },
    "active": true,
    "object": "voucher"
  },
  "metadata": {
    "source": "web",
    "campaign_ref": "summer-2024"
  }
}
```

---

## 🔑 Fields Reference

| Field | Type | คำอธิบาย |
|---|---|---|
| `id` | `string` | Unique ID ของ publication |
| `object` | `string` | จะเป็น `"publication"` เสมอ |
| `created_at` | `string` | เวลาที่ publish (ISO 8601) |
| `updated_at` | `string\|null` | เวลาที่ update ล่าสุด |
| `result` | `string` | `SUCCESS` หรือ `FAILURE` |
| `channel` | `string` | ช่องทาง เช่น `email`, `sms`, `api` |
| `customer_id` | `string` | Voucherify customer ID |
| `customer` | `object` | Customer ที่ได้รับ voucher |
| `voucher` | `object` | Voucher ที่ถูก assign |
| `metadata` | `object` | Custom data เพิ่มเติม |

---

## 🔗 Links

- [Publication Object - Voucherify API Reference](https://docs.voucherify.io/api-reference/publications/publication-object)
- [List Publications](https://docs.voucherify.io/api-reference/publications/list-publications)
- [Create Publication](https://docs.voucherify.io/api-reference/publications/create-publication)
