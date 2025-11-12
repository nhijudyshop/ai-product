# 🔧 TROUBLESHOOTING GUIDE - Vision AI

## ⚠️ Các Lỗi Thường Gặp & Cách Fix

---

## 1. 🔴 Error 429: Quota Exceeded

### Nguyên nhân:
```
Key đã hết quota 250 requests/ngày
```

### Giải pháp:
✅ **Tự động:** Hệ thống sẽ tự động chuyển sang key khác
✅ **Manual:** Nếu tất cả 5 keys đều hết quota:
- Đợi đến ngày mai (reset 00:00 Pacific Time)
- Hoặc hệ thống tự động fallback sang Florence-2

### Logs Console:
```javascript
⚠️ Key 4 failed with 429
🔄 Quota exceeded, trying next key...
🔑 Trying Gemini key 5/5...
```

---

## 2. 🟡 Error 503: Model Overloaded

### Nguyên nhân:
```
Server Gemini đang quá tải, nhiều người dùng cùng lúc
```

### Giải pháp:
✅ **Tự động:** Hệ thống retry với delay 3 giây
✅ **Manual:** 
- Đợi vài giây rồi thử lại
- Hoặc dùng Florence-2 (luôn available)

### Logs Console:
```javascript
⚠️ Key 1 failed with 503
⏳ Server overloaded, waiting 3 seconds...
🔑 Trying Gemini key 2/5...
```

---

## 3. 🔴 Error 403: Forbidden

### Nguyên nhân:
```
Key không hợp lệ hoặc bị khóa
```

### Giải pháp:
✅ **Tự động:** System skip key này và thử key khác
✅ **Manual:** Nếu tất cả keys đều 403:
- Check xem keys có bị revoke không
- Hoặc sử dụng Florence-2

---

## 4. ❌ "All Gemini keys failed"

### Nguyên nhân:
```
Tất cả 5 Gemini keys đều không khả dụng:
- Hết quota (429)
- Server overload (503)
- Keys invalid (403)
```

### Giải pháp:
✅ **Auto-Fallback đã kích hoạt!**

Khi thấy message này, hệ thống sẽ:
```javascript
⚠️ Gemini không khả dụng: All Gemini keys failed
🔄 Đang chuyển sang Florence-2...
[Phân tích với Florence-2 thành công]
```

### Không cần làm gì cả! System tự động handle.

---

## 5. 🟠 Florence-2 Error 503

### Nguyên nhân:
```
Model đang loading vào memory (cold start)
```

### Giải pháp:
✅ **Tự động:** System retry 3 lần với delay 3 giây
✅ **Manual:** Đợi 5-10 giây rồi thử lại

### Logs Console:
```javascript
Model loading, retrying in 3 seconds...
Florence attempt 2/3...
✅ Success!
```

---

## 6. 🔵 Image Too Large

### Nguyên nhân:
```
Ảnh quá lớn (> 10MB)
```

### Giải pháp:
✅ Resize ảnh trước khi upload
✅ Dùng tool online để compress: tinypng.com
✅ Hoặc resize trong code:

```javascript
function resizeImage(file, maxSize = 2048) {
    return new Promise((resolve) => {
        const img = new Image();
        img.onload = () => {
            const canvas = document.createElement('canvas');
            let width = img.width;
            let height = img.height;
            
            if (width > maxSize || height > maxSize) {
                if (width > height) {
                    height *= maxSize / width;
                    width = maxSize;
                } else {
                    width *= maxSize / height;
                    height = maxSize;
                }
            }
            
            canvas.width = width;
            canvas.height = height;
            canvas.getContext('2d').drawImage(img, 0, 0, width, height);
            
            canvas.toBlob(resolve, 'image/jpeg', 0.8);
        };
        img.src = URL.createObjectURL(file);
    });
}
```

---

## 🎯 Flow Chart - Error Handling

```
User uploads image
    ↓
Try Gemini Key 1
    ├─ Success → Display result ✅
    ├─ 429 (quota) → Try Key 2
    ├─ 503 (overload) → Wait 3s → Try Key 2
    └─ 403 (forbidden) → Skip → Try Key 2
    
Try Gemini Key 2-5
    ├─ Success → Display result ✅
    └─ All failed → Auto-fallback
    
Auto-Fallback to Florence-2
    ├─ 503 → Retry 3x with 3s delay
    ├─ Success → Display result ✅
    └─ Failed → Show error message ❌
```

---

## 📊 Status Codes Reference

| Code | Meaning | Auto-Action | Wait Time |
|------|---------|-------------|-----------|
| **200** | ✅ Success | None | - |
| **403** | 🔴 Forbidden | Skip key | 0.5s |
| **429** | 🔴 Quota exceeded | Next key | 0.5s |
| **503** | 🟡 Overloaded | Retry/Next key | 3s |
| **500** | 🔴 Server error | Retry | 2s |

---

## 🔍 Debugging Tips

### 1. Enable Console Logs
```javascript
// Mở Chrome DevTools: F12 → Console
// Logs sẽ hiển thị:
🔑 Trying Gemini key 1/5...
⚠️ Key 1 failed with 429
🔄 Quota exceeded, trying next key...
✅ Gemini success with key 2
```

### 2. Check Network Tab
```
F12 → Network → Filter: generateContent
→ Click vào request → Preview → Xem error detail
```

### 3. Manual Key Test
```javascript
// Test 1 key trong console:
const testKey = "AIzaSyC...";
fetch(`https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=${testKey}`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
        contents: [{ parts: [{ text: "test" }] }]
    })
}).then(r => r.json()).then(console.log);
```

---

## ⚡ Performance Tips

### 1. Giảm Load Time
```javascript
// Resize image trước khi gửi
const maxSize = 1024; // 1024x1024 max
// Compress quality: 0.8 (80%)
```

### 2. Cache Results
```javascript
const cache = new Map();
if (cache.has(imageHash)) {
    return cache.get(imageHash);
}
// Analyze & cache
cache.set(imageHash, result);
```

### 3. Batch Processing
```javascript
// Add delay giữa requests
for (const image of images) {
    await analyzeImage(image);
    await sleep(500); // 0.5s delay
}
```

---

## 🆘 Still Having Issues?

### Check These:
1. ✅ Internet connection stable?
2. ✅ Image file < 10MB?
3. ✅ Image format: JPG, PNG, WEBP?
4. ✅ Console logs có errors gì?
5. ✅ Browser: Chrome/Edge (latest)?

### Quick Fixes:
- 🔄 Hard refresh: Ctrl + Shift + R
- 🗑️ Clear cache: Ctrl + Shift + Delete
- 🔄 Try different browser
- 📱 Try on mobile device

### Manual Fallback:
```javascript
// If auto-fallback không work, force Florence-2:
currentProvider = "huggingface";
analyzeImage();
```

---

## 📈 Expected Behavior

### Normal Flow (Success):
```
1. Upload ảnh
2. Gemini Key 1 → Success (1-2s)
3. Display result ✅
```

### With Fallback (Some keys failed):
```
1. Upload ảnh
2. Gemini Key 1 → 429 (quota)
3. Gemini Key 2 → Success (2-3s)
4. Display result ✅
```

### Full Fallback (All Gemini failed):
```
1. Upload ảnh
2. Try all 5 Gemini keys → All failed
3. Warning: "Gemini không khả dụng"
4. Auto-switch to Florence-2
5. Florence-2 → Success (3-5s)
6. Display result ✅
```

---

## 📞 Support Info

### Files to Check:
- `vision-ai-free.html` - Main demo
- `n2shop-vision-integration.js` - Integration code
- `GEMINI-KEYS-INFO.md` - Keys documentation

### Console Commands:
```javascript
// Check current stats
getStats()

// Check key status
console.log('Current Gemini key:', currentGeminiKeyIndex)
console.log('Failed keys:', Array.from(failedGeminiKeys))

// Force provider
currentProvider = "huggingface" // or "gemini"
```

---

**Remember:** Hệ thống được thiết kế để auto-handle 99% errors. Chỉ cần click và đợi kết quả! 🎉

---

Last Updated: 2024
Version: 1.0
Status: ✅ Production Ready
