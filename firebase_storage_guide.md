# دليل تفعيل Firebase Storage وتعديل حجم أيقونة التطبيق

## الجزء الأول: تفعيل Firebase Storage

لتفعيل خدمة Firebase Storage بشكل صحيح في تطبيقك، اتبع الخطوات التالية:

### 1. تفعيل خدمة Firebase Storage في لوحة تحكم Firebase

1. قم بزيارة [لوحة تحكم Firebase](https://console.firebase.google.com/) وحدد مشروعك
2. من القائمة الجانبية، انقر على "Storage"
3. انقر على "البدء" أو "Get Started" إذا لم تكن قد فعلت الخدمة بعد
4. اختر موقع تخزين البيانات المناسب لك (يفضل اختيار الموقع الأقرب لمستخدميك)
5. قم بتأكيد إعدادات قواعد الأمان (يمكنك البدء بالوضع التجريبي ثم تعديلها لاحقاً)

### 2. تحديث قواعد الأمان لـ Firebase Storage

بعد تفعيل الخدمة، قم بتعديل قواعد الأمان للسماح بتحميل الصور:

```
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    match /task_images/{taskId}/{imageId} {
      // السماح بالقراءة للجميع، والكتابة للمستخدمين المسجلين فقط
      allow read;
      allow write: if request.auth != null;
    }
  }
}
```

### 3. التأكد من إضافة التبعيات الصحيحة في ملف pubspec.yaml

تأكد من وجود التبعيات التالية في ملف pubspec.yaml:

```yaml
dependencies:
  flutter:
    sdk: flutter
  firebase_core: ^latest_version
  firebase_storage: ^latest_version
  cloud_firestore: ^latest_version
  firebase_auth: ^latest_version
  image_picker: ^latest_version
  path: ^latest_version
  uuid: ^latest_version
```

بعد إضافة التبعيات، قم بتنفيذ الأمر:

```
flutter pub get
```

### 4. تهيئة Firebase Storage في التطبيق

تأكد من تهيئة Firebase في ملف main.dart:

```dart
import 'package:firebase_core/firebase_core.dart';
import 'package:flutter/material.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp();
  runApp(MyApp());
}
```

### 5. حل مشاكل عرض الصور من Firebase Storage

إذا كنت تواجه مشاكل في عرض الصور من Firebase Storage، تأكد من:

1. **التعامل مع الروابط بشكل صحيح**: تأكد من أن الروابط المخزنة في `imagePath` صحيحة وتم تقسيمها بشكل صحيح باستخدام الفاصلة.

2. **معالجة الأخطاء**: أضف معالجة أخطاء عند تحميل الصور من Firebase Storage.

3. **التحقق من صلاحيات الإنترنت**: أضف صلاحية الإنترنت في ملف AndroidManifest.xml:

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

4. **التحقق من حجم الصور**: تأكد من أن حجم الصور المرفوعة ليس كبيرًا جدًا لتجنب مشاكل الأداء.

## الجزء الثاني: تعديل حجم أيقونة التطبيق

لتعديل حجم أيقونة التطبيق لتظهر بشكل كامل، اتبع الخطوات التالية:

### 1. إنشاء أيقونة بالحجم المناسب

1. قم بإنشاء أيقونة مربعة بأبعاد 1024×1024 بكسل
2. تأكد من أن التصميم يظهر بشكل جيد عند تصغيره
3. احفظ الأيقونة بصيغة PNG مع خلفية شفافة

### 2. استبدال ملفات الأيقونة في مجلدات الموارد

قم باستبدال ملفات الأيقونة في المجلدات التالية:

- `android/app/src/main/res/mipmap-hdpi/ic_launcher.png`
- `android/app/src/main/res/mipmap-mdpi/ic_launcher.png`
- `android/app/src/main/res/mipmap-xhdpi/ic_launcher.png`
- `android/app/src/main/res/mipmap-xxhdpi/ic_launcher.png`
- `android/app/src/main/res/mipmap-xxxhdpi/ic_launcher.png`

### 3. تعديل ملف AndroidManifest.xml

تأكد من أن ملف AndroidManifest.xml يحتوي على الإعدادات الصحيحة للأيقونة:

```xml
<application
    android:label="elevatorsitracker"
    android:name="${applicationName}"
    android:icon="@mipmap/ic_launcher">
    ...
</application>
```

### 4. استخدام أداة Flutter لإنشاء الأيقونات

يمكنك استخدام حزمة `flutter_launcher_icons` لتسهيل عملية إنشاء الأيقونات بأحجام مختلفة:

1. أضف الحزمة إلى ملف pubspec.yaml:

```yaml
dev_dependencies:
  flutter_launcher_icons: ^latest_version
```

2. أضف إعدادات الأيقونة في ملف pubspec.yaml:

```yaml
flutter_icons:
  android: true
  ios: true
  image_path: "assets/icon/icon.png"
  adaptive_icon_background: "#FFFFFF" # لون الخلفية للأيقونة التكيفية
  adaptive_icon_foreground: "assets/icon/icon_foreground.png" # صورة المقدمة للأيقونة التكيفية
```

3. قم بتنفيذ الأمر لإنشاء الأيقونات:

```
flutter pub get
flutter pub run flutter_launcher_icons:main
```

### 5. إعادة بناء التطبيق

بعد تعديل الأيقونات، قم بإعادة بناء التطبيق:

```
flutter clean
flutter pub get
flutter build apk
```

## ملاحظات إضافية

1. **تأكد من تفعيل الخدمات في Firebase**: تأكد من تفعيل خدمات Authentication وFirestore وStorage في لوحة تحكم Firebase.

2. **تحقق من ملف google-services.json**: تأكد من وجود ملف google-services.json المحدث في مجلد android/app.

3. **اختبار التطبيق**: قم باختبار التطبيق على أجهزة مختلفة للتأكد من عمل الصور والأيقونة بشكل صحيح.

4. **مراقبة استهلاك Firebase Storage**: راقب استهلاك التخزين في لوحة تحكم Firebase لتجنب تجاوز الحد المجاني.

5. **تحسين أداء تحميل الصور**: فكر في تطبيق تقنيات ضغط الصور قبل رفعها لتحسين الأداء وتقليل استهلاك البيانات.