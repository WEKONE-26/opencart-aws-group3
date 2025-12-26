# 🔧 CI/CD SETUP GUIDE - Deploy Tới Tất Cả ASG Instances (Cách 1)

## 📋 Tóm Tắt Nhanh

**File workflow:** `.github/workflows/deploy.yml` ✅  
**Method:** AWS SSM + Auto Scaling Group  
**Deploy Mode:** **TẤT CẢ instances trong ASG cùng lúc**

**Lợi ích:**
- ✅ Push code 1 lần → tất cả instances update (2, 3, 4, hay bao nhiêu instances cũng được)
- ✅ Instance mới scale up sẽ tự động nhận latest code  
- ✅ Không cần hardcode instance IDs
- ✅ Production-ready (automatic & reliable)

---

## 📝 Workflow Logic

```
Push code to main branch
    ↓
GitHub Actions detects push
    ↓
AWS CLI retrieves all ASG instances
    ↓
SSM send-command to all instances (parallel):
  - Instance 1: git pull + restart
  - Instance 2: git pull + restart
  - Instance 3: git pull + restart (nếu scale up)
    ↓
Tất cả instances cập nhật code ✅
```

---

## 🔧 SETUP STEPS (7 bước, ~30 phút)

### **STEP 1: Tạo AWS IAM User Cho GitHub Actions**

**AWS Console:**
```
1. Go to: https://console.aws.amazon.com/iam/
2. Left sidebar → Users
3. Click "Create user"
4. Username: github-actions-deployer
5. Click "Next"
6. Select "Attach policies directly"
```

### **Attach Required Policies:**

**Policy 1: SSM Access**
```
Search box: "AmazonSSMFullAccess"
Click checkbox next to it
```

**Policy 2: Auto Scaling Access**
```
Search box: "AutoScalingFullAccess"  
Click checkbox next to it
```

**Click "Create user" button**

---

### **STEP 2: Generate Access Keys**

**AWS Console (Tiếp tục):**
```
1. Click on user "github-actions-deployer"
2. Click "Security credentials" tab
3. Scroll down → "Access keys"
4. Click "Create access key"
5. Select "Command Line Interface (CLI)"
6. Click checkbox "I understand the above recommendation"
7. Click "Create access key"
```

**📥 Download CSV file immediately:**
```
Click "Download .csv file"
Save to your computer
```

**File CSV content example:**
```
User name,Access key ID,Secret access key
github-actions-deployer,AKIA1234567890AB,wJalrXUtnFEMI/K7MDENG/ExampleKey
```

⚠️ **Save this file - you'll need it!**

---

### **STEP 3: Add AWS Credentials to GitHub Secrets**

**Go to GitHub:**
```
1. Your repository
2. Settings tab
3. Left sidebar → "Secrets and variables" → "Actions"
4. Click "New repository secret"
```

**Add Secret 1:**
```
Name: AWS_ACCESS_KEY_ID
Value: AKIA... (copy from CSV file, first value)
Click "Add secret"
```

**Add Secret 2:**
```
Name: AWS_SECRET_ACCESS_KEY  
Value: wJalr... (copy from CSV file, second value)
Click "Add secret"
```

⚠️ **Copy from CSV file exactly - don't miss any characters!**

---

### **STEP 4: Configure EC2 IAM Role (For ASG Instances)**

**AWS Console:**
```
1. Go to: https://console.aws.amazon.com/iam/
2. Left sidebar → Roles
3. Find role: "Group3_EC2_S3_Role" (or your EC2 role)
4. Click on it
```

**Add SSM Permission:**
```
1. Click "Add permissions" → "Attach policies directly"
2. Search: "AmazonSSMManagedInstanceCore"
3. Click checkbox
4. Click "Add permissions"
```

**Verify Role Has:**
```
✅ AmazonSSMManagedInstanceCore (for SSM)
✅ AmazonEC2FullAccess or AmazonS3FullAccess (existing)
```

---

### **STEP 5: Configure Git on EC2**

**SSH vào 1 trong các ASG instances:**

```bash
# Get EC2 public IP from AWS Console
# EC2 → Instances → find Group3-OpenCart-ASG instance → copy Public IPv4 address

ssh -i your-key.pem ec2-user@YOUR-EC2-PUBLIC-IP
```

**Run these commands:**

```bash
# 1. Install Git (nếu chưa có)
sudo yum install -y git

# 2. Go to OpenCart directory
cd /var/www/html

# 3. Configure Git
sudo git config --global user.email "github-actions@example.com"
sudo git config --global user.name "GitHub Actions"

# 4. Initialize Git repository
sudo git init
sudo git remote add origin https://github.com/WEKONE-26/opencart-aws-group3.git

# 5. Fetch and checkout main branch
sudo git fetch origin main
sudo git checkout main

# 6. Set permissions
sudo chown -R apache:apache /var/www/html
sudo chmod -R 755 /var/www/html

# 7. Create deployment log file
sudo touch /var/log/deployment.log
sudo chmod 666 /var/log/deployment.log

# 8. Exit SSH
exit
```

---

### **STEP 6: Test Deployment**

**Option A: Push code to trigger workflow**

```powershell
# On your local machine
cd your-repo-folder

# Make a small change
echo "Test CI/CD ASG deployment" >> README.md

# Commit and push
git add README.md
git commit -m "Test: Trigger CI/CD ASG deployment"
git push origin main

# GitHub Actions will automatically trigger!
```

**Option B: Manual trigger from GitHub UI**

```
1. GitHub → Actions tab
2. Click "Deploy OpenCart to ASG (All Instances)"
3. Click "Run workflow"
4. Select branch "main"
5. Click "Run workflow"
```

---

### **STEP 7: Monitor Deployment**

**On GitHub Actions:**
```
1. Repository → Actions tab
2. See latest workflow run
3. Status:
   🟡 Yellow = Running
   🟢 Green = Success ✅
   🔴 Red = Error ❌
```

**Click workflow to see logs:**
```
- "Get all ASG instances" → shows which instances found
- "Deploy code to all ASG instances" → deployment output
- "Deployment Summary" → final status
```

**On Website:**
```
1. Open ALB DNS: http://Group3-OpenCart-ALB-XXXX.ap-southeast-1.elb.amazonaws.com
2. Refresh (F5)
3. Check if changes visible
4. If yes → deployment successful ✅
```

**Check EC2 Logs:**
```bash
# SSH to any ASG instance
ssh -i your-key.pem ec2-user@YOUR-EC2-IP

# View deployment logs
tail -50 /var/log/deployment.log

# Output example:
# === Deploying to ASG Instance ===
# ✅ Deployment completed at Thu Dec 26 11:45:32 UTC 2025
# ✅ Deployment completed at Thu Dec 26 11:46:15 UTC 2025
```

---

## 🚨 TROUBLESHOOTING

### ❌ Error: "InvalidParameterValue" / "Instances not found"

```
Nguyên Nhân: ASG name sai hoặc không có instances

Cách Fix:
1. AWS Console → Auto Scaling Groups
2. Verify ASG name: "Group3-OpenCart-ASG" ✅
3. Verify instances running (Desired: 2, Running: 2)
4. Trigger workflow lại
```

### ❌ Error: "AccessDenied" / "UnauthorizedOperation"

```
Nguyên Nhân: AWS credentials không có SSM permissions

Cách Fix:
1. AWS IAM → Users → github-actions-deployer
2. Verify policies: AmazonSSMFullAccess ✅
3. GitHub → Settings → Secrets
4. Verify both secrets are set correctly
5. Trigger workflow lại
```

### ❌ Error: "SSMInstanceNotExistException"

```
Nguyên Nhân: EC2 instances không có SSM agent permission

Cách Fix:
1. AWS IAM → Roles → Group3_EC2_S3_Role
2. Add policy: AmazonSSMManagedInstanceCore
3. Terminate old ASG instances (force new ones to launch)
4. Wait 2 minutes for new instances to start SSM agent
5. Trigger workflow lại
```

### ❌ Error: "git: not found"

```
Nguyên Nhân: EC2 không cài git

Cách Fix:
1. SSH to EC2
2. Run: sudo yum install -y git
3. Run STEP 5 lại (git config + init)
4. Trigger workflow lại
```

---

## ✅ COMPLETION CHECKLIST

```
AWS IAM Setup:
☐ github-actions-deployer user created
☐ AmazonSSMFullAccess attached
☐ AutoScalingFullAccess attached  
☐ Access keys downloaded (CSV file saved)

GitHub Secrets:
☐ AWS_ACCESS_KEY_ID set
☐ AWS_SECRET_ACCESS_KEY set

EC2 IAM Role:
☐ AmazonSSMManagedInstanceCore attached to Group3_EC2_S3_Role

EC2 Configuration:
☐ Git installed on at least 1 ASG instance
☐ Git repository initialized (/var/www/html)
☐ Main branch checked out
☐ Permissions set (apache:apache, 755)
☐ Log file created (/var/log/deployment.log)

Testing:
☐ First deployment triggered (push or manual)
☐ GitHub Actions workflow runs ✅ (green)
☐ Logs show "Deploying to ASG Instance"
☐ Website shows changes
☐ All instances in ASG received update

✅ CI/CD FOR ASG READY!
```

---

## 📊 DAILY WORKFLOW (After Setup)

**Every day, deploy code like this:**

```bash
# 1. Edit file on local machine
# Example: catalog/controller/common/home.php

# 2. Test locally (if you have Docker/XAMPP)

# 3. Commit and push
git add .
git commit -m "Feature: Add new product filter"
git push origin main

# 4. ✅ DONE! 
#    GitHub Actions automatically:
#    - Detects push
#    - Gets all ASG instances
#    - Deploys to ALL instances in parallel
#    - Website updates in 1-2 minutes

# 5. Verify
#    - Check GitHub Actions logs ✅
#    - Refresh website, see changes ✅
```

---

## 🎯 ASG Scale-Up Scenario

**What happens when new instance launches:**

```
Current state: 2 instances (i-111, i-222) both with OpenCart

Auto Scaling triggers → New instance i-333 launches
    ↓
New instance i-333 starts with same AMI (has code)
    ↓
But code might be slightly old (AMI was created N hours ago)
    ↓
Next time you git push → workflow runs
    ↓
SSM discovers: 3 instances now (i-111, i-222, i-333)
    ↓
Deploys to ALL 3 at once → ALL get latest code ✅
    ↓
New instance i-333 is now in sync with i-111 and i-222 ✅
```

**Result:** Scales up don't cause out-of-sync issues!

---

## 💡 KEY DIFFERENCES: SSH vs SSM

| Aspect | SSH (Old) | SSM (New - Cách 1) |
|--------|-----------|------------------|
| **Targets** | 1 hardcoded IP | All ASG instances |
| **Scale-up** | Manual SSH each time | Auto-detected |
| **Setup** | SSH key | AWS credentials |
| **Security** | Requires SSH access | Uses IAM + AWS credentials |
| **Reliability** | Single point of failure | Parallel deployment |
| **Cost** | Free | Free (included in AWS) |

---

## 🚀 You're Ready!

Your CI/CD can now:
- ✅ Deploy to 2 instances
- ✅ Deploy to 3 instances (if scaled)
- ✅ Deploy to 10 instances (no extra setup!)
- ✅ Handle new instances automatically

**Push code once → All instances update automatically!** 🎉
