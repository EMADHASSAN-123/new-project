# دليل إعداد Supabase

## الخطوة 1: إنشاء مشروع Supabase

1. اذهب إلى [supabase.com](https://supabase.com)
2. سجل الدخول أو أنشئ حساب جديد
3. اضغط على "New Project"
4. املأ المعلومات:
   - **Project Name**: مركز الشفاء
   - **Database Password**: اختر كلمة مرور قوية
   - **Region**: اختر أقرب منطقة
5. اضغط "Create new project"

## الخطوة 2: الحصول على المفاتيح

1. بعد إنشاء المشروع، اذهب إلى **Settings** > **API**
2. انسخ:
   - **Project URL** (مثال: `https://xxxxx.supabase.co`)
   - **anon public** key

## الخطوة 3: تحديث ملفات التكوين

### تحديث `js/supabase-config.js`

```javascript
const SUPABASE_URL = 'YOUR_SUPABASE_URL'; // الصق Project URL هنا
const SUPABASE_ANON_KEY = 'YOUR_SUPABASE_ANON_KEY'; // الصق anon key هنا
```

### تحديث `js/auth.js`

```javascript
const SUPABASE_URL = 'YOUR_SUPABASE_URL';
const SUPABASE_ANON_KEY = 'YOUR_SUPABASE_ANON_KEY';
```

### تحديث `js/supabase-content.js`

```javascript
const SUPABASE_URL = 'YOUR_SUPABASE_URL';
const SUPABASE_ANON_KEY = 'YOUR_SUPABASE_ANON_KEY';
```

## الخطوة 4: إنشاء الجداول في Supabase

### جدول `content`

1. اذهب إلى **Table Editor** في Supabase Dashboard
2. اضغط **New Table**
3. اسم الجدول: `content`
4. أضف الأعمدة التالية:

| Column Name | Type | Default | Nullable |
|------------|------|---------|----------|
| id | uuid | gen_random_uuid() | No (Primary Key) |
| type | text | - | No |
| title | text | - | No |
| description | text | - | Yes |
| image_url | text | - | Yes |
| file_url | text | - | Yes |
| file_name | text | - | Yes |
| published | boolean | false | No |
| created_at | timestamp | now() | No |
| updated_at | timestamp | now() | No |

5. اضغط **Save**

### إنشاء Policy للوصول

1. اذهب إلى **Authentication** > **Policies**
2. للجدول `content`:
   - **SELECT**: Allow public read access
   - **INSERT**: Require authentication
   - **UPDATE**: Require authentication
   - **DELETE**: Require authentication

## الخطوة 5: إعداد Supabase Storage

### إنشاء Bucket

1. اذهب إلى **Storage** في Supabase Dashboard
2. اضغط **New bucket**
3. اسم الـ Bucket: `media`
4. اختر **Public bucket**
5. اضغط **Create bucket**

### إنشاء مجلدات

أنشئ المجلدات التالية داخل bucket `media`:
- `images/`
- `videos/`
- `audio/`
- `documents/`

### إعداد Policies للـ Storage

1. اذهب إلى **Storage** > **Policies**
2. للـ bucket `media`:
   - **SELECT**: Allow public read access
   - **INSERT**: Require authentication
   - **UPDATE**: Require authentication
   - **DELETE**: Require authentication

## الخطوة 6: إعداد Authentication

### تفعيل Email Authentication

1. اذهب إلى **Authentication** > **Providers**
2. فعّل **Email**
3. يمكنك تعطيل "Confirm email" للتطوير

### إنشاء مستخدم مدير

1. اذهب إلى **Authentication** > **Users**
2. اضغط **Add user** > **Create new user**
3. أدخل:
   - **Email**: admin@example.com
   - **Password**: اختر كلمة مرور قوية
4. اضغط **Create user**

## الخطوة 7: تحديث Row Level Security (RLS)

### للجدول `content`

```sql
-- Enable RLS
ALTER TABLE content ENABLE ROW LEVEL SECURITY;

-- Policy for SELECT (public read)
CREATE POLICY "Public content is viewable by everyone"
ON content FOR SELECT
USING (published = true);

-- Policy for INSERT (authenticated only)
CREATE POLICY "Users can insert content"
ON content FOR INSERT
WITH CHECK (auth.role() = 'authenticated');

-- Policy for UPDATE (authenticated only)
CREATE POLICY "Users can update content"
ON content FOR UPDATE
USING (auth.role() = 'authenticated');

-- Policy for DELETE (authenticated only)
CREATE POLICY "Users can delete content"
ON content FOR DELETE
USING (auth.role() = 'authenticated');
```

## الخطوة 8: اختبار التطبيق

1. افتح `admin/login.html`
2. سجل الدخول بالمستخدم الذي أنشأته
3. جرب إضافة محتوى جديد
4. تحقق من ظهوره في الموقع العام

## ملاحظات مهمة

- **الأمان**: في الإنتاج، استخدم Service Role Key فقط في Backend
- **النسخ الاحتياطي**: قم بعمل نسخ احتياطية دورية للبيانات
- **الحدود**: Supabase Free tier له حدود معينة، تحقق من الخطة المناسبة

## الدعم

للمزيد من المعلومات، راجع [وثائق Supabase](https://supabase.com/docs)

