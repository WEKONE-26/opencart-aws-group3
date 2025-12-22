# 🎯 D2 COMPLETE IMPLEMENTATION GUIDE - GROUP 3

**Project:** OpenCart on AWS Cloud - Enhanced Architecture  
**Team:** Group 3 (3 members)  
**Timeline:** 4 days (Dec 20-23, 2025)  
**Target Grade:** 70-79% (Grade A)  

---

## 📚 DOCUMENT STRUCTURE & NAVIGATION

<details>
<summary><b>📖 Click to view complete structure (Table of Contents)</b></summary>

```
📄 THIS DOCUMENT STRUCTURE
│
├── 📋 HEADER SECTION
│   ├── 🎯 Project Info (Team, Timeline, Target Grade)
│   ├── 👥 Team Roles & Responsibilities
│   └── 📊 System Architecture Overview
│       ├── 🔄 Traffic Flow Diagram (Multi-AZ)
│       ├── 📋 Data Flow Sequences (4 flows)
│       ├── 🏗️ Architecture Highlights
│       └── 🔒 Security Architecture
│
├── 📅 DAY 1: Foundation & Multi-AZ Deployment [5 hours]
│   ├── 📌 HOUR 1-2: GitHub Setup [9:00-11:00]
│   ├── 📌 HOUR 2-3: AWS Infrastructure Prep [11:00-12:00]
│   ├── 📌 HOUR 3-4: Create AMI & Launch EC2-B [13:00-14:00]
│   │   └── ⚠️ TROUBLESHOOTING: EC2-B Missing Files
│   ├── 📌 HOUR 4-5: Create Application Load Balancer [14:00-15:00]
│   │   └── ⚠️ CRITICAL: Update Config.php to ALB DNS
│   └── 📋 Final Verification Checklist
│
├── 📅 DAY 2: S3, CloudFront & Sessions [5 hours]
│   ├── 📌 HOUR 1-2: S3 Bucket & CloudFront
│   ├── 📌 HOUR 2-3: IAM Role for EC2 → S3
│   ├── 📌 HOUR 3-4: Install S3 Plugin
│   └── 📌 HOUR 4-5: Database Session Config
│
└── 📅 DAY 3-4: Monitoring & CI/CD [Placeholder]
```

**🗺️ Quick Navigation:**
- **Lỗi EC2-B thiếu code:** Search `⚠️ TROUBLESHOOTING: EC2-B Missing Files`
- **Cách fix CSS/JS lỗi:** Search `Part B: Update Config.php`
- **Target Group setup:** Search `STEP 1: Create Target Group`
- **Security Groups:** Search `STEP 3: Update Security Groups`
- **Final checklist:** Search `Final Verification Checklist`

**📝 Companion Documents:**
- [TECHNICAL_DOCUMENTATION.md](TECHNICAL_DOCUMENTATION.md) - Chi tiết architecture, flow, config cho người mới

</details>

---

## 👥 TEAM ROLES & RESPONSIBILITIES

```
Member 1 (You - Infrastructure Lead):
├─ AWS infrastructure setup (ALB, EC2s, networking)
├─ CI/CD pipeline (GitHub Actions)
├─ Monitoring & CloudWatch configuration
└─ Documentation & report writing

Member 2 (Application):
├─ OpenCart local setup & configuration
├─ S3 plugin integration
├─ Database migration to RDS
└─ Testing & QA

Member 3 (Documentation):
├─ Screenshots collection for report
├─ Cost analysis & scenarios
├─ Architecture diagrams
└─ Test cases documentation
```

---

ứ## 📊 SYSTEM ARCHITECTURE OVERVIEW - MULTI-AZ DESIGN

### **🔄 TRAFFIC FLOW DIAGRAM**

```
┌─────────────────────────────────────────────────────────────────────┐
│                         INTERNET USERS                               │
│                  (Customer browsing OpenCart)                        │
└──────────────────────────────┬──────────────────────────────────────┘
                               │ HTTPS/HTTP Request
                               │
                    ┌──────────▼──────────┐
                    │   Route 53 DNS      │
                    │  (Domain Resolver)  │
                    └──────────┬──────────┘
                               │ Returns ALB DNS
                               │
              ┌────────────────▼────────────────┐
              │     Internet Gateway            │
              │      Group3_IGW                 │
              │   (VPC Public Access)           │
              └────────────────┬────────────────┘
                               │
        ┏━━━━━━━━━━━━━━━━━━━━━▼━━━━━━━━━━━━━━━━━━━━━┓
        ┃    Application Load Balancer (MULTI-AZ)    ┃
        ┃      Group3_OpenCart_ALB                   ┃
        ┃                                             ┃
        ┃  ┌─────────────────────────────────────┐  ┃
        ┃  │ Listener: HTTP:80                   │  ┃
        ┃  │ Health Check: GET / (every 30s)     │  ┃
        ┃  │ Target Group: Group3_OpenCart_TG    │  ┃
        ┃  └─────────────────────────────────────┘  ┃
        ┃                                             ┃
        ┃  AZ Coverage: 1a ✅ + 1b ✅                ┃
        ┗━━━━━━━━━━━━┳━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━┛
                     │             │
         ┌───────────┴─────┐   ┌───┴──────────────┐
         │ Round Robin     │   │ Round Robin      │
         │ 50% traffic     │   │ 50% traffic      │
         └───────┬─────────┘   └────────┬─────────┘
                 │                      │
┌────────────────▼──────────────────────▼─────────────────────────────┐
│                VPC: Group3_VPC (10.0.0.0/16)                        │
│                Region: ap-southeast-1 (Singapore)                   │
│                                                                      │
│  ╔═══════════════════════════════════════════════════════════════╗ │
│  ║           AZ-1a (ap-southeast-1a)                             ║ │
│  ╠═══════════════════════════════════════════════════════════════╣ │
│  ║                                                                ║ │
│  ║  ┌──────────────────────────────────────────────────────┐    ║ │
│  ║  │  Public Subnet A (10.0.1.0/24)                       │    ║ │
│  ║  │                                                       │    ║ │
│  ║  │    ┌─────────────────────────────────┐              │    ║ │
│  ║  │    │  EC2-A (i-082cbe43b6ba19a6e)    │              │    ║ │
│  ║  │    │  Group3_WebServer1              │◄─────────────┼────╫─┤ Target 1
│  ║  │    │  ─────────────────────────      │              │    ║ │
│  ║  │    │  Public IP: 13.229.212.148      │              │    ║ │
│  ║  │    │  Private IP: 10.0.1.x           │              │    ║ │
│  ║  │    │                                  │              │    ║ │
│  ║  │    │  Stack:                          │              │    ║ │
│  ║  │    │  ├─ Amazon Linux 2023           │              │    ║ │
│  ║  │    │  ├─ Apache 2.4 (httpd)          │              │    ║ │
│  ║  │    │  ├─ PHP 8.4 + php-fpm           │              │    ║ │
│  ║  │    │  ├─ OpenCart 3.0.3.8            │              │    ║ │
│  ║  │    │  └─ AWS SDK for S3 upload       │              │    ║ │
│  ║  │    │                                  │              │    ║ │
│  ║  │    │  Roles:                          │              │    ║ │
│  ║  │    │  └─ Group3_EC2_S3_Role          │──────┐       │    ║ │
│  ║  │    └──────────────┬───────────────────┘      │       │    ║ │
│  ║  └───────────────────┼──────────────────────────┼───────┘    ║ │
│  ║                      │                          │             ║ │
│  ║                      │ Same-AZ                  │             ║ │
│  ║                      │ (Low latency)            │             ║ │
│  ║                      │                          │             ║ │
│  ║  ┌───────────────────▼──────────────────────────┼───────┐    ║ │
│  ║  │  Private Subnet A (10.0.11.0/24)             │       │    ║ │
│  ║  │                                               │       │    ║ │
│  ║  │    ┌─────────────────────────────────────┐   │       │    ║ │
│  ║  │    │  RDS MySQL 8.4.7 (Single-AZ) ⚠️     │◄──┘       │    ║ │
│  ║  │    │  group3-database                    │           │    ║ │
│  ║  │    │  ─────────────────────────────      │           │    ║ │
│  ║  │    │  Endpoint: group3-database.         │           │    ║ │
│  ║  │    │    cxcecm6wisku.ap-southeast-1.     │           │    ║ │
│  ║  │    │    rds.amazonaws.com:3306           │           │    ║ │
│  ║  │    │                                      │           │    ║ │
│  ║  │    │  Instance: db.t3.micro (FREE tier)  │           │    ║ │
│  ║  │    │  Storage: 20 GiB gp3 (encrypted)    │           │    ║ │
│  ║  │    │  Database: Group3_db                │           │    ║ │
│  ║  │    │                                      │           │    ║ │
│  ║  │    │  Tables:                             │           │    ║ │
│  ║  │    │  ├─ oc_product (19 items)           │           │    ║ │
│  ║  │    │  ├─ oc_session (shared sessions) ✅ │           │    ║ │
│  ║  │    │  ├─ oc_customer                     │           │    ║ │
│  ║  │    │  └─ oc_order                        │           │    ║ │
│  ║  │    │                                      │           │    ║ │
│  ║  │    │  Backups:                            │           │    ║ │
│  ║  │    │  ├─ Automated: 7 days retention     │           │    ║ │
│  ║  │    │  └─ Manual snapshots: On-demand     │           │    ║ │
│  ║  │    └──────────────────▲───────────────────┘           │    ║ │
│  ║  └───────────────────────┼───────────────────────────────┘    ║ │
│  ╚════════════════════════════════════════════════════════════════╝ │
│                             │                                        │
│                             │ Cross-AZ Connection                    │
│                             │ (~$0.01/GB data transfer)              │
│                             │                                        │
│  ╔═════════════════════════┼════════════════════════════════════╗   │
│  ║           AZ-1b (ap-southeast-1b)                            ║   │
│  ╠═════════════════════════┼════════════════════════════════════╣   │
│  ║                         │                                     ║   │
│  ║  ┌──────────────────────┼──────────────────────────────┐    ║   │
│  ║  │  Public Subnet B (10.0.2.0/24)                      │    ║   │
│  ║  │                      │                               │    ║   │
│  ║  │    ┌─────────────────▼───────────────┐              │    ║   │
│  ║  │    │  EC2-B (TBD - to be deployed)   │              │    ║   │
│  ║  │    │  Group3_WebServer2              │◄─────────────┼────╫───┤ Target 2
│  ║  │    │  ─────────────────────────      │              │    ║   │
│  ║  │    │  Public IP: TBD                 │              │    ║   │
│  ║  │    │  Private IP: 10.0.2.x           │              │    ║   │
│  ║  │    │                                  │              │    ║   │
│  ║  │    │  Stack: (Clone of EC2-A)        │              │    ║   │
│  ║  │    │  ├─ Amazon Linux 2023           │              │    ║   │
│  ║  │    │  ├─ Apache 2.4 (httpd)          │              │    ║   │
│  ║  │    │  ├─ PHP 8.4 + php-fpm           │              │    ║   │
│  ║  │    │  ├─ OpenCart 3.0.3.8            │              │    ║   │
│  ║  │    │  └─ AWS SDK for S3 upload       │              │    ║   │
│  ║  │    │                                  │              │    ║   │
│  ║  │    │  Roles:                          │              │    ║   │
│  ║  │    │  └─ Group3_EC2_S3_Role          │──────┐       │    ║   │
│  ║  │    └──────────────────────────────────┘      │       │    ║   │
│  ║  └─────────────────────────────────────────────┼───────┘    ║   │
│  ║                                                 │             ║   │
│  ╚═════════════════════════════════════════════════════════════╝   │
└──────────────────────────────────────────────────────────────────────┘
                                                    │
                                                    │ S3 Upload (via IAM role)
                                                    │
                                         ┌──────────▼──────────┐
                                         │  S3 Bucket           │
                                         │  group3-opencart-    │
                                         │  static              │
                                         │                      │
                                         │  Structure:          │
                                         │  ├─ catalog/         │
                                         │  │  └─ products/     │
                                         │  ├─ blog/            │
                                         │  └─ cache/           │
                                         │                      │
                                         │  Policy: Public Read │
                                         └──────────┬───────────┘
                                                    │ Origin
                                                    │
                                         ┌──────────▼───────────┐
                                         │  CloudFront CDN      │
                                         │  Distribution        │
                                         │                      │
                                         │  Domain: dXXXX.      │
                                         │  cloudfront.net      │
                                         │                      │
                                         │  Cache: Max-Age      │
                                         │  31536000 (1 year)   │
                                         │                      │
                                         │  Edge Locations:     │
                                         │  ├─ Singapore        │
                                         │  ├─ Hong Kong        │
                                         │  └─ Tokyo            │
                                         └──────────┬───────────┘
                                                    │
                                                    │ Cached delivery (<50ms)
                                                    │
                                         ┌──────────▼───────────┐
                                         │  Customer Browser    │
                                         │  (Product Images)    │
                                         └──────────────────────┘
```

### **📋 DATA FLOW SEQUENCES**

**FLOW 1: Customer Browse Website (Read-Only)**
```
1. Customer → http://ALB-DNS-NAME/
2. Route 53 → Resolves ALB DNS
3. Internet Gateway → Allows traffic into VPC
4. ALB → Health check both EC2s (every 30s)
5. ALB → Routes to healthy target (Round Robin):
   ├─ 50% → EC2-A (ap-southeast-1a)
   └─ 50% → EC2-B (ap-southeast-1b)
6. EC2-A or EC2-B → Processes PHP request
7. EC2 → Queries RDS MySQL:
   ├─ EC2-A → RDS (same-AZ, low latency ~1ms)
   └─ EC2-B → RDS (cross-AZ, ~2-3ms) ⚠️
8. RDS → Returns product data (19 products)
9. EC2 → Renders HTML with product info
10. EC2 → Returns response to ALB
11. ALB → Returns to customer browser
12. Browser → Loads images from CloudFront CDN
```

**FLOW 2: Customer Login / Add to Cart (Session Management)**
```
1. Customer logs in via ALB
2. ALB → Routes to EC2-A (random)
3. EC2-A → Creates session in PHP
4. EC2-A → Writes session to RDS:
   INSERT INTO oc_session (session_id, data, expire)
   VALUES ('abc123...', 'user_id=5|cart=[...]', NOW()+3600)
5. EC2-A → Returns OCSESSID cookie to browser
6. Customer refreshes page (might hit EC2-B!)
7. ALB → Routes to EC2-B (different instance!)
8. EC2-B → Reads cookie: OCSESSID=abc123...
9. EC2-B → Queries RDS (cross-AZ):
   SELECT data FROM oc_session WHERE session_id='abc123...'
10. RDS → Returns session data
11. EC2-B → Restores user context (still logged in!) ✅
12. Customer cart persists across both EC2s!
```

**FLOW 3: Admin Upload Product Image (S3 Static Assets)**
```
1. Admin uploads image via: http://ALB/admin/
2. ALB → Routes to EC2-A or EC2-B
3. EC2 → Receives uploaded file (temp storage)
4. EC2 → Calls S3Helper class (PHP)
5. S3Helper → Uses IAM role credentials (automatic)
6. EC2 → AWS SDK uploads to S3:
   s3://group3-opencart-static/catalog/products/2025/12/21/abc123.jpg
7. S3 → Stores object with public-read ACL
8. EC2 → Saves CloudFront URL in database:
   UPDATE oc_product SET image='https://dXXX.cloudfront.net/catalog/...'
9. CloudFront → Pulls image from S3 (origin fetch)
10. Next customer request → CloudFront serves cached image (<50ms)
11. Image stored in S3 (persistent) ✅
12. Image cached in CloudFront edges (fast delivery) ✅
```

**FLOW 4: Failure Scenario - AZ-1a Down (Disaster Recovery)**
```
SCENARIO: Entire AZ-1a fails (EC2-A + RDS down!)

1. ALB health checks EC2-A → FAIL (no response)
2. ALB → Marks EC2-A as "Unhealthy"
3. ALB → Routes 100% traffic to EC2-B (1b only) ✅
4. EC2-B tries to connect to RDS → FAIL (RDS in AZ-1a down!)
5. Website returns: "Database connection error" ❌

RECOVERY PROCEDURE:
6. Admin → Restores RDS from automated backup
7. Target AZ: ap-southeast-1b (different AZ from original)
8. Restore time: ~10-15 minutes (RTO: 15 min)
9. Data loss: 0-24 hours (RPO: last backup)
10. Admin → Updates EC2-B config.php with new RDS endpoint
11. Website operational again ✅

PREVENTION:
- Multi-AZ RDS ($25/month) → Automatic failover (0 sec RTO)
- Current: Single-AZ RDS ($0/month) → Manual restore (15 min RTO)
- Trade-off: Cost vs Availability
```

### **🏗️ ARCHITECTURE HIGHLIGHTS**

```
┌─────────────────────────────────────────────────────────────────┐
│                    MULTI-AZ DESIGN BENEFITS                      │
└─────────────────────────────────────────────────────────────────┘

✅ COMPUTE TIER (High Availability):
   ├─ 2x EC2 instances in different AZs
   ├─ ALB distributes traffic across both AZs
   ├─ If AZ-1b fails → EC2-A continues in AZ-1a
   ├─ If AZ-1a EC2 fails → EC2-B continues in AZ-1b
   └─ RTO: 0 seconds (automatic failover via ALB)

⚠️  DATABASE TIER (Single Point of Failure):
   ├─ RDS in ap-southeast-1a ONLY (Single-AZ)
   ├─ If AZ-1a fails → Database OFFLINE
   ├─ Restore from backup → NEW instance in AZ-1b
   ├─ RTO: ~15 minutes (manual restore)
   ├─ RPO: 0-24 hours (depends on backup frequency)
   └─ Acceptable for academic project (grade 70-79%)

📦 STATIC ASSETS (Global CDN):
   ├─ S3 bucket (11 9s durability)
   ├─ CloudFront caching (~50ms response)
   ├─ Images survive ALL AZ failures ✅
   └─ No single point of failure

🔐 SESSION MANAGEMENT (Shared State):
   ├─ Sessions stored in RDS (oc_session table)
   ├─ Both EC2s read/write to same database
   ├─ No sticky sessions needed
   └─ True stateless load balancing

💰 COST OPTIMIZATION:
   ├─ 2x t2.micro EC2: $0/month (750 hrs FREE tier)
   ├─ 1x db.t3.micro RDS: $0/month (750 hrs FREE tier)
   ├─ ALB: ~$16/month (NOT free tier)
   ├─ S3: ~$0.50/month (minimal usage)
   ├─ CloudFront: ~$1/month (first 1TB free tier)
   ├─ Cross-AZ transfer: ~$1-2/month
   └─ TOTAL: ~$18-20/month

📊 AVAILABILITY CALCULATION:
   ├─ ALB SLA: 99.99% (AWS guarantee)
   ├─ EC2 Multi-AZ: ~99.95% (2 instances)
   ├─ RDS Single-AZ: ~99.5% (manual recovery)
   └─ Overall: ~99.4% uptime (acceptable for grade A)

⚖️  TRADE-OFFS:
   ✅ PRO: Free tier eligible (cost = $0 for EC2+RDS)
   ✅ PRO: Multi-AZ compute (web tier survives AZ failure)
   ✅ PRO: Session sharing works perfectly
   ⚠️  CON: Database single point of failure
   ⚠️  CON: 15-minute RTO for database recovery
   ⚠️  CON: Potential data loss (0-24 hours based on backup)
   
   UPGRADE PATH:
   └─ Enable Multi-AZ RDS: +$25-30/month → 0 sec RTO + 0 data loss
```

### **🔒 SECURITY ARCHITECTURE**

```
SECURITY GROUP RULES:

Group3_ALB_SG (Load Balancer):
├─ Inbound:
│  ├─ HTTP (80) from 0.0.0.0/0 ✅
│  └─ HTTPS (443) from 0.0.0.0/0 ✅
└─ Outbound:
   └─ HTTP (80) to Group3_WebServer_SG

Group3_WebServer_SG (EC2 Instances):
├─ Inbound:
│  ├─ HTTP (80) from Group3_ALB_SG ONLY ✅
│  └─ SSH (22) from My IP (for management)
└─ Outbound:
   ├─ MySQL (3306) to Group3_RDS_SG
   └─ HTTPS (443) to 0.0.0.0/0 (for S3 API)

Group3_RDS_SG (Database):
├─ Inbound:
│  └─ MySQL (3306) from Group3_WebServer_SG ONLY ✅
└─ Outbound: None

IAM ROLES:
Group3_EC2_S3_Role:
└─ Permissions:
   ├─ s3:PutObject (upload images)
   ├─ s3:GetObject (download images)
   ├─ s3:DeleteObject (remove images)
   └─ s3:ListBucket (browse folders)

ENCRYPTION:
├─ RDS: Encrypted at rest (aws/rds KMS key) ✅
├─ S3: SSE-S3 encryption ✅
└─ CloudFront: HTTPS delivery ✅
```

---

## 🗓️ 4-DAY IMPLEMENTATION TIMELINE

### 📅 **DAY 1: Foundation & GitHub** `[Friday Dec 20 - 5 hours]`

#### 📌 **HOUR 1-2: GitHub Setup & Team Collaboration** `[9:00-11:00]`

**🎯 OBJECTIVE:** Create GitHub repository, download official OpenCart, push code for team collaboration

---

**📋 STEP 1.1: PREREQUISITES CHECK**

**Before starting, verify:**

```powershell
# Check 1: Git installed
git --version
# Expected: git version 2.x.x or higher

# Check 2: GitHub account ready
# - Have GitHub username/password
# - OR GitHub Personal Access Token (recommended for pushing)

# Check 3: Network access
ping github.com
# Expected: Reply from xxx.xxx.xxx.xxx

# Check 4: Workspace folder exists
Test-Path "C:\Users\ASUS\Documents\Sever Systems - Cloud\Cloud-project"
# Expected: True
```

**❌ TROUBLESHOOTING:**

| Problem | Check | Solution |
|---------|-------|----------|
| `git : The term 'git' is not recognized` | Git not installed | Download from https://git-scm.com/download/win |
| `Ping request could not find host github.com` | Network/DNS issue | Check internet, try `ping 8.8.8.8` |
| `Path does not exist` | Wrong folder | Create: `New-Item -ItemType Directory -Path "..." -Force` |
| `Permission denied` | Folder protected | Run PowerShell as Administrator |

---

**📋 STEP 1.2: CREATE GITHUB REPOSITORY**

**1. Open GitHub in browser:**
```
https://github.com
```

**2. Click "New" repository (top-right, green button)**

**3. Configure repository:**
```
Repository name: opencart-aws-group3
Description: OpenCart 3.0 AWS Multi-AZ Deployment - Group 3 Server Systems Project
Visibility: ⦿ Private (only team members can see)
Initialize: □ DO NOT check "Add README" (we'll create locally)
            □ DO NOT add .gitignore (we'll create custom)
            □ DO NOT choose license
Click: "Create repository"
```

**4. Invite team members:**
```
Repository → Settings → Collaborators and teams
→ Click "Add people"
→ Enter teammate GitHub usernames
→ Role: Write (can push code)
→ Click "Add <username> to this repository"

Repeat for all 3 team members
```

**✅ VERIFICATION:**
- Repository URL exists: `https://github.com/WEKONE-26/opencart-aws-group3`
- You see "Private" badge
- Team members listed under Collaborators

---

**📋 STEP 1.3: DOWNLOAD OFFICIAL OPENCART**

**1. Open PowerShell (not as admin needed):**
```powershell
# Navigate to project workspace
cd "C:\Users\ASUS\Documents\Sever Systems - Cloud\Cloud-project"

# Verify current directory
Get-Location
# Expected: ...\Cloud-project
```

**2. Download OpenCart official release:**
```powershell
# Download from GitHub official release
Invoke-WebRequest -Uri "https://github.com/opencart/opencart/releases/download/3.0.3.8/opencart-3.0.3.8.zip" -OutFile "opencart-3.0.3.8.zip"

# Check download completed (should be ~13-15 MB)
Get-Item "opencart-3.0.3.8.zip" | Select-Object Name, Length
```

**Expected output:**
```
Name                     Length
----                     ------
opencart-3.0.3.8.zip    14523648
```

**❌ TROUBLESHOOTING - Download fails:**

| Error | Cause | Solution |
|-------|-------|----------|
| `Unable to connect to the remote server` | Network timeout | Retry download, check firewall |
| `Access to the path ... is denied` | No write permission | Run as Administrator OR choose different folder |
| `The request was aborted: Could not create SSL/TLS secure channel` | TLS 1.2 not enabled | Run: `[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12` |

**3. Extract ZIP file:**
```powershell
# Extract to current directory
Expand-Archive -Path "opencart-3.0.3.8.zip" -DestinationPath "." -Force

# Verify extraction (should see "opencart-3.0.3.8" folder)
Get-ChildItem | Where-Object {$_.Name -like "opencart*"}
```

**Expected:** Folder `opencart-3.0.3.8` created with subdirectories

**4. Rename to standard name:**
```powershell
# Rename for consistency
Rename-Item "opencart-3.0.3.8" "opencart-3.0.x.x" -Force

# Verify rename
Test-Path "opencart-3.0.x.x"
# Expected: True
```

**5. Clean up ZIP file:**
```powershell
# Remove downloaded ZIP (save space)
Remove-Item "opencart-3.0.3.8.zip" -Force
```

**✅ VERIFICATION:**
```powershell
# Check folder size (should be ~70-80 MB)
Get-ChildItem "opencart-3.0.x.x" -Recurse | Measure-Object -Property Length -Sum | Select-Object @{Name="SizeMB";Expression={[math]::Round($_.Sum / 1MB, 2)}}
# Expected: SizeMB between 70-80

# Check key files exist
Test-Path "opencart-3.0.x.x/upload/index.php"
Test-Path "opencart-3.0.x.x/upload/catalog/view/theme/default"
# Both should return: True
```

---

**📋 STEP 1.4: INITIALIZE GIT REPOSITORY**

**1. Navigate into OpenCart folder:**
```powershell
cd "opencart-3.0.x.x"
Get-Location
# Expected: ...\Cloud-project\opencart-3.0.x.x
```

**2. Initialize Git:**
```powershell
git init
# Expected: Initialized empty Git repository in .../.git/
```

**3. Configure Git user (for commits):**
```powershell
git config user.name "Group3-Cloudproject"
git config user.email "huynhancool@gmail.com"

# Verify configuration
git config user.name
git config user.email
# Should echo your name and email
```

**4. Create .gitignore file:**
```powershell
# Create .gitignore to exclude temporary/sensitive files
@"
system/storage/cache/*
system/storage/logs/*
image/cache/*
config.php
admin/config.php
.env
*.pem
*.key
*.log
.DS_Store
Thumbs.db
"@ | Out-File -FilePath ".gitignore" -Encoding UTF8

# Verify .gitignore created
Get-Content ".gitignore"
```

**📖 Why exclude these files?**
```
config.php, admin/config.php → Contains DB credentials (security risk)
*.pem, *.key → SSH keys (never commit to Git!)
cache/, logs/ → Temporary files (changes constantly)
```

---

**📋 STEP 1.5: CREATE DOCUMENTATION STRUCTURE**

```powershell
# Create docs folders
New-Item -ItemType Directory -Path "docs/screenshots" -Force
New-Item -ItemType Directory -Path "docs/architecture" -Force
New-Item -ItemType Directory -Path "docs/testing" -Force
New-Item -ItemType Directory -Path "infrastructure" -Force

# Verify folders created
Get-ChildItem -Directory | Where-Object {$_.Name -eq "docs" -or $_.Name -eq "infrastructure"}
```

**Expected output:**
```
Mode    Name
----    ----
d----   docs
d----   infrastructure
```

**Create team README.md:**
```powershell
@"
# OpenCart AWS Deployment - Group 3

## Team Members
- Member 1: Infrastructure & DevOps (huynhancool@gmail.com)
- Member 2: Application Development
- Member 3: Documentation & Testing

## Project Overview
Multi-AZ OpenCart deployment on AWS with:
- Application Load Balancer (ALB)
- 2x EC2 instances (ap-southeast-1a + 1b)
- RDS MySQL database
- S3 + CloudFront for static assets
- Database session sharing

## Project Structure
\`\`\`
opencart-3.0.x.x/
├── upload/              # OpenCart application files
├── docs/
│   ├── screenshots/     # Project screenshots
│   ├── architecture/    # Architecture diagrams
│   └── testing/         # Test cases and results
├── infrastructure/      # AWS deployment scripts
└── README.md           # This file
\`\`\`

## Quick Start
1. See \`docs/SETUP.md\` for detailed AWS setup instructions
2. Check \`infrastructure/\` for deployment scripts
3. Review \`docs/architecture/\` for system design

## Current Progress
- [x] Local OpenCart setup (v3.0.3.8 Official Release)
- [x] GitHub repository created
- [ ] AWS VPC and networking
- [ ] EC2 instances and ALB
- [ ] RDS database
- [ ] S3 and CloudFront
- [ ] CI/CD pipeline
- [ ] CloudWatch monitoring

## Tech Stack
- **Application**: OpenCart 3.0.3.8
- **Web Server**: Apache 2.4 + PHP 8.4
- **Database**: MySQL 8.4 on RDS
- **CDN**: CloudFront + S3
- **Infrastructure**: AWS (ap-southeast-1)
- **Version Control**: Git + GitHub

## Documentation
See complete implementation guide in:
- \`D2_COMPLETE_IMPLEMENTATION_GUIDE.md\`
- \`TECHNICAL_DOCUMENTATION.md\`
"@ | Out-File -FilePath "README.md" -Encoding UTF8
```

**✅ VERIFICATION:**
```powershell
# Check README exists and has content
Get-Content "README.md" | Measure-Object -Line
# Expected: 40-50 lines

# Check folder structure
Get-ChildItem -Recurse -Directory | Select-Object FullName
```

---

**📋 STEP 1.6: INITIAL COMMIT & PUSH TO GITHUB**

**1. Stage all files:**
```powershell
git add .

# Check what will be committed
git status
```

**Expected output:**
```
Changes to be committed:
  new file:   .gitignore
  new file:   README.md
  new file:   docs/...
  new file:   upload/index.php
  ... (hundreds of files)
```

**2. Create initial commit:**
```powershell
git commit -m "Initial commit: OpenCart 3.0.3.8 official release with documentation structure"

# Verify commit created
git log --oneline
# Expected: Shows commit hash and message
```

**3. Connect to GitHub remote:**
```powershell
git remote add origin https://github.com/WEKONE-26/opencart-aws-group3.git

# Verify remote added
git remote -v
# Expected:
# origin  https://github.com/WEKONE-26/opencart-aws-group3.git (fetch)
# origin  https://github.com/WEKONE-26/opencart-aws-group3.git (push)
```

**4. Rename branch to main:**
```powershell
git branch -M main

# Verify branch name
git branch
# Expected: * main
```

**5. Push to GitHub:**
```powershell
# Push with force (first time only)
git push -u origin main --force
```

**⚠️ If authentication fails:**

**Option A: Use Personal Access Token (Recommended)**
```powershell
# GitHub blocks password authentication since Aug 2021
# Create token: GitHub → Settings → Developer settings → Personal access tokens → Tokens (classic)
# → Generate new token → Select "repo" scope → Generate

# When prompted for password, paste TOKEN (not GitHub password!)
```

**Option B: Use GitHub CLI**
```powershell
# Install GitHub CLI
winget install GitHub.cli

# Authenticate
gh auth login
# Follow prompts to authenticate via browser
```

**Expected output after successful push:**
```
Enumerating objects: 1500, done.
Counting objects: 100% (1500/1500), done.
Delta compression using up to 8 threads
Compressing objects: 100% (1200/1200), done.
Writing objects: 100% (1500/1500), 12.5 MiB | 2.5 MiB/s, done.
Total 1500 (delta 300), reused 0 (delta 0)
To https://github.com/WEKONE-26/opencart-aws-group3.git
 * [new branch]      main -> main
Branch 'main' set up to track remote branch 'main' from 'origin'.
```

---

**📋 STEP 1.7: VERIFY GITHUB REPOSITORY**

**1. Open GitHub in browser:**
```
https://github.com/WEKONE-26/opencart-aws-group3
```

**2. Verify repository contents:**
```
✅ Check "main" branch exists (dropdown shows "main")
✅ Check README.md displays on homepage
✅ Check folders visible: docs/, infrastructure/, upload/
✅ Check "X commits" count (should be 1)
✅ Check .gitignore exists (click to view)
```

**3. Test clone by teammate:**
```powershell
# Teammate should test clone:
cd "C:\Temp"
git clone https://github.com/WEKONE-26/opencart-aws-group3.git
cd opencart-aws-group3

# Verify files downloaded
Get-ChildItem
# Should see: docs/, infrastructure/, upload/, README.md, .gitignore
```

---

**✅ STEP 1 COMPLETION CHECKLIST:**

```
GitHub Repository:
☑ Repository created: opencart-aws-group3 (Private)
☑ All 3 team members invited as collaborators
☑ Repository accessible via: https://github.com/WEKONE-26/opencart-aws-group3

Local Setup:
☑ OpenCart 3.0.3.8 downloaded (~14 MB ZIP)
☑ Extracted to opencart-3.0.x.x folder (~75 MB)
☑ Git initialized in local folder
☑ Git user configured (name + email)

File Structure:
☑ .gitignore created (excludes config.php, *.pem, cache/)
☑ README.md created with project overview
☑ docs/ folders created (screenshots, architecture, testing)
☑ infrastructure/ folder created

Git Operations:
☑ All files staged (git add .)
☑ Initial commit created
☑ Remote origin added (GitHub URL)
☑ Branch renamed to "main"
☑ Code pushed to GitHub (1500+ files)

Verification:
☑ GitHub repository shows all files
☑ README.md displays on GitHub homepage
☑ Teammates can clone repository
☑ git log shows initial commit

Team Collaboration:
☑ Teammates have Write access
☑ Teammates can clone, commit, push
☑ Folder structure ready for Day 1-4 work
```

**💡 PRO TIPS:**

```
1. Daily Git Workflow (for next 4 days):
   git pull origin main          # Get teammates' changes
   git add .                      # Stage your changes
   git commit -m "Day X: ..."     # Commit with descriptive message
   git push origin main           # Share with team

2. Screenshot Organization:
   docs/screenshots/day1/         # Save EC2, ALB screenshots
   docs/screenshots/day2/         # S3, CloudFront screenshots
   docs/screenshots/day3/         # CloudWatch screenshots

3. Avoid Merge Conflicts:
   - Don't edit same files simultaneously
   - Pull before starting work
   - Push frequently (every hour)
```

---

**⏱️ TIME CHECK:** Should take 30-45 minutes total

**⏭️ NEXT:** HOUR 2-3 - AWS Infrastructure Preparation

---

### 📅 **DAY 3: GitHub Actions CI/CD Pipeline** `[Saturday Dec 22 - 5 hours]`

#### 📌 **HOUR 1-2: GitHub Secrets & IAM Setup** `[9:00-11:00]`

**🎯 OBJECTIVE:** Tạo AWS IAM user, access keys, và lưu vào GitHub Secrets để GitHub Actions có quyền deploy

---

**📋 STEP 3.1: TẠO IAM USER CHO GITHUB ACTIONS**

**1. AWS Console → IAM → Users:**
```
Click "Create user"
Username: github-actions-deployer
Next → Attach policies directly
```

**2. Attach SSM Policy:**
```
Search & select: AmazonSSMFullAccess
(Cho phép SSM SendCommand để deploy code vào EC2)
```

**3. Create Access Keys:**
```
IAM → Users → github-actions-deployer
→ Security credentials tab
→ Access keys → Create access key
→ Download CSV file

⚠️ SAVE BOTH:
- Access Key ID: AKIAXXXXXXXXXXXXXXXX
- Secret Access Key: wJalrXUtnFEMI/K7MDENG/...
```

---

**📋 STEP 3.2: CẤU HÌNH EC2 IAM ROLE**

**1. Add SSM policy vào EC2 role:**
```
IAM → Roles → Group3_EC2_S3_Role
→ Add permissions → Attach policies directly
→ Search: AmazonSSMManagedInstanceCore
→ Attach
```

**2. Reboot EC2 để apply:**
```
EC2 → Instances → EC2-A
→ Instance State → Reboot instance
(Chờ ~2 phút)
```

---

#### 📌 **HOUR 2-3: Tạo Deploy Workflow** `[11:00-12:00]`

**🎯 OBJECTIVE:** Tạo GitHub Actions workflow để auto-deploy code tới EC2

---

**📋 STEP 3.3: TẠO GITHUB SECRETS**

**1. GitHub → Settings → Secrets and variables → Actions**

**2. Add 2 secrets:**
```
Name: AWS_ACCESS_KEY_ID
Value: AKIAXXXXXXXXXXXXXXXX

---

Name: AWS_SECRET_ACCESS_KEY
Value: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
(Copy từ CSV file - chú ý ký tự đặc biệt!)
```

---

**📋 STEP 3.4: TẠO .github/workflows/deploy.yml**

**File được tạo tự động từ github actions template, chứa:**
- ✅ Checkout code từ GitHub
- ✅ Configure AWS credentials
- ✅ Deploy to EC2-A via SSM
- ✅ Deploy to EC2-B via SSM
- ✅ Deployment summary

---

#### 📌 **HOUR 3-4: Test & Fix Issues** `[13:00-14:00]`

**🎯 OBJECTIVE:** Trigger workflow, debug lỗi, verify deployment

---

**📋 STEP 3.5: TRIGGER WORKFLOW**

**Cách 1: Automatic (push code)**
```powershell
git commit --allow-empty -m "Trigger: Deploy workflow"
git push origin main
# Workflow tự động chạy!
```

**Cách 2: Manual (GitHub UI)**
```
https://github.com/WEKONE-26/opencart-aws-group3/actions
→ Click "Deploy to AWS EC2"
→ Click "Run workflow"
```

---

**📋 STEP 3.6: TROUBLESHOOTING WORKFLOW FAILURES**

**❌ Error 1: AccessDeniedException - ssm:SendCommand**
```
Nguyên Nhân: IAM user không có SSM permissions
Cách Fix:
1. AWS IAM → Users → github-actions-deployer
2. Add permissions → Attach AmazonSSMFullAccess
3. Trigger workflow lại
```

**❌ Error 2: Instances not in a valid state**
```
Nguyên Nhân: EC2 role không có SSM permissions
Cách Fix:
1. AWS IAM → Roles → Group3_EC2_S3_Role
2. Add permissions → Attach AmazonSSMManagedInstanceCore
3. EC2 → Reboot both instances
4. Trigger workflow lại
```

**❌ Error 3: Signature Mismatch**
```
Nguyên Nhân: AWS Secret Access Key sai (missing ký tự đặc biệt)
Cách Fix:
1. AWS IAM → Create NEW access key (delete old)
2. Copy từ CSV file chứ không phải screenshot
3. GitHub → Update AWS_SECRET_ACCESS_KEY secret
4. Trigger workflow lại
```

---

#### 📌 **HOUR 4-5: Verify & Celebrate** `[14:00-15:00]`

**🎯 OBJECTIVE:** Xác nhận deployment thành công

---

**📋 STEP 3.7: VERIFY DEPLOYMENT**

**1. Check workflow status:**
```
GitHub Actions → Latest run
Check: All steps ✅ (7 steps total)
Duration: ~14-30 seconds
```

**2. Test website:**
```
Browser: http://Group3-OpenCart-ALB-1956877542.ap-southeast-1.elb.amazonaws.com
Expected: ✅ Website loads perfectly
```

**3. Verify code updated:**
```powershell
SSH to EC2: ssh -i project.pem ec2-user@13.212.190.57
Check: ls -la /var/www/html/.github/workflows/
Should see: deploy.yml
```

**✅ STEP 3 COMPLETION CHECKLIST:**
```
GitHub Secrets:
☑ AWS_ACCESS_KEY_ID set
☑ AWS_SECRET_ACCESS_KEY set

IAM Configuration:
☑ github-actions-deployer user created
☑ EC2 role has AmazonSSMManagedInstanceCore
☑ EC2 rebooted

Workflow Execution:
☑ Workflow triggered successfully
☑ All 7 steps completed
☑ Logs show "deployment completed"
☑ No errors

Deployment Verification:
☑ Website loads via ALB
☑ Code updated on EC2
☑ Apache running

CI/CD Pipeline LIVE! 🎉
```

---

#### 📌 **HOUR 2-3: AWS Infrastructure Preparation** `[11:00-12:00]`

**🎯 OBJECTIVE:** Document existing D1 infrastructure, verify components, prepare for Multi-AZ expansion

---

**📋 STEP 2.1: PREREQUISITES CHECK**

**Before starting:**

```powershell
# Check 1: Have AWS credentials
# - AWS Account login (email + password)
# - OR AWS CLI configured

# Check 2: Know your D1 infrastructure
# - EC2 Instance ID from Day 1
# - RDS Endpoint from Day 1
# - VPC ID from Day 1
# - Security Group IDs

# Check 3: Have SSH key
Test-Path "C:\Users\ASUS\Documents\Sever Systems - Cloud\Cloud-project\project.pem"
# Expected: True

# Check 4: Can access EC2 from D1
# (Will test in next step)
```

---

**📋 STEP 2.2: INVENTORY EXISTING D1 INFRASTRUCTURE**

**🎯 Goal:** Document what was built in Day 1 to avoid duplicates

**1. Login to AWS Console:**
```
https://console.aws.amazon.com
Region: ap-southeast-1 (Singapore) ← Verify top-right dropdown!
```

**2. Document VPC:**

```
Services → VPC → Your VPCs

Find: VPC with tag "Group3_VPC" OR your custom name
Record:
├─ VPC ID: vpc-xxxxxxxxx
├─ CIDR: 10.0.0.0/16 (or your chosen range)
└─ Name tag: Group3_VPC
```

**3. Document Subnets:**

```
VPC → Subnets

Public Subnet A (for EC2-A):
├─ Subnet ID: subnet-xxxxxxxxx
├─ CIDR: 10.0.1.0/24
├─ AZ: ap-southeast-1a ✅
└─ Name: Group3_PublicSubnet_A

Private Subnet A (for RDS):
├─ Subnet ID: subnet-yyyyyyyyy
├─ CIDR: 10.0.11.0/24
├─ AZ: ap-southeast-1a
└─ Name: Group3_PrivateSubnet_A

⚠️ IMPORTANT: Note which AZ each subnet is in!
We'll create matching subnets in AZ-1b for Multi-AZ setup
```

**4. Document EC2 Instance (from D1):**

```
Services → EC2 → Instances

Find: Your running EC2 instance
Record:
├─ Instance ID: i-082cbe43b6ba19a6e (yours will differ!)
├─ Name: Group3_WebServer1 OR EC2-A
├─ Instance type: t2.micro
├─ Public IP: 13.229.212.148 (yours will differ!)
├─ Private IP: 10.0.1.x
├─ Availability Zone: ap-southeast-1a ✅
├─ VPC: Group3_VPC
├─ Subnet: Group3_PublicSubnet_A
├─ Security Group: Group3_WebServer_SG (sg-xxxxxxxxx)
└─ Key pair: project
```

**5. Document RDS Database:**

```
Services → RDS → Databases

Find: Your database instance
Record:
├─ DB identifier: group3-database (or your custom name)
├─ Endpoint: group3-database.cxcecm6wisku.ap-southeast-1.rds.amazonaws.com
├─ Port: 3306
├─ Engine: MySQL 8.4.7
├─ Instance class: db.t3.micro
├─ Availability Zone: ap-southeast-1a (Single-AZ!) ⚠️
├─ VPC: Group3_VPC
├─ Subnet group: Should have 2 subnets (for Multi-AZ option)
├─ Security Group: Group3_RDS_SG (sg-yyyyyyyyy)
├─ Master username: admin
└─ Database name: Group3_db
```

**6. Document Security Groups:**

```
EC2 → Security Groups

Group3_WebServer_SG:
├─ Security group ID: sg-xxxxxxxxx
├─ Inbound rules (current from D1):
│  ├─ HTTP (80) from 0.0.0.0/0 ← Will change later!
│  └─ SSH (22) from My IP
└─ Outbound rules:
   └─ All traffic to 0.0.0.0/0

Group3_RDS_SG:
├─ Security group ID: sg-yyyyyyyyy
├─ Inbound rules:
│  └─ MySQL (3306) from Group3_WebServer_SG ✅
└─ Outbound rules:
   └─ None (database doesn't initiate connections)
```

**7. Create inventory document:**

```powershell
# On your local machine
cd "C:\Users\ASUS\Documents\Sever Systems - Cloud\Cloud-project\opencart-3.0.x.x\infrastructure"

# Create inventory file
@"
# D1 Infrastructure Inventory
# Created: $(Get-Date -Format "yyyy-MM-dd HH:mm")

## VPC
- VPC ID: vpc-xxxxxxxxx
- VPC Name: Group3_VPC
- CIDR: 10.0.0.0/16
- Region: ap-southeast-1

## Subnets (Existing - AZ 1a only)
### Public Subnet A
- Subnet ID: subnet-xxxxxxxxx
- CIDR: 10.0.1.0/24
- AZ: ap-southeast-1a
- Name: Group3_PublicSubnet_A

### Private Subnet A
- Subnet ID: subnet-yyyyyyyyy
- CIDR: 10.0.11.0/24
- AZ: ap-southeast-1a
- Name: Group3_PrivateSubnet_A

## EC2 Instance (Existing)
- Instance ID: i-082cbe43b6ba19a6e
- Name: Group3_WebServer1
- Type: t2.micro
- Public IP: 13.229.212.148
- Private IP: 10.0.1.x
- AZ: ap-southeast-1a
- AMI: Amazon Linux 2023
- Key: project.pem

## RDS Database
- DB Identifier: group3-database
- Endpoint: group3-database.cxcecm6wisku.ap-southeast-1.rds.amazonaws.com
- Port: 3306
- Engine: MySQL 8.4.7
- Instance: db.t3.micro
- AZ: ap-southeast-1a (Single-AZ)
- Database: Group3_db
- Username: admin

## Security Groups
- WebServer SG: sg-xxxxxxxxx (allows HTTP + SSH)
- RDS SG: sg-yyyyyyyyy (allows MySQL from WebServer SG)

## TODO for Multi-AZ:
- [ ] Create Public Subnet B in ap-southeast-1b
- [ ] Create AMI from EC2-A
- [ ] Launch EC2-B in Subnet B
- [ ] Create Application Load Balancer
- [ ] Create Target Group
- [ ] Update Security Groups
"@ | Out-File -FilePath "d1-inventory.txt" -Encoding UTF8

# Display inventory
Get-Content "d1-inventory.txt"
```

---

**📋 STEP 2.3: VERIFY EC2-A FUNCTIONALITY**

**🎯 Goal:** Confirm EC2-A works before cloning to EC2-B

**1. SSH to EC2-A:**

```powershell
# From PowerShell (or use PuTTY)
cd "C:\Users\ASUS\Documents\Sever Systems - Cloud\Cloud-project"

ssh -i project.pem ec2-user@13.229.212.148
# Replace with YOUR EC2 Public IP from inventory!
```

**Expected:**
```
   ,     #_
   ~\_  ####_        Amazon Linux 2023
  ~~  \_#####\
  ~~     \###|
  ~~       \#/ ___   https://aws.amazon.com/linux/amazon-linux-2023
   ~~       V~' '->
    ~~~         /
      ~~._.   _/
         _/ _/
       _/m/'

[ec2-user@ip-10-0-1-xxx ~]$
```

**2. Verify installed services:**

```bash
# Check Apache running
sudo systemctl status httpd
# Expected: active (running) ✅

# Check PHP-FPM running
sudo systemctl status php-fpm
# Expected: active (running) ✅

# Check PHP version
php --version
# Expected: PHP 8.4.x ✅
```

**❌ If services stopped:**
```bash
sudo systemctl start httpd
sudo systemctl start php-fpm
sudo systemctl enable httpd  # Auto-start on boot
sudo systemctl enable php-fpm
```

**3. Verify OpenCart files:**

```bash
# Check web root size
du -sh /var/www/html/
# Expected: ~70-80M ✅

# Check critical files exist
ls -la /var/www/html/ | grep -E "index.php|config.php"
# Expected: Both files present ✅

# Check theme files (important for AMI)
ls -la /var/www/html/catalog/view/theme/default/
# Expected: stylesheet/, template/, image/ folders ✅

# Check image directory
du -sh /var/www/html/image/
# Expected: ~15-20M (demo products) ✅
```

**⚠️ CRITICAL - If files missing, AMI will fail!**

**4. Verify database connection:**

```bash
# Test MySQL connection to RDS
mysql -h group3-database.cxcecm6wisku.ap-southeast-1.rds.amazonaws.com \
      -u admin \
      -pcloudproject \
      -e "SELECT DATABASE(); SHOW TABLES;" \
      Group3_db
```

**Expected output:**
```
+------------+
| DATABASE() |
+------------+
| Group3_db  |
+------------+
+-------------------------+
| Tables_in_Group3_db     |
+-------------------------+
| oc_address              |
| oc_product              |
| oc_customer             |
| oc_session              |
... (50+ tables)
+-------------------------+
```

**❌ TROUBLESHOOTING:**

| Problem | Check | Solution |
|---------|-------|----------|
| `ERROR 2003: Can't connect` | RDS endpoint | Copy from AWS Console → RDS → Databases |
| `ERROR 1045: Access denied` | Password wrong | Default: `cloudproject` (no spaces) |
| `ERROR 2002: Connection timed out` | Security Group | RDS SG must allow MySQL from EC2 SG |
| `Database ... doesn't exist` | Wrong database name | Check RDS: Database name = `Group3_db` |

**5. Test website locally:**

```bash
# Test via localhost
curl -I http://localhost/
# Expected: HTTP/1.1 200 OK ✅

# Check if OpenCart displays
curl -s http://localhost/ | grep -i "opencart"
# Expected: Should find "OpenCart" in HTML ✅

# Exit SSH
exit
```

---

**📋 STEP 2.4: PLAN MULTI-AZ ARCHITECTURE**

**Create architecture planning document:**

```powershell
cd "C:\Users\ASUS\Documents\Sever Systems - Cloud\Cloud-project\opencart-3.0.x.x\docs\architecture"

@"
# Multi-AZ Architecture Plan

## Current State (Day 1)
\`\`\`
AZ: ap-southeast-1a ONLY
├─ Public Subnet A (10.0.1.0/24)
│  └─ EC2-A (i-082cbe43b6ba19a6e)
└─ Private Subnet A (10.0.11.0/24)
   └─ RDS MySQL (Single-AZ)

Problem: If AZ-1a fails → ENTIRE SYSTEM DOWN! ❌
\`\`\`

## Target State (After Day 1 Hour 3-5)
\`\`\`
MULTI-AZ DESIGN:

AZ: ap-southeast-1a
├─ Public Subnet A (10.0.1.0/24)
│  └─ EC2-A (existing)
└─ Private Subnet A (10.0.11.0/24)
   └─ RDS MySQL (still Single-AZ, but accessible cross-AZ)

AZ: ap-southeast-1b (NEW!)
└─ Public Subnet B (10.0.2.0/24) ← Will create in Hour 3
   └─ EC2-B (NEW!) ← Will create from AMI

Application Load Balancer (Multi-AZ)
├─ Subnet A (AZ-1a) ✅
├─ Subnet B (AZ-1b) ✅
└─ Distributes traffic:
   ├─ 50% → EC2-A (AZ-1a)
   └─ 50% → EC2-B (AZ-1b)

Benefit:
✅ If AZ-1a fails → ALB routes 100% to EC2-B
✅ If EC2-A fails → ALB routes 100% to EC2-B
✅ RTO: 0 seconds (automatic failover)
\`\`\`

## Implementation Steps (Hour 3-5)

### Hour 3: Create EC2-B
- [ ] Create AMI from EC2-A (No reboot!)
- [ ] Create Public Subnet B in ap-southeast-1b (CIDR: 10.0.2.0/24)
- [ ] Launch EC2-B from AMI in Subnet B
- [ ] Verify EC2-B has all files (~75MB OpenCart code)
- [ ] Test database connection from EC2-B → RDS (cross-AZ)

### Hour 4-5: Create ALB
- [ ] Create Target Group (Group3-OpenCart-TG)
- [ ] Register both EC2s as targets
- [ ] Create ALB spanning both AZs
- [ ] Create ALB Security Group (allow HTTP from 0.0.0.0/0)
- [ ] Update EC2 Security Group (allow HTTP from ALB SG only)
- [ ] ⚠️ CRITICAL: Update config.php to use ALB DNS (not EC2 IP!)
- [ ] Test load balancing (50/50 distribution)

## Success Criteria
- ✅ 2 EC2 instances in different AZs
- ✅ Both showing "Healthy" in Target Group
- ✅ ALB DNS resolves and loads website
- ✅ CSS/JS load correctly (no 404 errors)
- ✅ Refresh 10 times → see both EC2 IPs in responses
- ✅ Can stop EC2-A → website still works via EC2-B

## Cost Impact
- AMI storage: Free (first 5 AMIs)
- EC2-B: Free tier (750 hours/month)
- ALB: ~$16-18/month (NOT free tier) ⚠️
- Cross-AZ traffic: ~$0.01/GB (negligible for testing)
"@ | Out-File -FilePath "multi-az-plan.md" -Encoding UTF8

# Display plan
Get-Content "multi-az-plan.md"
```

---

**✅ STEP 2 COMPLETION CHECKLIST:**

```
Infrastructure Inventory:
☑ VPC ID documented (vpc-xxxxxxxxx)
☑ Subnet IDs documented (public + private in AZ-1a)
☑ EC2-A details documented (Instance ID, IP, AZ)
☑ RDS details documented (Endpoint, DB name, credentials)
☑ Security Group IDs documented (WebServer + RDS)
☑ Inventory saved: infrastructure/d1-inventory.txt

EC2-A Verification:
☑ SSH connection successful
☑ Apache running (systemctl status httpd → active)
☑ PHP-FPM running (systemctl status php-fpm → active)
☑ OpenCart files present (~75MB in /var/www/html/)
☑ Theme files intact (catalog/view/theme/default/)
☑ Database connection working (RDS accessible)
☑ Website loads locally (curl localhost → HTTP 200)

Planning Documents:
☑ Multi-AZ architecture plan created
☑ Implementation steps documented
☑ Success criteria defined
☑ Cost impact analyzed

Readiness for Hour 3:
☑ Know EC2-A Instance ID for AMI creation
☑ Know VPC ID for subnet creation
☑ Know RDS endpoint for testing
☑ Have project.pem key for SSH
☑ Understand Multi-AZ target architecture
```

**⏱️ TIME CHECK:** Should take 15-20 minutes

**⏭️ NEXT:** HOUR 3-4 - Create EC2 AMI & Launch EC2-B in AZ-1b

---

#### 📌 **HOUR 3-4: Create EC2 AMI & Launch EC2-B** `[13:00-14:00]`

**🎯 Goals:**
- ✅ Create AMI from D1 EC2 (EC2-A)
- ✅ Launch EC2-B from AMI
- ✅ Verify both instances work

---

**📋 STEP 3.1: PREREQUISITES CHECK - CREATE AMI**

**Before creating AMI:**

```powershell
# Check 1: Know EC2-A Instance ID
# From AWS Console → EC2 → Instances
# Example: i-082cbe43b6ba19a6e

# Check 2: Verify EC2-A is running
# Status: Running ✅ (NOT stopped or terminated!)

# Check 3: Verify OpenCart files exist on EC2-A
# SSH and check:
ssh -i project.pem ec2-user@13.229.212.148
du -sh /var/www/html/
# Expected: 70-80M
exit

# Check 4: Free up disk space on EC2-A
# AMI creation requires ~5-10 GB free space
```

**❌ TROUBLESHOOTING:**

| Problem | Check | Solution |
|---------|-------|----------|
| EC2 status: Stopped | Instance state | Start EC2, wait until "Running" |
| EC2 status: Terminated | Can't recover | Use different EC2 as source |
| Disk space warning | df -h shows >90% | Clean logs: `sudo journalctl --vacuum-time=1d` |
| /var/www/html empty | Files missing | Re-deploy OpenCart before AMI creation |

---

**📋 STEP 3.2: CREATE AMI FROM EC2-A**

**🎯 Goal:** Capture complete snapshot of EC2-A including OS, apps, and OpenCart code

**1. Open EC2 Console:**
```
https://console.aws.amazon.com/ec2/
→ Instances (left sidebar)
```

**2. Select source EC2:**
```
☑ Click checkbox next to: Group3_WebServer1 (or your EC2-A name)
   ├─ Instance ID: i-082cbe43b6ba19a6e
   ├─ Status: Running ✅
   └─ AZ: ap-southeast-1a
```

**3. Initiate AMI creation:**
```
Top menu: Actions → Image and templates → Create image
```

**4. Configure AMI settings:**

```
Image name: Group3_OpenCart_Golden_AMI
  ⚠️ Use descriptive name (will create multiple AMIs if debugging)
  ⚠️ No spaces allowed - use underscores or hyphens

Image description: OpenCart 3.0.3.8 with Apache 2.4 + PHP 8.4 + AWS SDK configured
  (Optional but recommended for team documentation)

No reboot: ✅ CHECK THIS BOX!
  ⚠️ CRITICAL: Prevents EC2-A downtime
  ⚠️ If unchecked: EC2-A will stop → website offline for 5-10 minutes!
  
  Why "No reboot"?
  ├─ Faster: AMI creates in 3-5 min (vs 10-15 min with reboot)
  ├─ Zero downtime: Website stays online
  └─ Sufficient: Filesystem consistency maintained by Linux journal

Instance volumes:
  [Default settings - no changes needed]
  └─ Volume 1: /dev/xvda (30 GB gp3) ✅ Included automatically

Tags:
  [Optional]
  Key: Project, Value: Group3-OpenCart
  Key: Environment, Value: Production
```

**5. Create the AMI:**
```
Click: "Create image" button (bottom-right)
```

**Expected popup:**
```
✅ Successfully created AMI ami-0a1b2c3d4e5f6g7h8
```

**6. Monitor AMI creation progress:**

```
Left sidebar: Images → AMIs

Find: Group3_OpenCart_Golden_AMI
├─ AMI ID: ami-0xxxxxxxxxxxxx (copy this!)
├─ Status: Pending... ⏳
│  ↓ (Wait 3-5 minutes)
├─ Status: Available ✅
└─ Block devices: /dev/xvda (30 GB snapshot created)
```

**✅ VERIFICATION - AMI Ready:**

**Check 1: Status = Available**
```
AMIs → Group3_OpenCart_Golden_AMI
Status: Available ✅ (NOT Pending or Failed!)
```

**Check 2: Snapshot exists**
```
Left sidebar: Elastic Block Store → Snapshots
Find: Snapshot for ami-0xxxxx
├─ Size: ~8-12 GB (compressed from 30 GB volume)
├─ Status: Completed ✅
└─ Description: Created by CreateImage(i-082cbe...) for ami-0xxxxx
```

**Check 3: Copy AMI ID**
```
AMI ID: ami-0xxxxxxxxxxxxx
Save to inventory file for later use!
```

**⏱️ DURATION:** 3-5 minutes for AMI creation + verification

---

**📋 STEP 3.3: CREATE PUBLIC SUBNET B (Multi-AZ Requirement)**

**🎯 Goal:** Create subnet in ap-southeast-1b for EC2-B placement

**⚠️ CRITICAL:** EC2-B MUST be in different AZ than EC2-A for true Multi-AZ!

**1. Open VPC Console:**
```
https://console.aws.amazon.com/vpc/
→ Subnets (left sidebar)
```

**2. Click "Create subnet"**

**3. Configure subnet:**

```
VPC ID: Select "Group3_VPC" (vpc-xxxxxxxxx)
  ⚠️ MUST match EC2-A's VPC!

Subnet settings:

Subnet name: Group3_PublicSubnet_B

Availability Zone: ap-southeast-1b ✅
  ⚠️ CRITICAL: Different from EC2-A (which is in 1a)!
  ⚠️ If you select 1a by mistake → NOT Multi-AZ!

IPv4 CIDR block: 10.0.2.0/24
  ⚠️ Different from Subnet A (10.0.1.0/24)!
  ⚠️ Must not overlap with existing subnets
  
  Subnet Planning:
  ├─ Subnet A (1a): 10.0.1.0/24 → 10.0.1.1 to 10.0.1.254
  ├─ Subnet B (1b): 10.0.2.0/24 → 10.0.2.1 to 10.0.2.254 ✅
  └─ Private A:    10.0.11.0/24 (for RDS)
```

**4. Click "Create subnet"**

**5. Make subnet public (enable internet access):**

```
Subnets → Select "Group3_PublicSubnet_B"
→ Actions → Edit route table association

Route table ID: Select the PUBLIC route table
  (The one with route: 0.0.0.0/0 → igw-xxxxx)
  ⚠️ NOT the default/main route table!
  
→ Save

Then:
→ Actions → Edit subnet settings
→ ✅ Enable auto-assign public IPv4 address
→ Save
```

**✅ VERIFICATION - Subnet Ready:**
```
Subnet: Group3_PublicSubnet_B
├─ State: Available ✅
├─ AZ: ap-southeast-1b ✅
├─ CIDR: 10.0.2.0/24 ✅
├─ Available IPs: 251 (out of 256)
├─ Route table: Has 0.0.0.0/0 → igw-xxxxx ✅
└─ Auto-assign public IP: Yes ✅
```

---

**📋 STEP 3.4: LAUNCH EC2-B FROM AMI**

**🎯 Goal:** Clone EC2-A into different AZ using saved AMI

**1. Navigate to AMIs:**
```
EC2 Console → Images → AMIs (left sidebar)
```

**2. Select AMI:**
```
☑ Click: Group3_OpenCart_Golden_AMI
   ├─ AMI ID: ami-0xxxxxxxxxxxxx
   ├─ Status: Available ✅
   └─ Owner: <your account ID>
```

**3. Launch from AMI:**
```
Top-right button: "Launch instance from AMI"
```

**4. Configure instance - Name and tags:**

```
Name: Group3_WebServer2

Additional tags (optional but recommended):
├─ Key: Role,        Value: WebServer
├─ Key: AZ,          Value: ap-southeast-1b
└─ Key: Environment, Value: Production
```

**5. Application and OS Images:**
```
[Already pre-filled from AMI]
├─ AMI: Group3_OpenCart_Golden_AMI (ami-0xxxxx)
├─ Architecture: 64-bit (x86)
└─ Root device: EBS (30 GB)
```

**6. Instance type:**
```
⦿ t2.micro
  ├─ 1 vCPU, 1 GB RAM
  ├─ Free tier eligible ✅
  └─ Same as EC2-A ✅
```

**7. Key pair:**
```
Key pair name: project ✅
  ⚠️ MUST use same key as EC2-A!
  ⚠️ If you select "Create new key" → You'll have 2 different keys!
```

**8. Network settings - CRITICAL FOR MULTI-AZ:**

```
Click: "Edit" (top-right of Network settings box)

VPC: Group3_VPC ✅
  (Auto-selected from AMI subnet, verify it matches!)

Subnet: Group3_PublicSubnet_B ✅
  ⚠️ CRITICAL: Select the NEW subnet in ap-southeast-1b!
  ⚠️ Verify dropdown shows: "Group3_PublicSubnet_B | ap-southeast-1b"
  ⚠️ If it shows ap-southeast-1a → WRONG! Change it!

Auto-assign public IP: Enable ✅
  (EC2-B needs public IP for SSH access)

Firewall (security groups):
⦿ Select existing security group

Select: Group3_WebServer_SG (sg-xxxxxxxxx) ✅
  ⚠️ Same security group as EC2-A!
  ⚠️ Allows: HTTP (80) + SSH (22)
```

**9. Configure storage:**
```
[Already pre-filled from AMI]

1x Volume:
├─ Size: 30 GB ✅ (from AMI snapshot)
├─ Volume type: gp3
├─ Delete on termination: Yes ✅
└─ Encrypted: No (optional for free tier)
```

**10. Advanced details:**
```
[Optional - can leave defaults]

IAM instance profile: None (will add IAM Role later in Day 2)
```

**11. Summary review:**
```
Verify in right sidebar:
├─ Name: Group3_WebServer2 ✅
├─ AMI: Group3_OpenCart_Golden_AMI ✅
├─ Instance type: t2.micro ✅
├─ Subnet: Group3_PublicSubnet_B (ap-southeast-1b) ✅ CRITICAL!
└─ Security group: Group3_WebServer_SG ✅

Number of instances: 1
```

**12. Launch!**
```
Click: "Launch instance" (bottom-right)
```

**Expected:**
```
✅ Successfully initiated launch of instance: i-0yyyyyyyyyyyyyyyy

Click: "View all instances"
```

**⏱️ DURATION:** 2-3 minutes for EC2 to start

---

**📋 STEP 3: Verify EC2-B** ⚠️ CRITICAL - DO NOT SKIP!

**🎯 Goal:** Ensure EC2-B launched correctly with all OpenCart files

---

**VERIFICATION CHECKLIST - 5 TESTS**

**1️⃣ GET EC2-B IP & SSH ACCESS:**

```bash
# AWS Console → EC2 → Instances → Click "Group3_WebServer2"
# Copy: Public IPv4 address (e.g., 13.212.xxx.xxx)
# Replace EC2_B_IP below with actual IP!

# Open terminal/PowerShell and SSH to EC2-B
ssh -i project.pem ec2-user@13.212.xxx.xxx
# Replace 13.212.xxx.xxx with your EC2-B Public IP!

# Expected: Should see Amazon Linux 2023 welcome screen ✅
```

**❌ If SSH fails:**

| Error | Solution |
|-------|----------|
| `Permission denied (publickey)` | Wrong key - use `project.pem` from Day 1 |
| `Connection timed out` | Wrong IP, check AWS Console for correct IP |
| `Network unreachable` | Check Security Group allows SSH (port 22) |
| `Host key verification failed` | First SSH - type `yes` to accept |

---

**2️⃣ VERIFY AVAILABILITY ZONE (MUST be 1b!):**

```bash
# From EC2-B SSH session:

# Get AZ metadata
TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
curl -H "X-aws-ec2-metadata-token: $TOKEN" -s http://169.254.169.254/latest/meta-data/placement/availability-zone

# Expected output: ap-southeast-1b ✅
# If shows ap-southeast-1a → EC2-B is in WRONG AZ! TERMINATE & RECREATE!
```

---

**3️⃣ VERIFY SERVICES ARE RUNNING:**

```bash
# Check Apache
sudo systemctl status httpd
# Expected: active (running) ✅

# Check PHP-FPM
sudo systemctl status php-fpm
# Expected: active (running) ✅

# If either stopped, restart:
sudo systemctl start httpd
sudo systemctl start php-fpm
sudo systemctl enable httpd    # Auto-start on reboot
sudo systemctl enable php-fpm
```

---

**4️⃣ 🚨 CRITICAL - VERIFY OPENCART FILES EXIST! 🚨**

This is the MOST IMPORTANT test. Many EC2-B setups fail here!

```bash
# Check if theme files exist (catalog files)
ls -la /var/www/html/catalog/view/theme/
# Expected: Should show "default" folder ✅
# If EMPTY or ERROR → Files missing! See Fix below!

# Check if image files exist
ls -la /var/www/html/image/catalog/
# Expected: Should show product images (20+ files) ✅
# If EMPTY or ERROR → Files missing! See Fix below!

# Check total size of /var/www/html/
du -sh /var/www/html/
# Expected: 70-80 MB (or close to 75 MB from EC2-A) ✅
# If < 1 MB or only a few KB → Files missing! See Fix below!
```

**🚨 IF FILES ARE MISSING:**

This is a SERIOUS PROBLEM! The AMI did not include OpenCart files.

**QUICK DIAGNOSIS:**

```bash
# Check what files ARE there
ls -la /var/www/html/ | head -20

# If you see only basic Apache files (index.html) but NO:
# ✗ admin/ folder
# ✗ catalog/ folder
# ✗ config.php file
# ✗ system/ folder

# → AMI FAILED TO COPY FILES!
```

**⚠️ SOLUTION - Fix Missing Files (3 options):**

**OPTION A: Copy files from EC2-A (Fastest, 10-15 minutes)**

```bash
# From EC2-B SSH session:
# Use rsync to copy files from EC2-A

rsync -avz --delete ec2-user@10.0.1.xxx:/var/www/html/ /home/ec2-user/
# Replace 10.0.1.xxx with EC2-A PRIVATE IP from AWS Console!

# If rsync not installed:
sudo yum install rsync -y

# Then copy to correct location:
sudo cp -r /home/ec2-user/html/* /var/www/html/
sudo chown -R apache:apache /var/www/html/
sudo chmod -R 755 /var/www/html/
```

**OPTION B: Re-download from GitHub (15-20 minutes)**

```bash
# Clone the repository into /var/www/html/
cd /var/www
sudo git clone https://github.com/WEKONE-26/opencart-aws-group3.git temp-repo
sudo cp -r temp-repo/upload/* /var/www/html/
sudo chown -R apache:apache /var/www/html/
sudo chmod -R 755 /var/www/html/
sudo rm -rf temp-repo/
```

**OPTION C: Terminate & Recreate (Most reliable, 10 minutes)**

```bash
# From AWS Console:
# 1. Terminate this broken EC2-B (i-xxxxxxx)
# 2. Create NEW AMI from EC2-A:
#    - Name: Group3_OpenCart_Golden_AMI_v2
#    - No reboot: ✅ CHECKED!
# 3. Wait 3-5 minutes for AMI to complete
# 4. Launch EC2-B again from AMI v2 (same settings as before)
# 5. Verify files exist (test 4 again)
```

**RECOMMENDED:** Start with Option A (rsync from EC2-A) - fastest if it works!

---

**5️⃣ VERIFY DATABASE CONNECTION (Cross-AZ test):**

```bash
# From EC2-B SSH, connect to RDS
mysql -h group3-database.cxcecm6wisku.ap-southeast-1.rds.amazonaws.com \
      -u admin \
      -pcloudproject \
      -e "SELECT VERSION(); SELECT COUNT(*) as products FROM oc_product;" \
      Group3_db

# Expected output:
# +--------------------+
# | VERSION()          |
# +--------------------+
# | 8.4.7-mysql        |
# +--------------------+
# +----------+
# | products |
# +----------+
# |       19 |
# +----------+
```

**❌ If connection fails:**

| Error | Cause | Solution |
|-------|-------|----------|
| `ERROR 2003: Can't connect` | RDS endpoint wrong | Use exact endpoint from AWS RDS console |
| `ERROR 1045: Access denied` | Wrong password | Default password: `cloudproject` (no extra spaces) |
| `ERROR 2002: Connection timed out` | Security Group blocks MySQL | RDS SG must allow port 3306 from EC2 SG |

---

**6️⃣ VERIFY WEBSITE LOADS LOCALLY:**

```bash
# Test Apache is serving OpenCart
curl -I http://localhost/
# Expected: HTTP/1.1 200 OK ✅

# Check if OpenCart HTML is there
curl -s http://localhost/ | grep -i "opencart"
# Expected: Should find word "opencart" in HTML ✅

# More detailed check - look for admin panel
curl -s http://localhost/ | head -50
# Should show HTML with <head>, <body>, etc (NOT just empty page)
```

---

**✅ IF ALL 6 TESTS PASS:**

```
✅ EC2-B in correct AZ (ap-southeast-1b)
✅ Services running (Apache + PHP)
✅ OpenCart files present (~75 MB)
✅ Database connection works (cross-AZ)
✅ Website loads locally
✅ Ready for ALB integration!

Exit SSH session:
exit
```

---

**📋 FINAL HOUR 3-4 CHECKLIST:**

```
✅ AMI created from EC2-A
   ├─ Name: Group3_OpenCart_Golden_AMI
   ├─ Status: Available
   └─ AMI ID: ami-0xxxxxxxxxxxxx (saved for records)

✅ Public Subnet B created
   ├─ CIDR: 10.0.2.0/24
   ├─ AZ: ap-southeast-1b
   ├─ Auto-assign public IP: Enabled
   └─ Route table: Has 0.0.0.0/0 → IGW

✅ EC2-B launched from AMI
   ├─ Instance ID: i-yyyyyyyyyyyyy
   ├─ AZ: ap-southeast-1b ✅ (verified!)
   ├─ Subnet: Group3_PublicSubnet_B
   ├─ Security Group: Group3_WebServer_SG
   ├─ Key pair: project
   └─ Status: Running

✅ EC2-B verified
   ├─ SSH access works
   ├─ Services running (httpd + php-fpm)
   ├─ OpenCart files present (~75 MB)
   ├─ Database connection works (cross-AZ)
   ├─ Website loads locally
   └─ Ready for Multi-AZ ALB!
```

**⏱️ HOUR 3-4 TOTAL TIME:** 15-25 minutes (depending on if AMI needs recreation)

**⏭️ NEXT:** HOUR 4-5 - Create Application Load Balancer & Configure Multi-AZ Routing

---

#### 📌 **HOUR 4-5: Create Application Load Balancer** `[14:00-15:00]`

**🎯 OBJECTIVES:**
- ✅ Create Target Group to register EC2 instances
- ✅ Create Multi-AZ Application Load Balancer
- ✅ Update Security Groups for ALB routing
- ✅ Configure DNS URLs in OpenCart config files
- ✅ Test load balancing across both AZs

---

**📋 STEP 1: CREATE TARGET GROUP** (2-3 minutes)

**🎯 Purpose:** Target Group is the "pool" of EC2 instances that ALB will distribute traffic to

**1. Open AWS Console:**
```
https://console.aws.amazon.com/ec2/
→ Left sidebar: Target Groups (under Load Balancing section)
→ Click "Create target group"
```

**2. Step 1 - Basic configuration:**

```
Target type: ⦿ Instances (Not "IP addresses")

Target group name: Group3-OpenCart-TG
  └─ Naming convention: ProjectName_Service_Purpose

Protocol: HTTP (not HTTPS yet)
Port: 80

VPC: Group3_VPC ✅
  └─ Must match your EC2 instances' VPC!

Protocol version: HTTP1 (leave default)
```

**3. Step 2 - Health check settings:**

```
Health check protocol: HTTP
Health check path: /
  └─ ALB will regularly request "/" to check if EC2 is alive

Advanced health check settings:
├─ Healthy threshold: 2
│  └─ EC2 must pass 2 consecutive health checks to be "Healthy"
│
├─ Unhealthy threshold: 3
│  └─ EC2 marked "Unhealthy" after 3 failed checks
│
├─ Timeout: 5 seconds
│  └─ Wait max 5 sec for response
│
├─ Interval: 30 seconds
│  └─ Check every 30 seconds (so every 2.5 minutes total wait before marking unhealthy)
│
└─ Success codes: 200
   └─ Only HTTP 200 counts as "healthy" (not 301, 404, etc.)
```

**4. Step 3 - Register targets:**

```
Available instances:
├─ ☑ Group3_WebServer1 (EC2-A) - i-082cbe43b6ba19a6e
└─ ☑ Group3_WebServer2 (EC2-B) - i-XXXXXXX

Port: 80 (for both)

Click: "Include as pending below"
```

**5. Click "Create target group"**

**✅ Expected:**
```
✅ Target Group created: Group3-OpenCart-TG
✅ 2 targets pending
✅ Status will change to "Healthy" within 30-60 seconds

Wait for:
Target Groups → Group3-OpenCart-TG → Targets tab
├─ EC2-A: Status = Healthy ✅ (green checkmark)
└─ EC2-B: Status = Healthy ✅ (green checkmark)
```

**❌ If targets show "Unhealthy":**

| Issue | Check | Solution |
|-------|-------|----------|
| Status: `Unhealthy` | HTTP 200 response | Verify Apache running on EC2s: `sudo systemctl status httpd` |
| Status: `Unused` | Not registered | Go back to Step 3, register both EC2s |
| Health check timeout | Network/Security Group | EC2 SG should allow HTTP (80) from ALB SG (will fix in Step 3) |

---

**📋 STEP 2: CREATE APPLICATION LOAD BALANCER** (3-5 minutes)

**🎯 Purpose:** ALB distributes traffic across both EC2 instances in different AZs

**1. Create ALB:**

```
AWS Console → EC2 → Load Balancers (left sidebar)
→ Click "Create load balancer"
→ Select "Application Load Balancer"
```

**2. Basic configuration:**

```
Name: Group3-OpenCart-ALB
  └─ Naming: ProjectName_Service_Type

Scheme: ⦿ Internet-facing
  └─ Public accessible (not internal-only)

IP address type: IPv4
  └─ Use IPv4 (IPv6 not needed for this project)
```

**3. Network mapping - ⚠️ CRITICAL:**

```
VPC: Group3_VPC ✅

Availability Zones (MUST select BOTH!):
├─ ✅ ap-southeast-1a → Group3_PublicSubnet_A
│  └─ This is where EC2-A is
│
└─ ✅ ap-southeast-1b → Group3_PublicSubnet_B
   └─ This is where EC2-B is

⚠️ If you only select one AZ → NOT Multi-AZ! ❌
⚠️ Both must be checked!
```

**4. Security groups:**

```
Security Groups: Create new security group

Group3_ALB_SG:

Inbound rules:
├─ HTTP (80) from 0.0.0.0/0 ✅ (anyone on internet)
└─ HTTPS (443) from 0.0.0.0/0 ✅ (for future upgrade)

Outbound rules:
└─ All traffic to 0.0.0.0/0 ✅ (ALB can send to anywhere)
```

**5. Listeners and routing:**

```
Add listener:
├─ Protocol: HTTP
├─ Port: 80
├─ Default action: Forward to → Group3-OpenCart-TG
└─ Click "Add"
```

**6. Review and create:**

```
Summary check:
├─ Name: Group3-OpenCart-ALB ✅
├─ VPC: Group3_VPC ✅
├─ AZs: 1a + 1b ✅ (BOTH!)
├─ Security Group: Group3_ALB_SG ✅
├─ Listener: HTTP:80 → Group3-OpenCart-TG ✅
└─ Click "Create"
```

**✅ Expected:**
```
✅ ALB created successfully
✅ DNS name: Group3-OpenCart-ALB-xxxxx.ap-southeast-1.elb.amazonaws.com
✅ State: Provisioning... (wait 2 minutes)
✅ State: Active ✅

Copy DNS name for next steps!
```

---

**📋 STEP 3: UPDATE SECURITY GROUPS** ⚠️ CRITICAL STEP

**Problem:** EC2s currently accept HTTP from 0.0.0.0/0 (old Day 1 setup)
Now they should ONLY accept from ALB! Otherwise ALB becomes pointless.

**Solution:**

```
AWS Console → EC2 → Security Groups
→ Find: Group3_WebServer_SG

Current inbound rules (OLD):
├─ HTTP (80) from 0.0.0.0/0 ← DELETE THIS!
└─ SSH (22) from My IP

New inbound rules (NEW - Fixed):
├─ HTTP (80) from Group3_ALB_SG ← ADD THIS!
└─ SSH (22) from My IP

Steps:
1. Select: Group3_WebServer_SG
2. Inbound rules tab → Edit
3. DELETE the "HTTP from 0.0.0.0/0" rule
4. ADD new rule:
   ├─ Type: HTTP
   ├─ Protocol: TCP
   ├─ Port: 80
   ├─ Source: Custom → Select "Group3_ALB_SG"
   └─ Save
```

**✅ Verification:**

```
Group3_WebServer_SG inbound rules should now show:
├─ HTTP (80) from sg-xxxxxxxx (Group3_ALB_SG) ✅
└─ SSH (22) from 118.69.158.111/32 (your IP)
```

---

**📋 STEP 4: UPDATE CONFIG.PHP** 🚨 MOST CRITICAL STEP!

**⚠️ WHY THIS STEP MATTERS:**

Without this, OpenCart still thinks it's on EC2-A IP (13.229.212.148), so:
- CSS/JS files fail to load (404 errors)
- Images broken
- Admin panel might not work properly
- Links broken

**Root issue:** Security Group now blocks direct EC2 IP access (only allows ALB)

**Solution:** Update config.php to use ALB DNS instead of EC2 IP

**STEP 4A: Get ALB DNS:**

```
AWS Console → Load Balancers
→ Select: Group3-OpenCart-ALB
→ Copy DNS name: Group3-OpenCart-ALB-1956877542.ap-southeast-1.elb.amazonaws.com
```

**STEP 4B: Update EC2-A config:**

```bash
ssh -i project.pem ec2-user@13.229.212.148

# Backup BEFORE editing!
sudo cp /var/www/html/config.php /var/www/html/config.php.backup.$(date +%s)
sudo cp /var/www/html/admin/config.php /var/www/html/admin/config.php.backup.$(date +%s)

# Edit main config.php
sudo nano /var/www/html/config.php

# Find these lines (usually around line 12-15):
define('HTTP_SERVER', 'http://13.229.212.148/');
define('HTTPS_SERVER', 'http://13.229.212.148/');

# Replace with ALB DNS (NOTE: Replace XXXXX with your actual ALB DNS!):
define('HTTP_SERVER', 'http://Group3-OpenCart-ALB-1956877542.ap-southeast-1.elb.amazonaws.com/');
define('HTTPS_SERVER', 'http://Group3-OpenCart-ALB-1956877542.ap-southeast-1.elb.amazonaws.com/');

# Save: Press Ctrl+X, then Y, then Enter

# Edit admin config.php  
sudo nano /var/www/html/admin/config.php

# Find these 3 lines (around line 12-20):
define('HTTP_SERVER', 'http://13.229.212.148/admin/');
define('HTTPS_SERVER', 'http://13.229.212.148/admin/');
define('HTTP_CATALOG', 'http://13.229.212.148/');

# Replace ALL 3 with:
define('HTTP_SERVER', 'http://Group3-OpenCart-ALB-1956877542.ap-southeast-1.elb.amazonaws.com/admin/');
define('HTTPS_SERVER', 'http://Group3-OpenCart-ALB-1956877542.ap-southeast-1.elb.amazonaws.com/admin/');
define('HTTP_CATALOG', 'http://Group3-OpenCart-ALB-1956877542.ap-southeast-1.elb.amazonaws.com/');

# Save: Ctrl+X, Y, Enter

# Restart Apache to apply changes
sudo systemctl restart httpd

# Verify restart successful
sudo systemctl status httpd
# Should show: active (running) ✅

exit
```

**STEP 4C: Update EC2-B config (SAME changes!):**

```bash
# Get EC2-B public IP from AWS Console
ssh -i project.pem ec2-user@<EC2_B_PUBLIC_IP>

# Backup configs
sudo cp /var/www/html/config.php /var/www/html/config.php.backup.$(date +%s)
sudo cp /var/www/html/admin/config.php /var/www/html/admin/config.php.backup.$(date +%s)

# Edit main config - SAME changes as EC2-A
sudo nano /var/www/html/config.php
# Change lines 12-15 to use ALB DNS

# Edit admin config - SAME changes as EC2-A
sudo nano /var/www/html/admin/config.php
# Change lines 12-20 to use ALB DNS

# Restart Apache
sudo systemctl restart httpd

exit
```

---

**📋 STEP 5: TEST LOAD BALANCING** ✅

**TEST 1: Access via ALB DNS:**

```
1. Clear browser cache: Ctrl + Shift + Delete
2. Open URL: http://Group3-OpenCart-ALB-1956877542.ap-southeast-1.elb.amazonaws.com
   (Replace XXXXX with YOUR actual ALB DNS!)

Expected:
✅ Website loads (OpenCart homepage)
✅ Full theme displays (not broken layout)
✅ No "Connection refused" or timeout errors
```

**TEST 2: Check for CSS/JS errors:**

```
1. Open browser console: F12
2. Go to Console tab
3. Refresh page: F5

Expected:
✅ No red error messages
✅ No "ERR_CONNECTION_TIMED_OUT"
✅ All files load successfully
```

**TEST 3: Verify Load Balancing (Round Robin):**

```
1. SSH to EC2-A, check Apache access log:
   ssh -i project.pem ec2-user@13.229.212.148
   tail -f /var/log/httpd/access_log &
   exit

2. Refresh ALB page multiple times (10 times)
3. Check which EC2 IP logged requests (should alternate!)

Expected pattern:
├─ Request 1 → logged by 10.0.1.xxx (EC2-A)
├─ Request 2 → logged by 10.0.2.xxx (EC2-B)
├─ Request 3 → logged by 10.0.1.xxx (EC2-A)
├─ Request 4 → logged by 10.0.2.xxx (EC2-B)
... (50/50 alternating pattern) ✅

This proves Round Robin load balancing works!
```

**TEST 4: Verify Multi-AZ Failover (Optional advanced test):**

```
1. STOP EC2-A (temporarily):
   AWS Console → EC2 → Instances → Stop Instance i-082cbe43b6ba19a6e

2. Refresh ALB page in browser:
   http://Group3-OpenCart-ALB-XXX.amazonaws.com

Expected:
✅ Website still loads (no downtime!)
✅ EC2-B serving all traffic (100%)
✅ Proves Multi-AZ works!

3. START EC2-A again:
   AWS Console → Instances → Start Instance
   
4. Wait 30 sec for health checks to pass
5. Refresh page again - traffic should rebalance 50/50
```

---

**✅ FINAL DAY 1 VERIFICATION CHECKLIST:**

```
TARGET GROUP:
☑ Group3-OpenCart-TG created
☑ EC2-A: Status = Healthy ✅
☑ EC2-B: Status = Healthy ✅
☑ Port: 80

APPLICATION LOAD BALANCER:
☑ Group3-OpenCart-ALB created
☑ State: Active
☑ Network: Multi-AZ (1a + 1b) ✅
☑ DNS: Group3-OpenCart-ALB-XXXXX.amazonaws.com (saved!)
☑ Listener: HTTP:80 → Group3-OpenCart-TG ✅

SECURITY GROUPS:
☑ Group3_ALB_SG: Allows HTTP from 0.0.0.0/0 ✅
☑ Group3_WebServer_SG: HTTP (80) from ALB SG only ✅ (NOT from 0.0.0.0/0)
☑ EC2s: Can reach RDS MySQL on port 3306 ✅

CONFIG FILES:
☑ EC2-A config.php: HTTP_SERVER = ALB DNS ✅
☑ EC2-A admin/config.php: All URLs = ALB DNS ✅
☑ EC2-B config.php: HTTP_SERVER = ALB DNS ✅
☑ EC2-B admin/config.php: All URLs = ALB DNS ✅

WEBSITE TESTING:
☑ Access via ALB: Works ✅
☑ Full OpenCart theme displays ✅
☑ No CSS/JS 404 errors ✅
☑ Load balancing: Round Robin distribution ✅
☑ Direct EC2 IPs: Blocked by SG (as intended) ✅

INFRASTRUCTURE:
☑ 2 EC2 instances in different AZs (1a, 1b) ✅
☑ 1 RDS database (Single-AZ, in 1a) ✅
☑ Both EC2s can connect to RDS (cross-AZ) ✅
☑ ALB distributes 50/50 traffic ✅
☑ Automatic failover works (stop one EC2, traffic routes to other) ✅
```

**⏱️ HOUR 4-5 TOTAL TIME:** 20-30 minutes

**🎉 END OF DAY 1!**

```
DAY 1 ACHIEVEMENTS:
✅ GitHub repository setup (OpenCart code pushed)
✅ Multi-AZ deployment (EC2-A in 1a, EC2-B in 1b)
✅ Application Load Balancer (traffic distribution)
✅ Health checks (automatic failover)
✅ Config files updated (ALB DNS integration)
✅ Load balancing verified (Round Robin)
✅ Zero-downtime failover tested

READY FOR DAY 2:
→ S3 for static assets (images)
→ CloudFront CDN (global distribution)
→ Database sessions (cross-EC2 sharing)
→ CI/CD pipeline (GitHub Actions)
```

**Save Day 1 progress to Git:**

```bash
cd "C:\Users\ASUS\Documents\Sever Systems - Cloud\Cloud-project\opencart-3.0.x.x"

# Add documentation
git add docs/
git add infrastructure/

# Commit
git commit -m "Day 1 Complete: Multi-AZ ALB deployment with config updates"

# Push
git push origin main

echo "✅ Day 1 progress saved to GitHub!"
```

---

### 📅 **DAY 2: S3, CloudFront & Sessions** `[Saturday Dec 21 - 5 hours]`

**🎯 DAY 2 OBJECTIVES:**
- ✅ Add S3 bucket for static asset storage (images, CSS, JS)
- ✅ Enable CloudFront CDN for global distribution (400+ edge locations)
- ✅ Setup IAM roles for secure EC2→S3 access (no hardcoded credentials!)
- ✅ Integrate AWS SDK into OpenCart for automatic S3 uploads
- ✅ Configure database sessions for cross-EC2 state sharing
- ✅ Test Multi-AZ session persistence

**⏰ TIMELINE:** 5 hours (9:00-14:00)

**📊 WHY DAY 2 IS CRITICAL:**

```
DAY 1 Problem - Broken Images on EC2-B:
───────────────────────────────────────
Admin uploads image to EC2-A:
  ├─ File saved: /var/www/html/image/product.jpg
  ├─ Customer #1 via ALB → EC2-A: ✅ Works
  └─ Customer #2 via ALB → EC2-B: ❌ 404 (file not synced!)

ROOT CAUSE: Each EC2 has separate local disk!

DAY 2 Solution - Centralized S3 + Global CDN:
──────────────────────────────────────────────
Image uploaded → S3 bucket (single source of truth!)
              → CloudFront caches at 400+ edge locations
              → Delivered to customers in 5-10ms
              → Works for EC2-A AND EC2-B simultaneously ✅

PLUS: Database Sessions:
└─ User logs into EC2-A
└─ Session stored in RDS (shared database!)
└─ ALB switches to EC2-B
└─ EC2-B reads session from RDS
└─ User still logged in! ✅ (no random logouts!)
```

---
   ☐ Both can connect to RDS (19 products)

2. Target Group:
   ☐ Group3-OpenCart-TG created
   ☐ Both EC2s registered as targets
   ☐ Both showing "Healthy" status
   ☐ Health check path: / (HTTP 200)

3. Application Load Balancer:
   ☐ Group3-OpenCart-ALB created
   ☐ Listener: HTTP:80 → Forward to TG
   ☐ Network mapping: BOTH AZs (1a + 1b) ⚠️
   ☐ State: Active
   ☐ DNS: Group3-OpenCart-ALB-XXXXX.elb.amazonaws.com

4. Security Groups:
   ☐ Group3_ALB_SG: HTTP from 0.0.0.0/0
   ☐ Group3_WebServer_SG: HTTP from ALB_SG only
   ☐ Group3_RDS_SG: MySQL from WebServer_SG only

5. Config Files:
   ☐ EC2-A config.php: HTTP_SERVER = ALB DNS ✅
   ☐ EC2-A admin/config.php: All URLs = ALB DNS ✅
   ☐ EC2-B config.php: HTTP_SERVER = ALB DNS ✅
   ☐ EC2-B admin/config.php: All URLs = ALB DNS ✅

✅ FUNCTIONAL TESTING:

6. Website Access:
   ☐ http://ALB-DNS/ loads OpenCart homepage
   ☐ Full theme/CSS displayed (not plain HTML)
   ☐ Browser Console (F12): No errors
   ☐ All CSS/JS load from ALB DNS (not EC2 IP)

7. Load Balancing:
   ☐ Refresh 10 times → traffic distributed 50/50
   ☐ Can see both EC2 IPs in responses:
      - ip-10-0-1-xxx (EC2-A)
      - ip-10-0-2-xxx (EC2-B)

8. Database Connectivity:
   ☐ Products page shows 19 items
   ☐ Both EC2s can query RDS
   ☐ Cross-AZ connection works (EC2-B → RDS)

✅ DOCUMENTATION:

9. Save inventory:
   ☐ EC2-A IP, Instance ID, AZ documented
   ☐ EC2-B IP, Instance ID, AZ documented
   ☐ ALB DNS documented
   ☐ Screenshots taken (at least 15)
```

**Common Issues & Quick Fixes:**

```
❌ ISSUE: Website shows plain HTML (no CSS)
✅ FIX: Update config.php to use ALB DNS (see above)

❌ ISSUE: EC2-B has no files (ls shows empty)
✅ FIX: Terminate EC2-B, create new AMI, relaunch

❌ ISSUE: Target shows "Unhealthy"
✅ FIX: Check Security Group allows ALB → EC2 (port 80)

❌ ISSUE: Can't access ALB DNS
✅ FIX: Wait 2-3 minutes for targets to become Healthy

❌ ISSUE: Only 1 EC2 gets traffic
✅ FIX: Check both targets registered and Healthy
```

**Save progress to GitHub:**

```bash
# Document what we did
cd "C:\Users\ASUS\Documents\Sever Systems - Cloud\Cloud-project"

cat > docs/day1-progress.md << 'EOF'
# Day 1 Progress - COMPLETE ✅

## Completed
- ✅ GitHub repo setup
- ✅ EC2-B launched from AMI in ap-southeast-1b
- ✅ Application Load Balancer created (Multi-AZ)
- ✅ Both EC2s registered and healthy
- ✅ Config.php updated to use ALB DNS (CRITICAL!)
- ✅ Website accessible via ALB with full theme
- ✅ Load balancing verified (50/50 distribution)

## Resources Created
- AMI: Group3_OpenCart_Golden_AMI (or v2 if recreated)
- EC2-A: i-082cbe43b6ba19a6e (ap-southeast-1a)
- EC2-B: i-XXXXXXX (ap-southeast-1b) ← DIFFERENT AZ!
- Target Group: Group3-OpenCart-TG
- ALB: Group3-OpenCart-ALB
- Security Group: Group3_ALB_SG

## Access Points
- ALB URL: http://Group3-OpenCart-ALB-[ID].ap-southeast-1.elb.amazonaws.com
- EC2-A Direct: http://13.229.212.148/ (blocked by SG now)
- EC2-B Direct: http://EC2_B_IP/ (blocked by SG now)

## Architecture Achieved
- ✅ Multi-AZ deployment (EC2s in 1a + 1b)
- ✅ Auto failover via ALB health checks
- ✅ Round Robin load distribution
- ✅ Single-AZ RDS (cost optimized)
- ✅ Cross-AZ database connection working

## Critical Lessons Learned
1. ALWAYS verify EC2-B has files after AMI launch
2. MUST update config.php to use ALB DNS (not EC2 IP)
3. ALB must span BOTH AZs for true Multi-AZ
4. Security Groups: EC2s only accept traffic from ALB SG

## Screenshots Checklist
[✅] EC2-A details (showing AZ-1a)
[✅] EC2-B details (showing AZ-1b)
[✅] Target Group with 2 healthy targets
[✅] ALB details (Multi-AZ mapping)
[✅] Website via ALB DNS (full theme)
[✅] Browser console (no errors)
[✅] Load balancing test (showing both IPs)
EOF

git add .
git commit -m "Day 1 complete: Multi-AZ ALB with config.php fixed"
git push
```

**End of Day 1!** 🎉

**NEXT: Day 2 will add S3, CloudFront, and DB sessions for true stateless architecture!**

---

### 📅 **DAY 2: S3, CloudFront & Sessions** `[Saturday Dec 21 - 5 hours]`

#### 📌 **HOUR 1-2: S3 Bucket & CloudFront Setup** `[09:00-11:00]`

**🎯 OBJECTIVES:**
- ✅ Create S3 bucket for OpenCart static assets
- ✅ Configure bucket for public image delivery
- ✅ Enable CloudFront global CDN
- ✅ Test S3 and CloudFront functionality

---

**💡 THE MULTI-AZ STORAGE PROBLEM**

**DAY 1 Issue:**
```
Admin uploads image on EC2-A:
  ├─ File saved: /var/www/html/image/catalog/product.jpg
  └─ Database: image_url = "/image/catalog/product.jpg"

Customer request #1 via ALB → EC2-A:
  ├─ Local file exists (/var/www/html/image/)
  ├─ Image displays ✅
  └─ Response time: 50-100ms

Customer request #2 via ALB → EC2-B (Round Robin!):
  ├─ Local file doesn't exist on EC2-B ❌
  ├─ Returns 404 error (image not found)
  └─ Customer sees broken image icon

THE PROBLEM:
Each EC2 instance has separate local storage!
Local disks are NOT shared between EC2-A and EC2-B
```

**DAY 2 SOLUTION: S3 + CloudFront**

```
Admin uploads image:
  ├─ OpenCart FileManager receives POST
  ├─ Server uploads to S3 bucket
  ├─ Database: image_url = "https://cdn.cloudfront.net/...jpg"
  └─ S3 has single source of truth ✅

Customer request via ALB (can hit ANY EC2):
  ├─ Browser requests: https://cdn.cloudfront.net/...jpg
  ├─ CloudFront caches image at 400+ edge locations
  ├─ Response time: 20-50ms (cached!) vs 100ms (origin)
  └─ Both EC2-A and EC2-B serve same image ✅

BENEFITS:
✅ Single source of truth (S3)
✅ Global distribution (CloudFront)
✅ Faster delivery (CDN caching)
✅ Cost optimized (no server disk usage)
✅ Unlimited scalability
```

---

**📋 STEP 1: Create S3 Bucket** (3 minutes)

**1️⃣ Navigate to S3 Console:**

```
https://console.aws.amazon.com/s3/
```

**2️⃣ Click "Create bucket" button**

**3️⃣ Configure Bucket Settings:**

```
GENERAL CONFIGURATION:
├─ Bucket name: group3-opencart-static
│  └─ ⚠️ MUST be globally unique (across all AWS accounts)
│  └─ Use lowercase, numbers, hyphens only
│  └─ No spaces, underscores, or capital letters!
│
├─ AWS Region: ap-southeast-1 (Singapore)
│  └─ ⚠️ CRITICAL: Same region as EC2 & RDS (reduces latency!)
│  └─ Cross-region transfer costs money
│
└─ Object Ownership: ACLs disabled ✅ (recommended)
   └─ Simplifies permissions (uses bucket policy instead)

VERSIONING:
└─ Bucket Versioning: Disable ✅ (not needed for this project)

ENCRYPTION:
└─ Server-side encryption: SSE-S3 ✅ (Amazon managed keys, no cost)
```

**4️⃣ Block Public Access Settings - ⚠️ CRITICAL STEP**

**❌ WRONG (if you check this):**
```
☑ Block all public access  ← This blocks your bucket!
  └─ No one can access images
  └─ Website shows broken images
```

**✅ CORRECT (uncheck this):**
```
☐ Block all public access  ← UNCHECK THIS!
```

**Individual checkboxes** (leave ALL UNCHECKED):
```
☐ Block public access to buckets and objects granted through new ACLs
☐ Block public access to buckets and objects granted through any ACLs  
☐ Block public access to buckets and objects granted through new policies
☐ Block public and cross-account access through any policies
```

**Why?** Product images MUST be publicly readable by customers' browsers!

**5️⃣ Acknowledge Public Access:**

```
☑ I acknowledge that the current settings might result in 
  this bucket and objects becoming public
  
(This is CORRECT - we WANT it public for images!)
```

**6️⃣ Click "Create bucket"**

**✅ Expected Output:**

```
✅ Success
Bucket "group3-opencart-static" has been created.
```

---

**📋 STEP 2: Configure Bucket Policy** (2 minutes)

**🎯 Purpose:** Allow public read access (no authentication needed for image viewing)

**1️⃣ Navigate to bucket:**

```
S3 Console
→ Buckets list
→ Click: group3-opencart-static
→ Permissions tab
→ Bucket policy section
→ Click "Edit"
```

**2️⃣ Paste Bucket Policy JSON:**

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::group3-opencart-static/*"
        }
    ]
}
```

**📖 Policy Breakdown:**

```
"Version": "2012-10-17"
  → Standard AWS policy language version (always use this)

"Statement": [{ ... }]
  → Array of permission statements

"Sid": "PublicReadGetObject"
  → Statement ID (just a label for documentation)

"Effect": "Allow"
  → Permission type (Allow | Deny)
  → "Allow" = grant this permission

"Principal": "*"
  → WHO gets this permission?
  → "*" = ANYONE (everyone on internet!)
  
"Action": "s3:GetObject"
  → WHAT can they do?
  → "s3:GetObject" = READ files (not upload/delete)
  → Users can ONLY read, nothing else
  
"Resource": "arn:aws:s3:::group3-opencart-static/*"
  → WHICH files?
  → "group3-opencart-static/*" = ALL files in bucket
  → /* means "all objects" (recursive)
```

**3️⃣ Click "Save changes"**

**✅ Expected Output:**

```
✅ Successfully edited bucket policy
```

**❌ If error: "Invalid principal in policy"**
→ Solution: Make sure "Principal" is exactly: "*" (with quotes)

---

**📋 STEP 3: Create Folder Structure** (2 minutes)

**🎯 Purpose:** Organize images logically for maintainability

**1️⃣ Create "catalog" folder:**

```
S3 Console → group3-opencart-static
→ Click "Create folder"
  ├─ Folder name: catalog
  └─ Click "Create folder"
```

**2️⃣ Create "products" subfolder inside "catalog":**

```
→ Double-click "catalog" folder to enter
→ Click "Create folder"
  ├─ Folder name: products
  └─ Click "Create folder"
```

**3️⃣ Go back and create other folders:**

```
← Click back button (or breadcrumb) to return to root

Create "blog" folder:
→ Click "Create folder" → name: blog → Create

Create "cache" folder:
→ Click "Create folder" → name: cache → Create
```

**4️⃣ Verify final structure:**

```
S3 Console → group3-opencart-static should show:

Folder structure:
├─ blog/             (for blog post images)
├─ cache/            (for cache/temp files)
└─ catalog/
   └─ products/      (for product images)
   
Total: 3 root folders, 1 subfolder
```

**✅ Folder structure ready for image organization!**

---

**📋 STEP 4: Create CloudFront Distribution** (10 minutes)

**🎯 Purpose:** Global CDN to cache and deliver images from nearest edge location

**⏱️ Note:** CloudFront deployment takes 5-10 minutes. Start now!

**1️⃣ Open CloudFront Console:**

```
https://console.aws.amazon.com/cloudfront/
```

**2️⃣ Click "Create distribution" button**

**3️⃣ STEP 1: Choose Origin**

```
Origin type: ⦿ Amazon S3 bucket

Select S3 bucket:
├─ Click "Browse S3"
├─ Select: group3-opencart-static ✅
└─ Click "Choose"

Result:
├─ Origin domain: group3-opencart-static.s3.ap-southeast-1.amazonaws.com
└─ Origin path: (leave empty)

S3 Access:
└─ ☑ Origin access control (recommended)
   └─ Simplifies security (no public bucket policy needed in theory,
      but ours is already public, so this is redundant but fine)
```

**4️⃣ STEP 2: Default Cache Behavior**

```
Viewer protocol policy:
├─ ⦿ Redirect HTTP to HTTPS ✅
│  └─ Customers access via HTTPS (secure + preferred)
│  └─ HTTP requests auto-redirect to HTTPS
│
Allowed HTTP methods:
├─ ⦿ GET, HEAD (recommended) ✅
│  └─ Only read operations
│  └─ EC2 will upload via SDK (not CloudFront)
│
Cache policy:
├─ ⦿ CachingOptimized ✅
│  ├─ Default TTL: 24 hours (1 day)
│  ├─ Max TTL: 365 days (1 year)
│  └─ CloudFront caches aggressively (good for product images)
│
Origin request policy:
└─ ⦿ CORS-S3Origin ✅
   └─ Allows CORS requests if needed
```

**5️⃣ STEP 3: Settings**

```
Viewer protocol policy: Redirect HTTP to HTTPS ✅

Root object: (leave empty)
  → Not needed (we serve images, not single root file)

Description: (optional)
  └─ CloudFront CDN for OpenCart product images

Price class:
├─ ⦿ Use all edge locations (default) ✅
│  └─ Best performance (images closest to user)
│  └─ Costs more ($0.08-0.12 per GB vs $0.085 per GB)
│
Enable: IPv6 ✅
  └─ Support modern IPv6 clients

Web ACL: None (WAF disabled)
  └─ Not needed for image delivery (no DDoS protection needed)
```

**6️⃣ STEP 4: Create Distribution**

```
Summary should show:
├─ Origin domain: group3-opencart-static.s3.ap-southeast-1.amazonaws.com ✅
├─ Viewer protocol: Redirect HTTP to HTTPS ✅  
├─ Cache policy: CachingOptimized ✅
├─ Price class: Use all edge locations ✅
└─ Click "Create distribution"
```

**⏳ Wait for Deployment (~5-10 minutes)**

```
CloudFront Distributions list shows:
├─ Status: Deploying... 🔄
├─ Progress bar increases
└─ Status: Enabled ✅ (when done)
```

**🎉 While waiting, continue to STEP 5!**

---

**📋 STEP 5: Test S3 & CloudFront Delivery** (5 minutes)

**🎯 Purpose:** Verify S3 and CloudFront are working correctly before integration with OpenCart

**1️⃣ Prepare test image:**

```
Find ANY image on your computer:
├─ Screenshot (easiest)
├─ Photo from Downloads folder
├─ PNG or JPG preferred
└─ Filename: test-product.png (or .jpg)

Maximum size: 5MB (for testing)
```

**2️⃣ Upload image to S3:**

```
S3 Console
→ Buckets → group3-opencart-static
→ Click folder: catalog/products/
→ Click "Upload" button
→ Click "Add files"
→ Select your test image
→ Click "Upload" button

Wait for:
✅ Upload complete (progress bar disappears)
✅ File shows in list: test-product.png
→ Click "Close"
```

**3️⃣ TEST 1 - S3 Direct Access:**

```
Open browser, visit this URL directly:

https://group3-opencart-static.s3.ap-southeast-1.amazonaws.com/catalog/products/test-product.png
```

**Expected result:**
```
✅ Image displays in browser
✅ Prove bucket policy works (public readable)
```

**❌ If you see error:**
```
Error: Access Denied
  → Solution: Bucket policy not applied
  → Go back to STEP 2, verify policy saved
```

**4️⃣ WAIT: Check CloudFront Status**

```
AWS Console → CloudFront → Distributions

Look for your distribution:
├─ Status: Enabled ✅ (must show "Enabled", not "Deploying")
├─ Domain name: dxxxxxxx.cloudfront.net (copy this!)
│  └─ Example: d1234abcd.cloudfront.net
└─ ⏰ If still "Deploying", wait 2-3 more minutes
```

**5️⃣ TEST 2 - CloudFront URL:**

**Replace `dxxxxxxx` with YOUR CloudFront domain:**

```
https://dxxxxxxx.cloudfront.net/catalog/products/test-product.png

Example (YOURS WILL BE DIFFERENT):
https://d1234abcd.cloudfront.net/catalog/products/test-product.png
```

**Expected result:**
```
✅ Image displays in browser
✅ First request slower (~500ms) - CloudFront fetches from S3
✅ Second request faster (~50ms) - CloudFront cache hit
```

**6️⃣ TEST 3 - Verify CloudFront Caching:**

```
Open DevTools: Press F12
→ Network tab
→ Refresh page: F5
→ Click on "test-product.png" request
→ Click "Headers" tab
→ Scroll down to "Response Headers"

Look for "X-Cache" header:

FIRST REQUEST:
  X-Cache: Miss from cloudfront
  └─ CloudFront fetched from S3 (no cache yet)

SECOND REQUEST (refresh again):
  X-Cache: Hit from cloudfront  ← ✅ PROVES CACHING WORKS!
  └─ CloudFront served from cache
  └─ Response time: < 100ms
```

**Check timing:**

```
First request:
  └─ Time: 300-500ms

Second request:
  └─ Time: 20-100ms

Improvement: 5-10x faster! ✅
```

---

**🔧 TROUBLESHOOTING TABLE**

| Issue | Symptom | Cause | Solution |
|-------|---------|-------|----------|
| **S3 Access Denied** | `<Error><Code>AccessDenied</Code>` | Bucket policy not applied | Go to S3 Console → Permissions → Bucket policy → Verify "Principal": "*" |
| **CloudFront still Deploying** | Status shows "Deploying..." (20+ min) | Normal delays | Wait up to 15 minutes. If longer, create new distribution |
| **CloudFront 403 Forbidden** | `<Error><Code>AccessDenied</Code>` at cloudfront.net | Origin not accessible | Check S3 bucket public access (block public access must be unchecked) |
| **Slow first request** | Takes > 2 seconds | Normal (S3 cold fetch) | Expected on first request. Cache will speed up subsequent requests |
| **Image cached forever** | Old image still showing after delete | Cache TTL | Wait 24 hours (default TTL) or invalidate cache |

---

**✅ HOUR 1-2 VERIFICATION CHECKLIST**

```
Infrastructure Setup:
☑ S3 bucket created: group3-opencart-static
☑ Region: ap-southeast-1 (Singapore)
☑ Block public access: ☐ (UNCHECKED) ✅
☑ Bucket policy: Public read enabled ✅
  └─ Action: "s3:GetObject"
  └─ Principal: "*"

Folder Structure:
☑ Folders created:
  ├─ /blog/
  ├─ /cache/
  └─ /catalog/products/

CloudFront CDN:
☑ Distribution created
☑ Status: Enabled ✅
☑ Domain saved: dxxxxxxx.cloudfront.net
☑ Origin: group3-opencart-static.s3.ap-southeast-1.amazonaws.com ✅
☑ Cache policy: CachingOptimized ✅

Testing Results:
☑ Test image uploaded: test-product.png
☑ S3 URL works: https://group3-opencart-static.s3.ap-southeast-1.amazonaws.com/... ✅
☑ CloudFront URL works: https://dxxxxxxx.cloudfront.net/...
  └─ First request: 300-500ms (cache miss)
  └─ Second request: 20-100ms (cache hit) ✅
☑ X-Cache header shows: Hit from cloudfront ✅

Performance Metrics:
☑ CloudFront response time: < 100ms (cached) ✅
☑ Improvement over S3: 5-10x faster ✅
☑ Global edge locations ready: 400+ deployed ✅
```

**⏱️ TIME TAKEN:** 25-35 minutes
**🎉 HOUR 1-2 COMPLETE!**

**Next:** HOUR 2-3 - IAM Role for EC2 → S3 Access

---

#### 📌 **HOUR 2-3: IAM Role for EC2 → S3 Access** `[11:00-13:00]`

**🎯 OBJECTIVES:**
- ✅ Create IAM role with S3 permissions
- ✅ Attach role to both EC2 instances
- ✅ Test S3 upload/download from EC2 command line
- ✅ Verify AWS SDK auto-authentication works

---

**💡 WHY IAM ROLES INSTEAD OF ACCESS KEYS?**

**❌ THE OLD (INSECURE) WAY:**

```php
// BAD: Hardcoded credentials in PHP code
$s3Client = new S3Client([
    'credentials' => [
        'key'    => 'AKIAIOSFODNN7EXAMPLE',      // ⚠️ Hardcoded!
        'secret' => 'wJalrXUtnFEMI/K7MDENG...'   // ⚠️ Visible!
    ]
]);

PROBLEMS:
├─ Keys visible in source code (security breach!)
├─ Keys in Git commits (forever in history!)
├─ Leaked if developer leaves company
├─ Manual key rotation every 90 days (tedious)
├─ Single key across all EC2 instances
├─ No audit trail (who did what?)
└─ All developers need same credentials
```

**✅ THE NEW (SECURE) WAY - IAM ROLE:**

```php
// GOOD: No credentials needed (auto-provided by EC2 role)
$s3Client = new S3Client([
    'region'  => 'ap-southeast-1',
    'version' => 'latest'
    // NO credentials! AWS SDK auto-discovers from EC2 role ✅
]);

BENEFITS:
├─ Zero credentials in code! ✅
├─ Auto-rotating temporary credentials (every 6 hours)
├─ Different permissions per EC2 instance
├─ Full audit trail via CloudTrail
├─ Easy per-instance revocation
├─ AWS security best practice ✅
└─ Credentials never visible in logs/code
```

---

**📋 STEP 1: CREATE IAM ROLE** (3 minutes)

**1️⃣ Open IAM Console:**

```
https://console.aws.amazon.com/iam/
```

**2️⃣ Navigate to Roles section:**

```
IAM Dashboard (left sidebar)
→ Click: "Roles"
→ Click: "Create role" button
```

**3️⃣ SELECT TRUSTED ENTITY:**

```
Trusted entity type: ⦿ AWS service ✅
  └─ This role will be used BY AWS services (like EC2)

Service or use case:
  ├─ Select "EC2" from the list
  └─ Description: "Allows EC2 instances to call AWS services on your behalf"

Click: "Next"
```

**4️⃣ ATTACH PERMISSIONS POLICY:**

```
Search box: Type "S3"

Search Results (filter for S3):
├─ AmazonS3FullAccess         ← ☑ SELECT THIS ONE
├─ AmazonS3ReadOnlyAccess     (not enough - we need upload)
└─ AmazonS3OutpostsFullAccess (not needed)

Policy Summary:
├─ Type: AWS managed policy
├─ Permissions: s3:*, s3-object-lambda:*
├─ Effect: Allow
└─ ✅ This allows ANY S3 action on ANY bucket
```

**⚠️ FOR PRODUCTION - Use Least Privilege:**

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": [
      "s3:PutObject",       // Upload
      "s3:GetObject",       // Download
      "s3:DeleteObject"     // Delete
    ],
    "Resource": "arn:aws:s3:::group3-opencart-static/*"
  }]
}
```

**For learning project, AmazonS3FullAccess is fine.**

**5️⃣ NAME AND CREATE ROLE:**

```
Role Details:

Role name: Group3_EC2_S3_Role
  └─ Naming: Project_Service_Purpose_Role

Description: Allows Group3 EC2 instances to upload images to S3 bucket

Tags: (optional, skip for now)

Review:
├─ Trusted entities: AWS service: ec2.amazonaws.com ✅
├─ Permissions: AmazonS3FullAccess ✅
└─ Click "Create role"
```

**✅ ROLE CREATED!**

```
Confirmation:
├─ Role name: Group3_EC2_S3_Role
├─ Role ARN: arn:aws:iam::796174527414:role/Group3_EC2_S3_Role
└─ Instance profile ARN: arn:aws:iam::796174527414:instance-profile/Group3_EC2_S3_Role
   └─ Auto-created for EC2 attachment ✅
```

---

**📋 STEP 2: ATTACH ROLE TO EC2-A** (2 minutes)

**1️⃣ Navigate to EC2 Instances:**

```
AWS Console → EC2 → Instances
→ Instances list
```

**2️⃣ SELECT EC2-A:**

```
☑ Select the instance labeled:
  ├─ Name: Group3_WebServer1 (or OpenCart-EC2-A)
  ├─ Instance ID: i-082cbe43b6ba19a6e (example)
  └─ AZ: ap-southeast-1a
```

**3️⃣ MODIFY IAM ROLE:**

```
Top toolbar: Click "Actions" dropdown
→ Security
→ Modify IAM role

Dialog Box:
├─ Current IAM role: - (none currently)
├─ IAM role dropdown: [Select Group3_EC2_S3_Role]
│  └─ Should appear in list
│  └─ Click to select
│
└─ Click "Update IAM role"
```

**✅ ROLE ATTACHED TO EC2-A!**

```
Confirmation:
├─ Role successfully updated
├─ Instance details page refreshes
└─ Security tab shows: IAM role = Group3_EC2_S3_Role ✅
```

---

**📋 STEP 3: ATTACH ROLE TO EC2-B** (2 minutes)

**REPEAT STEP 2 for EC2-B:**

```
EC2 Instances list
→ ☑ Select EC2-B
  ├─ Name: Group3_WebServer2 (or OpenCart-EC2-B)
  ├─ Instance ID: i-02a745f790cb007a3 (example)
  └─ AZ: ap-southeast-1b

→ Actions → Security → Modify IAM role
→ IAM role: Group3_EC2_S3_Role ✅
→ Update IAM role

Result:
├─ EC2-A: IAM role = Group3_EC2_S3_Role ✅
└─ EC2-B: IAM role = Group3_EC2_S3_Role ✅
```

---

**📋 STEP 4: TEST S3 ACCESS FROM EC2-A** (10 minutes)

**⏱️ WAIT 30 SECONDS:**
Role attachment needs time to propagate. Wait before testing!

**1️⃣ SSH TO EC2-A:**

```powershell
# From Windows PowerShell
ssh -i project.pem ec2-user@13.229.212.148
```

**Replace IP with YOUR EC2-A public IP!**

**2️⃣ VERIFY EC2 ROLE:**

```bash
# Check IAM role is attached
curl http://169.254.169.254/latest/meta-data/iam/info

# Expected output:
{
  "Code" : "Success",
  "LastUpdated" : "2025-12-22T10:30:45Z",
  "InstanceProfileArn" : "arn:aws:iam::796174527414:instance-profile/Group3_EC2_S3_Role",
  "InstanceProfileId" : "AIPAW..."
}

✅ Proves IAM role is attached and accessible!
```

**3️⃣ TEST 1: LIST ALL S3 BUCKETS**

```bash
aws s3 ls
```

**Expected output:**

```
2025-12-21 14:05:53 group3-opencart-static
```

**✅ SUCCESS! EC2 can access S3 via IAM role (no credentials needed!)**

**❌ If error:**

```
Unable to locate credentials
→ Solution: Role not attached or credentials not propagated yet
→ Wait 30 more seconds, try again
→ Or restart EC2 instance
```

**4️⃣ TEST 2: LIST BUCKET CONTENTS**

```bash
aws s3 ls s3://group3-opencart-static/
```

**Expected output:**

```
                           PRE blog/
                           PRE cache/
                           PRE catalog/
2025-12-21 15:25:28     122753 test-product.png
```

**✅ Shows folders and files!**

**5️⃣ TEST 3: UPLOAD FILE TO S3**

```bash
# Create test file
echo "Test upload from EC2-A via IAM role at $(date)" > test-ec2a.txt

# Upload to S3
aws s3 cp test-ec2a.txt s3://group3-opencart-static/test-ec2a.txt
```

**Expected output:**

```
upload: ./test-ec2a.txt to s3://group3-opencart-static/test-ec2a.txt
```

**✅ File uploaded successfully!**

**6️⃣ TEST 4: VERIFY FILE IN S3**

```bash
# Check file exists in S3
aws s3 ls s3://group3-opencart-static/ | grep test-ec2a
```

**Expected output:**

```
2025-12-22 11:35:14         65 test-ec2a.txt
```

**✅ File size ~65 bytes (correct!)**

---

**📋 STEP 5: TEST CROSS-AZ FILE SHARING** (5 minutes)

**🎯 GOAL:** Prove EC2-B can read files uploaded by EC2-A (shared S3)

**1️⃣ EXIT EC2-A:**

```bash
exit
```

**2️⃣ SSH TO EC2-B:**

```powershell
ssh -i project.pem ec2-user@<EC2_B_PUBLIC_IP>
```

**Replace with YOUR EC2-B public IP!**

**3️⃣ VERIFY ROLE ON EC2-B:**

```bash
curl http://169.254.169.254/latest/meta-data/iam/info

# Should show: Group3_EC2_S3_Role ✅
```

**4️⃣ DOWNLOAD FILE FROM EC2-A:**

```bash
# EC2-B downloads file that EC2-A uploaded
aws s3 cp s3://group3-opencart-static/test-ec2a.txt test-from-eca.txt

# Verify contents
cat test-from-eca.txt
```

**Expected output:**

```
download: s3://group3-opencart-static/test-ec2a.txt to ./test-from-eca.txt
Test upload from EC2-A via IAM role at Sun Dec 22 11:35:14 UTC 2025
```

**✅ CROSS-AZ FILE SHARING WORKS!**

**WHAT THIS PROVES:**
```
EC2-A (AZ 1a) ──upload──→ S3 (shared across regions)
                              ↓
                        EC2-B (AZ 1b) ←──download──
                        
Both EC2s can access SAME S3 bucket!
Perfect for Multi-AZ deployment ✅
```

**5️⃣ CLEANUP TEST FILES:**

```bash
# Delete test files
aws s3 rm s3://group3-opencart-static/test-ec2a.txt
rm test-from-eca.txt

# Verify deletion
aws s3 ls s3://group3-opencart-static/ | grep test
# Should show nothing (file deleted) ✅

exit
```

---

**🔧 TROUBLESHOOTING TABLE**

| Issue | Symptom | Cause | Solution |
|-------|---------|-------|----------|
| **Unable to locate credentials** | Error when running `aws s3 ls` | IAM role not attached or not propagated | 1) Go to EC2 instance 2) Actions → Security → Modify IAM role → Select Group3_EC2_S3_Role 3) Wait 30 seconds, try again |
| **No access to bucket** | `An error occurred (AccessDenied)` | IAM policy doesn't have S3 permissions | 1) Check role has AmazonS3FullAccess policy 2) Re-attach policy if missing |
| **aws CLI not found** | `aws: command not found` | AWS CLI not installed | Amazon Linux 2 has it by default. If missing: `sudo yum install aws-cli -y` |
| **Wrong bucket name** | `An error occurred (NoSuchBucket)` | Typo in bucket name | Verify bucket name in S3 console (should be `group3-opencart-static`) |
| **Role shows "none"** | Instance details show "- (no role)" | Role attachment didn't complete | Refresh page (F5) or restart instance and wait 60 seconds |
| **Credentials expired** | Error after 6 hours | Temporary credentials expired (normal) | AWS SDK auto-refreshes. Just retry command. |

---

**✅ HOUR 2-3 VERIFICATION CHECKLIST**

```
IAM Configuration:
☑ IAM role created: Group3_EC2_S3_Role
☑ Role ARN saved: arn:aws:iam::796174527414:role/Group3_EC2_S3_Role
☑ Instance profile created: arn:aws:iam::796174527414:instance-profile/Group3_EC2_S3_Role
☑ Trust policy: AWS service: ec2.amazonaws.com ✅
☑ Permissions: AmazonS3FullAccess ✅

EC2 Attachment:
☑ EC2-A attached: Group3_EC2_S3_Role ✅
  └─ Verified via curl metadata API
☑ EC2-B attached: Group3_EC2_S3_Role ✅
  └─ Verified via curl metadata API

AWS CLI Testing:
☑ EC2-A: aws s3 ls → Lists buckets ✅
  ├─ Shows: group3-opencart-static
  └─ No credentials in command!
  
☑ EC2-A: aws s3 ls s3://bucket/ → Lists objects ✅
☑ EC2-A: aws s3 cp → Upload works ✅
  └─ Uploaded test file successfully

☑ EC2-B: aws s3 cp → Download works ✅
  └─ Downloaded file from EC2-A

Cross-AZ Verification:
☑ EC2-A uploaded: test-ec2a.txt
☑ EC2-B downloaded: same file ✅
☑ File contents preserved ✅
☑ Sharing verified: Both EC2s access same S3 ✅

Security Validation:
☑ Zero hardcoded credentials ✅
☑ No access keys in code ✅
☑ Auto-rotating temp credentials ✅
☑ CloudTrail will log all S3 actions ✅
☑ Per-instance role control ✅
```

**📊 ARCHITECTURE AFTER HOUR 2-3:**

```
EC2-A (AZ 1a)
├─ IAM Role: Group3_EC2_S3_Role
├─ EC2 metadata API → temporary credentials (auto-refresh every 6 hours)
└─ AWS SDK reads credentials from metadata
   └─ No code changes needed! ✅

EC2-B (AZ 1b)
├─ IAM Role: Group3_EC2_S3_Role (same role)
├─ EC2 metadata API → temporary credentials (different credentials!)
└─ AWS SDK reads credentials from metadata

Both EC2s ↓ (same S3 bucket, different temporary credentials)

S3: group3-opencart-static
├─ Permission: Both EC2s can PutObject (upload) ✅
├─ Permission: Both EC2s can GetObject (download) ✅
├─ Permission: Both EC2s can DeleteObject (cleanup) ✅
└─ Single source of truth for shared files!
```

**⏱️ TIME TAKEN:** 20-25 minutes
**🎉 HOUR 2-3 COMPLETE!**

**Next:** HOUR 3-4 - AWS SDK Integration & S3 Upload
                     └─ This role will be used by AWS services like EC2

Use case:
├─ Service or use case: ⦿ EC2
└─ Description: "Allows EC2 instances to call AWS services on your behalf"

Click "Next"
```

**4. Attach permissions policy:**

```
Permissions policies (search box): Type "S3"

Search results:
├─ AmazonS3FullAccess         ← Select this one ☑
├─ AmazonS3ReadOnlyAccess     (not enough - we need upload)
└─ AmazonS3OutpostsFullAccess (not needed)

Policy summary:
├─ Type: AWS managed policy
├─ Description: "Provides full access to all buckets via the AWS Management Console"
└─ Permissions: s3:*, s3-object-lambda:* ✅

Click "Next"
```

**⚠️ PRODUCTION NOTE:**

```
For production, use least-privilege policy:

{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": [
      "s3:PutObject",
      "s3:GetObject",
      "s3:DeleteObject"
    ],
    "Resource": "arn:aws:s3:::group3-opencart-static/*"
  }]
}

For learning, AmazonS3FullAccess is fine.
```

**5. Name and create role:**

```
Role details:
├─ Role name: Group3_EC2_S3_Role
│             ↑ Naming convention: Project_Service_Purpose_Role
│
├─ Description: Allows Group3 EC2 instances to upload images to S3 bucket
│
├─ Step 1: Trusted entities:
│  └─ Trusted entity type: AWS service: ec2.amazonaws.com ✅
│
├─ Step 2: Permissions:
│  └─ Permissions policies (1): AmazonS3FullAccess ✅
│
└─ Tags (optional): Skip for now

Click "Create role"
```

**6. Verify role created:**

```
IAM → Roles → Search: "Group3_EC2_S3_Role"

Role ARN: arn:aws:iam::796174527414:role/Group3_EC2_S3_Role
          ↑ Your account ID will be different

Instance profile ARN: arn:aws:iam::796174527414:instance-profile/Group3_EC2_S3_Role
                      ↑ Auto-created for EC2 attachment ✅
```

---

**📋 STEP 2: Attach IAM Role to EC2 Instances**

**Attach to EC2-A:**

**1. Navigate to EC2 Console:**

```
EC2 Console → Instances
           → Select: ☑ OpenCart-EC2-A (i-082cbe43b6ba19a6e)
```

**2. Modify IAM role:**

```
Actions button (top)
└─ Security
   └─ Modify IAM role

IAM role settings:
├─ Current IAM role: - (none attached)
├─ IAM role: [Dropdown]
│  └─ Select: Group3_EC2_S3_Role ✅
│
└─ Click "Update IAM role"
```

**3. Verify attachment:**

```
Instance details (EC2-A):
└─ Security tab
   └─ IAM role: Group3_EC2_S3_Role ✅
```

**Repeat for EC2-B:**

```
EC2 Console → Instances
           → Select: ☑ OpenCart-EC2-B (i-02a745f790cb007a3)
           → Actions → Security → Modify IAM role
           → IAM role: Group3_EC2_S3_Role
           → Update IAM role

Verify: Instance details → Security tab → IAM role: Group3_EC2_S3_Role ✅
```

**Both EC2s should now have the role attached! ✅**

---

**📋 STEP 3: Test S3 Access from EC2-A**

**1. SSH to EC2-A:**

```powershell
ssh -i project.pem ec2-user@13.229.212.148
```

**Replace IP with your EC2-A public IP!**

**2. Test 1 - List all S3 buckets:**

```bash
aws s3 ls
```

**Expected output:**

```
2025-12-21 14:05:53 group3-opencart-static
```

**If error:** `Unable to locate credentials`  
**Solution:** IAM role not attached. Go back to Step 2.

**3. Test 2 - List bucket contents:**

```bash
aws s3 ls s3://group3-opencart-static/
```

**Expected output:**

```
                           PRE blog/
                           PRE cache/
                           PRE catalog/
2025-12-21 15:25:28     122753 Screenshot 2025-06-12 095458.png
```

**Shows folders and any files ✅**

**4. Test 3 - Upload a test file:**

```bash
echo "Test upload from EC2-A via IAM role" > test-ec2a.txt
aws s3 cp test-ec2a.txt s3://group3-opencart-static/test-ec2a.txt
```

**Expected output:**

```
upload: ./test-ec2a.txt to s3://group3-opencart-static/test-ec2a.txt
```

**5. Test 4 - Verify upload:**

```bash
aws s3 ls s3://group3-opencart-static/ | grep test-ec2a
```

**Expected output:**

```
2025-12-21 16:13:14         36 test-ec2a.txt
```

**File size ~36 bytes ✅**

**6. Test 5 - Download from EC2-B (Cross-AZ test):**

```bash
# Exit EC2-A
exit

# SSH to EC2-B
ssh -i project.pem ec2-user@<EC2-B-PUBLIC-IP>

# Download file uploaded by EC2-A
aws s3 cp s3://group3-opencart-static/test-ec2a.txt test-downloaded.txt

# Verify contents
cat test-downloaded.txt
```

**Expected output:**

```
download: s3://group3-opencart-static/test-ec2a.txt to ./test-downloaded.txt
Test upload from EC2-A via IAM role
```

**Cross-AZ file sharing works! ✅**

**7. Test 6 - Clean up test files:**

```bash
# From EC2-B
aws s3 rm s3://group3-opencart-static/test-ec2a.txt
rm test-downloaded.txt
```

**Expected output:**

```
delete: s3://group3-opencart-static/test-ec2a.txt
```

---

**🔧 TROUBLESHOOTING:**

| **Error** | **Cause** | **Solution** |
|-----------|-----------|--------------|
| `Unable to locate credentials` | IAM role not attached | EC2 Console → Instance → Actions → Security → Modify IAM role → Attach Group3_EC2_S3_Role |
| `An error occurred (AccessDenied)` | Policy missing S3 permissions | IAM Console → Roles → Group3_EC2_S3_Role → Permissions → Verify AmazonS3FullAccess attached |
| `aws: command not found` | AWS CLI not installed | Run: `sudo yum install aws-cli -y` (Amazon Linux 2 has it by default) |
| `Could not connect to the endpoint URL` | Wrong region or bucket name | Verify bucket exists: `aws s3 ls` (should show group3-opencart-static) |
| `upload failed: Unable to locate credentials` | Role not propagated yet | Wait 30 seconds, try again (IAM role attachment takes ~10-30 sec) |

---

**✅ VERIFICATION CHECKLIST:**

```
Hour 2-3 Deliverables:

IAM Configuration:
□ IAM role created: Group3_EC2_S3_Role
□ Policy attached: AmazonS3FullAccess ✅
□ Role ARN saved: arn:aws:iam::796174527414:role/Group3_EC2_S3_Role

EC2 Attachment:
□ EC2-A IAM role: Group3_EC2_S3_Role ✅
□ EC2-B IAM role: Group3_EC2_S3_Role ✅

Testing Results:
□ EC2-A: aws s3 ls → Lists buckets ✅
□ EC2-A: aws s3 cp → Upload works ✅
□ EC2-B: aws s3 cp → Download works ✅
□ Cross-AZ file sharing verified ✅
□ No hardcoded credentials needed ✅
```

---

## 📌 HOUR 3-4: AWS SDK Integration & S3 Upload (13:00-15:00)

**Goals:**
- ✅ Install Composer (PHP dependency manager)
- ✅ Install AWS SDK for PHP
- ✅ Create S3Helper class for image uploads
- ✅ Integrate S3 upload into OpenCart FileManager
- ✅ Test end-to-end upload → S3 → CloudFront
- ✅ Create admin user for testing

**Architecture After Hour 3-4:**

```
Admin uploads image:
├─ Browser → ALB → EC2 (A or B)
├─ FileManager.php receives upload
├─ Calls S3Helper class
├─ S3Helper → AWS SDK → S3 PutObject
├─ S3 stores file, returns URL
├─ Database saves CloudFront URL
└─ Frontend loads image from CloudFront ✅
```

---

### **STEP 1: Install Composer on EC2-A**

**📋 PREREQUISITES CHECK:**
```bash
# SSH to EC2-A
ssh -i project.pem ec2-user@13.229.212.148

# ✅ VERIFY: You're on EC2-A (check hostname)
hostname
# Expected: ip-10-0-1-xxx.ap-southeast-1.compute.internal
# ❌ If shows ip-10-0-2-xxx → You're on EC2-B, exit and SSH to EC2-A!

# ✅ VERIFY: PHP is installed
php --version
# Expected: PHP 8.4.2 or higher
# ❌ If "command not found" → Install PHP first!

# ✅ VERIFY: You're in correct directory
cd /var/www/html
pwd
# Expected: /var/www/html

# ✅ VERIFY: OpenCart files exist
ls -la index.php
# Expected: -rw-r--r-- 1 apache apache 9xxx Dec 20 10:30 index.php
# ❌ If "No such file" → Wrong directory or OpenCart not installed!
```

**📦 INSTALL COMPOSER:**
```bash
# Download and install Composer
sudo curl -sS https://getcomposer.org/installer | sudo php -- --install-dir=/usr/local/bin --filename=composer
```

**Expected installation output:**
```
All settings correct for using Composer
Downloading...

Composer (version 2.9.2) successfully installed to: /usr/local/bin/composer
Use it: php /usr/local/bin/composer
```

**✅ VERIFICATION:**
```bash
# Test Composer command
composer --version
```

**Expected output:**
```
Composer version 2.9.2 2025-12-15 14:13:42
PHP version 8.4.2
```

**❌ TROUBLESHOOTING:**

| Problem | Cause | Solution |
|---------|-------|----------|
| `composer: command not found` | Not in PATH | Use full path: `/usr/local/bin/composer --version` |
| `curl: command not found` | curl not installed | `sudo yum install curl -y` then retry |
| `Permission denied` | Not using sudo | Add `sudo` before command |
| Download fails | Network issue | Check internet: `ping google.com`, check Security Group allows outbound HTTPS |

---

### **STEP 2: Install AWS SDK for PHP**

**📋 PREREQUISITES CHECK:**
```bash
# ✅ VERIFY: Still in /var/www/html
pwd
# Expected: /var/www/html

# ✅ VERIFY: Composer working
composer --version
# Expected: Composer version 2.9.2...
# ❌ If error → Go back to STEP 1

# ✅ VERIFY: No existing vendor folder (fresh install)
ls -la vendor/
# Expected: "No such file or directory" (OK) OR existing vendor/ (also OK)
```

**📦 INSTALL AWS SDK:**
```bash
# Install AWS SDK via Composer
sudo composer require aws/aws-sdk-php
```

**⏱️ EXPECTED DURATION: 1-2 minutes**

**Expected installation output:**
```
Loading composer repositories with package information
Updating dependencies
Lock file operations: 14 installs, 0 updates, 0 removals
  - Locking aws/aws-crt-php (v1.2.7)
  - Locking aws/aws-sdk-php (3.369.0)
  - Locking guzzlehttp/guzzle (7.x)
  - Locking guzzlehttp/promises (2.x)
  - Locking guzzlehttp/psr7 (2.x)
  - Locking mtdowling/jmespath.php (2.x)
  - Locking psr/http-client (1.x)
  - Locking psr/http-factory (1.x)
  - Locking psr/http-message (2.x)
  - Locking ralouphie/getallheaders (3.x)
  - Locking symfony/deprecation-contracts (3.x)
  - Locking symfony/polyfill-mbstring (1.x)
  - Locking symfony/polyfill-php80 (1.x)
  - Locking symfony/polyfill-php81 (1.x)
Installing dependencies from lock file
Package operations: 14 installs, 0 updates, 0 removals
  - Installing aws/aws-sdk-php (3.369.0): Downloading (100%)
Generating autoload files
14 packages you are using are looking for funding.
Use the `composer fund` command to find out more!
```

**✅ VERIFICATION (3 checks):**

**Check 1: Vendor folder exists**
```bash
ls -la vendor/
```
**Expected:**
```
drwxr-xr-x  6 root   root    4096 Dec 22 10:30 .
drwxr-xr-x 15 apache apache  4096 Dec 22 10:25 ..
drwxr-xr-x  3 root   root    4096 Dec 22 10:30 aws
-rw-r--r--  1 root   root     176 Dec 22 10:30 autoload.php
drwxr-xr-x  2 root   root    4096 Dec 22 10:30 composer
drwxr-xr-x 14 root   root    4096 Dec 22 10:30 guzzlehttp
...
```

**Check 2: AWS SDK specifically**
```bash
ls -la vendor/aws/
```
**Expected:**
```
drwxr-xr-x 4 root root 4096 Dec 22 10:30 .
drwxr-xr-x 6 root root 4096 Dec 22 10:30 ..
drwxr-xr-x 3 root root 4096 Dec 22 10:30 aws-crt-php
drwxr-xr-x 5 root root 4096 Dec 22 10:30 aws-sdk-php
```
**✅ Both folders present = SUCCESS**

**Check 3: Count total packages**
```bash
ls vendor/ | wc -l
```
**Expected:** 14-20 (should be > 10)
**❌ If < 5 → Installation incomplete, retry!**

**❌ TROUBLESHOOTING:**

| Problem | Cause | Solution |
|---------|-------|----------|
| `composer: command not found` | Composer not in PATH | Use: `sudo /usr/local/bin/composer require aws/aws-sdk-php` |
| Download timeout | Network slow | Retry with: `sudo composer require aws/aws-sdk-php --timeout=900` |
| `Your requirements could not be resolved` | PHP version too old | Check PHP: `php -v`, need PHP 7.2+ |
| Permission denied writing vendor/ | Wrong permissions | `sudo chown -R apache:apache /var/www/html/` then retry |
| Only 1-2 packages installed | Interrupted download | Delete vendor/: `sudo rm -rf vendor/` and reinstall |

---

### **STEP 3: Create S3Helper Class**

```bash
sudo nano /var/www/html/system/library/s3helper.php
```

**Paste this code:**

```php
<?php
/**
 * Group3 S3 Helper Class
 * Handles image uploads to S3 bucket using IAM role authentication
 * No hardcoded AWS credentials needed!
 */

require '/var/www/html/vendor/autoload.php';

use Aws\S3\S3Client;
use Aws\Exception\AwsException;

class S3Helper {
    private $s3Client;
    private $bucket = 'group3-opencart-static';
    private $cloudfront = 'https://dt1v1qszn6knb.cloudfront.net';
    private $region = 'ap-southeast-1';
    
    public function __construct() {
        // IAM role provides credentials automatically - NO manual credentials!
        $this->s3Client = new S3Client([
            'version' => 'latest',
            'region'  => $this->region
            // No 'credentials' key needed - EC2 IAM role provides them!
        ]);
    }
    
    /**
     * Upload file to S3
     * @param string $localFile - Local file path
     * @param string $s3Key - S3 object key
     * @return string|false - CloudFront URL on success, false on failure
     */
    public function uploadImage($localFile, $s3Key) {
        try {
            $result = $this->s3Client->putObject([
                'Bucket' => $this->bucket,
                'Key'    => $s3Key,
                'SourceFile' => $localFile,
                // ⚠️ ACL line REMOVED - causes error with ACLs disabled!
                // Bucket policy handles public read access
                'ContentType' => mime_content_type($localFile),
                'CacheControl' => 'max-age=31536000, public, immutable'
            ]);
            
            // Return CloudFront URL
            return $this->cloudfront . '/' . $s3Key;
            
        } catch (AwsException $e) {
            error_log("S3 Upload Error: " . $e->getMessage());
            return false;
        }
    }
    
    /**
     * Generate S3 key with date-based folder structure
     */
    public function generateKey($filename) {
        $ext = pathinfo($filename, PATHINFO_EXTENSION);
        $unique = uniqid();
        $date = date('Y/m/d');
        
        return "catalog/products/{$date}/{$unique}.{$ext}";
    }
}
```

**Save:** Ctrl+X, Y, Enter

**⚠️ CRITICAL NOTE - ACL Error Fix:**

If you see this error:
```
AccessControlListNotSupported: The bucket does not allow ACLs
```

**Cause:** S3 bucket created with "ACLs disabled (recommended)" but code has `'ACL' => 'public-read'`

**Fix:** Remove the ACL parameter from putObject() call (already done in code above)

---

### **STEP 4: Create Test Script**

```bash
sudo nano /var/www/html/test-s3-upload.php
```

**Paste this code:**

```php
<?php
require '/var/www/html/vendor/autoload.php';
require '/var/www/html/system/library/s3helper.php';

echo "Uploading to S3...\n";

// Create test file
$testContent = 'Test upload from EC2! Time: ' . date('Y-m-d H:i:s');
$testFile = '/tmp/test-upload-' . time() . '.txt';
file_put_contents($testFile, $testContent);

// Upload
$s3 = new S3Helper();
$s3Key = $s3->generateKey('test-upload.txt');
$cloudfrontUrl = $s3->uploadImage($testFile, $s3Key);

if ($cloudfrontUrl) {
    echo "✅ Upload successful!\n";
    echo "CloudFront URL: $cloudfrontUrl\n";
} else {
    echo "❌ Upload failed!\n";
}

unlink($testFile);
?>
```

**Save:** Ctrl+X, Y, Enter

---

### **STEP 5: Test S3 Upload**

```bash
php /var/www/html/test-s3-upload.php
```

**Expected SUCCESS output:**
```
Uploading to S3...
✅ Upload successful!
CloudFront URL: https://dt1v1qszn6knb.cloudfront.net/catalog/products/2025/12/22/69483417a0a4f.txt
```

**Verification:**
1. Open CloudFront URL in browser → Should show test file content ✅
2. S3 Console → group3-opencart-static/catalog/products/2025/12/22/ → File exists ✅

---

### **STEP 6: Sync Files to EC2-B**

**Option 1: Direct SSH to EC2-B (Recommended if Security Group blocks EC2-A → EC2-B SSH)**

```powershell
# From Windows PowerShell
ssh -i project.pem ec2-user@47.129.37.211
```

**On EC2-B, repeat Steps 1-2:**

```bash
cd /var/www/html

# Install Composer
sudo curl -sS https://getcomposer.org/installer | sudo php -- --install-dir=/usr/local/bin --filename=composer
composer --version

# Install AWS SDK
sudo composer require aws/aws-sdk-php
ls -la vendor/aws/

# Create S3Helper (same code as EC2-A)
sudo nano /var/www/html/system/library/s3helper.php
# Paste SAME code from EC2-A
# Save

# Create test script
sudo nano /var/www/html/test-s3-upload.php
# Paste SAME test script
# Save

# Test
php /var/www/html/test-s3-upload.php
# ✅ Upload successful! (different CloudFront URL)
```

**Both EC2s can upload to S3! ✅**

---

### **STEP 7: Integrate into OpenCart FileManager**

**On EC2-B (or EC2-A):**

```bash
# Backup original file
sudo cp admin/controller/common/filemanager.php admin/controller/common/filemanager.php.backup

# Find upload section
grep -n "move_uploaded_file" admin/controller/common/filemanager.php
# Output: 277:                                    move_uploaded_file($file['tmp_name'], $directory . '/' . $filename);

# Edit file
sudo nano +277 admin/controller/common/filemanager.php
```

**Replace line 277 with this block:**

```php
                                if (!$json) {
                                        // Upload to S3
                                        require_once(DIR_SYSTEM . 'library/s3helper.php');
                                        $s3 = new S3Helper();
                                        $s3Key = $s3->generateKey($filename);
                                        $cloudfrontUrl = $s3->uploadImage($file['tmp_name'], $s3Key);
                                        
                                        // Also save locally for compatibility
                                        move_uploaded_file($file['tmp_name'], $directory . '/' . $filename);
                                        
                                        if ($cloudfrontUrl) {
                                                error_log("Image uploaded to S3: " . $cloudfrontUrl);
                                        }
                                }
```

**Save:** Ctrl+X, Y, Enter

---

### **STEP 8: Sync FileManager to Other EC2**

**Option A: Via S3 (if SSH blocked):**

```bash
# On EC2-B
sudo cp admin/controller/common/filemanager.php /tmp/
aws s3 cp /tmp/filemanager.php s3://group3-opencart-static/temp/

# On EC2-A
aws s3 cp s3://group3-opencart-static/temp/filemanager.php /tmp/
sudo cp /tmp/filemanager.php admin/controller/common/filemanager.php
```

**Option B: Direct SCP (if SSH works):**

```bash
# From EC2-A
scp -i ~/.ssh/id_rsa admin/controller/common/filemanager.php ec2-user@10.0.2.6:/tmp/
ssh ec2-user@10.0.2.6 "sudo mv /tmp/filemanager.php /var/www/html/admin/controller/common/"
```

---

### **STEP 9: Create Admin User for Testing**

**🎯 WHY THIS STEP?**
OpenCart installed but admin user was never created during installation. Need admin account to test S3 upload via admin panel.

---

**📋 STEP 9.1: CHECK IF ADMIN USER EXISTS**

```bash
# SSH to EC2-A (if not already connected)
ssh -i project.pem ec2-user@13.229.212.148

# Connect to RDS database
mysql -h group3-database.cxcecm6wisku.ap-southeast-1.rds.amazonaws.com -u admin -p
```
**Prompt:** `Enter password:`  
**Type:** `cloudproject` (password is hidden, just type and press Enter)

**Expected connection output:**
```
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 42
Server version: 8.4.7 Source distribution

mysql>
```

**❌ TROUBLESHOOTING - Connection Failed:**

| Error | Cause | Solution |
|-------|-------|----------|
| `ERROR 2003: Can't connect to MySQL server` | Wrong endpoint | Verify RDS endpoint in AWS Console → RDS → Databases |
| `ERROR 1045: Access denied for user 'admin'` | Wrong password | Check password (default: `cloudproject`) |
| `ERROR 2002: Connection timed out` | Security Group blocks | EC2 Security Group must allow outbound MySQL (3306) |

**Now check user table:**
```sql
USE Group3_db;
```
**Expected:** `Database changed`

```sql
-- Check if admin user exists
SELECT user_id, username, firstname, lastname, email, status FROM oc_user;
```

**SCENARIO A - User exists:**
```
+---------+----------+-----------+----------+---------------------+--------+
| user_id | username | firstname | lastname | email               | status |
+---------+----------+-----------+----------+---------------------+--------+
|       1 | admin    | Admin     | User     | admin@localhost.com |      1 |
+---------+----------+-----------+----------+---------------------+--------+
1 row in set (0.001 sec)
```
**✅ ACTION:** Skip to STEP 10 (user already exists!)

**SCENARIO B - Empty table:**
```
Empty set (0.001 sec)
```
**❌ ACTION:** Continue to STEP 9.2 to create user

---

**📋 STEP 9.2: GENERATE PASSWORD HASH**

**ℹ️ WHY:** OpenCart uses triple SHA1 with salt for password security (not bcrypt/argon2)

```bash
# Exit MySQL first
exit;

# Create password hash generator script
cat > /tmp/create-admin.php << 'EOF'
<?php
$password = 'admin123';
$salt = substr(md5(uniqid(rand(), true)), 0, 9);
$hash = sha1($salt . sha1($salt . sha1($password)));

echo "Password: admin123\n";
echo "Salt: $salt\n";
echo "Hash: $hash\n";
echo "\nSQL Command:\n";
echo "DELETE FROM oc_user WHERE username = 'admin';\n";
echo "INSERT INTO oc_user (user_group_id, username, password, salt, firstname, lastname, email, status, date_added) VALUES (1, 'admin', '$hash', '$salt', 'Admin', 'User', 'admin@localhost.com', 1, NOW());\n";
EOF

# Run generator
php /tmp/create-admin.php
```

**Example output:**
```
Password: admin123
Salt: b19ec3959
Hash: e995f3e3ce256fa8172b86300e95a26f434a5221

SQL Command:
DELETE FROM oc_user WHERE username = 'admin';
INSERT INTO oc_user (user_group_id, username, password, salt, firstname, lastname, email, status, date_added) VALUES (1, 'admin', 'e995f3e3ce256fa8172b86300e95a26f434a5221', 'b19ec3959', 'Admin', 'User', 'admin@localhost.com', 1, NOW());
```

**⚠️ IMPORTANT:** Your hash and salt will be DIFFERENT (randomly generated each time). This is CORRECT!

---

**📋 STEP 9.3: INSERT ADMIN USER INTO DATABASE**

```bash
# Reconnect to MySQL
mysql -h group3-database.cxcecm6wisku.ap-southeast-1.rds.amazonaws.com -u admin -p
# Enter password: cloudproject
```

```sql
USE Group3_db;

-- COPY the SQL command from your script output above
-- Replace YOUR_HASH and YOUR_SALT with actual values
DELETE FROM oc_user WHERE username = 'admin';

INSERT INTO oc_user (
    user_group_id, 
    username, 
    password, 
    salt, 
    firstname, 
    lastname, 
    email, 
    status, 
    date_added
) VALUES (
    1, 
    'admin', 
    'YOUR_GENERATED_HASH_HERE',  -- Example: e995f3e3ce256fa8172b86300e95a26f434a5221
    'YOUR_GENERATED_SALT_HERE',   -- Example: b19ec3959
    'Admin', 
    'User', 
    'admin@localhost.com', 
    1, 
    NOW()
);
```

**Expected output:**
```
Query OK, 1 row affected (0.003 sec)
```

**✅ VERIFICATION:**
```sql
SELECT user_id, username, firstname, lastname, email, status FROM oc_user WHERE username = 'admin';
```

**Expected output:**
```
+---------+----------+-----------+----------+---------------------+--------+
| user_id | username | firstname | lastname | email               | status |
+---------+----------+-----------+----------+---------------------+--------+
|       1 | admin    | Admin     | User     | admin@localhost.com |      1 |
+---------+----------+-----------+----------+---------------------+--------+
1 row in set (0.001 sec)
```

**✅ SUCCESS CRITERIA:**
- ✅ 1 row returned
- ✅ username = 'admin'
- ✅ status = 1 (enabled)

**❌ If no rows:** Re-run INSERT command, check for typos in hash/salt

```sql
-- Exit MySQL
exit;
```

---

**📋 STEP 9.4: TEST LOGIN CREDENTIALS**

**ℹ️ NOTE:** We'll test actual login in STEP 10, but save these credentials:

```
👤 Username: admin
🔑 Password: admin123
📧 Email: admin@localhost.com
```

**⚠️ REMEMBER:** If you lose these credentials:
1. Reconnect to MySQL
2. Re-run STEP 9.2 to generate new hash
3. UPDATE oc_user SET password='NEW_HASH', salt='NEW_SALT' WHERE username='admin';

---

### **STEP 10: Test Admin Panel**

```
1. Open browser: http://Group3-OpenCart-ALB-xxxx.elb.amazonaws.com/admin

2. Login:
   Username: admin
   Password: admin123

3. First login will ask to move storage directory
   → Select "Automatically Move"
   → Click "Move"

4. Access dashboard ✅
```

---

### **🔧 TROUBLESHOOTING**

**Issue 1: ACL Error**

**Error:**
```
AccessControlListNotSupported: The bucket does not allow ACLs
```

**Cause:** Bucket created with "ACLs disabled", but code has `'ACL' => 'public-read'`

**Fix:** Remove ACL parameter from S3Helper.php putObject() call (already fixed in code above)

---

**Issue 2: SSH Timeout Between EC2s**

**Error:**
```
ssh: connect to host 10.0.2.6 port 22: Connection timed out
```

**Cause:** Security Group missing inbound rule for SSH from EC2-A subnet (10.0.1.0/24)

**Fix Options:**
- Option 1: SSH directly from laptop to EC2-B ✅ (Recommended)
- Option 2: Update Security Group to allow SSH from 10.0.1.0/24
- Option 3: Use S3 as intermediate storage for file transfer

---

**Issue 3: Admin Login Fails**

**Error:**
```
No match for Username and/or Password
```

**Cause:** oc_user table empty, no admin user exists

**Fix:** Create admin user with proper OpenCart password hash (triple SHA1 with salt) - see STEP 9

---

**✅ VERIFICATION CHECKLIST:**

```
Hour 3-4 Deliverables:

Software Installation:
□ Composer installed on EC2-A ✅
□ Composer installed on EC2-B ✅
□ AWS SDK for PHP installed on EC2-A ✅
□ AWS SDK for PHP installed on EC2-B ✅

Code Implementation:
□ S3Helper class created (without ACL parameter) ✅
□ Test script created and working ✅
□ FileManager.php modified for S3 upload ✅
□ Files synced between EC2-A and EC2-B ✅

Testing Results:
□ S3 upload works from EC2-A ✅
□ S3 upload works from EC2-B ✅
□ CloudFront URLs accessible ✅
□ Admin user created successfully ✅
□ Admin panel accessible ✅

Troubleshooting Resolved:
□ ACL error fixed (removed ACL parameter) ✅
□ SSH timeout bypassed (direct SSH to EC2-B) ✅
□ Admin password hash created correctly ✅
```

---

**🎉 HOUR 2-3 COMPLETE!**

**Time taken:** ~20-30 minutes  
**Next:** Hour 3-4 - Install AWS SDK & S3 Plugin for OpenCart

---

**2. Attach Role to EC2 Instances:**

```
AWS Console → EC2 → Instances

For EC2-A (Group3_WebServer1):
├─ Select instance
├─ Actions → Security → Modify IAM role
├─ IAM role: Group3_EC2_S3_Role
└─ Update IAM role

Repeat for EC2-B (Group3_WebServer2)


**Deliverables:**
- ✅ IAM role created
- ✅ Both EC2s can access S3 without hardcoded credentials
- ✅ Security best practice achieved!

---

#### 📌 **HOUR 3-4: Install AWS SDK & S3 Integration** `[12:00-13:00]`

**🎯 GOALS:**

```
✅ Install Composer and AWS SDK for PHP on both EC2s
✅ Create S3Helper PHP class for S3 uploads
✅ Test S3 upload functionality with IAM role authentication
✅ Integrate S3 upload into OpenCart admin filemanager
✅ Sync all changes between EC2-A and EC2-B
```

**💡 WHAT WE'RE BUILDING:**

```
Admin uploads image via OpenCart admin panel:
    ↓
FileManager.php (modified)
    ↓
S3Helper.php (our custom class)
    ↓
AWS SDK for PHP (installed via Composer)
    ↓
IAM Role credentials (auto-provided by EC2)
    ↓
S3 Bucket (group3-opencart-static)
    ↓
CloudFront CDN (dt1v1qszn6knb.cloudfront.net)
    ↓
Customer browser (fast delivery!)
```

---

**📋 STEP 1: Install Composer on EC2-A**

**1. SSH to EC2-A:**

```powershell
# From Windows PowerShell
ssh -i project.pem ec2-user@13.229.212.148
```

**Replace with YOUR EC2-A public IP!**

**2. Navigate to web root:**

```bash
cd /var/www/html
```

**3. Download and install Composer:**

```bash
# Download Composer installer
sudo curl -sS https://getcomposer.org/installer | sudo php -- --install-dir=/usr/local/bin --filename=composer
```

**Expected output:**

```
All settings correct for using Composer
Downloading...

Composer (version 2.9.2) successfully installed to: /usr/local/bin/composer
Use it: php /usr/local/bin/composer
```

**4. Verify installation:**

```bash
composer --version
```

**Expected output:**

```
Composer version 2.9.2 2025-12-15 14:13:42
PHP version 8.1.31
```

**Composer installed successfully! ✅**

---

**📋 STEP 2: Install AWS SDK for PHP on EC2-A**

**1. Install AWS SDK via Composer:**

```bash
# Still in /var/www/html
sudo composer require aws/aws-sdk-php
```

**Expected output:**

```
Loading composer repositories with package information
Updating dependencies
Lock file operations: 14 installs, 0 updates, 0 removals
  - Locking aws/aws-crt-php (v1.2.7)
  - Locking aws/aws-sdk-php (3.369.0)
  - Locking guzzlehttp/guzzle (7.10.0)
  ...
Writing lock file
Installing dependencies from lock file
Package operations: 14 installs, 0 updates, 0 removals
  - Installing symfony/polyfill-mbstring (v1.33.0)
  - Installing aws/aws-sdk-php (3.369.0)
Generating autoload files
```

**2. Verify vendor folder created:**

```bash
ls -la vendor/aws/
```

**Expected output:**

```
drwxr-xr-x. 3 root   root     78 Dec 21 17:30 aws-crt-php
drwxr-xr-x. 6 root   root    172 Dec 21 17:30 aws-sdk-php
```

**AWS SDK installed successfully! ✅**

---

**📋 STEP 3: Create S3Helper Class on EC2-A**

**1. Create the S3Helper PHP file:**

```bash
sudo nano /var/www/html/system/library/s3helper.php
```

**2. Paste this complete code:**

**2. Create S3 Helper Class:**

```bash
# On EC2-A
sudo nano /var/www/html/system/library/s3helper.php

# Paste this code:
```

```php
<?php
/**
 * Group3 S3 Helper Class
 * Handles image uploads to S3 bucket
 */

require '/var/www/html/vendor/autoload.php';

use Aws\S3\S3Client;
use Aws\Exception\AwsException;

class S3Helper {
    private $s3Client;
    private $bucket = 'group3-opencart-static';
    private $cloudfront = 'https://YOUR_CLOUDFRONT_ID.cloudfront.net'; // UPDATE THIS!
    private $region = 'ap-southeast-1';
    
    public function __construct() {
        // IAM role provides credentials automatically
        $this->s3Client = new S3Client([
            'version' => 'latest',
            'region'  => $this->region
        ]);
    }
    
    /**
     * Upload file to S3
     * @param string $localFile - Local file path
     * @param string $s3Key - S3 object key (path in bucket)
     * @return string - CloudFront URL
     */
    public function uploadImage($localFile, $s3Key) {
        try {
            $result = $this->s3Client->putObject([
                'Bucket' => $this->bucket,
                'Key'    => $s3Key,
                'SourceFile' => $localFile,
                'ACL'    => 'public-read',
                'ContentType' => mime_content_type($localFile),
                'CacheControl' => 'max-age=31536000, public, immutable'
            ]);
            
            // Return CloudFront URL
            return $this->cloudfront . '/' . $s3Key;
            
        } catch (AwsException $e) {
            error_log("S3 Upload Error: " . $e->getMessage());
            return false;
        }
    }
    
    /**
     * Generate S3 key with date-based structure
     */
    public function generateKey($filename) {
        $ext = pathinfo($filename, PATHINFO_EXTENSION);
        $unique = uniqid();
        $date = date('Y/m/d');
        
        return "catalog/products/{$date}/{$unique}.{$ext}";
    }
    
    /**
     * Delete file from S3
     */
    public function deleteImage($s3Key) {
        try {
            $this->s3Client->deleteObject([
                'Bucket' => $this->bucket,
                'Key'    => $s3Key
            ]);
            return true;
        } catch (AwsException $e) {
            error_log("S3 Delete Error: " . $e->getMessage());
            return false;
        }
    }
}
```

**3. Update CloudFront URL in code:**

```bash
# Get your CloudFront distribution ID
# AWS Console → CloudFront → Distributions
# Copy the "Distribution domain name" (e.g., d1234abcd.cloudfront.net)

# Update the file
sudo nano /var/www/html/system/library/s3helper.php

# Change line:
# private $cloudfront = 'https://YOUR_CLOUDFRONT_ID.cloudfront.net';
# To:
# private $cloudfront = 'https://d1234abcd.cloudfront.net';

# Save: Ctrl+X, Y, Enter
```

**4. Modify OpenCart Image Upload Controller:**

```bash
# Backup original file first
sudo cp /var/www/html/admin/controller/common/filemanager.php /var/www/html/admin/controller/common/filemanager.php.backup

# Edit the file
sudo nano /var/www/html/admin/controller/common/filemanager.php

# Find the upload function (around line 100-150)
# Add this code after the file validation section:
```

```php
// ADD THIS CODE AFTER LINE ~130 (after file validation)

// Upload to S3 instead of local storage
require_once(DIR_SYSTEM . 'library/s3helper.php');
$s3 = new S3Helper();

// Generate S3 key
$s3Key = $s3->generateKey($file['name']);

// Upload to S3
$cloudfrontUrl = $s3->uploadImage($file['tmp_name'], $s3Key);

if ($cloudfrontUrl) {
    // Success - store CloudFront URL in database
    $json['success'] = 'Image uploaded to S3 successfully!';
    $json['url'] = $cloudfrontUrl;
    
    // Log for monitoring
    error_log("S3 Upload Success: " . $cloudfrontUrl);
} else {
    // Fallback to local storage
    $json['error'] = 'S3 upload failed, using local storage';
    // ... original local upload code ...
}
```

**5. Test Upload via OpenCart Admin:**

```bash
# Access OpenCart admin via ALB
# http://Group3-OpenCart-ALB-xxxx.elb.amazonaws.com/admin

1. Login to admin panel
2. Navigate: Catalog → Products → Edit any product
3. Click "Image" tab
4. Click "Upload" button
5. Select a test image
6. Upload

Expected behavior:
- Image uploads to S3
- URL in database: https://dxxxx.cloudfront.net/catalog/products/2025/12/21/xxxxx.jpg
- Admin shows: "Image uploaded to S3 successfully!" ✅

Verify in AWS:
- S3 Console → group3-opencart-static → catalog/products/2025/12/21/
- Image should be there! ✅
```

**Deliverables:**
- ✅ AWS SDK installed on both EC2s
- ✅ S3 helper class created
- ✅ Image uploads go to S3 → CloudFront
- ✅ Tested and working!

---

#### 📌 **HOUR 4-5: Database Session Configuration** `[15:00-16:00]`

**🎯 OBJECTIVES:**
- ✅ Fix DIR_STORAGE path configuration
- ✅ Configure OpenCart to use RDS database sessions
- ✅ Enable session sharing across EC2-A and EC2-B
- ✅ Test Multi-AZ session persistence
- ✅ Verify no logout when ALB switches between instances

**📚 BACKGROUND - Why Database Sessions?**

```
Problem with File Sessions (Default OpenCart):
┌─────────────────────────────────────────────┐
│ Request #1 → ALB → EC2-A                    │
│   → Session: /var/www/storage/session/abc  │
│   → Stores: user_id=5, cart=[1,2,3]        │
└─────────────────────────────────────────────┘
┌─────────────────────────────────────────────┐
│ Request #2 → ALB → EC2-B (Round Robin)     │
│   → Session file NOT FOUND on EC2-B! ❌    │
│   → User logged out, cart empty! ❌        │
└─────────────────────────────────────────────┘

Solution - Database Sessions (RDS Shared Storage):
┌─────────────────────────────────────────────┐
│ Request #1 → ALB → EC2-A                    │
│   → INSERT INTO oc_session (RDS)            │
│   → Session stored in database ✅           │
└─────────────────────────────────────────────┘
┌─────────────────────────────────────────────┐
│ Request #2 → ALB → EC2-B                    │
│   → SELECT FROM oc_session (RDS)            │
│   → Session found! User still logged in ✅  │
└─────────────────────────────────────────────┘
```

---

### **STEP 1: Fix DIR_STORAGE Path Configuration**

**⚠️ CRITICAL ISSUE DISCOVERED:**

During testing, OpenCart admin panel showed security warning to move storage directory outside web root. Initial path `/var/www/html/system/storage/` was changed to `/var/www/storage/` but both EC2s still had old path in config.

**Problem symptoms:**
```
- Admin panel shows 500 Internal Server Error
- Apache error log: No PHP fatal errors (misleading!)
- OpenCart error log: File path errors
- Root cause: DIR_STORAGE points to non-existent directory
```

**Solution - Update storage path on BOTH EC2 instances:**

```bash
# ========================================
# STEP 1.1: UPDATE EC2-A STORAGE PATH
# ========================================

ssh -i project.pem ec2-user@13.212.190.57

# Check current storage path
grep DIR_STORAGE /var/www/html/config.php

# Output should show:
# define('DIR_STORAGE', '/var/www/html/system/storage/');  ← OLD (wrong!)

# Fix storage path using sed
sudo sed -i "s|define('DIR_STORAGE', '/var/www/html/system/storage/');|define('DIR_STORAGE', '/var/www/storage/');|" /var/www/html/config.php /var/www/html/admin/config.php

# Verify change
grep DIR_STORAGE /var/www/html/config.php

# Expected output:
# define('DIR_STORAGE', '/var/www/storage/');  ← NEW (correct!) ✅

# Restart Apache
sudo systemctl restart httpd

# Exit EC2-A
exit

# ========================================
# STEP 1.2: UPDATE EC2-B STORAGE PATH
# ========================================

ssh -i project.pem ec2-user@54.169.147.163

# Same fix on EC2-B
sudo sed -i "s|define('DIR_STORAGE', '/var/www/html/system/storage/');|define('DIR_STORAGE', '/var/www/storage/');|" /var/www/html/config.php /var/www/html/admin/config.php

# Verify
grep DIR_STORAGE /var/www/html/config.php

# Expected: define('DIR_STORAGE', '/var/www/storage/');  ✅

# Restart Apache
sudo systemctl restart httpd

# Exit EC2-B
exit
```

**✅ VERIFICATION:**

```bash
# Test admin panel access via ALB
# Open browser:
http://Group3-OpenCart-ALB-1956877542.ap-southeast-1.elb.amazonaws.com/admin

# Expected:
# ✅ Admin panel loads successfully
# ✅ Security popup shown (move storage directory)
# ✅ Click dropdown → Select /var/www/ → Move
# ✅ OpenCart moves storage and updates config automatically
```

---

### **STEP 2: Verify RDS Session Table**

### **STEP 2: Verify RDS Session Table**

**Check if oc_session table exists and has correct structure:**

```bash
# SSH to EC2-A (or EC2-B, doesn't matter)
ssh -i project.pem ec2-user@13.212.190.57

# Connect to RDS MySQL
mysql -h group3-database.cxcecm6wisku.ap-southeast-1.rds.amazonaws.com \
      -u admin \
      -pcloudproject \
      -D Group3_db \
      -e "DESCRIBE oc_session;"
```

**Expected output:**

```
+------------+-------------+------+-----+---------+-------+
| Field      | Type        | Null | Key | Default | Extra |
+------------+-------------+------+-----+---------+-------+
| session_id | varchar(32) | NO   | PRI | NULL    |       |
| data       | text        | NO   |     | NULL    |       |
| expire     | datetime    | NO   |     | NULL    |       |
+------------+-------------+------+-----+---------+-------+
```

**✅ Table structure correct! OpenCart 3.0 created this table during initial installation.**

**Key fields:**
- `session_id`: Primary key, 32-char session identifier (cookie value)
- `data`: TEXT field storing serialized session data (JSON format)
- `expire`: Datetime when session expires (for garbage collection)

---

### **STEP 3: Check OpenCart Session Handler**

**Verify database session handler file exists:**

```bash
# Still on EC2-A
ls -la /var/www/html/system/library/session/db.php
```

**Expected output:**

```
-rw-r--r-- 1 apache apache 1793 Dec 20 19:48 /var/www/html/system/library/session/db.php
```

**✅ File exists - OpenCart 3.0 has built-in database session support**

**Inspect session handler code:**

```bash
cat /var/www/html/system/library/session/db.php
```

**Key methods in Session\DB class:**

```php
<?php
namespace Session;

final class DB {
    public $maxlifetime;

    public function __construct($registry) {
        $this->db = $registry->get('db');
        $this->maxlifetime = ini_get('session.gc_maxlifetime') ?: 1440;
        $this->gc();  // Garbage collection on init
    }

    public function read($session_id) {
        $query = $this->db->query("SELECT `data` FROM `" . DB_PREFIX . "session` WHERE `session_id` = '" . $this->db->escape($session_id) . "' AND `expire` > '" . date('Y-m-d H:i:s') . "'");
        
        if ($query->num_rows) {
            return json_decode($query->row['data'], true);  // Returns array
        } else {
            return false;  // No session found
        }
    }

    public function write($session_id, $data) {
        if ($session_id) {
            $this->db->query("REPLACE INTO `" . DB_PREFIX . "session` SET `session_id` = '" . $this->db->escape($session_id) . "', `data` = '" . $this->db->escape(json_encode($data)) . "', `expire` = '" . date('Y-m-d H:i:s', time() + $this->maxlifetime) . "'");
        }
        return true;
    }

    public function destroy($session_id) {
        $this->db->query("DELETE FROM `" . DB_PREFIX . "session` WHERE `session_id` = '" . $this->db->escape($session_id) . "'");
        return true;
    }
}
```

**How it works:**
1. `read()`: Fetches session from RDS, decodes JSON → array
2. `write()`: Encodes data to JSON, REPLACE INTO RDS (upsert)
3. `destroy()`: DELETE from RDS on logout
4. Garbage collection deletes expired sessions automatically

---

### **STEP 4: Enable Database Sessions on EC2-A**

```bash
# Edit main OpenCart config
sudo nano /var/www/html/config.php

# Find the database configuration section (around line 25-30):
# define('DB_DRIVER', 'mysqli');
# define('DB_HOSTNAME', 'group3-database.cxcecm6wisku.ap-southeast-1.rds.amazonaws.com');
# define('DB_USERNAME', 'admin');
# define('DB_PASSWORD', 'cloudproject');
# define('DB_DATABASE', 'Group3_db');
# define('DB_PORT', '3306');
# define('DB_PREFIX', 'oc_');

# ADD THIS LINE AFTER DB_DATABASE:
define('session_engine', 'db');

# ⚠️ IMPORTANT: Use lowercase 'session_engine' (not SESSION_ENGINE)
# OpenCart framework.php reads: $config->get('session_engine')

# Save: Ctrl+X, Y, Enter
```

**Also update admin config:**

```bash
sudo nano /var/www/html/admin/config.php

# Find DB configuration section
# ADD after DB_DATABASE:
define('session_engine', 'db');

# Save: Ctrl+X, Y, Enter
```

**Verify config added:**

```bash
grep session_engine /var/www/html/config.php
```

**Expected output:**

```
define('session_engine', 'db');
```

**✅ Config correct on EC2-A**

---

### **STEP 5: Sync Config to EC2-B via S3**

**Use S3 as intermediate storage for config sync:**

```bash
# Still on EC2-A
# Upload configs to S3 temp folder
aws s3 cp /var/www/html/config.php s3://group3-opencart-static/temp/config.php
aws s3 cp /var/www/html/admin/config.php s3://group3-opencart-static/temp/admin-config.php
```

**Expected output:**

```
upload: ../../var/www/html/config.php to s3://group3-opencart-static/temp/config.php
upload: ../../var/www/html/admin/config.php to s3://group3-opencart-static/temp/admin-config.php
```

**Exit EC2-A and apply to EC2-B:**

```bash
# Exit EC2-A
exit

# SSH to EC2-B
ssh -i project.pem ec2-user@54.169.147.163

# Download configs from S3
aws s3 cp s3://group3-opencart-static/temp/config.php /tmp/config.php
aws s3 cp s3://group3-opencart-static/temp/admin-config.php /tmp/admin-config.php

# Apply configs
sudo cp /tmp/config.php /var/www/html/config.php
sudo cp /tmp/admin-config.php /var/www/html/admin/config.php

# Set correct permissions
sudo chown apache:apache /var/www/html/config.php /var/www/html/admin/config.php

# Verify session_engine present
grep session_engine /var/www/html/config.php

# Expected output:
# define('session_engine', 'db');  ✅

# Exit EC2-B
exit
```

---

### **STEP 6: Clear Old Sessions and Restart Apache**

```bash
# Clear all existing file-based sessions from database
ssh -i project.pem ec2-user@13.212.190.57

# Delete all sessions (fresh start)
mysql -h group3-database.cxcecm6wisku.ap-southeast-1.rds.amazonaws.com \
      -u admin \
      -pcloudproject \
      -D Group3_db \
      -e "DELETE FROM oc_session;"

# Expected output: Query OK, X rows affected

# Restart Apache on EC2-A
sudo systemctl restart httpd

# Check status
sudo systemctl status httpd

# Expected: Active: active (running) ✅

# Exit EC2-A
exit

# Restart Apache on EC2-B
ssh -i project.pem ec2-user@54.169.147.163
sudo systemctl restart httpd
sudo systemctl status httpd

# Expected: Active: active (running) ✅

exit
```

---

### **STEP 7: Test Database Session Sharing**

**🎯 Test Scenario:**
1. Login to admin panel via ALB
2. Refresh page 20-30 times rapidly
3. ALB Round Robin will route requests to EC2-A and EC2-B randomly
4. Expected: Stay logged in throughout (session shared via RDS)

**Execute test:**

```bash
# Open browser
http://Group3-OpenCart-ALB-1956877542.ap-southeast-1.elb.amazonaws.com/admin

# Login with admin credentials:
Username: admin
Password: admin123  (or your password)

# After successful login:
1. Press F5 rapidly 20-30 times
2. Navigate: Catalog → Products → Edit product
3. Navigate: Sales → Orders
4. Navigate back to Dashboard
5. Refresh again 10 times

# ✅ EXPECTED RESULT:
- Stay logged in throughout
- No random logouts
- Dashboard data loads consistently
- Session persists across EC2 switches
```

**Verify session in database:**

```bash
ssh -i project.pem ec2-user@13.212.190.57

# Check sessions in RDS
mysql -h group3-database.cxcecm6wisku.ap-southeast-1.rds.amazonaws.com \
      -u admin \
      -pcloudproject \
      -D Group3_db \
      -e "SELECT session_id, LEFT(data, 80) as session_data, expire FROM oc_session ORDER BY expire DESC LIMIT 3;"
```

**Expected output:**

```
+----------------------------+--------------------------------------------------------------------------------+---------------------+
| session_id                 | session_data                                                                   | expire              |
+----------------------------+--------------------------------------------------------------------------------+---------------------+
| 3c23651137cc90836187916d66 | {"language":"en-gb","currency":"USD"}                                          | 2025-12-22 04:02:04 |
| b4e3ba3b9a1283279812340cf4 | {"language":"en-gb","currency":"USD","user_id":"2","firstname":"Admin",...}    | 2025-12-22 04:01:59 |
+----------------------------+--------------------------------------------------------------------------------+---------------------+
```

**✅ Session data includes:**
- `language`: Current language setting
- `currency`: Current currency
- `user_id`: Admin user ID (proves logged in!)
- `firstname`: Admin user's first name
- `user_token`: Security token for admin actions

**Check browser cookie:**

```javascript
// Open browser DevTools (F12) → Console
document.cookie

// Expected output:
"OCSESSID=b4e3ba3b9a1283279812340cf4; language=en-gb; currency=USD"
```

**Cookie explanation:**
- `OCSESSID`: Session ID (matches session_id in database)
- Browser sends this with every request
- ALB routes to EC2-A or EC2-B
- Both EC2s query RDS with this session_id
- Session data retrieved, user stays logged in ✅

---

### **TROUBLESHOOTING NOTES**

During implementation, several issues were encountered and resolved:

**Issue #1: 500 Internal Server Error after enabling sessions**

```
Symptom: Admin panel showed HTTP ERROR 500
Root cause: Wrong configuration constant name
Fix: Changed SESSION_ENGINE to session_engine (lowercase)
```

**Issue #2: DIR_STORAGE path mismatch**

```
Symptom: 500 error even with correct session config
Root cause: Storage path pointed to /var/www/html/system/storage/ 
           but OpenCart moved it to /var/www/storage/
Fix: Updated DIR_STORAGE in config.php on both EC2s
```

**Issue #3: Session file syntax errors from manual editing**

```
Symptom: PHP Parse error when creating custom session handler
Root cause: PowerShell escaping issues with heredoc
Fix: Created session handler file locally, uploaded via SCP
      Then reverted to built-in OpenCart db.php (which is correct)
```

**Key Learning:**
- OpenCart 3.0 database session handler (`system/library/session/db.php`) is **already correct**
- No need to modify it - uses JSON encoding/decoding properly
- Session wrapper class (`system/library/session.php`) expects **array** from `read()` method
- Config key is `session_engine` (lowercase), not `SESSION_ENGINE`

---

**✅ VERIFICATION CHECKLIST:**

```
Hour 4-5 Deliverables:

Storage Configuration:
□ DIR_STORAGE path fixed on EC2-A: /var/www/storage/ ✅
□ DIR_STORAGE path fixed on EC2-B: /var/www/storage/ ✅
□ Admin panel accessible without 500 errors ✅

Database Session Setup:
□ oc_session table verified in RDS (3 fields) ✅
□ Session handler file exists: system/library/session/db.php ✅
□ session_engine config added to EC2-A config.php ✅
□ session_engine config added to EC2-B config.php ✅
□ Config synced via S3 intermediate storage ✅

Apache Restart:
□ EC2-A Apache restarted successfully ✅
□ EC2-B Apache restarted successfully ✅

Testing Results:
□ Old sessions cleared from database ✅
□ Admin login successful via ALB ✅
□ Refreshed 20+ times without logout ✅
□ Sessions stored in RDS with JSON data ✅
□ Session contains user_id (proves login persistence) ✅
□ Browser cookie OCSESSID matches database session_id ✅

Multi-AZ Verification:
□ ALB Round Robin distributing traffic 50/50 ✅
□ EC2-A can read sessions created by EC2-B ✅
□ EC2-B can read sessions created by EC2-A ✅
□ Cross-AZ session sharing working ✅
```

---

**📊 ARCHITECTURE AFTER HOUR 4-5:**

```
User Login Flow with Database Sessions:

Browser → ALB → EC2-A (first request)
│
├─ POST /admin/index.php?route=common/login
├─ PHP validates credentials
├─ Session::start() called
│  ├─ Generates session_id: b4e3ba3b9a1283279812340cf4
│  └─ Creates $session->data array: {user_id: 2, user_token: xxx}
│
├─ Session::close() called (on request end)
│  └─ Session\DB::write() executes:
│     └─ REPLACE INTO oc_session SET
│        session_id = 'b4e3ba3b9a1283279812340cf4',
│        data = '{"language":"en-gb","currency":"USD","user_id":"2"...}',
│        expire = '2025-12-22 04:30:00'
│
└─ Response to browser:
   Set-Cookie: OCSESSID=b4e3ba3b9a1283279812340cf4; path=/; HttpOnly

Subsequent Request (might hit EC2-B):

Browser → ALB → EC2-B (Round Robin!)
│
├─ GET /admin/index.php?route=common/dashboard
├─ Cookie sent: OCSESSID=b4e3ba3b9a1283279812340cf4
│
├─ Session::start('b4e3ba3b9a1283279812340cf4')
│  └─ Session\DB::read('b4e3ba3b9a1283279812340cf4')
│     └─ SELECT data FROM oc_session WHERE session_id = 'b4e3ba3b...'
│        AND expire > NOW()
│     └─ Found! Returns: {"user_id":"2",...}
│
├─ OpenCart checks: $session->data['user_id'] → 2 ✅
├─ User is logged in! Load dashboard
│
└─ Response: Dashboard HTML (user still logged in) ✅

Cross-AZ Session Flow:
EC2-A (AZ 1a) ←→ RDS (AZ 1a) ←→ EC2-B (AZ 1b)
    2-4ms latency      2-4ms latency
    
Total: ~4-8ms for session read (negligible!)
```

---

**💡 BENEFITS ACHIEVED:**

```
1. True Load Balancing:
   ✅ No sticky sessions needed
   ✅ ALB Round Robin works perfectly
   ✅ Traffic distributed 50/50 between EC2s

2. High Availability:
   ✅ If EC2-A fails → EC2-B still has all sessions
   ✅ Users don't lose login/cart on failover
   ✅ Zero session data loss

3. Scalability:
   ✅ Can add EC2-C, EC2-D without session issues
   ✅ All instances share same RDS session storage
   ✅ Horizontal scaling ready

4. Multi-AZ Resilience:
   ✅ Sessions survive AZ failure
   ✅ RDS Multi-AZ can failover to standby
   ✅ Complete data durability
```

---

**📝 SAVE DAY 2 HOUR 4-5 PROGRESS:**

# Should show:
# +--------------------------+
# | Tables_in_opencart (oc_session) |
# +--------------------------+
# | oc_session                |
# +--------------------------+

# Check table structure
DESCRIBE oc_session;

# Should have columns:
# session_id (varchar)
# data (text)
# expire (int)

# If table doesn't exist, create it:
CREATE TABLE IF NOT EXISTS `oc_session` (
  `session_id` varchar(32) NOT NULL,
  `data` text NOT NULL,
  `expire` datetime NOT NULL,
  PRIMARY KEY (`session_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

# Exit MySQL
exit;
```

**3. Update session handler in OpenCart:**

```bash
# Edit startup file
sudo nano /var/www/html/system/startup.php

# Find the session initialization code (around line 80-100)
# It should already have DB session support
# Verify this section exists:

if (SESSION_ENGINE == 'db') {
    $registry->set('session', new Session('db', $registry));
} else {
    $registry->set('session', new Session('native'));
}

# If not present, the define we added should make it work
# OpenCart 3.0 has built-in DB session support
```

**4. Sync changes to EC2-B:**

```bash
# From EC2-A, copy config to EC2-B
scp /var/www/html/config.php ec2-user@10.0.11.XXX:/tmp/
scp /var/www/html/admin/config.php ec2-user@10.0.11.XXX:/tmp/

# SSH to EC2-B
ssh ec2-user@10.0.11.XXX

# Move files to correct location
sudo mv /tmp/config.php /var/www/html/config.php
sudo mv /tmp/admin/config.php /var/www/html/admin/config.php

# Set permissions
sudo chown apache:apache /var/www/html/config.php
sudo chown apache:apache /var/www/html/admin/config.php

# Restart Apache on both EC2s
sudo systemctl restart httpd
```

**5. Test Session Persistence:**

```bash
# Test scenario:
# 1. Login via ALB (might hit EC2-A)
# 2. Refresh page (might hit EC2-B)
# 3. Should still be logged in!

# Open browser:
http://Group3-OpenCart-ALB-xxxx.elb.amazonaws.com/admin

# Login with admin credentials

# Open browser console (F12)
# Go to Application → Cookies
# Find: OCSESSID cookie value (e.g., abc123...)

# Refresh page multiple times
# Watch Network tab to see which EC2 responds

# Check RDS to see session:
mysql -h RDS_ENDPOINT -u admin -p
USE opencart;
SELECT session_id, LEFT(data, 50), expire FROM oc_session ORDER BY expire DESC LIMIT 5;

# Should see your session! ✅
# Data contains: user_id, firstname, etc.
```

**Verification Test:**

```
Test 1: Login Persistence
├─ Login to frontend: /index.php?route=account/login
├─ Refresh 10 times rapidly
├─ Expected: Stay logged in ✅
└─ Check: ALB access logs show different target IPs

Test 2: Cart Persistence
├─ Add product to cart
├─ Navigate to different pages
├─ Cart should persist ✅
└─ Even if different EC2 serves request

Test 3: Admin Panel
├─ Login to admin
├─ Navigate: Catalog → Products → Edit
├─ Should work seamlessly ✅
└─ No random logouts
```

**📝 SAVE DAY 2 HOUR 4-5 PROGRESS:**

```bash
cd "C:\Users\ASUS\Documents\Sever Systems - Cloud\Cloud-project"

# Update progress document
cat >> docs/day2-progress.md << 'EOF'

## Hour 4-5: Database Session Configuration ✅

### Completed Tasks:
- ✅ Fixed DIR_STORAGE path on both EC2 instances
- ✅ Verified oc_session table structure in RDS
- ✅ Enabled database sessions via session_engine config
- ✅ Synced config files from EC2-A to EC2-B via S3
- ✅ Cleared old file-based sessions
- ✅ Restarted Apache on both instances
- ✅ Tested Multi-AZ session persistence
- ✅ Verified cross-EC2 session sharing working

### Test Results:
- Login persistence: PASSED ✅ (30+ refreshes, no logout)
- Cross-AZ sharing: PASSED ✅ (EC2-A ↔ RDS ↔ EC2-B)
- Session data format: JSON ✅
- Cookie mechanism: OCSESSID working ✅

### Configuration Changes:
| File | Change | Both EC2s |
|------|--------|-----------|
| config.php | define('session_engine', 'db'); | ✅ |
| admin/config.php | define('session_engine', 'db'); | ✅ |
| config.php | DIR_STORAGE = '/var/www/storage/' | ✅ |

### Database Schema:
Table: oc_session
├── session_id VARCHAR(32) PRIMARY KEY
├── data TEXT (JSON format)
└── expire DATETIME

Sample session data:
{
  "language": "en-gb",
  "currency": "USD",
  "user_id": "2",
  "firstname": "Admin",
  "user_token": "hHNmK7PaQVk5vU5XF768wk8KlZO4Kvi4"
}

EOF

# Commit Day 2 Hour 4-5 completion
git add docs/day2-progress.md
git commit -m "Day 2 Hour 4-5: Database session configuration complete - Multi-AZ session sharing working"
git push origin main
```

---

**⏱️ TIME CHECK:** Hour 4-5 should take 45-60 minutes total (including troubleshooting)

---

**🎉 DAY 2 COMPLETE!**

```
✅ Day 2 Summary - All Hours Completed:

Hour 1-2: S3 & CloudFront Setup
├─ S3 bucket: group3-opencart-static ✅
├─ CloudFront distribution: dt1v1qszn6knb.cloudfront.net ✅
└─ Static assets globally distributed ✅

Hour 2-3: IAM Role for EC2 → S3 Access
├─ IAM role: Group3_EC2_S3_Role ✅
├─ EC2-A and EC2-B attached to role ✅
└─ Passwordless S3 access from EC2s ✅

Hour 3-4: AWS SDK Integration
├─ Composer installed ✅
├─ AWS SDK for PHP installed ✅
├─ S3 image upload working ✅
└─ Admin can upload images to CloudFront ✅

Hour 4-5: Database Session Configuration
├─ DIR_STORAGE path fixed ✅
├─ Database sessions enabled ✅
├─ Multi-AZ session sharing working ✅
└─ Login persistence across EC2 switches ✅

Architecture Achieved:
┌──────────────────────────────────────────┐
│ Internet Users                           │
└────────┬─────────────────────────────────┘
         │
    ┌────▼────┐
    │   ALB   │ (Multi-AZ: 1a + 1b)
    └────┬────┘
         │
    ┌────┴────┐
    │         │
┌───▼───┐ ┌───▼───┐
│ EC2-A │ │ EC2-B │ (Both have IAM role)
│ AZ-1a │ │ AZ-1b │
└───┬───┘ └───┬───┘
    │         │
    └────┬────┘
         │
    ┌────▼────┐
    │   RDS   │ (Sessions shared!)
    │  MySQL  │
    └─────────┘

    ┌──────────────────┐
    │ S3 Bucket        │ ← Images
    │ (Multi-AZ Auto)  │
    └────────┬─────────┘
             │
    ┌────────▼─────────┐
    │   CloudFront     │ ← Global CDN
    │  (400+ Edges)    │
    └──────────────────┘

Key Achievements:
✅ High Availability (Multi-AZ EC2 + ALB)
✅ Session Persistence (RDS shared sessions)
✅ Global Performance (CloudFront CDN)
✅ Scalable Storage (S3 unlimited)
✅ Secure Access (IAM roles, no credentials)
✅ Cost Optimized (S3 lifecycle, CloudFront caching)
```

---

**⏭️ NEXT: DAY 3 - Monitoring & CI/CD**

```
Day 3 Planned Activities:

Hour 1-2: CloudWatch Monitoring Setup
├─ CloudWatch Agent installation
├─ Log streaming (Apache, OpenCart)
├─ Custom metrics (CPU, Memory, Disk)
└─ SNS alarm notifications

Hour 2-3: CloudWatch Dashboard & Alarms
├─ Create unified dashboard
├─ EC2 health metrics
├─ ALB request count
├─ RDS connections
└─ S3/CloudFront usage

Hour 3-4: GitHub Actions CI/CD Pipeline
├─ Automated deployment on git push
├─ Build, test, deploy workflow
├─ Blue-green deployment strategy
└─ Rollback capability

Hour 4-5: Comprehensive Testing
├─ Load testing (JMeter/Gatling)
├─ Failover testing (terminate EC2-A)
├─ Session persistence testing
└─ Performance benchmarking
```

---

**📸 SCREENSHOTS TO COLLECT (Day 2):**

```
S3 & CloudFront:
□ S3 bucket overview (object count, size)
□ CloudFront distribution (domain, status)
□ Sample image URL (CloudFront vs direct S3)

IAM & Security:
□ IAM role policy (AmazonS3FullAccess)
□ EC2 instance profile attached
□ S3 bucket policy (public read)

AWS SDK Integration:
□ Composer.json with aws/aws-sdk-php
□ Admin panel file upload interface
□ Network tab showing CloudFront image URL

Database Sessions:
□ oc_session table data (SELECT query result)
□ Admin panel staying logged in after 20+ refreshes
□ Browser DevTools → Cookie showing OCSESSID
□ Session data JSON in database

Testing Evidence:
□ ALB access logs showing Round Robin distribution
□ CloudWatch metrics (EC2-A vs EC2-B request count)
□ Session persistence test results
```

---

## Testing Results
- ✅ Images upload to S3
- ✅ CloudFront delivers cached images
- ✅ Sessions persist across EC2s
- ✅ No random logouts

## Screenshots
[Attach in docs/screenshots/day2/]
EOF

git add .
git commit -m "Day 2 complete: S3, CloudFront, DB sessions"
git push
```

**🎉 END OF DAY 2!** 

---

## 📊 DAY 2 COMPLETION SUMMARY

**✅ All 4 HOURS Complete (5 hours total achievement)**

### HOUR 1-2: S3 & CloudFront Setup ✅ (25-35 min)
```
✅ S3 bucket created: group3-opencart-static
✅ Bucket policy: Public read (s3:GetObject for *)
✅ Folder structure: /blog/, /cache/, /catalog/products/
✅ CloudFront distribution deployed
✅ Status: Enabled (400+ edge locations)
✅ Testing: S3 direct + CloudFront caching verified
✅ Performance: 5-10x faster with CloudFront caching
```

### HOUR 2-3: IAM Role for EC2→S3 Access ✅ (20-25 min)
```
✅ IAM role created: Group3_EC2_S3_Role
✅ Trust policy: AWS service: ec2.amazonaws.com
✅ Permissions: AmazonS3FullAccess
✅ EC2-A attached to role
✅ EC2-B attached to role
✅ AWS CLI tests passed:
   ├─ aws s3 ls (list buckets)
   ├─ aws s3 cp (upload files)
   └─ Cross-AZ sharing verified
✅ ZERO hardcoded credentials in code!
✅ Auto-rotating temporary credentials (6-hour refresh)
```

### HOUR 3-4: AWS SDK Integration ✅ (30-40 min)
```
✅ Composer installed on both EC2s
✅ AWS SDK for PHP installed (via Composer)
✅ S3Helper class created (custom PHP wrapper)
✅ Test script created and tested
✅ FileManager.php modified for S3 uploads
✅ Both EC2s can upload to S3 bucket
✅ CloudFront URLs generated automatically
✅ Admin user created for testing
```

### HOUR 4-5: Database Session Configuration ✅ (45-60 min)
```
✅ DIR_STORAGE path fixed: /var/www/storage/
✅ oc_session table verified in RDS
✅ Session handler: system/library/session/db.php
✅ session_engine = 'db' configured
✅ Config files synced between EC2s (via S3 temp)
✅ Admin panel accessible and working
✅ Sessions tested: 30+ rapid refreshes without logout
✅ Multi-AZ session sharing proven working
✅ ALB Round Robin + Sessions = Perfect load distribution!
```

---

## 📈 ARCHITECTURE ACHIEVED BY END OF DAY 2

```
LAYER 1: CDN & Storage
┌───────────────────────────────────────┐
│ CloudFront Distribution               │
│ (400+ edge locations globally)        │
│ Domain: dxxxxxxx.cloudfront.net       │
└────────────────┬──────────────────────┘
                 ↓
      ┌──────────────────────┐
      │  S3 Bucket           │
      │  group3-opencart-    │
      │  static              │
      │  (Images, static     │
      │   assets)            │
      └──────────────────────┘

LAYER 2: Compute & Load Balancing
┌────────────────────────────────────────────┐
│            ALB                             │
│  (Multi-AZ 1a + 1b)                       │
│  Health checks every 30 sec                │
└─────┬──────────────────┬──────────────────┘
      ↓                  ↓
┌───────────────┐   ┌───────────────┐
│   EC2-A       │   │   EC2-B       │
│   (AZ 1a)     │   │   (AZ 1b)     │
│   IAM Role    │   │   IAM Role    │
│   (S3 access) │   │   (S3 access) │
└───────────────┘   └───────────────┘

LAYER 3: Data & Sessions
┌──────────────────────────────────────────┐
│            RDS MySQL                     │
│ ├─ OpenCart database (19 products)       │
│ ├─ oc_session table (shared sessions)    │
│ └─ Cross-AZ accessible                   │
└──────────────────────────────────────────┘

DATA FLOW:
Admin uploads image
  ├─ EC2 receives POST via ALB
  ├─ S3Helper uploads to S3 bucket (via IAM role)
  ├─ CloudFront caches image
  └─ Database URL stored in RDS

Customer views image
  ├─ Browser requests CloudFront domain
  ├─ CloudFront serves cached version (< 100ms)
  ├─ Works from ANY EC2 (both have IAM role!)
  └─ Perfect Multi-AZ distribution!

User session management
  ├─ User logs in via ALB (hits EC2-A)
  ├─ Session saved to RDS (shared!)
  ├─ ALB Round Robin switches to EC2-B
  ├─ EC2-B reads session from RDS
  └─ User stays logged in! (Multi-AZ stateless!)
```

---

## 🎯 KEY ACHIEVEMENTS - DAY 2

| Achievement | Benefit | Impact |
|-------------|---------|--------|
| **S3 + CloudFront** | Single source of truth for images | No more 404 errors on EC2-B |
| **Global CDN** | 5-10x faster delivery | 20-50ms response time (cached) |
| **IAM Role** | Zero credentials in code | Security best practice ✅ |
| **AWS SDK** | Automatic S3 integration | Admin can upload images directly |
| **Database Sessions** | Sessions shared via RDS | Multi-AZ truly stateless |
| **Cross-AZ** | Automatic failover | Stop EC2-A → Site works via EC2-B |

---

## 💾 DAY 2 DELIVERABLES

**Infrastructure:**
- ✅ S3 bucket: `group3-opencart-static`
- ✅ CloudFront distribution: `dxxxxxxx.cloudfront.net`
- ✅ IAM role: `Group3_EC2_S3_Role`
- ✅ AWS SDK for PHP installed on both EC2s
- ✅ Session table: `oc_session` in RDS

**Code Changes:**
- ✅ S3Helper.php class created
- ✅ FileManager.php modified for S3 uploads
- ✅ config.php updated with session_engine = 'db'
- ✅ Both EC2s synchronized

**Testing Results:**
- ✅ S3 direct URL: Works
- ✅ CloudFront URL: Works + Cached
- ✅ AWS CLI S3 access: Works (no credentials needed!)
- ✅ Cross-AZ file sharing: Proven
- ✅ Session persistence: Multi-AZ tested
- ✅ Admin panel: Fully functional

**Cost Impact:**
- ALB: ~$21/month
- S3: ~$0.26/month
- CloudFront: ~$6/month
- **Total DAY 2 cost: ~$27/month**

---

## ✨ READY FOR DAY 3!

Architecture is now:
- ✅ **Distributed** (S3 + CloudFront global)
- ✅ **Stateless** (Database sessions in RDS)
- ✅ **Highly Available** (Multi-AZ + ALB + Auto-failover)
- ✅ **Scalable** (Can add more EC2s easily)
- ✅ **Secure** (IAM roles, no hardcoded credentials)

Next: DAY 3 - Monitoring & CI/CD Pipeline

---

## DAY 3 (Sunday Dec 22) - 5 hours: Monitoring & CI/CD

### **Hour 1-2: CloudWatch Agent Installation (9:00-11:00)**

**Goals:**
- ✅ Install CloudWatch agent on both EC2s
- ✅ Configure custom metrics collection
- ✅ Set up log streaming

**1. Install CloudWatch Agent on EC2-A:**

```bash
# SSH to EC2-A
ssh -i project.pem ec2-user@EC2_A_PUBLIC_IP

# Download CloudWatch agent
wget https://s3.amazonaws.com/amazoncloudwatch-agent/amazon_linux/amd64/latest/amazon-cloudwatch-agent.rpm

# Install
sudo rpm -U ./amazon-cloudwatch-agent.rpm

# Verify installation
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
    -a query -m ec2 -c default -s
```

**2. Create IAM role for CloudWatch:**

```
AWS Console → IAM → Roles → Create
- Trusted entity: EC2
- Permissions: CloudWatchAgentServerPolicy
- Name: Group3_EC2_CloudWatch_Role

# Attach role to EC2 (in AWS Console)
EC2 → Instance → Actions → Security → Modify IAM role
Select: Group3_EC2_CloudWatch_Role
```

**3. Configure CloudWatch Agent:**

```bash
sudo nano /opt/aws/amazon-cloudwatch-agent/etc/config.json
```

**Paste configuration:**

```json
{
  "agent": {
    "metrics_collection_interval": 60,
    "run_as_user": "root"
  },
  "logs": {
    "logs_collected": {
      "files": {
        "collect_list": [
          {
            "file_path": "/var/log/httpd/access_log",
            "log_group_name": "/aws/ec2/group3/apache/access",
            "log_stream_name": "{instance_id}",
            "retention_in_days": 7
          },
          {
            "file_path": "/var/log/httpd/error_log",
            "log_group_name": "/aws/ec2/group3/apache/error",
            "log_stream_name": "{instance_id}",
            "retention_in_days": 7
          },
          {
            "file_path": "/var/www/html/system/storage/logs/error.log",
            "log_group_name": "/aws/ec2/group3/opencart/error",
            "log_stream_name": "{instance_id}",
            "retention_in_days": 7
          }
        ]
      }
    }
  },
  "metrics": {
    "namespace": "Group3/EC2",
    "metrics_collected": {
      "cpu": {
        "measurement": [
          {"name": "cpu_usage_idle", "rename": "CPU_IDLE", "unit": "Percent"},
          {"name": "cpu_usage_active", "rename": "CPU_ACTIVE", "unit": "Percent"}
        ],
        "metrics_collection_interval": 60
      },
      "disk": {
        "measurement": [
          {"name": "used_percent", "rename": "DISK_USED", "unit": "Percent"},
          {"name": "free", "rename": "DISK_FREE", "unit": "Gigabytes"}
        ],
        "metrics_collection_interval": 60,
        "resources": ["*"]
      },
      "mem": {
        "measurement": [
          {"name": "mem_used_percent", "rename": "MEMORY_USED", "unit": "Percent"},
          {"name": "mem_available", "rename": "MEMORY_AVAILABLE", "unit": "Megabytes"}
        ],
        "metrics_collection_interval": 60
      }
    }
  }
}
```

**4. Start CloudWatch Agent:**

```bash
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
    -a fetch-config -m ec2 -s \
    -c file:/opt/aws/amazon-cloudwatch-agent/etc/config.json

# Enable on boot
sudo systemctl enable amazon-cloudwatch-agent
```

**5. Repeat for EC2-B** (same steps)

**6. Verify Metrics in CloudWatch:**

```
AWS Console → CloudWatch → Metrics → Group3/EC2
Should see: CPU_ACTIVE, MEMORY_USED, DISK_USED for both instances ✅
```

---

### **Hour 2-3: CloudWatch Dashboard & Alarms (11:00-12:00)**

**1. Create CloudWatch Dashboard:**

```
AWS Console → CloudWatch → Dashboards → Create dashboard

Name: Group3-OpenCart-Dashboard

Add widgets:
├─ Line graph: EC2 CPU Usage
├─ Line graph: Memory Usage
├─ Number: ALB Requests/min
├─ Line graph: ALB Response Time
├─ Number: Healthy Targets
└─ Line graph: RDS Performance
```

**2. Create SNS Topic:**

```
AWS Console → SNS → Topics → Create topic

Name: Group3-CloudWatch-Alerts
Protocol: Email
Endpoint: your-email@example.com
Confirm subscription ✅
```

**3. Create CloudWatch Alarms:**

```
Alarm 1: High CPU
├─ Metric: Group3/EC2 → CPU_ACTIVE
├─ Threshold: > 80%
├─ Datapoints: 2 out of 2
└─ Action: SNS → Group3-CloudWatch-Alerts

Alarm 2: Unhealthy Targets
├─ Metric: ALB → HealthyHostCount
├─ Threshold: < 2
└─ Action: SNS → Group3-CloudWatch-Alerts

Alarm 3: High Response Time
├─ Metric: ALB → TargetResponseTime
├─ Threshold: > 1000ms
└─ Action: SNS → Group3-CloudWatch-Alerts
```

**Deliverables:**
- ✅ Dashboard with 6+ widgets
- ✅ 3+ critical alarms
- ✅ SNS email notifications

---

### **Hour 3-4: GitHub Actions CI/CD (12:00-13:00)**

**1. Generate SSH key for GitHub:**

```bash
ssh-keygen -t rsa -b 4096 -C "github-deploy@group3" -f github-deploy-key
```

**2. Add public key to EC2s:**

```bash
# SSH to EC2-A
cat >> ~/.ssh/authorized_keys << 'EOF'
ssh-rsa AAAAB... github-deploy@group3
EOF

chmod 600 ~/.ssh/authorized_keys

# Repeat for EC2-B
```

**3. Add secrets to GitHub:**

```
GitHub → Repo → Settings → Secrets

Add:
├─ EC2_A_HOST: [EC2-A Public IP]
├─ EC2_B_HOST: [EC2-B Private IP]
├─ EC2_SSH_KEY: [Private key content]
└─ EC2_USERNAME: ec2-user
```

**4. Create GitHub Actions workflow:**

```bash
mkdir -p .github/workflows
cat > .github/workflows/deploy.yml
```

**Workflow content:**

```yaml
name: Deploy to AWS EC2

on:
  push:
    branches: [ main ]
  workflow_dispatch:

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v3
      
    - name: Setup SSH
      run: |
        mkdir -p ~/.ssh
        echo "${{ secrets.EC2_SSH_KEY }}" > ~/.ssh/deploy_key
        chmod 600 ~/.ssh/deploy_key
        
    - name: Deploy to EC2-A
      run: |
        rsync -avz --delete \
          -e "ssh -i ~/.ssh/deploy_key" \
          --exclude='.git' \
          --exclude='config.php' \
          ./ ${{ secrets.EC2_USERNAME }}@${{ secrets.EC2_A_HOST }}:/var/www/html/
        
    - name: Restart Services
      run: |
        ssh -i ~/.ssh/deploy_key ${{ secrets.EC2_USERNAME }}@${{ secrets.EC2_A_HOST }} \
          'sudo systemctl restart httpd'
```

**5. Test deployment:**

```bash
git add .
git commit -m "Add CI/CD pipeline"
git push

# Watch: https://github.com/YOUR_REPO/actions
# Should see workflow running ✅
```

---

### **Hour 4-5: Testing & Documentation (13:00-14:00)**

**1. Comprehensive Testing:**

```
Test Case 1: Basic Functionality
□ Homepage loads via ALB ✅
□ Images load from CloudFront ✅
□ Admin panel accessible ✅

Test Case 2: High Availability
□ Both targets healthy ✅
□ Stop EC2-A → Site still works ✅

Test Case 3: S3 Integration
□ Upload image via admin ✅
□ Image in S3 bucket ✅
□ CloudFront URL in database ✅

Test Case 4: Monitoring
□ CloudWatch metrics present ✅
□ Logs streaming ✅
□ Alarms configured ✅

Test Case 5: CI/CD
□ Code push triggers deploy ✅
□ Changes visible on ALB URL ✅
```

**2. Take Screenshots:**

```
Screenshot Checklist:
├─ Architecture (VPC, EC2, ALB, RDS, S3, CloudFront)
├─ Application (Homepage, Admin panel, Image upload)
├─ Monitoring (Dashboard, Metrics, Alarms)
├─ Testing (Failover, CloudFront cache)
└─ CI/CD (GitHub workflow)

Total: 25+ screenshots
```

**Deliverables:**
- ✅ All test cases passed
- ✅ 25+ screenshots collected
- ✅ Ready for Part 3 presentation

---

## DAY 4 (Monday Dec 23) - 5 hours: Polish & Report

### **Hour 1-2: Cost Analysis (9:00-11:00)**

**Current Infrastructure Cost:**

```
Resource Breakdown (ap-southeast-1):

1. EC2 (2x t2.micro): $16.94/month → $0 (free tier)
2. EBS (60GB): $5.76/month → $0 (free tier)
3. ALB: $21.43/month
4. RDS (db.t3.micro): $14.71/month
5. S3: $0.26/month
6. CloudFront: $6.08/month
7. CloudWatch: $8.50/month
8. Data Transfer: $2.40/month

TOTAL WITH FREE TIER: ~$36/month
TOTAL WITHOUT FREE TIER: ~$53/month
```

**Scenario Analysis:**

| Scenario | Monthly Cost | vs Current | Use Case |
|----------|--------------|------------|----------|
| Current (free tier) | $36 | Baseline | Assignment/POC |
| 10x Traffic | $135 | +275% | Successful launch |
| Multi-AZ | $100 | +178% | Production |
| Global (3 regions) | $165 | +358% | International |
| Cost-Optimized | $25 | -31% | Startup budget |

---

### **Hour 2-3: Report Writing (11:00-12:00)**

(Use template in separate report file)

---

### **Hour 3-5: Final Polish (12:00-14:00)**

**Final Verification:**

```
□ All resources tagged "Group3"
□ Both EC2s healthy
□ ALB accessible
□ S3/CloudFront working
□ CloudWatch monitoring active
□ CI/CD pipeline functional
□ Documentation complete
□ Screenshots organized
□ Cost analysis done
□ Ready for presentation
```

**End of Day 4!** 🎉🎉🎉

**YOU DID IT!** The complete D2 implementation is finished! 🚀

