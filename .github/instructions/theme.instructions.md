---
applyTo: '**'
---
# 🎨 Theme Guidelines (Single Source of Truth)

ไฟล์นี้คือแหล่งอ้างอิงหลักเรื่องธีม (แทนที่ `flutter_defaultTheme.instructions.md`).
โค้ดใหม่ทั้งหมดให้ยึดตามแนวทาง Material 3 + iOS-like look & feel ด้วย ColorScheme + TextTheme ที่ปรับแต่ง

## ✅ เป้าหมาย
- ใช้ Material 3 (`useMaterial3: true`) เพื่อให้ได้ component behavior ล่าสุด
- รักษาโทน iOS (spacing, สี primary, ช่วงสีเทา, rounded sheet/dialog)
- ลดการปรับแต่งกระจัดกระจาย: สี/ตัวอักษร/shape รวมศูนย์
- รองรับ Light/Dark theme เต็มรูปแบบ

## 🎯 หลักการสำคัญ
1. ใช้ `ColorScheme` เป็น base—ห้ามใช้ `ThemeData.backgroundColor`, `errorColor`, หรือ textTheme แบบ Material 2 (headline1 ฯลฯ) ในโค้ดใหม่
2. Typography ใช้ชื่อ Material 3: `displayLarge`, `titleMedium`, `bodyLarge`, `labelLarge` ฯลฯ
3. ปรับ iOS feel ผ่านสีและ radius (12–16, sheet top radius 24)
4. หลีกเลี่ยง `withOpacity()` ในชุด palette หลัก → ถ้าต้องการ variant กำหนดสีเพิ่มใน extension หรือใช้ `withValues()` (ดู extension)
5. หลีกเลี่ยงใส่สีตรง ๆ ใน Widget (hardcode) ให้ใช้จาก `Theme.of(context)` / `ColorScheme` / `TextTheme`

## 🎨 Color Scheme (Light)
ตัวอย่าง (ปรับตาม Branding ได้):
```dart
const _lightScheme = ColorScheme(
	brightness: Brightness.light,
	primary: Color(0xFF007AFF), // iOS Blue
	onPrimary: Color(0xFFFFFFFF),
	primaryContainer: Color(0xFFE3F1FF),
	onPrimaryContainer: Color(0xFF003763),
	secondary: Color(0xFF34C759),
	onSecondary: Color(0xFFFFFFFF),
	secondaryContainer: Color(0xFFCFFAD5),
	onSecondaryContainer: Color(0xFF062910),
	tertiary: Color(0xFF5AC8FA),
	onTertiary: Color(0xFF002431),
	tertiaryContainer: Color(0xFFC2F1FF),
	onTertiaryContainer: Color(0xFF003545),
	error: Color(0xFFFF3B30),
	onError: Color(0xFFFFFFFF),
	errorContainer: Color(0xFFFFDAD4),
	onErrorContainer: Color(0xFF410001),
	background: Color(0xFFF9F9F9),
	onBackground: Color(0xFF1C1C1E),
	surface: Color(0xFFFFFFFF),
	onSurface: Color(0xFF1C1C1E),
	surfaceVariant: Color(0xFFF2F2F7),
	onSurfaceVariant: Color(0xFF505055),
	outline: Color(0xFFE5E5EA),
	outlineVariant: Color(0xFFD0D0D4),
	shadow: Color(0x33000000),
	scrim: Color(0x66000000),
	inverseSurface: Color(0xFF2C2C2E),
	inverseOnSurface: Color(0xFFF2F2F7),
	inversePrimary: Color(0xFF0A84FF),
	surfaceTint: Color(0xFF007AFF),
);
```

## 🌙 Color Scheme (Dark)
```dart
const _darkScheme = ColorScheme(
	brightness: Brightness.dark,
	primary: Color(0xFF0A84FF),
	onPrimary: Color(0xFFFFFFFF),
	primaryContainer: Color(0xFF003A6E),
	onPrimaryContainer: Color(0xFFD3E8FF),
	secondary: Color(0xFF30D158),
	onSecondary: Color(0xFF00390F),
	secondaryContainer: Color(0xFF005322),
	onSecondaryContainer: Color(0xFFCFFAD5),
	tertiary: Color(0xFF64D2FF),
	onTertiary: Color(0xFF003546),
	tertiaryContainer: Color(0xFF004D63),
	onTertiaryContainer: Color(0xFFC2F1FF),
	error: Color(0xFFFF453A),
	onError: Color(0xFF690002),
	errorContainer: Color(0xFF930006),
	onErrorContainer: Color(0xFFFFDAD4),
	background: Color(0xFF1C1C1E),
	onBackground: Color(0xFFF2F2F7),
	surface: Color(0xFF2C2C2E),
	onSurface: Color(0xFFF2F2F7),
	surfaceVariant: Color(0xFF242426),
	onSurfaceVariant: Color(0xFFB5B5BB),
	outline: Color(0xFF3A3A3C),
	outlineVariant: Color(0xFF505055),
	shadow: Color(0x99000000),
	scrim: Color(0x99000000),
	inverseSurface: Color(0xFFF2F2F7),
	inverseOnSurface: Color(0xFF1C1C1E),
	inversePrimary: Color(0xFF007AFF),
	surfaceTint: Color(0xFF0A84FF),
);
```

## 🔤 Typography (iOS-like Mapping → Material 3)
```dart
TextTheme buildTextTheme(ColorScheme scheme, {bool dark = false}) {
	const font = 'SF Pro Display'; // หรือ Inter ถ้า cross-platform
	final baseColor = scheme.onSurface;
	return TextTheme(
		displayLarge: TextStyle(fontFamily: font, fontSize: 34, fontWeight: FontWeight.w700, color: baseColor),
		displayMedium: TextStyle(fontSize: 28, fontWeight: FontWeight.w600, fontFamily: font, color: baseColor),
		displaySmall: TextStyle(fontSize: 24, fontWeight: FontWeight.w600, fontFamily: font, color: baseColor),
		titleLarge: TextStyle(fontSize: 22, fontWeight: FontWeight.w600, fontFamily: font, color: baseColor),
		titleMedium: TextStyle(fontSize: 17, fontWeight: FontWeight.w600, fontFamily: font, color: baseColor),
		titleSmall: TextStyle(fontSize: 15, fontWeight: FontWeight.w600, fontFamily: font, color: baseColor),
		bodyLarge: TextStyle(fontSize: 17, fontWeight: FontWeight.w400, fontFamily: font, color: baseColor),
		bodyMedium: TextStyle(fontSize: 15, fontWeight: FontWeight.w400, fontFamily: font, color: baseColor.withOpacity(.85)),
		bodySmall: TextStyle(fontSize: 13, fontWeight: FontWeight.w400, fontFamily: font, color: baseColor.withOpacity(.65)),
		labelLarge: TextStyle(fontSize: 15, fontWeight: FontWeight.w600, fontFamily: font, color: baseColor),
		labelMedium: TextStyle(fontSize: 13, fontWeight: FontWeight.w600, fontFamily: font, color: baseColor.withOpacity(.85)),
		labelSmall: TextStyle(fontSize: 11, fontWeight: FontWeight.w600, fontFamily: font, color: baseColor.withOpacity(.65)),
	);
}
```

## 🧩 Component Themes (ตัวอย่างสำคัญ)
```dart
ThemeData buildLightTheme() {
	final scheme = _lightScheme;
	return ThemeData(
		useMaterial3: true,
		colorScheme: scheme,
		textTheme: buildTextTheme(scheme),
		scaffoldBackgroundColor: scheme.background,
		appBarTheme: AppBarTheme(
			backgroundColor: scheme.surface,
			elevation: 0,
			scrolledUnderElevation: 0,
			foregroundColor: scheme.onSurface,
			centerTitle: true,
			titleTextStyle: buildTextTheme(scheme).titleMedium,
		),
		inputDecorationTheme: InputDecorationTheme(
			filled: true,
			fillColor: scheme.surface,
			contentPadding: const EdgeInsets.symmetric(horizontal: 16, vertical: 14),
			border: OutlineInputBorder(
				borderRadius: BorderRadius.circular(16),
				borderSide: BorderSide(color: scheme.outlineVariant),
			),
			enabledBorder: OutlineInputBorder(
				borderRadius: BorderRadius.circular(16),
				borderSide: BorderSide(color: scheme.outline),
			),
			focusedBorder: OutlineInputBorder(
				borderRadius: BorderRadius.circular(16),
				borderSide: BorderSide(color: scheme.primary, width: 2),
			),
			hintStyle: buildTextTheme(scheme).bodyMedium,
		),
		elevatedButtonTheme: ElevatedButtonThemeData(
			style: ElevatedButton.styleFrom(
				elevation: 0,
				backgroundColor: scheme.primary,
				foregroundColor: scheme.onPrimary,
				minimumSize: const Size.fromHeight(48),
				shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(12)),
				textStyle: buildTextTheme(scheme).labelLarge,
			),
		),
		outlinedButtonTheme: OutlinedButtonThemeData(
			style: OutlinedButton.styleFrom(
				foregroundColor: scheme.primary,
				side: BorderSide(color: scheme.primary, width: 1.5),
				minimumSize: const Size.fromHeight(48),
				shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(12)),
				textStyle: buildTextTheme(scheme).labelLarge,
			),
		),
		snackBarTheme: SnackBarThemeData(
			backgroundColor: scheme.surface,
			behavior: SnackBarBehavior.floating,
			elevation: 1,
			shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(16)),
			contentTextStyle: buildTextTheme(scheme).bodyLarge,
			actionTextColor: scheme.primary,
		),
		dialogTheme: DialogTheme(
			backgroundColor: scheme.surface,
			shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(20)),
			titleTextStyle: buildTextTheme(scheme).titleLarge,
			contentTextStyle: buildTextTheme(scheme).bodyLarge,
		),
		bottomSheetTheme: const BottomSheetThemeData(
			shape: RoundedRectangleBorder(
				borderRadius: BorderRadius.vertical(top: Radius.circular(24)),
			),
		),
		tabBarTheme: TabBarTheme(
			indicator: UnderlineTabIndicator(
				borderSide: BorderSide(color: scheme.primary, width: 3),
				insets: const EdgeInsets.symmetric(horizontal: 16),
			),
			labelColor: scheme.primary,
			unselectedLabelColor: scheme.onSurfaceVariant,
			labelStyle: buildTextTheme(scheme).labelLarge,
			unselectedLabelStyle: buildTextTheme(scheme).labelLarge,
		),
		chipTheme: ChipThemeData(
			backgroundColor: scheme.surface,
			selectedColor: scheme.primary,
			labelStyle: buildTextTheme(scheme).bodyMedium!,
			secondaryLabelStyle: buildTextTheme(scheme).bodyMedium!.copyWith(color: scheme.onPrimary),
			shape: const StadiumBorder(),
			side: BorderSide(color: scheme.primary),
			padding: const EdgeInsets.symmetric(horizontal: 12, vertical: 6),
		),
	);
}

ThemeData buildDarkTheme() {
	final scheme = _darkScheme;
	return buildLightTheme().copyWith(
		colorScheme: scheme,
		textTheme: buildTextTheme(scheme, dark: true),
		scaffoldBackgroundColor: scheme.background,
	);
}
```

## 🧪 Validation Checklist
- [ ] ไม่มีการใช้ textTheme.headline1/bodyText1/button/caption ตรง ๆ ในโค้ดใหม่
- [ ] ทุกสีมาจาก ColorScheme (ยกเว้นเฉพาะบางกรณี overlay)
- [ ] ใช้ useMaterial3: true
- [ ] Widget ปรับสีผ่าน Theme / ColorScheme (ไม่ hardcode ในหลายที่)
- [ ] Light & Dark parity: contrast ≥ 4.5 ในข้อความหลัก

## 🛠️ Extensions ที่ควรมี
```dart
extension ColorX on Color {
	Color withValues({int? r,int? g,int? b,int? a}) => Color.fromARGB(
		a ?? alpha,
		r ?? red,
		g ?? green,
		b ?? blue,
	);
}
```

## 🚫 Anti-Patterns
- แก้ไขสีราย widget โดยไม่ผ่าน Theme
- ใช้ `copyWith` ของ Theme หลายชั้นใน widget ย่อยโดยไม่มีเหตุผล
- ใส่ฟอนต์คนละ family ในบางจุด (ยกเว้น branding จำเป็นจริง ๆ)

## 🔄 การย้ายจากเวอร์ชันเก่า
| สิ่งเดิม (Deprecated) | ใช้แทน |
| ---------------------- | ------- |
| ThemeData.backgroundColor | ColorScheme.background |
| errorColor | ColorScheme.error |
| textTheme.headline1 | textTheme.displayLarge |
| textTheme.bodyText1 | textTheme.bodyLarge |
| textTheme.button | textTheme.labelLarge |
| textTheme.caption | textTheme.bodySmall |

## 📦 การใช้งานใน `MaterialApp`
```dart
MaterialApp(
	theme: buildLightTheme(),
	darkTheme: buildDarkTheme(),
	themeMode: ThemeMode.system,
	// ...rest
);
```

## 📝 หมายเหตุฟอนต์
- iOS: สามารถไม่ระบุ fontFamily เพื่อให้ใช้ SF Pro System อัตโนมัติ (หรือคงไว้ถ้าลงไฟล์ถูกต้อง)
- Android/Web: ถ้าไม่มี SF Pro ให้ใช้ Inter / Roboto และ map ผ่าน fallback

## 🔜 Roadmap (ปรับปรุงต่อได้)
- Dynamic color extraction (Android 12) → Blend กับ base iOS palette
- High contrast theme variant
- Motion spec centralization

จบ — ปรับปรุง/เพิ่ม component ใหม่ให้อัปเดตที่ไฟล์นี้เท่านั้น
