---
mode: agent
---

# 🚀 Flutter Project Setup Guide

สร้างโปรเจค Flutter (iOS/Android/Web) ใหม่ตามมาตรฐาน

## 📋 Setup Requirements

**Project Info:**
- **Name**: ถามผู้ใช้จาก input chat
- **Package**: ใช้ snake_case จาก project name (เช่น my_awesome_app)
- **Platform**: iOS, Android, Web
- **Command**: `flutter create [project_name]` เสมอ

**Basic Dependencies:**
- GetX (state management + routing + DI)
- Dio (HTTP client)
- SharedPreferences, SecureStorage (local storage)
- flutter_i18n (internationalization)
- flutter_lints (code quality)

## ⚠️ Setup Rules

1. **Splash Screen**: ทุกครั้งเปิดแอปต้องมี Splash Screen ก่อนไปหน้าหลัก
2. **New Project Only**: สร้างโปรเจคใหม่เสมอ ห้ามแก้ไขโปรเจคเดิม
3. **Quality Check**: รัน `flutter analyze` และเช็คตาม `checklist.md`
4. **Follow Architecture**: ยึดตาม `flutter_default.instructions.md` และ `flutter_structure.instructions.md`

## 🗂️ Initial Project Structure

สร้างโครงสร้างตาม `flutter_structure.instructions.md`:

```
lib/
├── app/                # Entry point, routes, DI
├── core/               # Shared utilities, theme, network
├── shared/             # Reusable widgets
├── features/           # Feature modules
│   └── splash/         # Required: Splash screen
└── main.dart
```

## � Setup Checklist

1. ✅ Run `flutter create [project_name]`
2. ✅ Setup dependencies in `pubspec.yaml`
3. ✅ Create folder structure ตาม instructions
4. ✅ Setup theme ตาม `theme.instructions.md`
5. ✅ Create Splash Screen feature
6. ✅ Setup routing ด้วย GetX
7. ✅ Run `flutter analyze` เช็ค errors
8. ✅ Test บน iOS/Android/Web

---

**Reference:** Follow `.github/instructions/flutter_default.instructions.md` และ `flutter_structure.instructions.md`
