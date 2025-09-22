---
applyTo: '**'
---
.
├── lib/
│   ├── app/                                # แอประดับสูง (entry, env, routes, DI)
│   │   ├── main.dart                       # จุดเริ่มต้นของแอป
│   │   ├── env/                            # ค่าคอนฟิก per environment
│   │   │   ├── app_env.dart                # sealed class / enum Env { dev, stg, prod }
│   │   │   └── env_config.dart             # base + impl (API baseUrl, feature flags)
│   │   ├── config/                         # คอนฟิกรวมระดับแอป (AppConfig)
│   │   ├── routes/                         # เส้นทางรวมของทั้งแอป
│   │   │   └── app_routes.dart             # กำหนดชื่อ route และ path
│   │   └── injection.dart                  # composition root รวม DI เข้า GetX/get_it
│   │
│   ├── core/                               # cross-cutting (ไม่ผูกกับฟีเจอร์)
│   │   ├── constants/                      # ค่าคงที่, keys, timeouts
│   │   ├── error/                          # Failure, Exception mapping, guard
│   │   │   ├── failure.dart                # โครงสร้าง Failure ที่ใช้ใน domain
│   │   │   └── error_mapper.dart           # แปลง error จาก data เป็น failure
│   │   ├── network/                        # การจัดการ HTTP (Dio)
│   │   │   ├── dio_client.dart             # ตั้งค่า Dio client
│   │   │   └── interceptors/               # ตัวดักจับ request/response เช่น token, error
│   │   │       ├── access_token_interceptor.dart   # เพิ่ม access token ใน header
│   │   │       ├── refresh_token_interceptor.dart  # ดักจับ 401 แล้วรีเฟรช token
│   │   │       ├── error_interceptor.dart          # แปลง DioError เป็น Failure
│   │   │       └── user_agent_interceptor.dart     # เพิ่ม user-agent ใน header
│   │   ├── storage/                        # การจัดเก็บข้อมูลในเครื่อง
│   │   │   ├── shared_prefs_service.dart           # ใช้ shared_preferences
│   │   │   └── secure_storage_service.dart         # ใช้ flutter_secure_storage
│   │   ├── services/                       # cross-feature services (auth session, FCM, etc.)
│   │   │   ├── auth_session_service.dart   # จัดการ token/identity ที่ระดับแอป
│   │   │   ├── firebase_message_service.dart   # จัดการ FCM
│   │   │   ├── deeplink_service.dart   
│   │   │   └── analytics_service.dart      # จัดการ analytics (Firebase, etc.)
│   │   ├── extensions/                     # Dart extensions (date, num, log, hex)
│   │   ├── theme/                          # ธีม (theme, color, spacing, light/dark, typography)
│   │   ├── type/                           # Enums/Types ที่ใช้ร่วม
│   │   ├── util/                           # Utilities (validators, formatters, etc.)
│   │   └── result/                         # โครงสร้างผลลัพธ์แบบรวม (Result/Either)
│   │       └── result.dart
│   │
│   ├── shared/                             # สิ่งที่ UI ใช้ร่วมกัน (ไม่ผูกฟีเจอร์)
│   │   ├── widgets/                        # Reusable widgets 
│   │   │   ├── app_bar/                    # App Bar widgets
│   │   │   │   ├── app_bar_main_widget.dart # Main page app bar
│   │   │   │   └── app_bar_widget.dart     # Secondary page app bar
│   │   │   └── bottom_nav_bar/             # Bottom navigation bar 
│   │   │       └── bottom_nav_bar_widget.dart # Main page bottom navigation bar
│   │   └── bindings/                       # Global bindings ถ้าจำเป็น
│   │
│   ├── features/                           # แยกตามฟีเจอร์ (สเกลง่าย/แยกทีมทำงานได้)
│   │   ├── auth/                           # ตัวอย่างฟีเจอร์การเข้าสู่ระบบ
│   │   │   ├── domain/                     # business logic (pure Dart)
│   │   │   │   ├── entities/               # pure Dart entities
│   │   │   │   │   └── user.dart           # entity หลัก เช่น User
│   │   │   │   ├── repositories/           # contracts (abstract)
│   │   │   │   │   └── auth_repository.dart # interface
│   │   │   │   └── usecases/               # use cases
│   │   │   │       ├── login_use_case.dart # use case เช่น login
│   │   │   ├── data/                       # data layer (API, DB, cache)
│   │   │   │   ├── datasources/
│   │   │   │   │   ├── remote/             # API/HTTP/Firebase per feature
│   │   │   │   │   │   └── auth_api.dart   
│   │   │   │   │   └── local/              # cache/db per feature
│   │   │   │   │       └── auth_cache.dart 
│   │   │   │   ├── models/                 # DTOs responses
│   │   │   │   ├── models_request/         # DTOs requests
│   │   │   │   ├── mappers/                # map DTO <-> Entity
│   │   │   │   └── repositories/           # implementations
│   │   │   │       └── auth_repository_impl.dart   
│   │   │   └── presentation/               # UI (Flutter) + state management (GetX)
│   │   │       ├── pages/                  # หน้าจอ เช่น login
│   │   │       │   └── login_page.dart
│   │   │       ├── controllers/            # Controller/ViewModel
│   │   │       │   └── login_controller.dart
│   │   │       ├── bindings/               # Binding สำหรับ DI
│   │   │       │   └── auth_binding.dart
│   │   │       └── widgets/                # widgets เฉพาะฟีเจอร์
│   │   │           └── login_form.dart
│   │   └── <more_features>/...             # profile, order, payment, ...
│   │
│   └── shared_libraries.dart               # export barrel ถ้าต้องการ
│
├── assets/                                 # static files (font, icon, i18n)
│   ├── i18n/                               # (แนะนำ) ย้ายเป็น ARB: l10n/*.arb
│   │   └── en|th/*.yaml                    # ใช้ easy_localization หรือ custom loader
│   ├── fonts/                              # ฟอนต์ที่ใช้ในแอป
│   └── icon_*/                             # ไอคอน SVG หรือ PNG
│
├── test/                                   # unit + widget tests
│   ├── core/                               # unit tests สำหรับ core
│   └── features/                           # unit + widget tests สำหรับแต่ละฟีเจอร์
│       └── auth/                           # ตัวอย่างฟีเจอร์การเข้าสู่ระบบ
│           ├── domain/                     # ทดสอบ business logic
│           ├── data/                       # ทดสอบ data layer
│           └── presentation/               # ทดสอบ UI + state management
└── analysis_options.yaml                   # lints (flutter_lints + custom)

คำอธิบายโครงสร้าง:
App: จุดเริ่มแอป, ค่าคอนฟิก environment, Route รวม, Composition Root (DI)
Core: สิ่งที่ทุกฟีเจอร์ใช้ร่วมกัน (Dio + interceptors, Storage, Session/FCM/Deeplink/Analytics service, Error/Result, Theme/i18n)
Shared: Widgets/UI ที่นำกลับใช้ได้ ไม่ผูกฟีเจอร์
Features: แยกเป็นฟีเจอร์เดี่ยว ๆ (auth, profile ฯลฯ) ภายในยังคงแยก Domain/Data/Presentation

