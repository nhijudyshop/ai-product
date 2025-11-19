# GitHub Auto-Deployment Setup

Hướng dẫn setup tự động deploy Cloudflare Worker từ GitHub khi push code.

## 🎯 Lợi ích

- ✅ **Tự động deploy** khi push code lên GitHub
- ✅ **Không cần chạy lệnh thủ công** - GitHub Actions làm tất cả
- ✅ **CI/CD** - Deploy liên tục, nhanh chóng
- ✅ **Bảo mật** - API keys và secrets được mã hóa trên GitHub
- ✅ **Lịch sử deploy** - Theo dõi mọi lần deploy

## 📋 Các bước setup

### Bước 1: Tạo Cloudflare API Token

1. Đăng nhập vào [Cloudflare Dashboard](https://dash.cloudflare.com/)

2. Vào **My Profile** → **API Tokens**

3. Click **Create Token**

4. Chọn template **Edit Cloudflare Workers**

5. Hoặc tạo Custom Token với permissions:
   - **Account** → **Workers Scripts** → **Edit**
   - **Account** → **Account Settings** → **Read**

6. Click **Continue to summary** → **Create Token**

7. **Copy token** và lưu lại (chỉ hiện 1 lần!)

### Bước 2: Lấy Cloudflare Account ID

1. Vào [Cloudflare Dashboard](https://dash.cloudflare.com/)

2. Chọn bất kỳ website nào (hoặc Workers & Pages)

3. Scroll xuống phần **API** ở sidebar bên phải

4. Copy **Account ID**

Hoặc chạy lệnh:
```bash
wrangler whoami
```

### Bước 3: Thêm Secrets vào GitHub Repository

1. Vào GitHub repository của bạn: https://github.com/nhijudyshop/ai-product

2. Click **Settings** → **Secrets and variables** → **Actions**

3. Click **New repository secret**

4. Thêm các secrets sau:

#### Secret 1: CLOUDFLARE_API_TOKEN
- Name: `CLOUDFLARE_API_TOKEN`
- Value: `<Cloudflare API Token bạn vừa tạo>`
- Click **Add secret**

#### Secret 2: CLOUDFLARE_ACCOUNT_ID
- Name: `CLOUDFLARE_ACCOUNT_ID`
- Value: `<Cloudflare Account ID của bạn>`
- Click **Add secret**

#### Secret 3: HF_KEYS
- Name: `HF_KEYS`
- Value: `hf_key1,hf_key2,hf_key3` (API keys của HuggingFace, cách nhau bằng dấu phẩy)
- Click **Add secret**

#### Secret 4: GEMINI_KEYS
- Name: `GEMINI_KEYS`
- Value: `AIzaKey1,AIzaKey2,AIzaKey3` (API keys của Gemini, cách nhau bằng dấu phẩy)
- Click **Add secret**

### Bước 4: Kích hoạt GitHub Actions

1. Vào tab **Actions** trong GitHub repository

2. Nếu Actions chưa được bật, click **I understand my workflows, go ahead and enable them**

3. Workflow `Deploy Cloudflare Worker` sẽ xuất hiện

### Bước 5: Test Auto-Deploy

#### Cách 1: Push code vào main branch

```bash
# Sửa file trong folder cloudflare
echo "// Test deploy" >> cloudflare/server.js

# Commit và push
git add cloudflare/
git commit -m "Test auto-deployment"
git push origin main
```

#### Cách 2: Trigger thủ công

1. Vào tab **Actions**
2. Chọn workflow **Deploy Cloudflare Worker**
3. Click **Run workflow** → **Run workflow**

### Bước 6: Xem kết quả Deploy

1. Vào tab **Actions** trong GitHub

2. Click vào workflow run đang chạy

3. Xem logs để theo dõi quá trình deploy

4. Khi thành công, bạn sẽ thấy:
   ```
   ✅ Cloudflare Worker deployed successfully!
   🔗 Check your worker at: https://ai-api-proxy.your-subdomain.workers.dev
   ```

## 🔄 Workflow hoạt động như thế nào?

File `.github/workflows/deploy-cloudflare-worker.yml` sẽ:

1. **Trigger** khi:
   - Push vào branch `main` hoặc `master`
   - Có thay đổi trong folder `cloudflare/`
   - Hoặc trigger thủ công

2. **Steps**:
   - Checkout code từ GitHub
   - Setup Node.js v20
   - Install Wrangler CLI
   - Deploy lên Cloudflare Workers
   - Inject secrets (HF_KEYS, GEMINI_KEYS)

3. **Kết quả**:
   - Worker được deploy tự động
   - API keys được set từ GitHub Secrets
   - Logs hiển thị trên GitHub Actions

## 🛠️ Troubleshooting

### Lỗi: "Authentication error"

**Nguyên nhân:** API Token không hợp lệ hoặc thiếu permissions

**Giải pháp:**
- Kiểm tra lại CLOUDFLARE_API_TOKEN
- Tạo token mới với đúng permissions

### Lỗi: "Account ID not found"

**Nguyên nhân:** CLOUDFLARE_ACCOUNT_ID sai

**Giải pháp:**
- Kiểm tra lại Account ID
- Chạy `wrangler whoami` để lấy Account ID

### Lỗi: "Secrets not found"

**Nguyên nhân:** Thiếu HF_KEYS hoặc GEMINI_KEYS

**Giải pháp:**
- Thêm HF_KEYS và GEMINI_KEYS vào GitHub Secrets
- Format: `key1,key2,key3` (cách nhau bằng dấu phẩy, không có khoảng trắng)

### Workflow không chạy

**Nguyên nhân:** GitHub Actions chưa được bật

**Giải pháp:**
- Vào Settings → Actions → Enable Actions
- Hoặc push vào branch main để kích hoạt

## 📊 Workflow File

Xem file workflow tại: `.github/workflows/deploy-cloudflare-worker.yml`

```yaml
name: Deploy Cloudflare Worker

on:
  push:
    branches:
      - main
      - master
    paths:
      - 'cloudflare/**'
  workflow_dispatch: # Allow manual trigger

jobs:
  deploy:
    runs-on: ubuntu-latest
    name: Deploy to Cloudflare Workers
    # ... (xem file đầy đủ)
```

## 🔐 Bảo mật

- ✅ API tokens được mã hóa trong GitHub Secrets
- ✅ Không bao giờ log ra secrets
- ✅ Chỉ có repository admins mới thấy secrets
- ✅ Secrets không được expose trong public logs

## 🎉 Lợi ích của Auto-Deploy

1. **Tiết kiệm thời gian** - Không cần chạy `wrangler deploy` thủ công
2. **Ít lỗi hơn** - Tự động deploy, giảm sai sót
3. **Deploy nhanh** - Push code là deploy ngay
4. **Rollback dễ dàng** - Revert commit = revert deploy
5. **Theo dõi tốt hơn** - Lịch sử deploy rõ ràng

## 📚 Resources

- [Cloudflare Wrangler Action](https://github.com/cloudflare/wrangler-action)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Cloudflare Workers Documentation](https://developers.cloudflare.com/workers/)

## ⚡ Quick Setup Commands

```bash
# 1. Tạo Cloudflare API Token (manual)
# → Vào https://dash.cloudflare.com/profile/api-tokens

# 2. Get Account ID
wrangler whoami

# 3. Add secrets to GitHub
# → Vào https://github.com/nhijudyshop/ai-product/settings/secrets/actions

# 4. Push code để trigger deploy
git add .
git commit -m "Setup auto-deployment"
git push origin main

# 5. Xem kết quả
# → Vào https://github.com/nhijudyshop/ai-product/actions
```

---

**Hoàn thành setup, bạn sẽ có CI/CD tự động cho Cloudflare Worker!** 🚀
