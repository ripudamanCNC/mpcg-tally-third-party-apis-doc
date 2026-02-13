# Tally Integration API - Client & Third-Party Guide

## 📋 Table of Contents
1. [Overview](#overview)
2. [Quick Start](#quick-start)
3. [Authentication](#authentication)
4. [Invoice Integration](#invoice-integration)
5. [Response Structures](#response-structures)
6. [Common Use Cases](#common-use-cases)
7. [Error Handling](#error-handling)
8. [Best Practices](#best-practices)

---

## Overview

### Purpose
This API enables seamless integration between your accounting system (Tally) and the MPC Medical Platform invoice management system. It provides secure access to invoice data with all necessary fields for accounting, GST compliance, and financial reporting.

### Key Features
- ✅ Secure token-based authentication (1-year validity)
- ✅ Comprehensive invoice data with customer, payment, and GST details
- ✅ Advanced filtering (date range, status, branch, invoice type)
- ✅ Export tracking to prevent duplicate imports
- ✅ CSV and JSON export formats
- ✅ Support for full, partial, and top-up invoices
- ✅ Complete HSN code mapping for GST compliance

### Base URL
```
Development: http://mpc.codesncoffee.com
```

---

## Quick Start

### Step 1: Authenticate
```bash
curl -X POST "https://mpc.codesncoffee.com/api/tally/auth/login" \
  -H "Content-Type: application/json" \
  -d '{
    "username": "your_tally_username",
    "password": "your_secure_password"
  }'
```

### Step 2: Get Invoices
```bash
curl -X GET "https://mpc.codesncoffee.com/api/invoices/tally/integration?fromDate=2026-02-01&toDate=2026-02-13&status=UPLOADED&includeExported=false" \
  -H "Authorization: Bearer YOUR_TOKEN_HERE"
```

### Step 3: Export to CSV
```bash
curl -X GET "https://mpc.codesncoffee.com/api/invoices/tally/export?fromDate=2026-02-01&toDate=2026-02-13&status=UPLOADED" \
  -H "Authorization: Bearer YOUR_TOKEN_HERE" \
  --output invoices.csv
```

---

## Authentication

### 1️⃣ Login - Get Access Token

**Endpoint:** `POST /api/tally/auth/login`

**Description:** Authenticate to receive a bearer token valid for 1 year.

**Request:**
```bash
curl -X POST "https://mpc.codesncoffee.com/api/tally/auth/login" \
  -H "Content-Type: application/json" \
  -d '{
    "username": "tally_user",
    "password": "SecurePassword123!"
  }'
```

**Response Structure:**
```json
{
  "success": true,
  "message": "Login successful",
  "data": {
    "user": {
      "uid": 1,
      "username": "tally_user",
      "email": "tally@example.com",
      "block": 0,
      "branch_ids": [1, 2, 3],
      "permissions": ["VIEW_INVOICES", "EXPORT_INVOICES"]
    },
    "access_token": "tly_1a2b3c4d5e6f7g8h9i0j",
    "token_expires_at": "2027-02-13T10:30:00.000Z"
  }
}
```

**Important Fields:**
- `access_token`: Use this in all subsequent API calls as Bearer token
- `token_expires_at`: Token validity (1 year from login)
- `branch_ids`: Array of branch IDs you have access to

---

### 2️⃣ Refresh Token (Optional)

**Endpoint:** `POST /api/tally/auth/refresh`

**Description:** Extend token expiry without re-login.

**Request:**
```bash
curl -X POST "https://mpc.codesncoffee.com/api/tally/auth/refresh" \
  -H "Authorization: Bearer tly_1a2b3c4d5e6f7g8h9i0j"
```

**Response:**
```json
{
  "success": true,
  "message": "Token refreshed successfully",
  "data": {
    "access_token": "tly_9z8y7x6w5v4u3t2s1r0q",
    "token_expires_at": "2028-02-13T10:30:00.000Z"
  }
}
```

---

### 3️⃣ Get Profile

**Endpoint:** `GET /api/tally/auth/profile`

**Request:**
```bash
curl -X GET "https://mpc.codesncoffee.com/api/tally/auth/profile" \
  -H "Authorization: Bearer tly_1a2b3c4d5e6f7g8h9i0j"
```

**Response:**
```json
{
  "success": true,
  "data": {
    "uid": 1,
    "username": "tally_user",
    "email": "tally@example.com",
    "branch_ids": [1, 2, 3],
    "permissions": ["VIEW_INVOICES", "EXPORT_INVOICES"],
    "created_at": "2026-01-01T00:00:00.000Z"
  }
}
```

---

### 4️⃣ Logout

**Endpoint:** `POST /api/tally/auth/logout`

**Request:**
```bash
curl -X POST "https://mpc.codesncoffee.com/api/tally/auth/logout" \
  -H "Authorization: Bearer tly_1a2b3c4d5e6f7g8h9i0j"
```

**Response:**
```json
{
  "success": true,
  "message": "Logged out successfully"
}
```

---

## Invoice Integration

### 🔍 Get Invoices (JSON Format)

**Endpoint:** `GET /api/invoices/tally/integration`

**Description:** Retrieve structured invoice data with all fields needed for accounting and GST compliance.

#### Query Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `fromDate` | string (YYYY-MM-DD) | No | - | Start date for filtering invoices |
| `toDate` | string (YYYY-MM-DD) | No | - | End date for filtering invoices |
| `invoiceNumber` | string | No | - | Filter by invoice number (supports wildcards: `INV-2026-*`) |
| `status` | string | No | `UPLOADED` | Invoice status: `PENDING`, `GENERATED`, `UPLOADED`, `FAILED` |
| `invoiceType` | string | No | - | Invoice type: `FULL`, `PARTIAL`, `TOPUP` |
| `branchId` | number | No | - | Filter by specific branch ID |
| `limit` | number | No | `100` | Results per page (max recommended: 500) |
| `page` | number | No | `1` | Page number for pagination |
| `includeExported` | string | No | `false` | Include already exported invoices: `true` or `false` |

#### Example Request

**Get today's new invoices:**
```bash
curl -X GET "https://mpc.codesncoffee.com/api/invoices/tally/integration?fromDate=2026-02-13&toDate=2026-02-13&status=UPLOADED&includeExported=false" \
  -H "Authorization: Bearer tly_1a2b3c4d5e6f7g8h9i0j"
```

**Get all invoices for February 2026:**
```bash
curl -X GET "https://mpc.codesncoffee.com/api/invoices/tally/integration?fromDate=2026-02-01&toDate=2026-02-28&status=UPLOADED&limit=500" \
  -H "Authorization: Bearer tly_1a2b3c4d5e6f7g8h9i0j"
```

**Search specific invoice pattern:**
```bash
curl -X GET "https://mpc.codesncoffee.com/api/invoices/tally/integration?invoiceNumber=INV-2026-0001*" \
  -H "Authorization: Bearer tly_1a2b3c4d5e6f7g8h9i0j"
```

**Get branch-specific invoices:**
```bash
curl -X GET "https://mpc.codesncoffee.com/api/invoices/tally/integration?branchId=1&fromDate=2026-02-01&toDate=2026-02-13" \
  -H "Authorization: Bearer tly_1a2b3c4d5e6f7g8h9i0j"
```

---

### 📥 Export Invoices (CSV Format)

**Endpoint:** `GET /api/invoices/tally/export`

**Description:** Export invoices in Tally-compatible CSV format and optionally mark them as exported.

#### Query Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `fromDate` | string (YYYY-MM-DD) | No | - | Start date for filtering |
| `toDate` | string (YYYY-MM-DD) | No | - | End date for filtering |
| `invoiceNumber` | string | No | - | Filter by invoice number |
| `status` | string | No | `UPLOADED` | Filter by status |
| `invoiceType` | string | No | - | Filter by type |
| `branchId` | number | No | - | Filter by branch |
| `markAsExported` | string | No | `true` | Mark as exported: `true` or `false` |

#### Example Requests

**Export and mark as exported (recommended for daily sync):**
```bash
curl -X GET "https://mpc.codesncoffee.com/api/invoices/tally/export?fromDate=2026-02-13&toDate=2026-02-13&status=UPLOADED&markAsExported=true" \
  -H "Authorization: Bearer tly_1a2b3c4d5e6f7g8h9i0j" \
  --output invoices_2026-02-13.csv
```

**Preview export without marking:**
```bash
curl -X GET "https://mpc.codesncoffee.com/api/invoices/tally/export?fromDate=2026-02-01&toDate=2026-02-28&markAsExported=false" \
  -H "Authorization: Bearer tly_1a2b3c4d5e6f7g8h9i0j" \
  --output preview_feb_2026.csv
```

**Re-export with already exported invoices:**
```bash
curl -X GET "https://mpc.codesncoffee.com/api/invoices/tally/export?fromDate=2026-02-01&toDate=2026-02-28&includeExported=true&markAsExported=false" \
  -H "Authorization: Bearer tly_1a2b3c4d5e6f7g8h9i0j" \
  --output re_export_feb_2026.csv
```

#### CSV Format Structure
```csv
Invoice Number,Invoice Date,Type,Customer Name,Phone,Email,Subtotal,Tax,Total,Paid,Pending,Payment Method,Transaction ID,Branch,Service,Status,Exported At
"INV-2026-000123","13-Feb-2026","FULL","John Doe","+91 9876543210","john@example.com","5000","900","5900","5900","0","UPI","TXN123456","Main Clinic","Physiotherapy","UPLOADED","13-Feb-2026 10:30"
```

---

## Response Structures

### 📊 Invoice Integration Response (JSON)

#### Complete Response Structure
```json
{
  "success": true,
  "message": "Invoices for Tally integration retrieved successfully",
  "data": {
    "invoices": [
      {
        "invoice_id": 123,
        "invoice_number": "INV-2026-000123",
        "invoice_date": "2026-02-13T10:30:00.000Z",
        "invoice_type": "FULL",
        "customer": {
          "user_id": 456,
          "name": "John Doe",
          "phone": "+91 9876543210",
          "email": "john.doe@example.com",
          "address": "123 Sample Street, Apartment 4B",
          "city": "Mumbai",
          "region": "Maharashtra"
        },
        "company": {
          "name": "Global BodyFix My Pain Clinic LLP",
          "address": "Unit B-1, V. N. Sphere Mall, Linking Road, Bandra West, Mumbai, Maharashtra",
          "phone": "+91 9189400905",
          "email": "contact@mypainclinicglobal.com",
          "gstin": "27ABCDE1234F1Z5"
        },
        "branch": {
          "branch_id": 1,
          "branch_name": "Main Clinic - Mumbai Bandra"
        },
        "amounts": {
          "subtotal": 5000.00,
          "tax_amount": 900.00,
          "total_amount": 5900.00,
          "amount_paid": 5900.00,
          "pending_amount": 0.00
        },
        "items": [
          {
            "description": "Physiotherapy Package - 10 Sessions",
            "sessions": "10 sessions",
            "price": "Rs. 500.00",
            "amount": 5000.00,
            "hsn_code": "998314",
            "gst_percent": 18
          }
        ],
        "payment": {
          "payment_method": "UPI",
          "transaction_id": "TXN123456789",
          "cart_id": 789,
          "payment_id": 101
        },
        "service_description": "Physiotherapy Package",
        "remarks": "Full payment completed. Thank you for your business!",
        "bank": {
          "name": "HDFC Bank",
          "branch": "Bandra (West) Linking Road",
          "account": "50200059739211",
          "ifsc": "HDFC0001234"
        },
        "s3_pdf_url": "https://s3.amazonaws.com/bucket/invoices/INV-2026-000123.pdf",
        "tally_exported_at": null,
        "tally_exported_by": null,
        "status": "UPLOADED",
        "generated_at": "2026-02-13T10:30:05.000Z"
      }
    ],
    "summary": {
      "total_invoices": 50,
      "total_amount": 295000.00,
      "total_paid": 250000.00,
      "total_pending": 45000.00,
      "exported_count": 0,
      "pending_export_count": 50
    }
  },
  "meta": {
    "total": 50,
    "limit": 100,
    "page": 1,
    "paginated": true,
    "filters": {
      "fromDate": "2026-02-13",
      "toDate": "2026-02-13",
      "invoiceNumber": null,
      "status": "UPLOADED",
      "invoiceType": null,
      "branchId": null,
      "includeExported": "false"
    }
  }
}
```

#### Field Descriptions

##### Invoice Object Fields

| Field | Type | Description |
|-------|------|-------------|
| `invoice_id` | number | Unique invoice ID in the system |
| `invoice_number` | string | Human-readable invoice number (e.g., INV-2026-000123) |
| `invoice_date` | string (ISO 8601) | Invoice generation timestamp |
| `invoice_type` | string | Type: `FULL`, `PARTIAL`, or `TOPUP` |
| `status` | string | Status: `PENDING`, `GENERATED`, `UPLOADED`, or `FAILED` |
| `generated_at` | string (ISO 8601) | When invoice was generated |
| `s3_pdf_url` | string | Direct URL to PDF invoice document |
| `service_description` | string | Summary of services provided |
| `remarks` | string | Additional notes and payment details |
| `tally_exported_at` | string/null | Timestamp when exported to Tally (null if not exported) |
| `tally_exported_by` | number/null | User ID who exported to Tally |

##### Customer Object

| Field | Type | Description |
|-------|------|-------------|
| `user_id` | number | Unique customer ID |
| `name` | string | Customer full name |
| `phone` | string | Phone with country code |
| `email` | string | Customer email address |
| `address` | string | Customer street address |
| `city` | string | Customer city |
| `region` | string | Customer state/region |

##### Company Object

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Company legal name |
| `address` | string | Company registered address |
| `phone` | string | Company contact number |
| `email` | string | Company email |
| `gstin` | string | GST Identification Number |

##### Amounts Object

| Field | Type | Description |
|-------|------|-------------|
| `subtotal` | number | Amount before tax |
| `tax_amount` | number | Total GST amount |
| `total_amount` | number | Grand total (subtotal + tax) |
| `amount_paid` | number | Amount paid in this invoice |
| `pending_amount` | number | Remaining balance (0 for FULL invoices) |

##### Items Array

| Field | Type | Description |
|-------|------|-------------|
| `description` | string | Service/product description |
| `sessions` | string | Session details (e.g., "10 sessions") |
| `price` | string | Unit price formatted |
| `amount` | number | Line item total |
| `hsn_code` | string | HSN/SAC code for GST |
| `gst_percent` | number | GST percentage applied |

##### Payment Object

| Field | Type | Description |
|-------|------|-------------|
| `payment_method` | string | Payment mode (UPI, Cash, Card, etc.) |
| `transaction_id` | string | Payment transaction reference |
| `cart_id` | number | Associated cart ID |
| `payment_id` | number | Payment record ID |

##### Summary Object

| Field | Type | Description |
|-------|------|-------------|
| `total_invoices` | number | Total invoices matching filters |
| `total_amount` | number | Sum of all invoice totals |
| `total_paid` | number | Sum of all payments |
| `total_pending` | number | Sum of all pending amounts |
| `exported_count` | number | Number of already exported invoices |
| `pending_export_count` | number | Number of never-exported invoices |

---

### 🧾 Partial Invoice Response Structure

For **partial payment invoices**, additional fields provide complete payment context:

```json
{
  "invoice_id": 124,
  "invoice_number": "INV-2026-000124",
  "invoice_date": "2026-02-13T14:30:00.000Z",
  "invoice_type": "PARTIAL",
  
  "customer": {
    "user_id": 457,
    "name": "Jane Smith",
    "phone": "+91 9876543211",
    "email": "jane.smith@example.com",
    "address": "456 Test Road",
    "city": "Delhi",
    "region": "Delhi"
  },
  
  "company": {
    "name": "Global BodyFix My Pain Clinic LLP",
    "gstin": "27ABCDE1234F1Z5"
  },
  
  "amounts": {
    "subtotal": 1694.92,
    "tax_amount": 305.08,
    "total_amount": 2000.00,
    "amount_paid": 2000.00,
    "pending_amount": 0.00
  },
  
  "cart_context": {
    "cart_total": 10000.00,
    "total_paid_so_far": 5000.00,
    "cart_pending_amount": 5000.00,
    "current_payment": 2000.00
  },
  
  "items": [
    {
      "description": "Physiotherapy Package",
      "sessions": "50 sessions",
      "price": "Rs. 200.00/session",
      "amount": 1694.92,
      "hsn_code": "998314",
      "gst_percent": 18
    }
  ],
  
  "full_cart_items": [
    {
      "description": "Physiotherapy Package",
      "sessions": "50 sessions",
      "price": "Rs. 200.00/session",
      "amount": "Rs. 10,000.00",
      "hsnCode": "998314",
      "gstPercent": 18
    }
  ],
  
  "payment": {
    "payment_method": "Cash",
    "transaction_id": "CASH-TXN-789",
    "cart_id": 890,
    "payment_id": 202
  },
  
  "remarks": "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━\n🧾 PARTIAL PAYMENT RECEIPT #2\n━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━\n\nCURRENT PAYMENT ALLOCATION:\n  • Physiotherapy Package: Rs.2000.00 (100%)\n\n━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━\n📊 CART SUMMARY\n━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━\nTotal Cart Value: Rs.10,000.00\nThis Payment: Rs.2,000.00\nTotal Paid So Far: Rs.5,000.00\nRemaining Balance: Rs.5,000.00\nPayment Progress: 50.0%",
  
  "s3_pdf_url": "https://s3.amazonaws.com/bucket/invoices/INV-2026-000124.pdf",
  "tally_exported_at": null,
  "status": "UPLOADED"
}
```

#### Understanding Partial Invoice Fields

| Field | Description | Example Value |
|-------|-------------|---------------|
| `amounts.total_amount` | Amount paid in THIS invoice only | 2000.00 |
| `amounts.pending_amount` | Always 0 (this invoice is fully paid) | 0.00 |
| `cart_context.cart_total` | Full cart/package total | 10000.00 |
| `cart_context.current_payment` | This payment amount | 2000.00 |
| `cart_context.total_paid_so_far` | Cumulative payments | 5000.00 |
| `cart_context.cart_pending_amount` | Remaining cart balance | 5000.00 |
| `items[].amount` | Proportionally allocated amount for this payment | 1694.92 |
| `full_cart_items` | Original cart items with full amounts | Array |
| `remarks` | Complete payment history with visual breakdown | String |

#### Tally Import Guidelines for Partial Invoices

✅ **Import Each Partial Invoice as Standalone Sale**
- Each partial invoice is fully paid (pending_amount = 0)
- Import to Sales ledger with actual payment amount
- Use HSN codes for accurate GST accounting

✅ **Use Cart Context for Customer Account Reconciliation**
- Track multiple partial payments against same cart_id
- Use remarks for complete audit trail
- Reference cart_context for payment progress

✅ **GST Compliance**
- Use hsn_code and gst_percent from items array
- Tax amount is correctly calculated for this payment
- Each partial invoice is a valid GST transaction

---

## Common Use Cases

### Use Case 1: Daily Invoice Sync (Recommended)

**Scenario:** Sync today's new invoices at end of business day

```bash
# Step 1: Get today's date
TODAY=$(date +%Y-%m-%d)

# Step 2: Fetch new invoices (not yet exported)
curl -X GET "https://mpc.codesncoffee.com/api/invoices/tally/integration?fromDate=${TODAY}&toDate=${TODAY}&status=UPLOADED&includeExported=false" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  --output daily_invoices_${TODAY}.json

# Step 3: Export to CSV and mark as exported
curl -X GET "https://mpc.codesncoffee.com/api/invoices/tally/export?fromDate=${TODAY}&toDate=${TODAY}&status=UPLOADED&markAsExported=true" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  --output daily_invoices_${TODAY}.csv
```

**Benefits:**
- ✅ Prevents duplicate imports
- ✅ Maintains daily backup in JSON format
- ✅ Automated tracking of export status

---

### Use Case 2: Monthly Reconciliation

**Scenario:** Export all invoices for month-end accounting

```bash
# February 2026 invoices
curl -X GET "https://mpc.codesncoffee.com/api/invoices/tally/integration?fromDate=2026-02-01&toDate=2026-02-28&status=UPLOADED&limit=500" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  --output feb_2026_invoices.json

# Export to CSV for Tally import
curl -X GET "https://mpc.codesncoffee.com/api/invoices/tally/export?fromDate=2026-02-01&toDate=2026-02-28&status=UPLOADED" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  --output feb_2026_tally.csv
```

**Use Cases:**
- Monthly GST reconciliation
- Financial reporting
- Audit trail generation

---

### Use Case 3: Branch-wise Reports

**Scenario:** Generate invoices for specific branch

```bash
# Branch ID 1 invoices for February
curl -X GET "https://mpc.codesncoffee.com/api/invoices/tally/integration?branchId=1&fromDate=2026-02-01&toDate=2026-02-28&status=UPLOADED" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  --output branch_1_feb_2026.json
```

**Requirements:**
- Your Tally user must have access to the branch ID
- Use `branch_ids` from your profile response to check access

---

### Use Case 4: Search and Re-export Specific Invoices

**Scenario:** Find and re-export specific invoice(s) without marking as exported

```bash
# Search for invoices matching pattern
curl -X GET "https://mpc.codesncoffee.com/api/invoices/tally/integration?invoiceNumber=INV-2026-0001*" \
  -H "Authorization: Bearer YOUR_TOKEN"

# Re-export without marking (for corrections)
curl -X GET "https://mpc.codesncoffee.com/api/invoices/tally/export?invoiceNumber=INV-2026-000123&markAsExported=false" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  --output corrected_invoice.csv
```

**Use Cases:**
- Invoice corrections
- Customer queries
- Audit requirements

---

### Use Case 5: Automated Backup Script

**Complete bash script for daily automated sync:**

```bash
#!/bin/bash

# Configuration
API_URL="https://mpc.codesncoffee.com"
TOKEN="YOUR_TALLY_TOKEN_HERE"
BACKUP_DIR="/path/to/backups"
TODAY=$(date +%Y-%m-%d)

# Create backup directory
mkdir -p "${BACKUP_DIR}/${TODAY}"

# Fetch JSON backup
echo "Fetching invoice data..."
curl -X GET "${API_URL}/api/invoices/tally/integration?fromDate=${TODAY}&toDate=${TODAY}&status=UPLOADED&includeExported=false" \
  -H "Authorization: Bearer ${TOKEN}" \
  --output "${BACKUP_DIR}/${TODAY}/invoices.json"

# Export CSV and mark as exported
echo "Exporting to CSV..."
curl -X GET "${API_URL}/api/invoices/tally/export?fromDate=${TODAY}&toDate=${TODAY}&status=UPLOADED&markAsExported=true" \
  -H "Authorization: Bearer ${TOKEN}" \
  --output "${BACKUP_DIR}/${TODAY}/invoices.csv"

echo "Backup completed: ${BACKUP_DIR}/${TODAY}/"
```

**Setup:**
```bash
# Make executable
chmod +x daily_sync.sh

# Add to crontab (run daily at 11:59 PM)
crontab -e
# Add line: 59 23 * * * /path/to/daily_sync.sh
```

---

## Error Handling

### Common Error Responses

#### 401 Unauthorized
```json
{
  "success": false,
  "message": "Authentication required",
  "error": "UNAUTHORIZED"
}
```
**Solution:** Check your Bearer token, login again if expired.

---

#### 403 Forbidden
```json
{
  "success": false,
  "message": "Access denied to branch",
  "error": "FORBIDDEN"
}
```
**Solution:** Verify your `branch_ids` from profile endpoint.

---

#### 400 Bad Request
```json
{
  "success": false,
  "message": "Invalid date format",
  "error": "VALIDATION_ERROR"
}
```
**Solution:** Use YYYY-MM-DD format for dates.

---

#### 404 Not Found
```json
{
  "success": false,
  "message": "No invoices found matching criteria",
  "data": {
    "invoices": [],
    "summary": {
      "total_invoices": 0,
      "total_amount": 0
    }
  }
}
```
**Note:** This is not an error - no invoices match your filters.

---

#### 500 Internal Server Error
```json
{
  "success": false,
  "message": "Internal server error",
  "error": "SERVER_ERROR"
}
```
**Solution:** Contact API support team.

---

### Error Handling Best Practices

```bash
# Example with error handling
response=$(curl -s -w "\n%{http_code}" -X GET "https://mpc.codesncoffee.com/api/invoices/tally/integration?fromDate=2026-02-13&toDate=2026-02-13" \
  -H "Authorization: Bearer YOUR_TOKEN")

http_code=$(echo "$response" | tail -n1)
body=$(echo "$response" | head -n-1)

if [ "$http_code" -eq 200 ]; then
    echo "Success: $body"
    echo "$body" > invoices.json
elif [ "$http_code" -eq 401 ]; then
    echo "Error: Authentication failed. Please login again."
elif [ "$http_code" -eq 403 ]; then
    echo "Error: Access denied. Check branch permissions."
else
    echo "Error: HTTP $http_code - $body"
fi
```

---

## Best Practices

### 🔒 Security

1. **Token Storage:**
   - Store tokens securely (encrypted storage, environment variables)
   - Never commit tokens to version control
   - Use environment variables in scripts:
     ```bash
     export TALLY_TOKEN="tly_your_token_here"
     curl -H "Authorization: Bearer ${TALLY_TOKEN}" ...
     ```

2. **Token Rotation:**
   - Tokens valid for 1 year
   - Set reminder to refresh 1 week before expiry
   - Use refresh endpoint to extend without re-login

3. **HTTPS Only:**
   - Always use HTTPS in production
   - Verify SSL certificates in API calls

---

### 📊 Data Management

1. **Daily Sync Schedule:**
   - Run sync at end of business day (11:00 PM - 11:59 PM)
   - Use `includeExported=false` to fetch only new invoices
   - Always use `markAsExported=true` in daily sync

2. **Pagination:**
   - Default limit: 100 invoices
   - Maximum recommended: 500 invoices per page
   - For large date ranges, use pagination:
     ```bash
     # Page 1
     curl "...&limit=500&page=1" ...
     # Page 2
     curl "...&limit=500&page=2" ...
     ```

3. **Backup Strategy:**
   - Keep JSON backups for 90 days minimum
   - Store CSV exports for audit trail
   - Archive monthly for long-term compliance

---

### 🔍 Filtering Best Practices

1. **Date Range:**
   - Use specific date ranges instead of large periods
   - For month-end: Use exact month boundaries
   - For daily sync: Use current date only

2. **Status Filter:**
   - Always use `status=UPLOADED` for production data
   - Only include other statuses for debugging

3. **Branch Filter:**
   - Use `branchId` for multi-branch accounting
   - Verify access via profile endpoint first

---

### 📈 Performance Optimization

1. **Limit Results:**
   - Use appropriate `limit` based on your needs
   - Avoid fetching all invoices in single call

2. **Export Tracking:**
   - Use `includeExported=false` to reduce payload
   - Mark exports properly to avoid reprocessing

3. **Caching:**
   - Cache invoice data locally after fetch
   - Don't re-fetch the same date range multiple times

---

### ✅ GST Compliance

1. **HSN Code Mapping:**
   - All items include `hsn_code` field
   - Map to appropriate GST categories in Tally
   - Common codes:
     - `998314`: Medical and physiotherapy services (18% GST)

2. **Tax Calculation:**
   - `tax_amount` is pre-calculated
   - `gst_percent` provided per item
   - Use for GST return filing

3. **GSTIN Verification:**
   - Company GSTIN in `company.gstin`
   - Customer GSTIN (if B2B) in customer details

---

### 🔄 Integration Workflow

**Recommended Daily Workflow:**
```
1. Authenticate (once per day or reuse token)
   ↓
2. Fetch new invoices (JSON for backup)
   ↓
3. Export to CSV with markAsExported=true
   ↓
4. Import CSV to Tally
   ↓
5. Verify import success
   ↓
6. Archive backup files
```

**Monthly Workflow:**
```
1. Export full month data (all invoices including exported)
   ↓
2. Reconcile with Tally ledger
   ↓
3. Generate GST reports
   ↓
4. Archive for compliance
```

---

## Invoice Types Reference

### FULL Invoice
- Complete payment for entire cart/package
- `amounts.pending_amount` = 0
- Single invoice for complete transaction

### PARTIAL Invoice
- Part payment of larger cart/package
- Multiple invoices for same cart_id
- Includes `cart_context` for full picture
- Each invoice is fully paid (its own pending = 0)

### TOPUP Invoice
- Wallet credit/recharge invoice
- No service items
- Payment mode recorded

---

## Support & Contact

### Technical Support
- **Email:** ripudaman@codesncoffee.com
- **Response Time:** 24-48 hours
- **Emergency:** Contact your account manager

### API Issues
- Check API status page
- Verify token validity
- Review error messages carefully

### Feature Requests
- Submit via support email
- Include use case description
- Expected timeline discussion

---

## Appendix

### A. Invoice Status Values

| Status | Description | Action Required |
|--------|-------------|-----------------|
| `PENDING` | Invoice created but PDF not generated | Wait for generation |
| `GENERATED` | PDF created but not uploaded | Wait for upload |
| `UPLOADED` | Ready for integration | **Use for Tally import** |
| `FAILED` | Generation/upload failed | Contact support |

### B. Payment Methods

Common payment method values:
- `UPI` - Unified Payments Interface
- `Cash` - Cash payment
- `Card` - Credit/Debit card
- `Net Banking` - Direct bank transfer
- `PineLabs` - Payment gateway transactions

### C. HSN/SAC Codes

Common codes for medical services:
- `998314` - Physiotherapy services
- `999293` - Consultation services
- `999296` - Other paramedical services

### D. Date Format Reference

**Required Format:** `YYYY-MM-DD`

**Examples:**
- ✅ Correct: `2026-02-13`
- ❌ Wrong: `13-02-2026`, `02/13/2026`, `2026/02/13`

**Timezone:** All dates in UTC (ISO 8601 format)

---

## Quick Reference Card

### Essential Endpoints

| Operation | Endpoint | Method |
|-----------|----------|--------|
| Login | `/api/tally/auth/login` | POST |
| Get Invoices (JSON) | `/api/invoices/tally/integration` | GET |
| Export CSV | `/api/invoices/tally/export` | GET |
| Refresh Token | `/api/tally/auth/refresh` | POST |
| Profile | `/api/tally/auth/profile` | GET |
| Logout | `/api/tally/auth/logout` | POST |

### Key Parameters

| Parameter | Purpose | Example |
|-----------|---------|---------|
| `fromDate` | Start date filter | `2026-02-01` |
| `toDate` | End date filter | `2026-02-13` |
| `status` | Invoice status | `UPLOADED` |
| `includeExported` | Include already exported | `false` |
| `markAsExported` | Mark as exported | `true` |
| `limit` | Results per page | `100` |

### Authentication Header
```
Authorization: Bearer tly_your_token_here
```

---

## Changelog

### Version 1.0 - February 2026
- Initial release
- Full, Partial, and Top-up invoice support
- CSV and JSON export formats
- Export tracking functionality
- Branch-based access control
- HSN code integration for GST compliance

---

**Document Version:** 1.0  
**Last Updated:** February 13, 2026  
**API Version:** 1.0  
**Prepared For:** Tally Integration Clients & Third Parties
