# Voucherify API — Create Publication (GET Method)

> 📄 Source: https://docs.voucherify.io/api-reference/publications/create-publication-with-get

---

## 📌 คืออะไร

**GET /v1/publications/create** — สร้าง publication โดยใช้ GET method แทน POST  
เหมาะสำหรับ **client-side integration** ที่ต้องการ embed ใน URL หรือ redirect flow  
ทำหน้าที่เหมือนกับ `POST /v1/publications` ทุกประการ แต่ส่ง parameters ผ่าน query string แทน

> ⚠️ **ใช้เฉพาะกรณีที่จำเป็น** — ปกติแนะนำใช้ `POST /v1/publications` แทนเพราะ secure กว่า

---

## 🔧 Endpoint

```
GET https://{cluster}.voucherify.io/v1/publications/create
```

---

## 📋 Query Parameters

| Parameter | Type | Required | คำอธิบาย |
|---|---|---|---|
| `customer` | `string` | ✅ | Customer source_id หรือ ID |
| `campaign` | `string` | ✅ (ถ้าไม่มี voucher) | ชื่อ campaign |
| `voucher` | `string` | ✅ (ถ้าไม่มี campaign) | Voucher code ที่ต้องการ assign |
| `channel` | `string` | - | ช่องทาง เช่น `api`, `email` |
| `metadata` | `string` (JSON) | - | Custom data (JSON encoded) |

---

## 📦 Response Structure

Response เหมือนกับ `POST /v1/publications` ทุกประการ:

```json
{
  "id": "pub_abc123",
  "object": "publication",
  "created_at": "2024-06-10T09:59:00Z",
  "customer_id": "cust_xyz789",
  "customer": {
    "id": "cust_xyz789",
    "source_id": "user-001",
    "object": "customer"
  },
  "voucher": {
    "id": "v_abc456",
    "code": "PROMO10",
    "object": "voucher"
  },
  "channel": "api",
  "result": "SUCCESS",
  "campaign": "Summer Campaign",
  "metadata": {}
}
```

---

## ⚡ cURL ตัวอย่าง

```bash
curl --request GET \
  --url 'https://{cluster}.voucherify.io/v1/publications/create?customer=user-001&campaign=Summer+Campaign&channel=api' \
  --header 'X-App-Id: YOUR_APP_ID' \
  --header 'X-App-Token: YOUR_APP_TOKEN'
```

---

## 🚀 Implementation Notes

### เมื่อไหร่ควรใช้ GET vs POST

| | POST /v1/publications | GET /v1/publications/create |
|---|---|---|
| **Security** | ✅ ดีกว่า — body ไม่ถูก log | ⚠️ params ถูก log ใน server log |
| **Use case** | server-side ทั่วไป | redirect URL, email link |
| **Parameters** | JSON body | query string |
| **แนะนำ** | ✅ ใช้งานทั่วไป | เฉพาะกรณีจำเป็น |

### TypeScript Function

```typescript
async function createPublicationWithGet(params: {
  customer: string;
  campaign?: string;
  voucher?: string;
  channel?: string;
}): Promise<Publication> {
  const query = new URLSearchParams();
  query.set('customer', params.customer);
  if (params.campaign) query.set('campaign', params.campaign);
  if (params.voucher) query.set('voucher', params.voucher);
  if (params.channel) query.set('channel', params.channel);

  return voucherifyRequest(`/v1/publications/create?${query.toString()}`);
}
```

---

## ⚠️ ข้อควรระวัง

| เรื่อง | รายละเอียด |
|---|---|
| **Parameters ถูก log** | query string ปรากฏใน server/proxy logs — อย่าใส่ข้อมูล sensitive |
| **ใช้ POST แทนได้เสมอ** | ถ้าไม่มีความจำเป็นพิเศษ ให้ใช้ POST |
| **URL encoding** | ต้อง encode campaign name ที่มีช่องว่าง เช่น `Summer+Campaign` |

---

## ✅ Checklist

```
[ ] ใช้ POST แทนถ้าเป็น server-side call
[ ] Encode query parameters ให้ถูกต้อง
[ ] Handle 409 Conflict กรณี publish ซ้ำ
[ ] ตรวจสอบ result === 'SUCCESS'
[ ] อย่าใส่ข้อมูล sensitive ใน query params
```

---

## 🔗 Links

- [Create Publication (GET) - Voucherify API Reference](https://docs.voucherify.io/api-reference/publications/create-publication-with-get)
- [Create Publication (POST)](https://docs.voucherify.io/api-reference/publications/create-publication)
- [Publication Object](https://docs.voucherify.io/api-reference/publications/publication-object)
- [Errors](https://docs.voucherify.io/api-reference/errors)
