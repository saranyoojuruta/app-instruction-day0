---
applyTo: '**'
---
- บังคับโครงสร้างนี้กับโปรเจกต์ Flutter/Dart ทุกครั้งที่มีการ prompt
- ทุกครั้งที่มีการ prompt ให้ทำ todolist เสมอก่อนเริ่มทำงาน เพื่อแบ่งขั้นตอนการทำงานให้ชัดเจน
## 📝 Dependency Rule

- Dependency Rule จาก นอก → ใน เท่านั้น
```Presentation → Domain``` และ ```Data → Domain```
- Domain ไม่รู้จัก Flutter/Dio/Storage ใด ๆ (Pure Dart)
- ติดต่อข้ามเลเยอร์ผ่าน interfaces (Repository contracts) เท่านั้น


### TabBar/TabView Usage
- ถ้ามีการใช้ `DefaultTabController` หรือ `TabBarView` ในหน้าใด ๆ **ต้อง**
  - กำหนดและควบคุม `TabController` ผ่าน controller หลักของหน้านั้น (เช่น GetX controller)
  - ห้ามสร้าง TabController ใน widget โดยตรง ให้ inject หรือดึงจาก controller หลักเท่านั้น
  - แต่ละ tab/page ที่อยู่ใน TabBarView **ต้องแยกเป็นไฟล์และ widget ของตัวเอง**
  - Logic ของแต่ละ tab/page ให้แยก controller ของตัวเอง (child controller) และ inject ผ่าน controller หลัก
  - หลีกเลี่ยงการเขียน logic ของแต่ละ tab/page ปะปนใน controller หลัก

ตัวอย่าง:

```dart
// ใน main controller
class MainController extends GetxController with GetSingleTickerProviderStateMixin {
  late TabController tabController;
  @override
  void onInit() {
    tabController = TabController(length: 2, vsync: this);
    super.onInit();
  }
  @override
  void onClose() {
    tabController.dispose();
    super.onClose();
  }
}

// ใน main page
class MainPage extends GetView<MainController> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: TabBar(controller: controller.tabController, ...),
      body: TabBarView(controller: controller.tabController, children: [Tab1Page(), Tab2Page()]),
    );
  }
}

// ในแต่ละ tab/page
class Tab1Page extends GetView<Tab1Controller> { ... }
class Tab2Page extends GetView<Tab2Controller> { ... }
```

> สรุป: ถ้าใช้ TabBar/TabBarView ต้องแยก controller และ logic ของแต่ละ tab/page ออกจากกันเสมอ

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
   - Controller access ผ่าน `controller` property ของ GetView เท่านั้น (ห้ามใช้ Get.find() ใน widget/build)
   - ให้ใช้ stateMixins ใน controller เพื่อแยก logic ที่ใช้ซ้ำได้
   - ถ้าหน้ามีการ โหลดข้อมูล ให้ใช้ `StateMixin<T>` และ `controller.obx(...)` ในหน้า Page
   - ถ้าหน้ามีการ โหลดข้อมูลหลายชุด ให้ใช้ `MultipleStatusMixin`
   - ถ้าหน้ามีการ โหลดข้อมูลที่ซับซ้อน (เช่น มี filter, search, pagination) ให้ใช้ `PagingMixin` หรือ `PagingController` (ถ้าใช้ infinite scroll)
   - ถ้าหน้ามีการ Scroll ที่ซับซ้อน (เช่น scroll to top/bottom, scroll to index) ให้ใช้ `ScrollMixin`
   - ถ้าหน้ามีการ Refresh (pull to refresh) ให้ใช้ `RefreshMixin
   - ถ้าหน้ามีการ โหลดข้อมูล + Refresh + Pagination ให้ใช้ `PagingRefreshMixin`
   - ถ้าหน้ามีการ โหลดข้อมูล + Refresh + Pagination + Scroll ให้ใช้ `PagingRefreshScrollMixin`
   - ถ้าหน้ามีการ โหลดข้อมูล + Refresh + Scroll ให้ใช้ `RefreshScrollMixin`
   - ถ้าหน้ามีการ โหลดข้อมูล + Pagination ให้ใช้ `PagingMixin`
   - ถ้าหน้ามีการ โหลดข้อมูล + Scroll ให้ใช้ `ScrollMixin`
   - ถ้าหน้ามีการ Refresh + Pagination ให้ใช้ `PagingRefreshMixin`
   - ถ้าหน้ามีการ Refresh + Scroll ให้ใช้ `RefreshScrollMixin`
   - ถ้าหน้ามีการ Pagination + Scroll ให้ใช้ `PagingScrollMixin`
   - ถ้าถ้ามีการ ใช้ state ที่ซับซ้อน ให้แยกเป็น controller ย่อย (child controller) แล้ว inject ผ่าน binding

5. **ทำ Widgets**
   - สร้าง widgets เฉพาะฟีเจอร์ใน `features/<feature>/presentation/widgets/`
   - ถ้า widgets นำกลับใช้ได้หลายฟีเจอร์ ให้ย้ายไป `shared/`
   - ห้ามสร้าง widgets ในหน้า Page โดยตรง

6. **ผูก DI และ Route**
   - ผูก dependencies ใน `features/<feature>/presentation/bindings/<feature>_binding.dart`
   - เพิ่มเส้นทางใน `app/routes/app_routes.dart`

7. **เขียน Tests**
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
        <!-- - AnalyticsService: ส่ง event ไปยัง Google Analytics, Firebase Analytics, หรือ Amplitude -->
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
  firebase_core: ^4.1.1
  firebase_messaging: ^16.0.2
  firebase_analytics: ^12.0.2
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
    firebase_core: ^4.1.1
    firebase_messaging: ^16.0.2
    firebase_analytics: ^12.0.2
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
  - ให้สร้างหน้าจอหลัก 1 page (main page) ที่มี TabBar/TabView
  - แต่ละ tab ให้แยกเป็น page ของตัวเอง และแต่ละ tab มี controller ของตัวเอง (ไม่ใช้ controller เดียวกัน) และใช้ TabController ของ Flutter ใน main page ควบคุมการเปลี่ยน tab และ TabController ของแต่ละ tab ควบคุม state ภายใน tab นั้น ๆ โดยที่ต้องอยู่ใน controller ของ tab นั้น ๆ
  - ตัวอย่าง: WG-10 (Tab1), WG-11 (Tab2) → main page + TabBar + WG-10_page.dart + WG-11_page.dart + controller แยกแต่ละ tab

- **แยก features:**
  - ถ้า SC-XX หมายถึงฟีเจอร์ใหม่ที่ไม่ซ้ำกับของเดิม ให้แยกเป็น features ใหม่ตามโครงสร้าง

> สรุป: การแยกหรือรวมหน้าจอให้ดูที่ logic จริงของ UI/UX ว่าเป็น state/tab เดียวกันหรือไม่ ไม่แยกตามหมายเลข SC/WG อย่างเดียว

- **Orchestrator Pattern:**
  - ใช้ OrchestratorService ใน core/services/orchestrator_service.dart เพื่อจัดการ WG widget ทั้งหมดในแอป
  - OrchestratorService จะทำหน้าที่:
    - สร้าง/ทำลาย WG widget ตามคำสั่งจาก controller
    - ป้องกันการแสดง toast ซ้ำซ้อน (toast cooldown)
    - จัดคิว popup ให้แสดงทีละอัน
    - ล็อกสถานะ modal เมื่อมี modal แสดงอยู่
  - ทุก controller ที่ต้องการแสดง WG widget ต้องเรียกผ่าน OrchestratorService เท่านั้น ห้ามสร้าง/แสดง WG widget โดยตรงใน controller/page

- **WG Widget Compliance:**
  - ต้องใช้ WG widget ตามที่กำหนดใน wireframe เท่านั้น
  - ห้ามสร้าง WG widget ใหม่ที่ไม่อยู่ในรายการที่กำหนด
  - ถ้าต้องการเพิ่ม WG widget ใหม่ ต้องขออนุมัติและอธิบายการใช้งานให้ชัดเจนก่อน

- **ReadMe:**
  - อธิบายการโปรงโครงสร้างแอป, การเพิ่มฟีเจอร์ใหม่, การใช้ OrchestratorService, และกฎการตั้งชื่อใน README.md ของโปรเจกต์เสมอ
  - อัพเดต README.md ทุกครั้งที่มีการเปลี่ยนแปลงโครงสร้างหรือกฎ
  - ตัวอย่าง README.md:
    ```markdown
    # Project Structure

    ## Folder Structure
    - lib/
      - app/: Entry, env, routes, DI
      - core/: Cross-cutting concerns (Dio, storage, theme, error, etc.)
      - shared/: Reusable widgets/UI (not feature-specific)
      - features/: Feature modules (auth, profile, etc.)
      - shared_libraries.dart

    ## Adding New Features
    1. Create feature structure using generator script.
    2. Implement Domain layer (Entities, Repository interfaces, UseCases).
    3. Implement Data layer (Models, Mappers, Datasources, Repository implementations).
    4. Implement Presentation layer (Pages, Controllers, Bindings, Widgets).
    5. Register DI in feature bindings.
    6. Add routes in app/routes/app_routes.dart.
    7. Write tests mirroring source structure.

    ## OrchestratorService
    - Manages WG widgets globally.
    - Prevents toast spamming.
    - Queues popups.
    - Locks modal state.

    ## Naming Conventions
    - UseCase: *_use_case.dart
    - Repository interface: *_repository.dart
    - Repository implementation: *_repository_impl.dart
    - Entity: *_entity.dart
    - DTO: data/models_request/ and data/models/
    - Controller: *_controller.dart
    - Page: *_page.dart
    - Widget: *_widget.dart
    - Binding: *_binding.dart
    - Mapper: *_mapper.dart
    - Service: *_service.dart
    - Datasource: *_datasource.dart
    - Interceptor: *_interceptor.dart
    - Model: *_model.dart
    ```

