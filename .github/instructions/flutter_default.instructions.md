---
applyTo: '**'
---
- บังคับโครงสร้างนี้กับโปรเจกต์ Flutter/Dart ทุกครั้งที่มีการ prompt
- ทุกครั้งที่มีการ prompt ให้ทำ todoli- แยก service ที่เป็น cross-feature (session/deeplink/analytics) ไว้ core/services/
    - หมายถึง พวก service ที่ไม่ได้เป็นของฟีเจอร์ใดฟีเจอร์หนึ่งโดยเฉพาะ แต่ใช้ร่วมกันหลายฟีเจอร์ในแอพ → เราไม่ควรไปผูกมันไว้ใน features/ แต่ควรเอาไปไว้ที่ core/services/ เพื่อให้ทุกฟีเจอร์เรียกใช้ได้ เช่น
        - SessionService: เก็บข้อมูล session ปัจจุบัน เช่น access token, refresh token, current user
        - AuthSessionService: ทุกฟีเจอร์ที่ต้องเรียก API ก็ใช้ AuthSessionService ดึง token ไม่ต้องเขียนเองซ้ำ
        - DeeplinkService: รองรับการเปิดแอพผ่านลิงก์ (เช่น เปิดหน้าสินค้า, เปิดหน้าชำระเงิน)มอก่อนเริ่มทำงาน เพื่อแบ่งขั้นตอนการทำงานให้ชัดเจน

## 🏗️ Clean Architecture Rules

**Dependency Rule**: `Presentation → Domain` และ `Data → Domain`
- Domain เป็น Pure Dart (ไม่รู้จัก Flutter/Dio/Storage)
- ติดต่อข้ามเลเยอร์ผ่าน interfaces เท่านั้น
- ทุก UseCase คืนค่าเป็น `Result<T>` เพื่อจัดการ error
- DTO ต้อง map เป็น Entity ก่อนออกจาก Data layer


## 📊 Data Flow

```
UI (Page/Widget)
  └─ Controller (GetX) → UseCase (Domain) → Repository (Interface)
      └─ Repository Impl (Data) → [API/Local] → Mappers → Result<T>
```

## 🛠️ Core Components

**Cross-cutting Services** (`core/services/`):
- **AuthSessionService**: จัดการ token & current user
- **DeeplinkService**: รับลิงก์เข้าแอป
- **AnalyticsService**: ส่งเหตุการณ์การใช้งาน

**Network & Error**:
- ใช้ Dio + interceptors (`access_token`, `refresh_token`, `error`)
- แปลง error เป็น `Failure` ด้วย `error_mapper`
- ใช้ `Result<T>` แทนการ `throw` error

**DI & Routing**:
- Composition Root: `app/injection.dart` 
- Feature Bindings: `features/<feature>/presentation/bindings/`
- Routes: `app/routes/app_routes.dart`  

## ⚙️ Environment & i18n

**Environment Config**: 
```bash
flutter run --dart-define=ENV=dev/stg/prod
```

**Internationalization**:
- ใช้ `flutter_i18n` กับไฟล์ `assets/i18n/{en,th}.yaml`
- เรียกใช้: `FlutterI18n.translate(context, "key")`

## 🧪 Testing Strategy

โครงสร้าง test mirror source: `test/features/<feature>/domain|data|presentation`
- **Domain**: ทดสอบ UseCase (mock repository)  
- **Data**: ทดสอบ Repository impl + Mapper + Datasource
- **Presentation**: Widget/Controller tests  

## 🔧 การเพิ่มฟีเจอร์ใหม่

1. **สร้างโครง**: `dart run tool/gen_feature.dart <FeatureName>`
2. **Domain**: สร้าง Entities, Repository (interface), UseCases
3. **Data**: สร้าง Models/Request, Mappers, Datasources, Repository Impl
4. **Presentation**: สร้าง Pages, Controllers, Bindings, Widgets
   - Controller เรียก UseCase เสมอ (ห้ามเรียก datasource ตรง)
   - ใช้ StateMixins สำหรับ UI state:
     - `StateMixin<T>` + `controller.obx()` สำหรับโหลดข้อมูลเบื้องต้น
     - `PagingMixin` สำหรับ pagination
     - `RefreshMixin` สำหรับ pull-to-refresh
     - `ScrollMixin` สำหรับ scroll control
5. **Widgets**: สร้างใน `features/<feature>/presentation/widgets/`
6. **DI & Route**: ผูกใน `<feature>_binding.dart` และเพิ่มใน `app_routes.dart`
7. **Tests**: เขียน test mirror source structure

---

## 📌 Naming Conventions & Rules

**การตั้งชื่อไฟล์:**
- Use cases: `*_use_case.dart`
- Repository interface: `*_repository.dart` / implementation: `*_repository_impl.dart`
- Entities: `*_entity.dart`
- Controllers: `*_controller.dart`
- Pages: `*_page.dart` / Widgets: `*_widget.dart`
- Bindings: `*_binding.dart` / Services: `*_service.dart`

**✅ Do:**
- UseCase ทำงานเดียว → controller เรียก usecase; usecase เรียก repository
- Error เดินทางด้วย Failure → UI ตัดสินใจจากประเภท/ข้อความ
- Import แบบ package: `import 'package:app_name/...'`
- แยก cross-feature services ไว้ `core/services/`
- Test mirror source structure

**❌ Don't:**
- ห้าม DTO หลุดเข้า domain/presentation → map ผ่าน mappers/
- Controller ห้ามเรียก datasource ตรง → ผ่าน repository/usecase
- ห้ามใช้ `withOpacity` → ใช้ `withValues`

## 🖼️ Wireframe Screen Mapping (SC-XX/WG-XX)

**Multi-state**: SC-XX หลายหมายเลขที่เป็นหน้าจอเดียวกัน → รวมเป็น 1 page + จัดการ state
**Multi-tab**: WG-XX แยก Tab → main page + TabBar + แต่ละ tab แยก controller
**Orchestrator Pattern**: ใช้ OrchestratorService ใน `core/services/` จัดการ WG widget

## 🧩 Dependencies ที่ต้องมี

```yaml
dependencies:
  get: ^4.6.5
  get_it: ^7.6.0
  injectable: ^2.3.0
  flutter_i18n: ^0.32.0
  dio: ^5.4.0
  shared_preferences: ^2.2.2
  flutter_secure_storage: ^9.0.0
```

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



