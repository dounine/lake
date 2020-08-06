---
description: 在Flutter 我们可以使用vibrate插件、可以兼容安卓与IOS。一般可用于振动反馈、比如按钮点击反馈、网络请求成功反馈等等
---

# 振动反馈

添加依赖到`pubspec.yaml`到文件当中

安卓需要添加下面的振动权限到`Android Manifest`中

```text
<uses-permission android:name="android.permission.VIBRATE"/>
```

使用

```dart
import 'package:vibrate/vibrate.dart';
//检查是否支持振动
bool canVibrate = await Vibrate.canVibrate;
Vibrate.vibrate();
```

间隔振动

```dart
final Iterable<Duration> pauses = [
    const Duration(milliseconds: 500),
    const Duration(milliseconds: 1000),
    const Duration(milliseconds: 500),
];
Vibrate.vibrateWithPauses(pauses);
```

触觉振动

```dart
enum FeedbackType {
  success,
  error,
  warning,
  selection,
  impact,
  heavy,
  medium,
  light
}

var _type = FeedbackType.impact;
Vibrate.feedback(_type);
```

