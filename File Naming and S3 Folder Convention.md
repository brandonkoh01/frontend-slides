# File Naming and S3 Folder Convention

## Purpose

Defines the S3 bucket structure, folder layout, object key naming convention, and file naming standard for all uploads in the 3 POCs. 

---

## S3 Bucket Strategy

We’ll use a single bucket for the POCs. 

**Recommended bucket name:** `gr-poc-<aws-account-id>-ap-southeast-1`

---

## Folder Structure

```
gr-poc-{account-id}-ap-southeast-1/
├── dos/
│   └── {gr_id}/
│       └── {timestamp}_{sanitised_filename}.{ext}
│
├── attachments/
│   └── {gr_id}/
│       └── {attachment_id}_{sanitised_filename}.{ext}
```

**Prefix Definitions**

| Prefix | Purpose | Key format |
| --- | --- | --- |
| `dos/{gr_id}/` | Delivery Order upload — one per GR | `dos/{gr_id}/{timestamp}_{filename}.{ext}` |
| `attachments/{gr_id}/` | Supporting documents — zero or many per GR | `attachments/{gr_id}/{attachment_id}_{filename}.{ext}` |

**Key Component Rules**

| Component | Format | Example |
| --- | --- | --- |
| `{gr_id}` | UUID v4, lowercase with hyphens | `a1b2c3d4-5678-90ab-cdef-111122223333` |
| `{attachment_id}` | UUID v4, lowercase with hyphens | `b2c3d4e5-6789-01bc-defg-222233334444` |
| `{timestamp}` | ISO 8601 compact UTC: `YYYYMMDDTHHmmSSZ` | `20260602T061200Z` |
| `{sanitised_filename}` | Original filename, lowercased, spaces replaced with `_`, truncated to 100 chars | `delivery_order_june.pdf` |
| `{ext}` | Lowercase file extension | `pdf`, `png`, `jpeg` |

**Example Keys**

```
dos/a1b2c3d4-5678-90ab-cdef-111122223333/20260602T061200Z_delivery_order.pdf

attachments/a1b2c3d4-5678-90ab-cdef-111122223333/b2c3d4e5-6789-01bc-defg-222233334444_packing_list.pdf
```

---

## Allowed File Types

| Format | MIME Type |
| --- | --- |
| PDF | `application/pdf` |
| PNG | `image/png` |
| JPEG | `image/jpeg` |

File type validation must be enforced server-side using MIME type inspection, not solely by checking the file extension. 

---

## Design Decisions & Rationale

| Decision | Rationale |
| --- | --- |
| UUID as `{gr_id}` prefix | Reduces collision risk, aligns with `goods_receipt.id` PK in DB |
| `{attachment_id}` in attachment keys | Prevents overwrite when two users upload files with identical names within the same GR |
| Timestamp in key | Provides readable upload ordering without querying the database |
| Flat lowercase filenames with `_` | Avoids case-sensitivity trouble and URL encoding requirements for spaces |
| Two-prefix structure (`dos/`, `attachments/`) | Enables S3 bucket policy conditions on prefix (`s3:prefix`), supports independent lifecycle rules per object type |