---
mode: agent
---
สร้างโปรเจค mobile app สำหรับ iOS และ Android และ webbrowser โดยใช้ flutter ตามรายละเอียดด้านล่าง ให้ยึดรูปแบบและโครงสร้างโปรเจคตามเอกสาร ไฟล์ md ที่เกี่ยวข้องเป็นสำคัญ

project name : ให้ระบุชื่อโปรเจคโดยถามผู้ใช้จาก input chat
package: ในส่วนของ android ให้อ้างอิง package name จาก ไฟล์ google-services.json ที่ให้มาเสมอ
ส่วน iOS ให้อ้างอิง package name จาก ไฟล์ GoogleService-Info.plist ที่ให้มาเสมอ และ package name ต้องตรงกับ android package name ถ้าไม่มีไฟล์นี้ให้ขใช้จาก project name แทน โดยใช้รูปแบบ snake_case
platform: iOS, Android, Web
architecture: Flutter Clean Architecture with GetX
state management: GetX
routing: GetX routes
dependency injection: GetX/get_it with injectable
http client: Dio
local storage: SharedPreferences, SecureStorage
form validation: form_field_validator
image: cached_network_image
json serialization: json_serializable
environment variables: flutter_dotenv
linting: flutter_lints
code format: dart format
code analysis: dart analyze
CI/CD: GitHub Actions, Bitrise, Codemagic
theme: ตาม theme.md
ในส่วนของ firebase crashlytics ให้ใช้ไฟล์ google-services.json และ GoogleService-Info.plist ที่ให้มาเสมอ ถ้าไม่มีไฟล์นี้ให้ข้ามไป
ในส่วนของ google authentication ให้ใช้ไฟล์ google-services.json และ GoogleService-Info.plist ที่ให้มาเสมอ ถ้าไม่มีไฟล์นี้ให้ข้ามไป

important: ต้องทำตาม  Key Features และ Theme Guideline ให้ถูกต้อง และทุกครั้งที่เปิด app ขึ้นมา จะต้องทำการเปิดหน้า Splash Screen เสมอ ก่อนจะแทนที่ด้วยหน้าหลัก
important2: ต้องเป็นการสร้างโปรเจคมาใหม่ทุกครั้ง ห้ามลบหรือแก้ไขโปรเจคอื่น ๆ ที่มีอยู่แล้ว จะต้องเป็นการ new project ขึ้นมาใหม่เสมอ
important3: เมื่อทำทุกอย่างเรียบร้อยแล้ว ให้ไปเช็คความถูกต้องของโปรเจคอีกครั้ง ตามรายละเอียดใน checklist.md และทำการ analyze โปรเจคเสมอเพื่อเช็คความผิดพลาด
important4: ต้องสร้างโปรเจคด้วยคำสั่ง flutter create [project_name] เสมอ


# 📱 Project Development & Structure Instructions

## 1. Folder Structure (ตามมาตรฐาน)

```
lib/
├── app/                # Entry, env, routes, DI (composition root)
│   ├── main.dart
│   ├── env/
│   ├── config/
│   ├── routes/
│   └── injection.dart
├── core/               # Cross-cutting concerns (Dio, storage, theme, error, etc.)
├── shared/             # Reusable widgets/UI (not feature-specific)
├── features/           # Feature modules (auth, profile, etc.)
│   └── <feature>/
│       ├── domain/
│       ├── data/
│       └── presentation/
└── shared_libraries.dart
assets/
├── i18n/
├── fonts/
└── icon_*/
test/
├── core/
└── features/
    └── <feature>/
        ├── domain/
        ├── data/
        └── presentation/
analysis_options.yaml
```

- **อธิบาย:**
  - `app/`: จุดเริ่มต้น, env, routes, DI
  - `core/`: สิ่งที่ทุกฟีเจอร์ใช้ร่วมกัน (Dio, storage, theme, error, result, services)
  - `shared/`: Widgets/UI ที่นำกลับใช้ได้ ไม่ผูกฟีเจอร์
  - `features/`: แยกเป็นฟีเจอร์ (auth, profile ฯลฯ) ภายในแยก Domain/Data/Presentation
  - `test/`: โครงสร้าง test mirror กับ lib/

## 2. Clean Architecture & Technologies
- **Clean Architecture:**
  - `domain/`: entities, repositories (abstract), usecases
  - `data/`: datasources (api/local db), models, mappers, repositories (impl)
  - `presentation/`: pages, controllers, bindings, widgets (per feature)
- **GetX:**
  - ใช้ GetX สำหรับ state, navigation, DI เท่านั้น
  - ทุก Controller/Service ต้องมี Binding file คู่กัน (ใน features/<feature>/presentation/bindings/)
  - Core services: `Get.put(Service(), permanent: true)`
  - Feature controllers: `Get.lazyPut(() => Controller(), fenix: true)`
  - ห้าม inject ใน widget/build, inject เฉพาะใน Binding
  - Controller access: ผ่าน `controller` property ของ GetView เท่านั้น
- **Theming:**
  - ใช้ semantic tokens ใน core/theme/app_color.dart เท่านั้น (ห้าม Color(...) ตรง)
  - ThemeData, typography, spacing, light/dark mode, custom font (อ้างอิงจาก theme.md)
  - เริ่มต้นโปรเจคครั้งแรก ให้เป็น light mode เสมอ
- **Localization:**
  - ใช้ flutter_i18n
  - assets/i18n/
- **API:**
  - ใช้ Dio (core/network/)
- **Storage:**
  - SharedPreferences, SecureStorage (core/storage/)
- **Routing:**
  - GetX routes (app/routes/app_routes.dart)
- **Form Validation:**
  - form_field_validator
- **Image:**
  - cached_network_image
- **JSON:**
  - json_serializable
- **Env:**
  - flutter_dotenv (app/env/)
- **CI/CD:**
  - GitHub Actions, Bitrise, Codemagic
- **Lint:**
  - flutter_lints, dart format/analyze
- **Secrets:**
  - .env, ห้าม hardcode
- **Versioning:**
  - semantic versioning ใน pubspec.yaml
- **Icon/Splash:**
  - flutter_launcher_icons, flutter_native_splash

## 3. Page, Controller, Binding Rules
- ทุก Page ต้องมี Controller และ Binding คู่กัน (features/<feature>/presentation/pages/, controllers/, bindings/)
- Page: extends `GetView<Controller>` เท่านั้น
- Controller: inject ผ่าน Binding เท่านั้น
- ถ้าต้องใช้หลาย controller: main controller เรียก `Get.find<OtherController>()` ภายใน
- ห้าม inject ใน widget/build
- Binding/Controller/Widget เฉพาะฟีเจอร์: อยู่ใน features/<feature>/presentation/

## 4. Asset & Data Management
- assets/images, assets/fonts, assets/i18n
- อ้างอิง assets ใน pubspec.yaml
- Optimize images
- Network: HTTPS only
- Licenses: include third-party licenses

## 5. Code Quality & Testing
- ทุก function/widget/page ใหม่ ต้องมี test (test/ โครงสร้าง mirror lib/)
- ใช้ flutter_test, mocktail, get_test
- >80% coverage ก่อน release
- Theme snapshot tests: light/dark mode (button, textfield, toast)
- Lint: flutter_lints/custom
- Document classes/methods/logic
- README ครบถ้วน
- Git flow, commit message ชัดเจน
- Performance profiling
- Accessibility: screen reader
- Error handling robust
- Security best practices
- Analytics: Firebase Analytics, Sentry/Crashlytics

## 6. File Organization & Naming
- Widget: lib/shared/widgets/
- Navigation: lib/app/routes/
- Utility: lib/core/util/
- Enums: lib/core/type/
- Snake_case for file names
- Single responsibility per file
- Split large files

## 7. App Config Files (lib/app/config/)
- app_color.dart: semantic tokens, ThemeData, light/dark
- app_image.dart: asset paths
- app_config.dart: constants, endpoints

## 8. Reuse Before Create
- ตรวจสอบก่อนสร้างใหม่ ใช้หรือ extend component เดิมถ้ามี

---
- อ้างอิงโครงสร้างและแนวทางจากไฟล์ `.github/instructions/flutter_structure.instructions.md` ทุกครั้ง
- ปรับปรุง/เพิ่มรายละเอียดตามที่ไฟล์นี้กำหนดเท่านั้น
