# 🔧 CI/CD SETUP GUIDE - GitHub Secrets & EC2 Configuration

## 📋 Tóm Tắt Nhanh

**File workflow đã fixed:** `.github/workflows/deploy.yml` ✅
**Method:** SSH trực tiếp (đơn giản, không cần IAM phức tạp)
**Yêu cầu:** 2 GitHub Secrets + Git config trên EC2

---

## 🔑 STEP 1: Lấy EC2 Host (Public IP)

**AWS Console:**
```
1. EC2 → Instances
2. Chọn 1 instance (hoặc bất kỳ ASG instance nào)
3. Tìm "Public IPv4 address"
   VD: 13.251.156.78
```

**Lưu giữ giá trị này → Dùng cho EC2_HOST secret**

---

## 🔑 STEP 2: Lấy EC2 SSH Private Key

**AWS Console:**
```
1. EC2 → Key Pairs
2. Tìm key pair bạn dùng (VD: Group3-Key.pem, project.pem)
3. Click Download key pair
4. Mở file .pem bằng Notepad/VS Code
5. Copy TẤT CẢ content
```

**Ví dụ nội dung file .pem:**
```
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUtbm9uZS1ub25lAAAAAA...
... (hàng dài ký tự)
-----END OPENSSH PRIVATE KEY-----
```

**Copy từ BEGIN đến END**

---

## 🔐 STEP 3: Thêm GitHub Secrets

**Trên GitHub Repository:**

```
1. Repository → Settings (tab)
2. Sidebar → "Secrets and variables" → "Actions"
3. Click "New repository secret"
```

### Secret 1: EC2_HOST

```
Name: EC2_HOST
Value: 13.251.156.78 (hoặc IP của bạn)
Click "Add secret"
```

### Secret 2: EC2_SSH_KEY

```
Name: EC2_SSH_KEY
Value: (Paste toàn bộ content từ .pem file)

⚠️ IMPORTANT:
- Bắt đầu: -----BEGIN OPENSSH PRIVATE KEY-----
- Kết thúc: -----END OPENSSH PRIVATE KEY-----
- Copy tất cả, không bỏ dòng nào

Click "Add secret"
```

---

## 🔧 STEP 4: Cấu Hình Git Trên EC2

**SSH vào 1 trong 2 EC2 instances:**

```bash
ssh -i your-key.pem ec2-user@13.251.156.78
```

**Chạy các lệnh sau:**

```bash
# 1. Cài Git (nếu chưa có)
sudo yum update -y
sudo yum install -y git

# 2. Vào thư mục OpenCart
cd /var/www/html

# 3. Cấu hình Git
sudo git config --global user.email "github-actions@example.com"
sudo git config --global user.name "GitHub Actions"

# 4. Khởi tạo repository (nếu chưa có)
sudo git init
sudo git remote add origin https://github.com/WEKONE-26/opencart-aws-group3.git

# 5. Lấy code từ main branch
sudo git fetch origin main
sudo git checkout main

# 6. Cấp quyền cho Apache
sudo chown -R apache:apache /var/www/html
sudo chmod -R 755 /var/www/html

# 7. Tạo log file cho deployment tracking
sudo touch /var/log/deployment.log
sudo chmod 666 /var/log/deployment.log

# 8. Exit SSH
exit
```

---

## ✅ STEP 5: Test Deployment Đầu Tiên

**Cách A: Push code tự động trigger**

```powershell
# Trên máy local
cd your-repo-folder

# Edit 1 file bất kỳ
echo "Test CI/CD" >> README.md

# Commit và push
git add .
git commit -m "Test: Trigger CI/CD deployment"
git push origin main

# GitHub Actions sẽ tự động chạy!
```

**Cách B: Manual trigger từ GitHub UI**

```
1. GitHub → Actions tab
2. Click "Deploy OpenCart to EC2" workflow
3. Click "Run workflow"
4. Chọn branch "main"
5. Click "Run workflow"
```

---

## 📊 STEP 6: Kiểm Tra Kết Quả

**Trên GitHub Actions:**

```
1. GitHub → Actions → Latest run
2. Xem status:
   🟡 Yellow = Đang chạy
   🟢 Green = Thành công ✅
   🔴 Red = Lỗi ❌
3. Click vào run để xem logs
```

**Trên Website:**

```
1. Mở: http://Group3-OpenCart-ALB-XXXX.ap-southeast-1.elb.amazonaws.com
2. F5 refresh
3. Xem file bạn vừa edit → Nếu thấy thì deployment thành công ✅
```

**Trên EC2 Instance:**

```bash
# SSH vào EC2
ssh -i your-key.pem ec2-user@13.251.156.78

# Xem deployment logs
tail -20 /var/log/deployment.log

# Output sẽ như:
# ✅ Deployment completed at Thu Dec 26 10:45:32 UTC 2025
# ✅ Deployment completed at Thu Dec 26 10:46:15 UTC 2025
```

---

## 🚨 TROUBLESHOOTING

### ❌ Error: "Permission denied (publickey)"

```
Nguyên Nhân: EC2_SSH_KEY sai

Cách Fix:
1. GitHub → Settings → Secrets
2. Delete old EC2_SSH_KEY
3. Copy lại .pem file (copy từ BEGIN đến END)
4. Thêm EC2_SSH_KEY mới
5. Trigger workflow lại
```

### ❌ Error: "Could not resolve hostname"

```
Nguyên Nhân: EC2_HOST IP sai

Cách Fix:
1. AWS Console → EC2 → Instances
2. Copy "Public IPv4 address" chính xác
3. GitHub → Settings → Secrets
4. Edit EC2_HOST
5. Trigger workflow lại
```

### ❌ Error: "fatal: Not a git repository"

```
Nguyên Nhân: EC2 chưa init Git

Cách Fix:
1. SSH vào EC2
2. Run STEP 4 lại
3. Trigger workflow lại
```

### ❌ Error: "permission denied" trên /var/www/html

```
Nguyên Nhân: Apache không có quyền

Cách Fix:
1. SSH vào EC2
2. Chạy:
   sudo chown -R apache:apache /var/www/html
   sudo chmod -R 755 /var/www/html
3. Trigger workflow lại
```

---

## 📋 CHECKLIST HOÀN THÀNH

```
GitHub Secrets:
☐ EC2_HOST added (public IP)
☐ EC2_SSH_KEY added (full .pem content)

EC2 Configuration:
☐ Git installed
☐ Git repository initialized
☐ Git user configured
☐ Main branch checked out
☐ Permissions set (apache:apache)
☐ Log file created

Testing:
☐ First deployment triggered
☐ GitHub Actions workflow ✅ (green)
☐ Website shows changes
☐ Logs show "Deployment completed"

✅ CI/CD READY!
```

---

## 📝 Quy Trình Hằng Ngày

Sau khi setup xong, cách bạn deploy mỗi ngày:

```bash
# 1. Edit file trên local
# VD: sửa catalog/controller/common/home.php

# 2. Commit và push
git add .
git commit -m "Fix: Homepage display issue"
git push origin main

# 3. ✅ DONE! GitHub Actions tự động deploy trong 1-2 phút
#    Không cần SSH, không cần thủ công gì khác!
```

**Kiểm tra:**
- GitHub Actions → xem workflow ✅
- Website → F5 → thấy thay đổi ✅
- Done!

---

## 🎯 Tóm Tắt CI/CD

| Vấn Đề | Chi Tiết |
|--------|----------|
| **Setup Time** | ~30 phút (lần đầu) |
| **Deploy Time** | 1-2 phút (tự động) |
| **Manual Work** | git add → git commit → git push |
| **Tất Cả Instances** | ASG tự động update (bao nhiêu instances cũng được) |
| **Rollback** | git revert + git push (1 phút) |
| **Cost** | Miễn phí (GitHub Actions free tier) |

---

## ✨ Workflow Logic

```
Push code to main branch
    ↓
GitHub Actions detects push
    ↓
Workflow runs:
  1. Checkout code
  2. Connect to EC2 via SSH
  3. cd /var/www/html
  4. git fetch origin main
  5. git reset --hard origin/main (update files)
  6. systemctl restart httpd (restart Apache)
  7. Log output
    ↓
Website automatically updated ✅
All instances in ASG get same code ✅
Customers see changes immediately ✅
```
