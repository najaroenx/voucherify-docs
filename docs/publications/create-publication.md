# Voucherify API — Create Publication

> 📄 Source: https://docs.voucherify.io/api-reference/publications/create-publication

---

## 📌 คืออะไร

**POST /v1/publications** — แจก voucher ให้ customer (assign voucher code ให้ลูกค้า)  
1 call = 1 voucher ที่ถูก assign ให้ 1 customer จาก campaign ที่กำหนด

> 💡 ต้องระบุอย่างน้อย `customer` + (`campaign` หรือ `voucher`) อย่างใดอย่างหนึ่ง

---

## 🔧 Endpoint

```
POST https://{cluster}.voucherify.io/v1/publications
```

---

## 📋 Request Body Fields

| Field | Type | Required | คำอธิบาย |
|---|---|---|---|
| `customer` | `object` | ✅ | Customer ที่จะได้รับ voucher |
| `customer.source_id` | `string` | ✅ (แนะนำ) | ID จากระบบของคุณ |
| `customer.id` | `string` | - | Voucherify customer ID |
| `customer.email` | `string` | - | อีเมล customer |
| `campaign` | `object` | ✅ (ถ้าไม่มี voucher) | Campaign ที่จะดึง voucher จาก |
| `campaign.name` | `string` | ✅ | ชื่อ campaign |
| `campaign.count` | `integer` | - | จำนวน voucher (default: 1) |
| `voucher` | `string` | ✅ (ถ้าไม่มี campaign) | Voucher code ที่ต้องการ assign โดยตรง |
| `channel` | `string` | - | ช่องทาง เช่น `api`, `email`, `sms` |
| `metadata` | `object` | - | Custom data เพิ่มเติม |

---

## 📦 Request Body ตัวอย่าง

### Publish จาก Campaign

```json
{
  "customer": {
    "source_id": "user-001",
    "name": "John Doe",
    "email": "john@example.com"
  },
  "campaign": {
    "name": "Summer Campaign",
    "count": 1
  },
  "channel": "api",
  "metadata": {
    "source": "checkout-page"
  }
}
```

### Publish Voucher Code โดยตรง

```json
{
  "customer": {
    "source_id": "user-001"
  },
  "voucher": "PROMO10",
  "channel": "api"
}
```

---

## 📦 Response Structure

```json
{
  "id": "pub_abc123",
  "object": "publication",
  "created_at": "2024-06-10T09:59:00Z",
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
  "channel": "api",
  "result": "SUCCESS",
  "campaign": "Summer Campaign",
  "metadata": {
    "source": "checkout-page"
  }
}
```

---

## ⚡ cURL ตัวอย่าง

```bash
curl --request POST \
  --url https://{cluster}.voucherify.io/v1/publications \
  --header 'X-App-Id: YOUR_APP_ID' \
  --header 'X-App-Token: YOUR_APP_TOKEN' \
  --header 'Content-Type: application/json' \
  --data '{
    "customer": {
      "source_id": "user-001",
      "email": "john@example.com"
    },
    "campaign": {
      "name": "Summer Campaign",
      "count": 1
    },
    "channel": "api",
    "metadata": {
      "source": "web"
    }
  }'
```

---

## 🚀 Implementation Notes

### 1) TypeScript Interfaces

```typescript
interface CreatePublicationPayload {
  customer: {
    source_id?: string;
    id?: string;
    name?: string;
    email?: string;
  };
  campaign?: {
    name: string;
    count?: number;
  };
  voucher?: string;
  channel?: string;
  metadata?: Record<string, unknown>;
}

interface Publication {
  id: string;
  object: 'publication';
  created_at: string;
  customer_id: string;
  customer: Record<string, unknown>;
  voucher: {
    id: string;
    code: string;
    object: 'voucher';
  };
  channel: string;
  result: 'SUCCESS' | 'FAILURE';
  campaign: string | null;
  metadata: Record<string, unknown>;
}
```

### 2) Publish Voucher Function

```typescript
async function publishVoucher(
  payload: CreatePublicationPayload
): Promise<Publication> {
  const res: Publication = await voucherifyRequest('/v1/publications', {
    method: 'POST',
    body: JSON.stringify(payload)
  });

  if (res.result !== 'SUCCESS') {
    throw new Error(`Publication failed for customer ${payload.customer.source_id}`);
  }

  return res;
}
```

### 3) Get or Publish — ป้องกัน duplicate

```typescript
async function getOrPublishVoucher(
  customerId: string,
  campaignName: string
): Promise<string> {
  // เช็คก่อนว่า customer มี voucher จาก campaign นี้แล้วไหม
  const existing = await voucherifyRequest(
    `/v1/publications?customer=${encodeURIComponent(customerId)}&campaign=${encodeURIComponent(campaignName)}&result=SUCCESS&limit=1`
  );

  if (existing.publications?.length > 0) {
    return existing.publications[0].voucher.code;
  }

  // ยังไม่มี → publish ใหม่
  const publication = await publishVoucher({
    customer: { source_id: customerId },
    campaign: { name: campaignName },
    channel: 'api'
  });

  return publication.voucher.code;
}
```

---

## ⚠️ ข้อควรระวัง

| เรื่อง | รายละเอียด |
|---|---|
| **1 customer ต่อ 1 campaign** | publish ซ้ำจะได้ error 409 — เช็คก่อน หรือใช้ get-or-publish |
| **campaign หรือ voucher อย่างน้อย 1** | ถ้าไม่ระบุเลยจะ error |
| **เก็บ voucher.code จาก response** | code นี้คือสิ่งที่ต้องส่งให้ user |
| **channel ควรระบุเสมอ** | ช่วยให้ audit trail ชัดเจน |

---

## ✅ Checklist

```
[ ] ระบุ customer.source_id จากระบบของคุณ
[ ] ระบุ campaign.name หรือ voucher อย่างใดอย่างหนึ่ง
[ ] Handle 409 Conflict กรณี publish ซ้ำ
[ ] เก็บ voucher.code จาก response เพื่อส่งให้ user
[ ] ตรวจสอบ result === 'SUCCESS' ก่อนใช้ข้อมูล
[ ] ระบุ channel ให้ถูกต้อง
```

---

## 🔗 Links

- [Create Publication - Voucherify API Reference](https://docs.voucherify.io/api-reference/publications/create-publication)
- [List Publications](https://docs.voucherify.io/api-reference/publications/list-publications)
- [Publication Object](https://docs.voucherify.io/api-reference/publications/publication-object)
- [Errors](https://docs.voucherify.io/api-reference/errors)
