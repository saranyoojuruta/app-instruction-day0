---
applyTo: '**'
---
- บังคับโครงสร้างนี้กับโปรเจกต์ Flutter/Dart ทุกครั้งที่มีการ prompt
- ทุกครั้งที่มีการ prompt ให้ทำ todoli- แยก service ที่เป็น cross-feature (session/deeplink/analytics) ไว้ core/services/
    - หมายถึง พวก service ที่ไม่ได้เป็นของฟีเจอร์ใดฟีเจอร์หนึ่งโดยเฉพาะ แต่ใช้ร่วมกันหลายฟีเจอร์ในแอพ → เราไม่ควรไปผูกมันไว้ใน features/ แต่ควรเอาไปไว้ที่ core/services/ เพื่อให้ทุกฟีเจอร์เรียกใช้ได้ เช่น
        - SessionService: เก็บข้อมูล session ปัจจุบัน เช่น access token, refresh token, current user
        - AuthSessionService: ทุกฟีเจอร์ที่ต้องเรียก API ก็ใช้ AuthSessionService ดึง token ไม่ต้องเขียนเองซ้ำ
        - DeeplinkService: รองรับการเปิดแอพผ่านลิงก์ (เช่น เปิดหน้าสินค้า, เปิดหน้าชำระเงิน)มอก่อนเริ่มทำงาน เพื่อแบ่งขั้นตอนการทำงานให้ชัดเจน

> UPDATE 2025-09: เอกสารฉบับนี้ปรับปรุงเพื่อ: (1) ชัดเจนเรื่อง DI (ใช้ get_it + injectable เป็น single source), (2) เพิ่มมาตรฐาน Result/Failure, (3) รวมธีมไปที่ `theme.instructions.md`, (4) อธิบาย token refresh concurrency, (5) ให้ทีมเลือกระบบ i18n เดียวทั้งโปรเจ็กต์, (6) ระบุการเลี่ยง API เก่า (Material 2) และรองรับ Material 3 โดยยังคง iOS look & feel.

## 🏗️ Clean Architecture Rules

**Dependency Rule**: `Presentation → Domain` และ `Data → Domain`
- Domain เป็น Pure Dart (ไม่รู้จัก Flutter/Dio/Storage)
- ติดต่อข้ามเลเยอร์ผ่าน interfaces เท่านั้น
- ทุก UseCase คืนค่าเป็น `Result<T>` (sealed) เพื่อจัดการ error แทน throw
- DTO ต้อง map เป็น Entity ก่อนออกจาก Data layer (ห้าม DTO หลุดขึ้น Presentation)


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
- ใช้ Dio + interceptors (`access_token`, `refresh_token`, `error`, `user_agent`)
- แปลง error เป็น `Failure` ด้วย `error_mapper` → mapping matrix (Network/Timeout/Server/Auth/Validation/Unknown)
- ใช้ `Result<T>` แทนการ `throw` (domain ไม่โยน exception ขึ้น UI)

**Token Refresh Concurrency Policy**:
1. Interceptor ตรวจ 401 (ยกเว้น endpoint refresh เอง) → ใส่ request ใน queue แล้ว trigger refresh ถ้ายังไม่มี refresh กำลังทำงาน
2. ใช้ mutex/flag ป้องกัน refresh ซ้อนกัน (เช่น `Completer<void>` รอผล)
3. สำเร็จ → replay queued requests ด้วย header ใหม่
4. ล้มเหลว (invalid/expired refresh) → เคลียร์ session + ส่งสัญญาณ logout (emit event จาก AuthSessionService)
5. จำกัดจำนวน replay/retry สูงสุด (เช่น 1 ครั้ง) ป้องกัน loop

Pseudo (แนวคิด):
```
if (response.statusCode == 401) {
  enqueue(request);
  if (!refreshing) {
    refreshing = true;
    final r = await authRepo.refresh();
    refreshing = false;
    if (r.isSuccess) replayAll(); else failAll(AuthFailure.expired());
  }
  return pendingResponseFuture; // รอจน refresh เสร็จ
}
```

**DI & Routing**:
- Composition Root: `app/injection.dart` 
- Feature Bindings: `features/<feature>/presentation/bindings/`
- Routes: `app/routes/app_routes.dart`  

## ⚙️ Environment & i18n

**Environment Config**: 
```bash
flutter run --dart-define=ENV=dev/stg/prod
```

**Internationalization (เลือก 1 แนวทางทั้งโปรเจ็กต์)**:
Option A (แนะนำระยะยาว - เลือกใช้ในโปรเจ็กต์นี้): Flutter gen-l10n + ARB (`l10n/*.arb`) → ประสิทธิภาพ & tooling ดีกว่า
Option B: `flutter_i18n` + YAML (`assets/i18n/{en,th}.yaml`) → แก้ไขง่าย ไม่ต้อง build step เพิ่ม
> โปรเจ็กต์นี้ปัจจุบัน: เลือก Option A (gen-l10n)
ใช้: `context.l10n.someKey`

โครงสร้างที่ต้องมี:
```
lib/
  l10n/
    app_en.arb
    app_th.arb
```
ตัวอย่าง `app_en.arb`:
```
{
  "appTitle": "Sample App",
  "welcomeMessage": "Welcome {userName}!",
  "@welcomeMessage": {
    "description": "Shown on the home screen greeting the user",
    "placeholders": {"userName": {"type": "String"}}
  }
}
```
การใช้งานในโค้ด:
```
import 'package:flutter_gen/gen_l10n/app_localizations.dart';

// MaterialApp
MaterialApp(
  localizationsDelegates: AppLocalizations.localizationsDelegates,
  supportedLocales: AppLocalizations.supportedLocales,
  onGenerateTitle: (ctx) => ctx.l10n.appTitle,
  // ...
);

// ใช้งานใน widget
final text = context.l10n.welcomeMessage(userName);
```
CI เพิ่ม build step (ถ้าใช้): `flutter gen-l10n` (รันอัตโนมัติผ่าน flutter build)

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
- Error เดินทางด้วย Failure → UI ตัดสินใจจากประเภท/ข้อความ ไม่จับ exception ที่ domain ไม่โยน
- Import แบบ package: `import 'package:app_name/...';`
- แยก cross-feature services ไว้ `core/services/`
- Test mirror source structure
- DI ทั้งหมด register ใน get_it (injectable) ส่วน GetX Binding = resolve เท่านั้น ไม่ new instance
- Theme: ใช้ `theme.instructions.md` เป็น single source; หลีกเลี่ยงแก้ ThemeData กระจัดกระจาย
- ใช้ ColorScheme (Material 3) แต่ปรับ palette ให้ iOS-like ได้

**❌ Don't:**
- ห้าม DTO หลุดเข้า domain/presentation → map ผ่าน mappers/
- Controller ห้ามเรียก datasource ตรง → ผ่าน repository/usecase
- หลีกเลี่ยงใช้ API Material 2 ที่ deprecated (เช่น textTheme.headline1) ในโค้ดใหม่
- ห้ามสร้าง instance service ซ้ำในหลายที่ (single source DI)
- อย่า refresh token ซ้อนกันหลายครั้ง (ต้องมี mutex)

### Result / Failure Contract (มาตรฐาน)
Result<T> มี 2 รูป:
```
sealed class Result<T> {
  const Result();
  bool get isSuccess => this is Success<T>;
  bool get isFailure => this is FailureResult<T>;
  T get data => (this as Success<T>).value; // ใช้เฉพาะกรณีแน่ใจ
  Failure get failure => (this as FailureResult<T>).error;
}
class Success<T> extends Result<T> { final T value; const Success(this.value); }
class FailureResult<T> extends Result<T> { final Failure error; const FailureResult(this.error); }
```
Failure (ตัวอย่าง):
```
sealed class Failure { final String message; final Object? cause; const Failure(this.message,{this.cause}); }
class NetworkFailure extends Failure { const NetworkFailure(String m,{Object? cause}) : super(m,cause:cause); }
class TimeoutFailure extends Failure { const TimeoutFailure(String m): super(m); }
class AuthFailure extends Failure { final bool expired; const AuthFailure(String m,{this.expired=false}): super(m); }
class ValidationFailure extends Failure { final Map<String,String>? fieldErrors; const ValidationFailure(String m,{this.fieldErrors}); }
class ServerFailure extends Failure { final int? status; const ServerFailure(String m,{this.status}); }
class UnknownFailure extends Failure { const UnknownFailure(String m,{Object? cause}): super(m,cause:cause); }
```
UseCase pattern:
```
abstract class UseCase<T, P> { Future<Result<T>> call(P params); }
``` 
Controller ตัวอย่าง:
```
final r = await loginUseCase(LoginParams(...));
r.when(
  success: (u) => state = LoggedIn(u),
  failure: (f) => showError(f.message),
);
```

### DI Policy
- ทุก dependency อยู่ใน `injection.dart` (generate ด้วย injectable)
- GetX Binding: `Get.put(GetIt.I<MyController>())` หรือ factory จาก usecases ที่ resolve แล้ว
- หลีกเลี่ยง manual singleton นอกเหนือจากการลงทะเบียนใน get_it

### Token Refresh Edge Cases
- Refresh 401 แล้ว 401 ซ้ำอีก → ยุติ flow และ logout
- Refresh endpoint timeout → retry 1 ครั้งสูงสุด
- เกิน max queued requests (เช่น 30) → fail ทั้งหมดด้วย AuthFailure

### สี / withValues
- ถ้าต้องการปรับ RGBA ให้ implement extension ชื่อ `withValues({int? r,int? g,int? b,int? a})` ใน `core/extensions/color_extension.dart`
- หลีกเลี่ยง `withOpacity(0.x)` บ่อย ๆ ในธีมหลักเพื่อไม่ทำ palette กระจัดกระจาย ใช้สีจาก ColorScheme

### THEME
เนื้อหาเต็มย้ายไปที่ `theme.instructions.md` (ไฟล์นี้ไม่ถือเป็น source of truth ด้านธีมแล้ว)

### Home Navigation (5 Tabs)
ดูรายละเอียดมาตรฐานการสร้างหน้า Home + Bottom Navigation 5 ปุ่ม ที่ไฟล์ `home_navigation.instructions.md` (โครงสร้าง controller, deeplink, analytics, badge, lazy init)

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
- หลีกเลี่ยงความโปร่งใสเกินไปในธีม 
<!-- ห้ามใช้ withOpacity สำหรับทำโปรงใสสี ให้ใช้ withValues -->



