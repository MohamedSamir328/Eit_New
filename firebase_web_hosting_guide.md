# دليل نشر تطبيق Flutter على Firebase Hosting

هذا الدليل يشرح كيفية نشر تطبيق Flutter على Firebase Hosting للحصول على عنوان URL يمكن الوصول إليه من أي متصفح.

## المتطلبات الأساسية

1. تثبيت Firebase CLI (واجهة سطر الأوامر)
2. تهيئة Firebase في مشروع Flutter
3. بناء إصدار الويب من تطبيق Flutter

## خطوات النشر

### 1. تثبيت Firebase CLI

قم بتثبيت Firebase CLI باستخدام npm:

```bash
npm install -g firebase-tools
```

### 2. تسجيل الدخول إلى Firebase

```bash
firebase login
```

### 3. تهيئة مشروع Firebase للويب

تأكد من أن ملف `index.html` في مجلد `web` يحتوي على سكريبتات Firebase (تم إضافتها بالفعل).

### 4. بناء إصدار الويب من التطبيق

```bash
flutter build web
```

هذا سينشئ مجلد `build/web` يحتوي على ملفات الويب الجاهزة للنشر.

### 5. تهيئة Firebase Hosting

إذا لم تقم بتهيئة Firebase Hosting بعد، قم بتنفيذ:

```bash
firebase init hosting
```

اتبع التعليمات وحدد:
- مشروع Firebase الخاص بك (elevatorsitracker)
- مجلد `build/web` كمجلد عام
- تكوين التطبيق كتطبيق صفحة واحدة (SPA): نعم
- تجاوز ملف index.html: لا

### 6. نشر التطبيق

```bash
firebase deploy --only hosting
```

### 7. الوصول إلى التطبيق

بعد اكتمال النشر، ستحصل على عنوان URL للتطبيق، مثل:

```
https://elevatorsitracker.web.app
```

يمكنك الآن الوصول إلى تطبيقك من أي متصفح باستخدام هذا العنوان.

## ملاحظات هامة

1. **تحديث التطبيق**: لتحديث التطبيق، قم ببناء إصدار جديد من الويب ثم أعد النشر باستخدام نفس الأمر.

2. **إعدادات Firebase**: تأكد من أن قواعد الأمان في Firebase (Firestore وStorage) تسمح بالوصول من تطبيق الويب.

3. **مجال مخصص**: يمكنك إعداد مجال مخصص (domain) لتطبيقك من خلال إعدادات Firebase Hosting في لوحة تحكم Firebase.

4. **تحسين الأداء**: استخدم خيارات تحسين Flutter للويب مثل:
   ```bash
   flutter build web --web-renderer canvaskit --release
   ```

5. **التوافق مع المتصفحات**: تأكد من اختبار التطبيق على متصفحات مختلفة للتأكد من التوافق.