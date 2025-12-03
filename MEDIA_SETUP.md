# إعداد نظام إدارة الوسائط
# Media Management System Setup

## إنشاء جدول Media في Supabase

قم بتنفيذ الاستعلام التالي في SQL Editor في Supabase:

```sql
-- إنشاء جدول الوسائط
CREATE TABLE IF NOT EXISTS media (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  type TEXT NOT NULL CHECK (type IN ('image', 'video', 'audio')),
  title TEXT NOT NULL,
  description TEXT,
  url TEXT,
  video_url TEXT,
  file_path TEXT,
  file_url TEXT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- إنشاء فهرس لتحسين الأداء
CREATE INDEX IF NOT EXISTS idx_media_type ON media(type);
CREATE INDEX IF NOT EXISTS idx_media_created_at ON media(created_at DESC);

-- تفعيل RLS (Row Level Security) - اختياري
ALTER TABLE media ENABLE ROW LEVEL SECURITY;

-- سياسة للقراءة العامة (للسماح بعرض الوسائط في المعرض)
CREATE POLICY "Allow public read access" ON media
  FOR SELECT
  USING (true);

-- سياسة للكتابة (للمسؤولين فقط - تحتاج لتعديلها حسب نظام المصادقة)
CREATE POLICY "Allow authenticated insert" ON media
  FOR INSERT
  WITH CHECK (auth.role() = 'authenticated');

CREATE POLICY "Allow authenticated update" ON media
  FOR UPDATE
  USING (auth.role() = 'authenticated');

CREATE POLICY "Allow authenticated delete" ON media
  FOR DELETE
  USING (auth.role() = 'authenticated');

-- دالة لتحديث updated_at تلقائياً
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ language 'plpgsql';

-- تفعيل الدالة على جدول media
CREATE TRIGGER update_media_updated_at BEFORE UPDATE ON media
    FOR EACH ROW EXECUTE FUNCTION update_updated_at_column();
```

## إعداد Supabase Storage

### 1. إنشاء Bucket باسم `media`

1. اذهب إلى Storage في Supabase Dashboard
2. اضغط على "New bucket"
3. اسم الـ bucket: `media`
4. اختر "Public bucket" للسماح بالوصول العام

### 2. إنشاء المجلدات التالية داخل bucket `media`:

- `images/` - للصور
- `audio/` - للملفات الصوتية
- `videos/` - للفيديوهات (إن كانت ستُرفع)

### 3. إعداد السياسات (Policies) للـ Storage:

```sql
-- السماح بالقراءة العامة
CREATE POLICY "Public Access" ON storage.objects
FOR SELECT
USING (bucket_id = 'media');

-- السماح بالرفع للمستخدمين المصادق عليهم
CREATE POLICY "Authenticated Upload" ON storage.objects
FOR INSERT
WITH CHECK (bucket_id = 'media' AND auth.role() = 'authenticated');

-- السماح بالحذف للمستخدمين المصادق عليهم
CREATE POLICY "Authenticated Delete" ON storage.objects
FOR DELETE
USING (bucket_id = 'media' AND auth.role() = 'authenticated');
```

## الميزات المضافة

### 1. إدارة الوسائط الكاملة
- ✅ إضافة صور (رفع إلى Supabase Storage)
- ✅ إضافة فيديو YouTube (حفظ الرابط فقط)
- ✅ إضافة ملفات صوتية (رفع إلى Supabase Storage)
- ✅ تعديل الوسائط (تغيير العنوان، الوصف، أو استبدال الملف)
- ✅ حذف الوسائط (مع حذف الملفات من Storage)

### 2. عرض ديناميكي في المعرض
- ✅ عرض الصور بشكل ديناميكي من Supabase
- ✅ عرض فيديوهات YouTube مع embed
- ✅ عرض ملفات صوتية مع مشغل HTML5

### 3. تحسينات الأداء
- ✅ Lazy Loading للصور
- ✅ Pagination في لوحة التحكم
- ✅ استعلامات محسّنة مع فهارس

### 4. تحسينات تجربة المستخدم
- ✅ معاينة قبل الحفظ (Preview)
- ✅ رسائل نجاح/خطأ واضحة
- ✅ واجهة سهلة ومتناسقة مع التصميم
- ✅ نافذة معاينة للصور عند النقر

## استخدام النظام

### في لوحة التحكم (admin/media.html):

1. **إضافة وسيط جديد:**
   - اضغط على "رفع ملفات جديدة"
   - اختر نوع الوسيط (صورة / فيديو YouTube / صوت)
   - أدخل العنوان والوصف
   - ارفع الملف أو أدخل رابط YouTube
   - اضغط "حفظ"

2. **تعديل وسيط:**
   - اضغط على أيقونة التعديل في بطاقة الوسيط
   - عدّل البيانات المطلوبة
   - يمكنك استبدال الملف/الرابط
   - اضغط "حفظ التعديلات"

3. **حذف وسيط:**
   - اضغط على أيقونة الحذف
   - أكّد الحذف
   - سيتم حذف الوسيط والملف من Storage

### في المعرض (gallery.html):

- يتم تحميل الوسائط تلقائياً من Supabase
- يمكن التبديل بين الصور والفيديوهات والتلاوات
- الصور تظهر مع Lazy Loading لتحسين الأداء
- يمكن النقر على الصور لعرضها بحجم كامل

## ملاحظات مهمة

1. **الأمان:** تأكد من إعداد RLS و Storage Policies بشكل صحيح
2. **الأداء:** النظام يستخدم Pagination لتجنب تحميل عدد كبير من الوسائط دفعة واحدة
3. **التوافق:** النظام متوافق مع الكود الموجود ولا يغيّر البنية الأساسية

