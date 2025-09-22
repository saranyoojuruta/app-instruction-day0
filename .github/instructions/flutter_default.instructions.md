---
applyTo: '**'
---
## 📝 Dependency Rule

- Dependency Rule จาก นอก → ใน เท่านั้น
```Presentation → Domain``` และ ```Data → Domain```
- Domain ไม่รู้จัก Flutter/Dio/Storage ใด ๆ (Pure Dart)
- ติดต่อข้ามเลเยอร์ผ่าน interfaces (Repository contracts) เท่านั้น

### เส้นทางข้อมูล (Data/Control Flow)

```
UI (Page/Widget)
  └─ Controller (GetX)            # จัดการ state/เหตุการณ์จาก UI
      └─ UseCase (Domain)         # 1 UseCase ต่อ 1 การทำงาน
          └─ Repository (Domain)  # เรียกใช้งานผ่านตัว Interface
              └─ Repository Impl (Data)
                  ├─ Datasource Remote (Dio)
                  ├─ Datasource Local (Cache/Prefs/DB)
                  └─ Mappers: DTO ↔ Entity
                       ↓ Result<T>
Controller อัปเดต state + render UI
```
- ทุก UseCase คืนค่าเป็น Result<T> (เช่น Ok<User> หรือ Err<Failure>) เพื่อให้ UI จัดการ error ได้คุมรูปแบบ
- DTO/response จะถูก map เป็น Entity เสมอก่อนออกจาก Data layer

---

### Cross-cutting Services (ใน `core/services/`)
- **AuthSessionService**: จัดการ access/refresh token & current user (ให้ interceptors และฟีเจอร์อื่นใช้งาน)  
- **FirebaseMessageService**: ตั้งค่า/รับ FCM push  
- **DeeplinkService**: รับลิงก์เข้า → กระจายไปยังฟีเจอร์ที่เกี่ยวข้อง  
- **AnalyticsService**: ส่งเหตุการณ์การใช้งานไปยัง Analytics provider  

---

### Network & Error Handling
- ใช้ **Dio** + interceptors (`access_token`, `refresh_token`, `user_agent`, `error`) ใน `core/network/`  
- แปลงข้อผิดพลาดเป็น **Failure กลาง** (เช่น `NetworkFailure`, `AuthFailure`) ด้วย `error_mapper`  
- หลีกเลี่ยงการ `throw` ทะลุ UI — ใช้ `Result<T>` เป็นสัญญารับ-ส่ง  

---

### DI & Routing (GetX)
- **Composition Root** อยู่ที่ `app/injection.dart` ลงทะเบียนของกลาง (Dio, Storage, Services)  
- แต่ละฟีเจอร์มี **Bindings** ของตัวเอง เพื่อลงทะเบียน datasource/repository/usecase/controller ตาม scope  
- เส้นทางนำทางรวมอยู่ที่ `app/routes/app_routes.dart`  

---

### Environment & Config
- `app/env/` เก็บค่า **per-environment** (`dev/stg/prod`) เช่น `baseUrl`, feature flags  
- รันด้วย  
  ```bash
  flutter run --dart-define=ENV=dev
  flutter run --dart-define=ENV=stg
  flutter run --dart-define=ENV=prod

---

### Internationalization / Theme
- ภาษาที่รองรับ: ไทย, อังกฤษ (เพิ่มได้ในอนาคต)
- **i18n** (YAML) อยู่ใน `assets/i18n/` และ helper ใน `core/`  
- **Theme / typography / spacing** อยู่ใน `core/theme/`  

---

### Testing Strategy
- โครงสร้าง test **mirror source**: `test/features/<feature>/domain|data|presentation`  
- **Domain**: ทดสอบ UseCase (mock repository)  
- **Data**: ทดสอบ Repository impl + Mapper + Datasource (mock Dio/Storage)  
- **Presentation**: Widget/Controller tests (GetX) เฉพาะ logic UI  

---
### วิธีเพิ่มฟีเจอร์ใหม่ (Recipe)

1. **สร้างโครงฟีเจอร์**
   - ใช้สคริปต์ generator:
     ```bash
     dart run tool/gen_feature.dart <FeatureName>
     ```
   - จะได้โครง `lib/features/<feature>/domain|data|presentation` พร้อมไฟล์ `presentation/bindings/<feature>_binding.dart`
   - สคริปต์จะสร้าง barrel `index.dart` ใน `domain/entities`, `domain/repositories`, `domain/usecases` ให้อัตโนมัติ

2. **เขียน Domain**
   - สร้าง **Entities**, **Repository (interface)**, และ **UseCases**
   - (ถ้าใช้) อ้างอิง `core/usecase/base_use_case.dart` หรือ `features/<feature>/domain/usecases/base_use_case.dart`

3. **ทำ Data**
   - สร้าง **Models/Request**, **Mappers**, **Datasources** (`remote`/`local`), และ **Repository Impl**
   - map DTO ↔ Entity ผ่าน `mappers/` เท่านั้น

4. **ทำ Presentation**
   - สร้าง **Pages / Controllers / Bindings / Widgets**
   - Controller เรียกใช้ UseCase เสมอ (ห้ามเรียก datasource ตรง)

5. **ผูก DI และ Route**
   - ผูก dependencies ใน `features/<feature>/presentation/bindings/<feature>_binding.dart`
   - เพิ่มเส้นทางใน `app/routes/app_routes.dart`

6. **เขียน Tests**
   - โครงสร้าง test แบบ **mirror source**:
     ```
     test/features/<feature>/
       ├─ domain/         # ทดสอบ UseCase (mock repository)
       ├─ data/           # ทดสอบ Repository impl + Mapper + Datasource (mock Dio/Storage)
       └─ presentation/   # Widget/Controller tests (เฉพาะ logic UI)
     ```

> ✅ หลักการคงที่: ใช้ `Result<T>`/`Failure` เป็นสัญญารับ-ส่งข้อผิดพลาด, ห้ามให้ DTO หลุดจาก Data layer, และแยก cross-cutting ไว้ที่ `core/`.

---

## 📌 ข้อตกลงการตั้งชื่อ (Conventions)

- Use cases ลงท้ายด้วย `_use_case.dart`
- Repository interfaces ลงท้ายด้วย `_repository.dart`; implementations ลงท้ายด้วย `_repository_impl.dart`
- Domain entities ลงท้ายด้วย `_entity.dart`
- Request/Response DTOs อยู่ใน `data/models_request` และ `data/models`
- Controllers ลงท้ายด้วย `_controller.dart`
- Pages ลงท้ายด้วย `_page.dart`
- Widgets ลงท้ายด้วย `_widget.dart`
- Bindings ลงท้ายด้วย `_binding.dart`
- Mappers ลงท้ายด้วย `_mapper.dart`
- Services ลงท้ายด้วย `_service.dart`
- Data sources ลงท้ายด้วย `_datasource.dart`
- Dio interceptors ลงท้ายด้วย `_interceptor.dart`
- Dio models ลงท้ายด้วย `_model.dart`

---

## 🧭 กฎเล็ก ๆ แต่ช่วย “กันโค้ดพัง”
### ✅ Do
- UseCase ทำงานเดียว → controller เรียก usecase; usecase เรียก repository
- Error เดินทางด้วย Failure → UI ตัดสินใจจากประเภท/ข้อความ
- Interceptor ไม่รู้เรื่อง UI → แค่โยน error ที่แปลงแล้วออกมา
- การ import ให้ import แบบ package ไม่ใช่ relative path อ้างจากชื่อแพ็กเกจใน pubspec.yaml ตามด้วยเส้นทางใต้ lib/
  - ใช้ relative เฉพาะกรณีพิเศษ
    - ไฟล์ part/part of ภายในโมดูลเดียวกัน
    - สคริปต์เล็กๆ/ตัวอย่างชั่วคราวในโฟลเดอร์เดียวกัน
    - แต่หลีกเลี่ยงการ “ผสม” relative และ package: ไปยังไฟล์เดียวกัน
- แยก service ที่เป็น cross-feature (session/fcm/deeplink/analytics) ไว้ core/services/
    - หมายถึง พวก service ที่ไม่ได้เป็นของฟีเจอร์ใดฟีเจอร์หนึ่งโดยเฉพาะ แต่ใช้ร่วมกันหลายฟีเจอร์ในแอพ → เราไม่ควรไปผูกมันไว้ใน features/ แต่ควรเอาไปไว้ที่ core/services/ เพื่อให้ทุกฟีเจอร์เรียกใช้ได้ เช่น
        - SessionService: เก็บข้อมูล session ปัจจุบัน เช่น access token, refresh token, current user
        - AuthSessionService: ทุกฟีเจอร์ที่ต้องเรียก API ก็ใช้ AuthSessionService ดึง token ไม่ต้องเขียนเองซ้ำ
        - DeeplinkService: รองรับการเปิดแอพผ่านลิงก์ (เช่น เปิดหน้าสินค้า, เปิดหน้าชำระเงิน)
        - AnalyticsService: ส่ง event ไปยัง Google Analytics, Firebase Analytics, หรือ Amplitude
- Test โครงสร้าง mirror source → test/features/<feature>/...


### ❌ Don't
- ห้าม DTO หลุดเข้า domain/presentation → map ผ่าน mappers/ เสมอ
- Controller ห้ามเรียก datasource ตรง → ผ่าน repository/usecase เท่านั้น
- ห้ามใช้ withOpacity สำหรับทำโปรงใสสี ให้ใช้ withValues

---

## 🌐 Internationalization (i18n)

- ใช้ [flutter_i18n](https://pub.dev/packages/flutter_i18n) สำหรับการจัดการภาษา
- ไฟล์ภาษาให้อยู่ใน `assets/i18n/` (รองรับ YAML/JSON/ARB ตามที่ flutter_i18n รองรับ)
- โครงสร้างไฟล์ตัวอย่าง:
  - `assets/i18n/en.yaml`
  - `assets/i18n/th.yaml`
- กำหนดใน pubspec.yaml:
  ```yaml
  flutter:
    assets:
      - assets/i18n/
  dependencies:
    flutter_i18n: ^0.32.0 # หรือเวอร์ชันล่าสุด
  ```
- ตั้งค่าใน main.dart:
  ```dart
  import 'package:flutter_i18n/flutter_i18n_delegate.dart';
  import 'package:flutter_localizations/flutter_localizations.dart';
  // ...existing code...
  MaterialApp(
    localizationsDelegates: [
      FlutterI18nDelegate(
        translationLoader: FileTranslationLoader(
          basePath: 'assets/i18n',
          fallbackFile: 'en',
          useCountryCode: false,
        ),
      ),
      GlobalMaterialLocalizations.delegate,
      GlobalWidgetsLocalizations.delegate,
      GlobalCupertinoLocalizations.delegate,
    ],
    supportedLocales: [Locale('en'), Locale('th')],
    // ...existing code...
  )
  ```
- เรียกใช้ในโค้ด:
  ```dart
  FlutterI18n.translate(context, "key")
  ```

---

## 🧩 Dependency Injection (DI) & Routing

- ต้องเพิ่ม dependency DI และ dependencies อื่น ๆ ที่เกี่ยวข้องกับโครงสร้างแอปใน pubspec.yaml ได้แก่:
  # --- Dependency Injection ---
  get_it: ^7.6.0
  injectable: ^2.3.0
  get: ^4.6.5
  # --- Internationalization ---
  flutter_i18n: ^0.32.0
  flutter_localizations:
    sdk: flutter
  # --- Network ---
  dio: ^5.4.0
  # --- Storage ---
  shared_preferences: ^2.2.2
  flutter_secure_storage: ^9.0.0
  # --- Theme/Font ---
  google_fonts: ^6.1.0
  # --- Firebase (ถ้าใช้ FCM/Analytics) ---
  firebase_core: ^2.24.2
  firebase_messaging: ^14.7.10
  firebase_analytics: ^10.8.0
  # --- อื่น ๆ ตามที่ใช้จริง ---

- ตัวอย่างใน pubspec.yaml:
  ```yaml
  dependencies:
    get_it: ^7.6.0
    injectable: ^2.3.0
    get: ^4.6.5
    flutter_i18n: ^0.32.0
    flutter_localizations:
      sdk: flutter
    dio: ^5.4.0
    shared_preferences: ^2.2.2
    flutter_secure_storage: ^9.0.0
    google_fonts: ^6.1.0
    firebase_core: ^2.24.2
    firebase_messaging: ^14.7.10
    firebase_analytics: ^10.8.0
    # ...
  dev_dependencies:
    injectable_generator: ^2.4.0
    build_runner: ^2.4.0
  ```
- ลงทะเบียน DI composition root ที่ `app/injection.dart` และแต่ละฟีเจอร์ผูก DI ใน bindings ของตัวเอง

---

## 🖼️ Wireframe Screen Mapping Rule (SC-XX/WG-XX)

- **Multi-state:**
  - หากใน wireframe มี SC-xx หรือ WG-xx หลายหมายเลข แต่จริง ๆ แล้วเป็นหน้าจอเดียวกันที่เปลี่ยนแปลงตามเงื่อนไข (เช่น state ก่อน/หลัง timeout, ปุ่มเปลี่ยนสถานะ ฯลฯ)
  - ให้รวมเป็น 1 feature/page เดียวกัน แล้วจัดการ state ภายใน controller/page
  - ตัวอย่าง: SC-01 (ก่อน timeout), SC-02 (หลัง timeout) → สร้าง 1 page + controller ที่รองรับหลาย state

- **Multi-tab:**
  - หาก SC-xx หรือ WG-xx หลายหมายเลขเป็นหน้าจอเดียวกันแต่แยกเป็น Tab (เช่น Tab ข้อมูลพื้นฐาน / Tab ข้อมูลอื่น ๆ)
  - ให้สร้างเป็น 1 page ที่มี TabBar/TabView และ controller เดียวกัน
  - ตัวอย่าง: WG-10 (Tab1), WG-11 (Tab2) → 1 page + TabBar + controller

- **แยก features:**
  - ถ้า SC-XX หมายถึงฟีเจอร์ใหม่ที่ไม่ซ้ำกับของเดิม ให้แยกเป็น features ใหม่ตามโครงสร้าง

> สรุป: การแยกหรือรวมหน้าจอให้ดูที่ logic จริงของ UI/UX ว่าเป็น state/tab เดียวกันหรือไม่ ไม่แยกตามหมายเลข SC/WG อย่างเดียว


