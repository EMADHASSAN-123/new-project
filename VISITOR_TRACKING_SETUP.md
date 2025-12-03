# Visitor Tracking Setup Guide
# دليل إعداد تتبع الزوار

## Supabase Table Setup
## إعداد جدول Supabase

You need to create a `visitors` table in your Supabase database. Run the following SQL in your Supabase SQL Editor:

تحتاج إلى إنشاء جدول `visitors` في قاعدة بيانات Supabase. قم بتشغيل SQL التالي في محرر SQL في Supabase:

```sql
-- Create visitors table
-- إنشاء جدول الزوار
CREATE TABLE IF NOT EXISTS visitors (
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    fingerprint TEXT NOT NULL,
    ip_address TEXT,
    user_agent TEXT,
    referrer TEXT,
    screen_resolution TEXT,
    timezone TEXT,
    language TEXT,
    visited_at TIMESTAMPTZ DEFAULT NOW(),
    page_url TEXT,
    page_path TEXT,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Create index on fingerprint for faster queries
-- إنشاء فهرس على fingerprint للاستعلامات الأسرع
CREATE INDEX IF NOT EXISTS idx_visitors_fingerprint ON visitors(fingerprint);

-- Create index on visited_at for date-based queries
-- إنشاء فهرس على visited_at للاستعلامات القائمة على التاريخ
CREATE INDEX IF NOT EXISTS idx_visitors_visited_at ON visitors(visited_at);

-- Create index on created_at
-- إنشاء فهرس على created_at
CREATE INDEX IF NOT EXISTS idx_visitors_created_at ON visitors(created_at);

-- Enable Row Level Security (RLS)
-- تفعيل أمان مستوى الصف (RLS)
ALTER TABLE visitors ENABLE ROW LEVEL SECURITY;

-- Create policy to allow authenticated users to read all visitors
-- إنشاء سياسة للسماح للمستخدمين المصادق عليهم بقراءة جميع الزوار
CREATE POLICY "Allow authenticated users to read visitors"
    ON visitors FOR SELECT
    TO authenticated
    USING (true);

-- Create policy to allow anonymous inserts (for tracking)
-- إنشاء سياسة للسماح بإدراج مجهول (للتتبع)
CREATE POLICY "Allow anonymous inserts for tracking"
    ON visitors FOR INSERT
    TO anon
    WITH CHECK (true);
```

## Features
## المميزات

### 1. Admin Session Verification
### 1. التحقق من جلسة المدير

- ✅ Enhanced session verification on all dashboard pages
- ✅ Displays admin name (full name or email) in dashboard header
- ✅ Automatic redirect to login if not authenticated
- ✅ Works consistently across all dashboard pages

- ✅ تحقق محسّن من الجلسة في جميع صفحات لوحة التحكم
- ✅ عرض اسم المدير (الاسم الكامل أو البريد الإلكتروني) في رأس لوحة التحكم
- ✅ إعادة توجيه تلقائية إلى صفحة تسجيل الدخول إذا لم يكن مصادقاً عليه
- ✅ يعمل بشكل متسق عبر جميع صفحات لوحة التحكم

### 2. Visitor Tracking
### 2. تتبع الزوار

- ✅ Tracks unique visitors on homepage
- ✅ Prevents duplicate entries using localStorage
- ✅ Uses fingerprinting for device identification
- ✅ Stores visitor data in Supabase `visitors` table
- ✅ Tracks IP address, user agent, referrer, and more

- ✅ يتتبع الزوار الفريدين في الصفحة الرئيسية
- ✅ يمنع الإدخالات المكررة باستخدام localStorage
- ✅ يستخدم البصمة لتحديد الجهاز
- ✅ يخزن بيانات الزوار في جدول `visitors` في Supabase
- ✅ يتتبع عنوان IP، وكيل المستخدم، المرجع، والمزيد

### 3. Analytics Dashboard
### 3. لوحة تحكم الإحصائيات

- ✅ Total unique visitors count
- ✅ Today's visitors count
- ✅ Last 7 days visitors count
- ✅ Average daily visitors
- ✅ Interactive line chart showing daily visitors
- ✅ Recent visitors table with details

- ✅ عدد إجمالي الزوار الفريدين
- ✅ عدد زوار اليوم
- ✅ عدد زوار آخر 7 أيام
- ✅ متوسط الزوار اليومي
- ✅ رسم بياني خطي تفاعلي يوضح الزوار اليوميين
- ✅ جدول آخر الزوار مع التفاصيل

## Files Created/Modified
## الملفات المُنشأة/المُعدّلة

### New Files
### ملفات جديدة

1. `js/admin-header.js` - Admin header module for displaying admin name
2. `js/visitor-tracking.js` - Visitor tracking module
3. `js/analytics.js` - Analytics dashboard module
4. `admin/analytics.html` - Analytics page
5. `VISITOR_TRACKING_SETUP.md` - This setup guide

### Modified Files
### ملفات معدّلة

1. `js/auth.js` - Enhanced session verification and admin user info
2. `js/dashboard-supabase.js` - Added admin header initialization
3. `js/content-management-supabase.js` - Added auth verification and admin header
4. `js/ai-tools.js` - Added auth verification and admin header
5. `js/media-management.js` - Added auth verification and admin header
6. `admin/dashboard.html` - Added analytics link and admin header script
7. `admin/content-management.html` - Added analytics link and admin header script
8. `admin/ai-tools.html` - Added analytics link, auth scripts, and admin header
9. `admin/media.html` - Added analytics link and admin header script
10. `index.html` - Added visitor tracking script

## Usage
## الاستخدام

### For Admins
### للمدراء

1. Log in to the admin dashboard
2. Navigate to "إحصائيات الزوار" (Analytics) from the sidebar
3. View visitor statistics and charts
4. Your name will be displayed in the dashboard header

1. سجل الدخول إلى لوحة تحكم المدير
2. انتقل إلى "إحصائيات الزوار" من الشريط الجانبي
3. عرض إحصائيات الزوار والرسوم البيانية
4. سيتم عرض اسمك في رأس لوحة التحكم

### For Visitors
### للزوار

- Visitors are automatically tracked when they visit the homepage
- Each unique device/session is tracked once per day
- No user interaction required

- يتم تتبع الزوار تلقائياً عند زيارة الصفحة الرئيسية
- يتم تتبع كل جهاز/جلسة فريدة مرة واحدة في اليوم
- لا يتطلب تفاعل المستخدم

## Notes
## ملاحظات

- The visitor tracking uses localStorage to prevent duplicate entries per day
- Fingerprinting is used to identify unique devices
- IP address is fetched from external services (may have rate limits)
- All tracking happens asynchronously and doesn't block page loading
- The analytics page requires admin authentication

- يستخدم تتبع الزوار localStorage لمنع الإدخالات المكررة يومياً
- يتم استخدام البصمة لتحديد الأجهزة الفريدة
- يتم جلب عنوان IP من خدمات خارجية (قد يكون لها حدود معدل)
- يحدث التتبع بشكل غير متزامن ولا يعيق تحميل الصفحة
- تتطلب صفحة الإحصائيات مصادقة المدير

