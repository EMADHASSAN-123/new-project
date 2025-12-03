# دليل إعداد متغيرات البيئة
# Environment Variables Setup Guide

## نظرة عامة
تم نقل جميع المفاتيح الحساسة من الكود إلى ملفات منفصلة محمية من Git لضمان الأمان عند رفع المشروع على GitHub.

## الملفات المُنشأة

### 1. `.env` (مُستبعد من Git)
يحتوي على المفاتيح الحقيقية. **لا ترفع هذا الملف إلى Git!**

### 2. `.env.example`
ملف قالب يحتوي على أسماء المتغيرات بدون قيم حقيقية. يمكن رفعه بأمان إلى Git.

### 3. `config.js` (مُستبعد من Git)
ملف JavaScript يحتوي على المفاتيح الحقيقية للاستخدام في المتصفح. **لا ترفع هذا الملف إلى Git!**

### 4. `config.example.js`
ملف قالب يحتوي على البنية بدون قيم حقيقية. يمكن رفعه بأمان إلى Git.

### 5. `.gitignore`
تم تحديثه لاستبعاد الملفات الحساسة:
- `.env`
- `config.js`

## خطوات الإعداد

### للمطورين الجدد:

1. **انسخ ملف القالب:**
   ```bash
   cp config.example.js config.js
   ```

2. **افتح `config.js` واملأ القيم الحقيقية:**
   ```javascript
   window.ENV_CONFIG = {
       SUPABASE_URL: "YOUR_SUPABASE_URL_HERE",
       SUPABASE_ANON_KEY: "YOUR_SUPABASE_ANON_KEY_HERE"
   };
   ```

3. **احصل على المفاتيح من Supabase:**
   - اذهب إلى [Supabase Dashboard](https://app.supabase.com)
   - اختر مشروعك
   - اذهب إلى **Settings** > **API**
   - انسخ:
     - **Project URL** → `SUPABASE_URL`
     - **anon public** key → `SUPABASE_ANON_KEY`

## كيفية عمل النظام

1. يتم تحميل `config.js` في جميع صفحات HTML قبل `supabase-config.js`
2. `supabase-config.js` يقرأ القيم من `window.ENV_CONFIG`
3. إذا لم يتم العثور على القيم، يتم عرض تحذير في Console

## التحقق من الإعداد

افتح Console المتصفح (F12) وتحقق من عدم وجود رسائل خطأ. يجب أن ترى:
```
Supabase initialized successfully (GLOBAL MODE)
```

إذا رأيت تحذيرًا حول عدم تكوين Supabase، تأكد من:
- وجود ملف `config.js`
- تحميل `config.js` قبل `supabase-config.js` في HTML
- صحة القيم في `config.js`

## الأمان

✅ **افعل:**
- ارفع `config.example.js` و `.env.example` إلى Git
- احتفظ بـ `config.js` و `.env` محليًا فقط
- استخدم `.gitignore` لحماية الملفات الحساسة

❌ **لا تفعل:**
- لا ترفع `config.js` إلى Git
- لا ترفع `.env` إلى Git
- لا تشارك المفاتيح الحقيقية في الكود أو في المحادثات العامة

## استكشاف الأخطاء

### المشكلة: "Supabase is not configured properly"
**الحل:** تأكد من وجود `config.js` وأنه يحتوي على القيم الصحيحة.

### المشكلة: الموقع لا يتصل بـ Supabase
**الحل:** 
1. تحقق من تحميل `config.js` قبل `supabase-config.js`
2. تحقق من صحة المفاتيح في `config.js`
3. افتح Console المتصفح للتحقق من الأخطاء

## ملاحظات

- ملف `config.js` مطلوب للعمل في المتصفح لأنه لا يمكن قراءة `.env` مباشرة في JavaScript
- تم تحديث جميع ملفات HTML لتضمين `config.js` قبل `supabase-config.js`
- النظام يعمل مع المشروع الحالي بدون تغييرات إضافية

