# Storage & Data Lifecycle Policy

## 1. Object Storage (S3 Compatible)

Kita pake S3-Compatible storage (Cloudflare R2 atau DigitalOcean Spaces) buat ngejar **Zero Egress Fee**.

**Rekomendasi Provider:**
| Provider | Egress Fee | Storage Cost | Recommendation |
|----------|------------|--------------|----------------|
| **Cloudflare R2** | **$0/GB** | $0.015/GB/mo | ✅ **Primary Choice** |
| DigitalOcean Spaces | $0/GB (within DO ecosystem) | $5/100GB/mo | Alternative if hosting on DO |
| AWS S3 | $0.09/GB | $0.023/GB/mo | ❌ Avoid (egress fee mahal) |

---

## 2. Bucket Structure & Hierarchy

Kita bagi bucket berdasarkan **Urgency**, **Privacy**, dan **Access Pattern**:

| Folder / Prefix | Isi | Access Level | Retention (Age) |
|-----------------|-----|--------------|-----------------|
| `/profiles` | Foto Profil Jagoan & Tasker | Public-Read | Selamanya (atau sampai akun didelete) |
| `/jobs-evidence` | Foto Sebelum/Sesudah Kerja | Presigned-URL (15 min expiry) | **30 Hari** setelah job selesai |
| `/disputes` | Bukti Sengketa (Sangat Sensitif) | Restricted/Private | **1 Tahun** (Setelah itu Hard Delete) |
| `/kyc-vault` | Foto KTP & Selfie (Enkripsi!) | Super-Private (AES-256) | Sesuai regulasi / Cold Storage |
| `/invoices` | PDF Invoice & Transaksi | Presigned-URL | 1 tahun → Glacier |
| `/logs` | System & Audit Logs | Private | 7 hari Standard → 30 hari Glacier |

---

## 3. Data Retention Rules

### 3.1 Automatic Cleanup Policy

| Data Type | Standard Storage | Archival (Glacier) | Deletion Policy |
|-----------|------------------|--------------------|-----------------|
| Job Photos (`/jobs-evidence`) | 30 Days | N/A | Hard Delete |
| Dispute Evidence (`/disputes`) | 1 Year | N/A | Hard Delete + Secure Wipe |
| Profile Photos (`/profiles`) | Indefinite | N/A | Delete on account deletion |
| KYC Documents (`/kyc-vault`) | 1 Year | 7 Years (Compliance) | Secure Delete after 7 years |
| Invoices (`/invoices`) | 1 Year | 6 Years (Glacier) | Delete after 7 years |
| System Logs (`/logs`) | 7 Days | 30 Days | Auto Cleanup |

### 3.2 S3 Lifecycle Rules Configuration

**Rule 1: Job Evidence Cleanup**
```json
{
  "Rule": {
    "ID": "DeleteJobEvidenceAfter30Days",
    "Prefix": "jobs-evidence/",
    "Status": "Enabled",
    "Expiration": { "Days": 30 },
    "NoncurrentVersionExpiration": { "NoncurrentDays": 1 }
  }
}
```

**Rule 2: Dispute Evidence Cleanup**
```json
{
  "Rule": {
    "ID": "DeleteDisputeEvidenceAfter1Year",
    "Prefix": "disputes/",
    "Status": "Enabled",
    "Expiration": { "Days": 365 },
    "NoncurrentVersionExpiration": { "NoncurrentDays": 1 }
  }
}
```

**Rule 3: Invoice Archival**
```json
{
  "Rule": {
    "ID": "ArchiveInvoicesAfter1Year",
    "Prefix": "invoices/",
    "Status": "Enabled",
    "Transitions": [
      {
        "Days": 365,
        "StorageClass": "GLACIER_INSTANT_RETRIEVAL"
      }
    ],
    "Expiration": { "Days": 2555 }
  }
}
```

**Rule 4: Log Rotation**
```json
{
  "Rule": {
    "ID": "LogRotation",
    "Prefix": "logs/",
    "Status": "Enabled",
    "Transitions": [
      { "Days": 7, "StorageClass": "GLACIER" }
    ],
    "Expiration": { "Days": 30 }
  }
}
```

---

## 4. Privacy & Security

### 4.1 Access Control

| Folder | Public | Presigned URL | IAM Policy |
|--------|--------|---------------|------------|
| `/profiles` | ✅ Read | ❌ | `s3:GetObject` for public |
| `/jobs-evidence` | ❌ | ✅ 15 min expiry | `s3:GetObject` restricted |
| `/disputes` | ❌ | ✅ 15 min expiry (jury only) | `s3:GetObject` restricted |
| `/kyc-vault` | ❌ | ❌ | `s3:GetObject` + decryption key |
| `/invoices` | ❌ | ✅ 15 min expiry | `s3:GetObject` restricted |

### 4.2 Presigned URL Requirements

Semua file di `/jobs-evidence`, `/disputes`, dan `/invoices` **WAJIB** diakses lewat **Presigned URL** dengan:

- **Expiry Time:** 15 menit maximum
- **HTTP Method:** GET only
- **IP Restriction:** Optional (untuk dispute evidence)
- **User Identity:** Logged in JWT required untuk generate

**Contoh Generate Presigned URL (Rust):**
```rust
use aws_sdk_s3::presigning::PresigningConfig;
use std::time::Duration;

async fn generate_evidence_presigned_url(
    s3_client: &s3::Client,
    bucket: &str,
    key: &str,
) -> Result<String> {
    let presign_config = PresigningConfig::builder()
        .expires_in(Duration::from_secs(900)) // 15 menit
        .build();
    
    let presigned = s3_client
        .get_object()
        .bucket(bucket)
        .key(key)
        .presigned(presign_config)
        .await?;
    
    Ok(presigned.uri().to_string())
}
```

### 4.3 Encryption Requirements

**At-Rest Encryption:**
- Semua file: AES-256 (S3 managed keys)
- `/kyc-vault`: Customer-Managed Keys (CMK) via AWS KMS / HashiCorp Vault

**In-Transit Encryption:**
- TLS 1.3 mandatory untuk semua S3 requests
- Certificate pinning di Flutter App

---

## 5. Cost Optimization

### 5.1 Image Compression (Client-Side)

**Flutter App Requirements:**
- **Max Resolution:** 1920x1080 (Full HD) untuk evidence foto
- **Max File Size:** 500KB - 1MB per image
- **Format:** JPEG quality 80% atau WebP
- **Compression:** Lakukan sebelum upload (bukan di server)

**Contoh Flutter Compression:**
```dart
import 'package:image_cropper/image_cropper.dart';
import 'package:image_compression/image_compression.dart';

Future<File> compressEvidenceImage(File imageFile) async {
  // Crop & resize first
  final cropped = await ImageCropper().cropImage(
    sourcePath: imageFile.path,
    maxWidth: 1920,
    maxHeight: 1080,
    compressQuality: 80,
  );
  
  // Compress to target size
  final compressed = await ImageCompression.compressWithFile(
    File(cropped!.path),
    targetSize: 500 * 1024, // 500KB
  );
  
  return compressed;
}
```

### 5.2 Upload Strategy

**Multipart Upload:**
- Trigger untuk file > 5MB
- Chunk size: 5MB per part
- Parallel upload: 3 concurrent parts

**Retry Logic:**
- Exponential backoff: 1s, 2s, 4s, 8s, 16s
- Max retries: 5
- Circuit breaker: Fail fast setelah 3 consecutive failures

### 5.3 Egress Fee Mitigation

**Strategi:**
1. **Cloudflare R2** untuk zero egress fee (prioritas utama)
2. **CDN Caching** untuk `/profiles` (public assets)
3. **Presigned URL Caching** di client (15 min cache)
4. **Bandwidth Monitoring** dengan alert threshold $50/bulan

---

## 6. Implementation Checklist

### 6.1 Backend (Rust)

- [ ] S3 client initialization dengan Cloudflare R2 endpoint
- [ ] Presigned URL generation untuk `/jobs-evidence`, `/disputes`, `/invoices`
- [ ] File upload endpoint dengan multipart support
- [ ] Image metadata stripping (EXIF removal untuk privacy)
- [ ] S3 Lifecycle Rules deployment via Terraform/CloudFormation

### 6.2 Frontend (Flutter)

- [ ] Image compression sebelum upload
- [ ] EXIF data removal (GPS location stripping)
- [ ] Multipart upload untuk large files
- [ ] Retry logic dengan exponential backoff
- [ ] Presigned URL caching (15 min)

### 6.3 Infrastructure

- [ ] S3 bucket creation dengan versioning enabled
- [ ] Lifecycle rules configuration
- [ ] CORS policy untuk Flutter App
- [ ] Bucket policy untuk public `/profiles`
- [ ] CloudWatch alarm untuk storage cost monitoring

---

## 7. Disaster Recovery

### 7.1 Backup Strategy

| Data Type | Backup Frequency | Retention | Recovery Point Objective |
|-----------|------------------|-----------|--------------------------|
| `/profiles` | Daily snapshot | 30 days | 24 hours |
| `/jobs-evidence` | None (ephemeral) | N/A | N/A (can be re-uploaded) |
| `/disputes` | Weekly snapshot | 90 days | 7 days |
| `/kyc-vault` | Daily snapshot + Cross-region | 7 years | 1 hour |
| `/invoices` | Daily snapshot | 7 years | 24 hours |

### 7.2 Data Recovery Procedure

**Scenario: Accidental Deletion**
1. Check S3 Versioning → restore previous version
2. If versioning disabled → restore from daily snapshot
3. If snapshot unavailable → request user re-upload (untuk ephemeral data)

**Scenario: Bucket Corruption**
1. Failover to cross-region replica (untuk `/kyc-vault`)
2. Restore from latest snapshot
3. Verify integrity dengan checksum validation

---

## 8. Compliance & Audit

### 8.1 Data Sovereignty

- Semua data disimpan di region **Indonesia (Jakarta)** untuk compliance
- Cross-border transfer: prohibited untuk KYC data
- Audit log: semua akses ke `/kyc-vault` dan `/disputes` dicatat

### 8.2 Right to Erasure (GDPR-like)

**User Account Deletion Flow:**
1. Delete `/profiles/{user_id}/*` immediately
2. Soft-delete `/jobs-evidence` (30 days grace period)
3. Hard-delete `/jobs-evidence` setelah 30 days
4. Retain `/kyc-vault` sesuai regulasi (7 years)
5. Anonymize `/disputes` (remove PII, retain evidence)

---

## 9. Monitoring & Alerting

### 9.1 Metrics to Track

| Metric | Threshold | Alert Channel |
|--------|-----------|---------------|
| Storage Growth Rate | > 10GB/hour | Slack #infra-alerts |
| Egress Cost | > $50/month | Email + Slack |
| Failed Upload Rate | > 1% | PagerDuty |
| Presigned URL Error Rate | > 5% | Slack #infra-alerts |

### 9.2 Cost Monitoring Dashboard

**Daily Report:**
- Total storage per prefix
- Egress bandwidth per prefix
- Estimated monthly cost projection
- Top 10 users by storage usage

**Weekly Report:**
- Storage growth trend
- Cost per user segment
- Lifecycle rule effectiveness (GB saved)

---

## 10. Related Documents

- [`TECHNICAL.md`](../TECHNICAL.md) - Technical specification overview
- [`TECHNICAL.md#security-architecture`](../TECHNICAL.md#10-security-architecture) - Security requirements
- [`TECHNICAL.md#infrastructure`](../TECHNICAL.md#11-infrastructure--deployment) - Deployment guidelines

---

**Document Control:**
- **Author:** Core Technical Team
- **Version:** 1.0
- **Date:** 2026-03-16
- **Next Review:** 2026-06-16

---

*End of Document*
