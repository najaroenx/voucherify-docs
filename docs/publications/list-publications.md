# Voucherify API — List Publications

> 📄 Source: https://docs.voucherify.io/api-reference/publications/list-publications

---

## 📌 คืออะไร

**GET /v1/publications** — ดึงรายการ publications ทั้งหมด พร้อม filter และ pagination  
Publication คือ event ที่ voucher ถูก assign ให้ customer — endpoint นี้ใช้ดูประวัติการแจก voucher ทั้งหมด

---

## 🔧 Endpoint

```
GET https://{cluster}.voucherify.io/v1/publications
```

---

## 📋 Query Parameters

### Pagination

| Parameter | Type | Default | Max | คำอธิบาย |
|---|---|---|---|---|
| `limit` | `integer` | `25` | `100` | จำนวน publications ต่อ page |
| `page` | `integer` | `1` | - | หน้าที่ต้องการ |

### Sorting

| Parameter | Type | คำอธิบาย |
|---|---|---|
| `order` | `string` | เช่น `created_at`, `-created_at` (- = descending) |

### Filtering

| Parameter | Type | คำอธิบาย |
|---|---|---|
| `customer` | `string` | filter ตาม customer ID หรือ source_id |
| `campaign` | `string` | filter ตามชื่อ campaign |
| `code` | `string` | filter ตาม voucher code |
| `result` | `string` | `SUCCESS` หรือ `FAILURE` |
| `created_at[gt]` | `string` | filter publications หลังจากวันที่นี้ (ISO 8601) |
| `created_at[lt]` | `string` | filter publications ก่อนวันที่นี้ (ISO 8601) |

---

## 📦 Response Structure

```json
{
  "object": "list",
  "total": 34,
  "data_ref": "publications",
  "publications": [
    {
      "id": "pub_abc123",
      "object": "publication",
      "created_at": "2024-03-22T12:58:28Z",
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
        "object": "voucher"
      },
      "channel": "api",
      "result": "SUCCESS",
      "campaign": "Summer Campaign",
      "metadata": {}
    }
  ]
}
```

| Field | Type | คำอธิบาย |
|---|---|---|
| `object` | `string` | จะเป็น `"list"` เสมอ |
| `total` | `integer` | จำนวน publications ทั้งหมดที่ตรงกับ filter |
| `data_ref` | `string` | จะเป็น `"publications"` — ชื่อ key ที่เก็บ array |
| `publications` | `Publication[]` | รายการ publication objects |

---

## ⚡ cURL ตัวอย่าง

### ดึง publications ทั้งหมด (หน้าแรก)

```bash
curl --request GET \
  --url 'https://{cluster}.voucherify.io/v1/publications?limit=20&page=1' \
  --header 'X-App-Id: YOUR_APP_ID' \
  --header 'X-App-Token: YOUR_APP_TOKEN'
```

### Filter ตาม customer

```bash
curl --request GET \
  --url 'https://{cluster}.voucherify.io/v1/publications?customer=user-001&limit=20' \
  --header 'X-App-Id: YOUR_APP_ID' \
  --header 'X-App-Token: YOUR_APP_TOKEN'
```

### Filter ตาม campaign + result

```bash
curl --request GET \
  --url 'https://{cluster}.voucherify.io/v1/publications?campaign=Summer+Campaign&result=SUCCESS&limit=20' \
  --header 'X-App-Id: YOUR_APP_ID' \
  --header 'X-App-Token: YOUR_APP_TOKEN'
```

### Filter ตาม date range

```bash
curl --request GET \
  --url 'https://{cluster}.voucherify.io/v1/publications?created_at%5Bgt%5D=2024-06-01T00%3A00%3A00Z&created_at%5Blt%5D=2024-08-31T23%3A59%3A59Z&limit=100' \
  --header 'X-App-Id: YOUR_APP_ID' \
  --header 'X-App-Token: YOUR_APP_TOKEN'
```

---

## 🚀 Implementation Notes

### 1) TypeScript Interfaces

```typescript
interface ListPublicationsParams {
  limit?: number;
  page?: number;
  order?: string;
  customer?: string;
  campaign?: string;
  code?: string;
  result?: 'SUCCESS' | 'FAILURE';
  createdAtGt?: string;
  createdAtLt?: string;
}

interface PublicationCustomer {
  id: string;
  source_id: string;
  name?: string;
  email?: string;
  object: 'customer';
}

interface PublicationVoucher {
  id: string;
  code: string;
  object: 'voucher';
}

interface Publication {
  id: string;
  object: 'publication';
  created_at: string;
  customer_id: string;
  customer: PublicationCustomer;
  voucher: PublicationVoucher;
  channel: string;
  result: 'SUCCESS' | 'FAILURE';
  campaign: string | null;
  metadata: Record<string, unknown>;
}

interface ListPublicationsResponse {
  object: 'list';
  total: number;
  data_ref: 'publications';
  publications: Publication[];
}
```

### 2) List Publications Function

```typescript
async function listPublications(
  params: ListPublicationsParams = {}
): Promise<ListPublicationsResponse> {
  const query = new URLSearchParams();

  if (params.limit) query.set('limit', String(params.limit));
  if (params.page) query.set('page', String(params.page));
  if (params.order) query.set('order', params.order);
  if (params.customer) query.set('customer', params.customer);
  if (params.campaign) query.set('campaign', params.campaign);
  if (params.code) query.set('code', params.code);
  if (params.result) query.set('result', params.result);
  if (params.createdAtGt) query.set('created_at[gt]', params.createdAtGt);
  if (params.createdAtLt) query.set('created_at[lt]', params.createdAtLt);

  return voucherifyRequest(`/v1/publications?${query.toString()}`);
}
```

### 3) Get All Publications ของ Customer (Pagination Loop)

```typescript
async function getAllPublicationsForCustomer(
  customerId: string
): Promise<Publication[]> {
  const allPublications: Publication[] = [];
  let page = 1;
  const limit = 100;

  while (true) {
    const res = await listPublications({
      customer: customerId,
      limit,
      page,
      order: '-created_at'
    });

    allPublications.push(...res.publications);

    if (res.publications.length < limit) break;
    page++;
  }

  return allPublications;
}
```

### 4) เช็คว่า Customer เคยได้ Voucher จาก Campaign นี้แล้วไหม

```typescript
async function hasCustomerReceivedVoucher(
  customerId: string,
  campaignName: string
): Promise<boolean> {
  const res = await listPublications({
    customer: customerId,
    campaign: campaignName,
    result: 'SUCCESS',
    limit: 1
  });

  return res.total > 0;
}
```

### 5) ดึง Voucher Code ที่ Customer ได้รับล่าสุดจาก Campaign

```typescript
async function getLatestVoucherForCustomer(
  customerId: string,
  campaignName: string
): Promise<string | null> {
  const res = await listPublications({
    customer: customerId,
    campaign: campaignName,
    result: 'SUCCESS',
    limit: 1,
    order: '-created_at'
  });

  if (res.publications.length === 0) return null;
  return res.publications[0].voucher.code;
}
```

---

## ⚠️ ข้อควรระวัง

| เรื่อง | รายละเอียด |
|---|---|
| **`data_ref` = `"publications"`** | ต้องอ่าน `response.publications` ไม่ใช่ `response.data` |
| **`total` ≠ จำนวนที่ได้ใน page** | `total` คือทั้งหมดที่ match filter |
| **date filter ต้อง encode** | `created_at[gt]` → `created_at%5Bgt%5D` ใน URL |
| **limit สูงสุด 100** | ถ้าต้องการมากกว่าต้องทำ pagination loop |
| **order ค่า default** | ถ้าไม่ระบุ order อาจได้ผลลัพธ์ไม่ consistent — ระบุเสมอ |

---

## ✅ Checklist การใช้ List Publications

```
[ ] อ่าน response.publications ไม่ใช่ response.data
[ ] ระบุ order parameter เสมอเพื่อผลลัพธ์ consistent
[ ] ใช้ limit=100 เพื่อลด API calls เมื่อ fetch ทั้งหมด
[ ] Handle pagination ด้วย page++ จนกว่า length < limit
[ ] Encode date filter parameters ให้ถูกต้องใน URL
[ ] Handle rate limit (429) ถ้า loop ดึงข้อมูลจำนวนมาก
[ ] ใช้ result=SUCCESS เพื่อกรองเฉพาะที่สำเร็จ
```

---

## 🔗 Links

- [List Publications - Voucherify API Reference](https://docs.voucherify.io/api-reference/publications/list-publications)
- [Publication Object](https://docs.voucherify.io/api-reference/publications/publication-object)
- [Create Publication](https://docs.voucherify.io/api-reference/publications/create-publication)
- [Listing & Pagination](https://docs.voucherify.io/api-reference/listing)
- [Errors](https://docs.voucherify.io/api-reference/errors)
