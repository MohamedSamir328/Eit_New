# تعديلات الكود لحل مشاكل Firebase Storage وأيقونة التطبيق

## أولاً: تعديلات لتفعيل Firebase Storage بشكل صحيح

بعد مراجعة الكود الحالي في ملف `edit_task_screen.dart`، لاحظت أن هناك بعض التعديلات التي يمكن إجرائها لضمان عمل Firebase Storage بشكل صحيح:

### 1. تعديل دالة _loadImages لتحسين تحميل الصور

```dart
Future<void> _loadImages() async {
  if (widget.task.imagePath == null || widget.task.imagePath!.isEmpty) return;

  setState(() => _isLoadingImages = true);

  try {
    // التحقق مما إذا كانت المسارات هي روابط Firebase Storage
    if (widget.task.imagePath!.contains('https://')) {
      // تقسيم الروابط المخزنة وإزالة المسافات الزائدة
      final urls = widget.task.imagePath!.split(',').map((url) => url.trim()).where((url) => url.isNotEmpty).toList();
      setState(() {
        imageUrls = urls;
      });
      print('تم تحميل ${imageUrls.length} رابط صورة من Firebase Storage');
    } else {
      // محاولة تحميل الصور من المسارات المحلية (للتوافق مع الإصدارات القديمة)
      final imagePaths = widget.task.imagePath!.split(',');
      for (var path in imagePaths) {
        final trimmedPath = path.trim();
        if (trimmedPath.isEmpty) continue;
        
        final file = File(trimmedPath);
        if (await file.exists()) {
          images.add(file);
        } else {
          print('الملف غير موجود في المسار المحلي: $trimmedPath');
        }
      }

      // إذا لم يتم العثور على أي صور محلية، محاولة تحميلها من Firebase Storage
      if (images.isEmpty && imagePaths.isNotEmpty) {
        print('محاولة تحميل الصور من Firebase Storage...');
      }
    }
  } catch (e) {
    print('حدث خطأ أثناء تحميل الصور: $e');
  } finally {
    setState(() => _isLoadingImages = false);
  }
}
```

### 2. تعديل دالة _uploadImagesToFirebase لتحسين رفع الصور

```dart
Future<List<String>> _uploadImagesToFirebase() async {
  List<String> uploadedUrls = [];

  // إضافة الروابط الموجودة مسبقاً
  uploadedUrls.addAll(imageUrls);

  // رفع الصور الجديدة
  for (var image in images) {
    try {
      // إنشاء اسم فريد للصورة
      final uuid = Uuid().v4();
      final fileName = '$uuid.${path.extension(image.path).replaceAll('.', '')}';

      // إنشاء مرجع للصورة في Firebase Storage
      final storageRef = FirebaseStorage.instance
          .ref()
          .child('task_images')
          .child(widget.task.id)
          .child(fileName);

      // رفع الصورة مع إظهار تقدم الرفع
      final uploadTask = storageRef.putFile(image);
      
      // متابعة تقدم الرفع (يمكن إضافة شريط تقدم هنا)
      uploadTask.snapshotEvents.listen((TaskSnapshot snapshot) {
        final progress = snapshot.bytesTransferred / snapshot.totalBytes;
        print('تقدم رفع الصورة: ${(progress * 100).toStringAsFixed(2)}%');
      });

      // انتظار اكتمال الرفع
      await uploadTask;

      // الحصول على رابط التنزيل
      final downloadUrl = await storageRef.getDownloadURL();
      uploadedUrls.add(downloadUrl);

      print('تم رفع الصورة بنجاح: $downloadUrl');
    } catch (e) {
      print('فشل في رفع الصورة: $e');
      // يمكن إضافة إشعار للمستخدم هنا
    }
  }

  return uploadedUrls;
}
```

### 3. تعديل طريقة عرض الصور لتحسين الأداء

```dart
Image.network(
  imageUrls[index],
  fit: BoxFit.cover,
  cacheWidth: 300, // تحديد عرض الصورة المخزنة مؤقتًا لتحسين الأداء
  cacheHeight: 300, // تحديد ارتفاع الصورة المخزنة مؤقتًا لتحسين الأداء
  loadingBuilder: (context, child, loadingProgress) {
    if (loadingProgress == null) return child;
    return Center(
      child: CircularProgressIndicator(
        value: loadingProgress.expectedTotalBytes != null
            ? loadingProgress.cumulativeBytesLoaded /
                loadingProgress.expectedTotalBytes!
            : null,
      ),
    );
  },
  errorBuilder: (context, error, stackTrace) {
    print('خطأ في تحميل الصورة: $error');
    return Center(
      child: Icon(
        Icons.error,
        color: Colors.red,
      ),
    );
  },
)
```

## ثانياً: تعديل حجم أيقونة التطبيق

لتعديل حجم أيقونة التطبيق، يمكنك اتباع الخطوات التالية:

### 1. إضافة حزمة flutter_launcher_icons

أضف الحزمة إلى ملف pubspec.yaml:

```yaml
dev_dependencies:
  flutter_launcher_icons: ^0.13.1
```

### 2. إضافة إعدادات الأيقونة في ملف pubspec.yaml

```yaml
flutter_icons:
  android: true
  ios: true
  image_path: "lib/assets/app_icon.png"
  adaptive_icon_background: "#FFFFFF"
  adaptive_icon_foreground: "lib/assets/app_icon_foreground.png"
  min_sdk_android: 21
```

### 3. إنشاء أيقونة بالحجم المناسب

1. قم بإنشاء أيقونة مربعة بأبعاد 1024×1024 بكسل
2. احفظها في المسار `lib/assets/app_icon.png`
3. قم بإنشاء نسخة للمقدمة (foreground) واحفظها في `lib/assets/app_icon_foreground.png`

### 4. توليد الأيقونات

قم بتنفيذ الأوامر التالية:

```
flutter pub get
flutter pub run flutter_launcher_icons
```

### 5. تعديل ملف AndroidManifest.xml (إذا لزم الأمر)

تأكد من أن ملف AndroidManifest.xml يحتوي على الإعدادات الصحيحة للأيقونة:

```xml
<application
    android:label="elevatorsitracker"
    android:name="${applicationName}"
    android:icon="@mipmap/ic_launcher">
    ...
</application>
```

## ثالثاً: نصائح إضافية لتحسين أداء التطبيق

### 1. ضغط الصور قبل رفعها

```dart
Future<void> pickImage() async {
  final pickedFile = await ImagePicker().pickImage(
    source: ImageSource.camera,
    imageQuality: 70, // ضغط الصورة لتقليل حجمها
    maxWidth: 1200, // تحديد العرض الأقصى
    maxHeight: 1200, // تحديد الارتفاع الأقصى
  );
  if (pickedFile != null) {
    setState(() {
      images.add(File(pickedFile.path));
    });
  }
}
```

### 2. إضافة مؤشر تقدم لرفع الصور

```dart
Widget _buildUploadProgressIndicator(UploadTask uploadTask) {
  return StreamBuilder<TaskSnapshot>(
    stream: uploadTask.snapshotEvents,
    builder: (context, snapshot) {
      if (snapshot.hasData) {
        final TaskSnapshot? data = snapshot.data;
        final progress = data != null ? data.bytesTransferred / data.totalBytes : 0.0;
        return Column(
          children: [
            LinearProgressIndicator(value: progress),
            Text('${(progress * 100).toStringAsFixed(1)}%'),
          ],
        );
      } else {
        return const SizedBox();
      }
    },
  );
}
```

### 3. تحسين التعامل مع الأخطاء

```dart
void _showErrorDialog(String message) {
  showDialog(
    context: context,
    builder: (context) => AlertDialog(
      title: Text('خطأ'),
      content: Text(message),
      actions: [
        TextButton(
          onPressed: () => Navigator.pop(context),
          child: Text('حسناً'),
        ),
      ],
    ),
  );
}
```

## رابعاً: التحقق من إعدادات Firebase

1. تأكد من وجود ملف `google-services.json` في مجلد `android/app`
2. تأكد من تهيئة Firebase في ملف `main.dart`
3. تحقق من قواعد الأمان في Firebase Storage
4. تأكد من تفعيل خدمات Authentication وFirestore وStorage في لوحة تحكم Firebase

بتطبيق هذه التعديلات، ستتمكن من حل مشاكل Firebase Storage وتعديل حجم أيقونة التطبيق بنجاح.