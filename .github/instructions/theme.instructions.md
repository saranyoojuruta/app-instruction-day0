---
applyTo: '**'
---
# 🎨 IMDb Theme Color Tokens Example

```dart
// lib/core/theme/app_color.dart
import 'package:flutter/material.dart';

class AppColor {
	static const Color appBase = Color(0xFF121212); // พื้นหลังหลัก
	static const Color surface = Color(0xFF1E1E1E); // Card
	static const Color elevated = Color(0xFF171717); // Elevated
	static const Color primary = Color(0xFFF5C518); // IMDb Yellow
	static const Color secondary = Color(0xFFFFD95C); // Highlight/Outline
	static const Color textPrimary = Color(0xFFFFFFFF);
	static const Color textSecondary = Color(0xFFB3B3B3);
	static const Color border = Color(0xFF2A2A2A);
	static const Color success = Color(0xFF4CAF50);
	static const Color error = Color(0xFFCF6679);
	static const Color info = Color(0xFF60A5FA);
}
```

```dart
// lib/core/theme/app_text_theme.dart
import 'package:flutter/material.dart';

class AppTextTheme {
	static const String fontFamily = 'Montserrat';

	static const TextStyle headline1 = TextStyle(fontSize: 28, fontWeight: FontWeight.bold, fontFamily: fontFamily, color: AppColor.textPrimary);
	static const TextStyle headline2 = TextStyle(fontSize: 20, fontWeight: FontWeight.w600, fontFamily: fontFamily, color: AppColor.textPrimary);
	static const TextStyle body1 = TextStyle(fontSize: 16, fontWeight: FontWeight.normal, fontFamily: fontFamily, color: AppColor.textPrimary);
	static const TextStyle body2 = TextStyle(fontSize: 14, fontWeight: FontWeight.normal, fontFamily: fontFamily, color: AppColor.textSecondary);
	static const TextStyle button = TextStyle(fontSize: 16, fontWeight: FontWeight.w500, fontFamily: fontFamily, color: Colors.black);
	static const TextStyle caption = TextStyle(fontSize: 12, fontWeight: FontWeight.normal, fontFamily: fontFamily, color: AppColor.textSecondary);
}
```

```dart
// lib/core/theme/app_theme.dart
import 'package:flutter/material.dart';
import 'app_color.dart';
import 'app_text_theme.dart';

final ThemeData imdbTheme = ThemeData(
	brightness: Brightness.dark,
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
			foregroundColor: Colors.black,
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
		secondaryLabelStyle: AppTextTheme.body2.copyWith(color: Colors.black),
		shape: StadiumBorder(),
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
		shape: RoundedRectangleBorder(borderRadius: BorderRadius.vertical(top: Radius.circular(24))),
	),
	// เพิ่มเติม: radius, elevation, spacing ตามดีไซน์
);
```

**Usage:**

```dart
MaterialApp(
	theme: imdbTheme,
	// ...
)
```

---

อ้างอิงจากไฟล์: `Re_Play – UX_UI Design Specification (IMDb Theme)_V0.01.md`