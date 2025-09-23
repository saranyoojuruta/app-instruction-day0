---
applyTo: '**'
---
# 🍏 iOS-like Theme (Light & Dark)

> ใช้ไฟล์นี้เฉพาะเมื่อ `theme.instructions.md` ไม่มีไฟล์ใน repo ถ้ามีไฟล์ `theme.instructions.md` ให้ใช้ไฟล์นั้นแทน หรือถ้ามีไฟล์ `theme.instructions.md` อยู่แต่ไม่มีข้อมูลเกี่ยวกับธีม ให้รวมข้อมูลจากไฟล์นี้เข้าไปใน `theme.instructions.md` แทน หรือถ้าไฟล์ `theme.instructions.md` มีข้อมูลเกี่ยวกับธีมอยู่แล้ว แต่ไม่ครบ ให้เพิ่มข้อมูลจากไฟล์นี้เข้าไปใน `theme.instructions.md` แทน

## Light Theme

```dart
// lib/core/theme/app_color.dart
import 'package:flutter/material.dart';

class AppColor {
  static const Color appBase = Color(0xFFF9F9F9); // พื้นหลังหลัก
  static const Color surface = Color(0xFFFFFFFF); // Card/Surface
  static const Color elevated = Color(0xFFF2F2F7); // Elevated
  static const Color primary = Color(0xFF007AFF); // iOS Blue
  static const Color secondary = Color(0xFF34C759); // iOS Green
  static const Color textPrimary = Color(0xFF1C1C1E);
  static const Color textSecondary = Color(0xFF8E8E93);
  static const Color border = Color(0xFFE5E5EA);
  static const Color success = Color(0xFF34C759);
  static const Color error = Color(0xFFFF3B30);
  static const Color info = Color(0xFF5AC8FA);
}
```

```dart
// lib/core/theme/app_text_theme.dart
import 'package:flutter/material.dart';
import 'app_color.dart';

class AppTextTheme {
  static const String fontFamily = 'SF Pro Display';

  static const TextStyle headline1 = TextStyle(fontSize: 28, fontWeight: FontWeight.bold, fontFamily: fontFamily, color: AppColor.textPrimary);
  static const TextStyle headline2 = TextStyle(fontSize: 22, fontWeight: FontWeight.w600, fontFamily: fontFamily, color: AppColor.textPrimary);
  static const TextStyle body1 = TextStyle(fontSize: 17, fontWeight: FontWeight.normal, fontFamily: fontFamily, color: AppColor.textPrimary);
  static const TextStyle body2 = TextStyle(fontSize: 15, fontWeight: FontWeight.normal, fontFamily: fontFamily, color: AppColor.textSecondary);
  static const TextStyle button = TextStyle(fontSize: 17, fontWeight: FontWeight.w500, fontFamily: fontFamily, color: Colors.white);
  static const TextStyle caption = TextStyle(fontSize: 13, fontWeight: FontWeight.normal, fontFamily: fontFamily, color: AppColor.textSecondary);
}
```

```dart
// lib/core/theme/app_theme.dart
import 'package:flutter/material.dart';
import 'app_color.dart';
import 'app_text_theme.dart';

final ThemeData iosLightTheme = ThemeData(
  brightness: Brightness.light,
  primaryColor: AppColor.primary,
  backgroundColor: AppColor.appBase,
  scaffoldBackgroundColor: AppColor.appBase,
  fontFamily: AppTextTheme.fontFamily,
  textTheme: TextTheme(
    headline1: AppTextTheme.headline1,
    headline2: AppTextTheme.headline2,
    bodyText1: AppTextTheme.body1,
    bodyText2: AppTextTheme.body2,
    button: AppTextTheme.button,
    caption: AppTextTheme.caption,
  ),
  cardColor: AppColor.surface,
  dividerColor: AppColor.border,
  errorColor: AppColor.error,
  elevatedButtonTheme: ElevatedButtonThemeData(
    style: ElevatedButton.styleFrom(
      backgroundColor: AppColor.primary,
      foregroundColor: Colors.white,
      shape: RoundedRectangleBorder(
        borderRadius: BorderRadius.circular(12),
      ),
      minimumSize: const Size.fromHeight(48),
      textStyle: AppTextTheme.button,
    ),
  ),
  outlinedButtonTheme: OutlinedButtonThemeData(
    style: OutlinedButton.styleFrom(
      foregroundColor: AppColor.primary,
      side: const BorderSide(color: AppColor.primary, width: 2),
      shape: RoundedRectangleBorder(
        borderRadius: BorderRadius.circular(12),
      ),
      textStyle: AppTextTheme.button,
    ),
  ),
  chipTheme: ChipThemeData(
    backgroundColor: Colors.transparent,
    selectedColor: AppColor.primary,
    labelStyle: AppTextTheme.body2,
    secondaryLabelStyle: AppTextTheme.body2.copyWith(color: Colors.white),
    shape: const StadiumBorder(),
    side: const BorderSide(color: AppColor.primary),
    padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 4),
  ),
  inputDecorationTheme: InputDecorationTheme(
    filled: true,
    fillColor: AppColor.surface,
    border: OutlineInputBorder(
      borderRadius: BorderRadius.circular(16),
      borderSide: BorderSide(color: AppColor.border),
    ),
    enabledBorder: OutlineInputBorder(
      borderRadius: BorderRadius.circular(16),
      borderSide: BorderSide(color: AppColor.border),
    ),
    focusedBorder: OutlineInputBorder(
      borderRadius: BorderRadius.circular(16),
      borderSide: BorderSide(color: AppColor.primary, width: 2),
    ),
    hintStyle: TextStyle(color: AppColor.textSecondary),
  ),
  tabBarTheme: TabBarTheme(
    labelColor: AppColor.primary,
    unselectedLabelColor: AppColor.textSecondary,
    indicator: UnderlineTabIndicator(
      borderSide: BorderSide(width: 3, color: AppColor.primary),
    ),
  ),
  snackBarTheme: SnackBarThemeData(
    backgroundColor: AppColor.surface,
    contentTextStyle: AppTextTheme.body1,
    actionTextColor: AppColor.primary,
    behavior: SnackBarBehavior.floating,
    shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(16)),
  ),
  dialogTheme: DialogTheme(
    backgroundColor: AppColor.surface,
    shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(16)),
    titleTextStyle: AppTextTheme.headline2,
    contentTextStyle: AppTextTheme.body1,
  ),
  bottomSheetTheme: BottomSheetThemeData(
    backgroundColor: AppColor.surface,
    shape: const RoundedRectangleBorder(borderRadius: BorderRadius.vertical(top: Radius.circular(24))),
  ),
);
```

## Dark Theme

```dart
// lib/core/theme/app_color.dart (ต่อ)
class AppColorDark {
  static const Color appBase = Color(0xFF1C1C1E);
  static const Color surface = Color(0xFF2C2C2E);
  static const Color elevated = Color(0xFF242426);
  static const Color primary = Color(0xFF0A84FF);
  static const Color secondary = Color(0xFF30D158);
  static const Color textPrimary = Color(0xFFF2F2F7);
  static const Color textSecondary = Color(0xFF8E8E93);
  static const Color border = Color(0xFF3A3A3C);
  static const Color success = Color(0xFF30D158);
  static const Color error = Color(0xFFFF453A);
  static const Color info = Color(0xFF64D2FF);
}
```

```dart
// lib/core/theme/app_text_theme.dart (ต่อ)
class AppTextThemeDark {
  static const String fontFamily = 'SF Pro Display';

  static const TextStyle headline1 = TextStyle(fontSize: 28, fontWeight: FontWeight.bold, fontFamily: fontFamily, color: AppColorDark.textPrimary);
  static const TextStyle headline2 = TextStyle(fontSize: 22, fontWeight: FontWeight.w600, fontFamily: fontFamily, color: AppColorDark.textPrimary);
  static const TextStyle body1 = TextStyle(fontSize: 17, fontWeight: FontWeight.normal, fontFamily: fontFamily, color: AppColorDark.textPrimary);
  static const TextStyle body2 = TextStyle(fontSize: 15, fontWeight: FontWeight.normal, fontFamily: fontFamily, color: AppColorDark.textSecondary);
  static const TextStyle button = TextStyle(fontSize: 17, fontWeight: FontWeight.w500, fontFamily: fontFamily, color: Colors.white);
  static const TextStyle caption = TextStyle(fontSize: 13, fontWeight: FontWeight.normal, fontFamily: fontFamily, color: AppColorDark.textSecondary);
}
```

```dart
// lib/core/theme/app_theme.dart (ต่อ)
final ThemeData iosDarkTheme = ThemeData(
  brightness: Brightness.dark,
  primaryColor: AppColorDark.primary,
  backgroundColor: AppColorDark.appBase,
  scaffoldBackgroundColor: AppColorDark.appBase,
  fontFamily: AppTextThemeDark.fontFamily,
  textTheme: TextTheme(
    headline1: AppTextThemeDark.headline1,
    headline2: AppTextThemeDark.headline2,
    bodyText1: AppTextThemeDark.body1,
    bodyText2: AppTextThemeDark.body2,
    button: AppTextThemeDark.button,
    caption: AppTextThemeDark.caption,
  ),
  cardColor: AppColorDark.surface,
  dividerColor: AppColorDark.border,
  errorColor: AppColorDark.error,
  elevatedButtonTheme: ElevatedButtonThemeData(
    style: ElevatedButton.styleFrom(
      backgroundColor: AppColorDark.primary,
      foregroundColor: Colors.white,
      shape: RoundedRectangleBorder(
        borderRadius: BorderRadius.circular(12),
      ),
      minimumSize: const Size.fromHeight(48),
      textStyle: AppTextThemeDark.button,
    ),
  ),
  outlinedButtonTheme: OutlinedButtonThemeData(
    style: OutlinedButton.styleFrom(
      foregroundColor: AppColorDark.primary,
      side: const BorderSide(color: AppColorDark.primary, width: 2),
      shape: RoundedRectangleBorder(
        borderRadius: BorderRadius.circular(12),
      ),
      textStyle: AppTextThemeDark.button,
    ),
  ),
  chipTheme: ChipThemeData(
    backgroundColor: Colors.transparent,
    selectedColor: AppColorDark.primary,
    labelStyle: AppTextThemeDark.body2,
    secondaryLabelStyle: AppTextThemeDark.body2.copyWith(color: Colors.white),
    shape: const StadiumBorder(),
    side: const BorderSide(color: AppColorDark.primary),
    padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 4),
  ),
  inputDecorationTheme: InputDecorationTheme(
    filled: true,
    fillColor: AppColorDark.surface,
    border: OutlineInputBorder(
      borderRadius: BorderRadius.circular(16),
      borderSide: BorderSide(color: AppColorDark.border),
    ),
    enabledBorder: OutlineInputBorder(
      borderRadius: BorderRadius.circular(16),
      borderSide: BorderSide(color: AppColorDark.border),
    ),
    focusedBorder: OutlineInputBorder(
      borderRadius: BorderRadius.circular(16),
      borderSide: BorderSide(color: AppColorDark.primary, width: 2),
    ),
    hintStyle: TextStyle(color: AppColorDark.textSecondary),
  ),
  tabBarTheme: TabBarTheme(
    labelColor: AppColorDark.primary,
    unselectedLabelColor: AppColorDark.textSecondary,
    indicator: UnderlineTabIndicator(
      borderSide: BorderSide(width: 3, color: AppColorDark.primary),
    ),
  ),
  snackBarTheme: SnackBarThemeData(
    backgroundColor: AppColorDark.surface,
    contentTextStyle: AppTextThemeDark.body1,
    actionTextColor: AppColorDark.primary,
    behavior: SnackBarBehavior.floating,
    shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(16)),
  ),
  dialogTheme: DialogTheme(
    backgroundColor: AppColorDark.surface,
    shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(16)),
    titleTextStyle: AppTextThemeDark.headline2,
    contentTextStyle: AppTextThemeDark.body1,
  ),
  bottomSheetTheme: BottomSheetThemeData(
    backgroundColor: AppColorDark.surface,
    shape: const RoundedRectangleBorder(borderRadius: BorderRadius.vertical(top: Radius.circular(24))),
  ),
);
```

**Usage:**

```dart
MaterialApp(
  theme: iosLightTheme,
  darkTheme: iosDarkTheme,
  // ...
)
```

---

- อ้างอิงจาก iOS 17 Human Interface Guidelines
- ใช้ font 'SF Pro Display' (หรือ Montserrat/Roboto แทนถ้าไม่มี)
- ปรับสี/spacing/shape ให้เหมาะกับ iOS look & feel