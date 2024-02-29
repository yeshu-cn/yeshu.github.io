# 如何在 Flutter 中动态修改应用启动图标

在 Android 中，动态修改应用的启动图标可以通过以下步骤实现：

## 1. 创建 `<activity-alias>`

在 `AndroidManifest.xml` 文件中，为每个想要切换的图标创建一个 `<activity-alias>`，并为每个 `<activity-alias>` 设置一个不同的图标。默认情况下，所有的 `<activity-alias>` 都被设置为 disabled 状态，只有主 `<activity>` 是 enabled。

```xml
<activity-alias
    android:name=".AliasIcon1"
    android:enabled="false"
    android:icon="@mipmap/ic_launcher_alias1"
    android:label="@string/app_name"
    android:targetActivity=".MainActivity">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity-alias>
```

## 2. 使用 `MethodChannel` 调用原生方法

在 Flutter 代码中，使用 `MethodChannel` 调用原生 Android 的方法来切换图标。这个方法会禁用当前启用的 `<activity-alias>`，然后启用我们想要切换的 `<activity-alias>`。这样，新启用的 `<activity-alias>` 就会成为新的启动图标。

```dart
import 'package:flutter/services.dart';

class IconSwitcher {
  static const platform = const MethodChannel('com.example.app/icon');

  static Future<void> switchIcon(int iconNumber) async {
    try {
      await platform.invokeMethod('switchIcon', {'iconNumber': iconNumber});
    } on PlatformException catch (e) {
      print("Failed to switch icon: '${e.message}'.");
    }
  }
}
```

## 3. 实现原生方法

在原生 Android 代码中，实现一个方法来处理 Flutter 的调用。这个方法会禁用所有的 `<activity-alias>`，然后启用传入的 `<activity-alias>`。

```java
private void switchIcon(int iconNumber) {
    PackageManager pm = getPackageManager();

    // Disable all alias activities
    pm.setComponentEnabledSetting(
      new ComponentName("com.example.app", "com.example.app.AliasIcon1"),
      PackageManager.COMPONENT_ENABLED_STATE_DISABLED,
      PackageManager.DONT_KILL_APP
    );
    pm.setComponentEnabledSetting(
      new ComponentName("com.example.app", "com.example.app.AliasIcon2"),
      PackageManager.COMPONENT_ENABLED_STATE_DISABLED,
      PackageManager.DONT_KILL_APP
    );

    // Enable the selected alias activity
    pm.setComponentEnabledSetting(
      new ComponentName("com.example.app", "com.example.app.AliasIcon" + iconNumber),
      PackageManager.COMPONENT_ENABLED_STATE_ENABLED,
      PackageManager.DONT_KILL_APP
    );
}
```

## 注意事项

这种方法的一个限制是，一些设备制造商可能不支持 `<activity-alias>`，或者不允许动态更改应用图标。在这些设备上，这种方法可能无法正常工作。

另外，如果你在 `<activity>` 或 `<activity-alias>` 标签中设置了图标，那么这个图标会覆盖 `<application>` 标签中设置的图标。如果你没有在 `<activity>` 或 `<activity-alias>` 标签中设置图标，那么 `<application>` 标签中设置的图标会被使用。