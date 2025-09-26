---
applyTo: '**'
---
# 🧭 Home Navigation (5-Tab Bottom Navigation) Guidelines

วัตถุประสงค์: กำหนดมาตรฐานการสร้างหน้า Home ที่มี bottom navigation 5 ปุ่ม ให้สอดคล้องกับ Clean Architecture, DI, analytics, deeplink และ state management (GetX) ในโปรเจ็กต์นี้

## ✅ ขอบเขต
- โครงสร้างไฟล์ / โค้ดสำหรับหน้า Home (Main Shell + Tabs)
- การจัดการ state / controller layer
- การ integrate กับ DeeplinkService / AnalyticsService / AuthSessionService
- Pattern สำหรับ lazy tab init + state retention
- การสื่อสารกับฟีเจอร์ย่อยผ่าน event/result

## 🧩 โครงสร้างไฟล์ที่ต้องมี
```
lib/
  presentation/
    home/
      pages/
        home_page.dart                # Scaffold + BottomNavigationBar / NavigationBar (Material 3)
      controllers/
        home_controller.dart          # จัดการ index, tab init, deeplink redirect
      bindings/
        home_binding.dart             # Resolve controller ผ่าน get_it
      widgets/
        home_nav_bar_widget.dart      # แยก widget bottom nav (custom look/iOS-like)
        home_tab_navigator_widget.dart# (ถ้าต้องการ per-tab Navigator แยก stack)
  features/
    <featureX>/ ... (tab content pages ของแต่ละฟีเจอร์)
```

## 🗂️ รายการแท็บ (เริ่มต้น 5 ปุ่ม)
| Index | Route name (ภายใน) | Feature / Module | Icon (light idea) | Note |
|-------|---------------------|------------------|-------------------|------|
| 0 | home_feed | feed / dashboard | home / grid | Default first tab |
| 1 | search | search | search | Support deeplink query |
| 2 | action_center | actions / quick | plus.circle / lightning | Floating central action optional |
| 3 | notifications | notifications | bell | Badge count driven by stream |
| 4 | profile | profile | person | Requires auth guard |

> สามารถปรับชื่อ index/icon ได้ แต่ต้องคง pattern route key และ analytics event

## 🔄 Lifecycle & Lazy Load
- โหลดเฉพาะ tab 0 ตอนเริ่มต้น
- Tab อื่น ๆ initial build เมื่อถูกเลือกครั้งแรก (lazy)
- ใช้ `IndexedStack` เพื่อ retain state (scroll position / form)
- ถ้า tab 2 เป็น central action (modal) → สามารถ override behavior ให้เปิด modal sheet แทนการสลับ index

Pseudo:
```dart
Widget build(BuildContext context) {
  return Obx(() => Scaffold(
    body: IndexedStack(
      index: c.currentIndex,
      children: c.tabs, // สร้างเฉพาะที่ถูก init แล้ว ที่เหลือ placeholder Container()
    ),
    bottomNavigationBar: HomeNavBarWidget(
      currentIndex: c.effectiveIndex,
      onTap: c.onTapNav,
      badges: c.badgeMap,
    ),
  ));
}
```

## 🧠 HomeController หน้าที่หลัก
| Responsibility | รายละเอียด |
|----------------|------------|
| index state | currentIndex + effectiveIndex (ในกรณี central action) |
| lazy init | initTabIfNeeded(i) เรียก repository/usecase ครั้งแรกเฉพาะ tab นั้น |
| deeplink | แปลง path → tab index + optional inner route (delegate ไปยัง tab navigator) |
| analytics | fire event เมื่อเปลี่ยน tab (throttle 300ms) |
| auth guard | ถ้ากด tab ที่ต้อง login แต่ยังไม่ login → push หน้า login (ไม่เปลี่ยน index) |
| badge sync | ฟัง stream (เช่น Notifications count) แล้ว update badgeMap |

## 🔐 Auth Guard แนวทาง
```dart
bool _requireAuth(int index) => index == 4 || index == 3; // profile + notifications
Future<bool> _ensureAuth() async {
  if (!session.isLoggedIn) {
    await Get.toNamed(AppRoutes.login);
    return session.isLoggedIn;
  }
  return true;
}
```

## 🔗 Deeplink Pattern
รูปแบบ deeplink:
```
myapp://tab/<tabKey>[?sub=<subRoute>&q=...]
```
ตัวอย่าง:
- myapp://tab/search?q=keyword
- myapp://tab/profile

Flow:
1. DeeplinkService emit event → HomeController.handleDeeplink(uri)
2. Map `<tabKey>` → index
3. If require auth → guard
4. Set index + ส่ง subRoute/query ไปยัง tab-specific controller (ผ่าน event bus / method call)

## 🧪 Analytics Events ตัวอย่าง
| Event | Params |
|-------|--------|
| tab_change | from, to, timestamp |
| tab_reselect | index |
| deeplink_tab_open | tab, source=deeplink |

## 🧵 Central Action (Tab กลางพิเศษ – ถ้าใช้)
- Index 2 สามารถเป็นปุ่ม FAB / translucent item ที่ไม่เปลี่ยน page
- onTap: เปิด BottomSheet หรือ Page ใหม่ → ไม่เปลี่ยน currentIndex
- effectiveIndex = currentIndex (แทน central) เพื่อไม่ให้ UI highlight central เสมอ

## 🧭 Per-Tab Navigator (ถ้าต้องการ stack แยก)
- ใช้ `Navigator` ซ้อนในแต่ละ tab (Key ต่อ tab)
- กลับจาก tab อื่นแล้ว previous stack ยังอยู่
- Long-press หรือ double tap บน tab (option) → popToRoot ของ stack นั้น

Snippet:
```dart
final _navKeys = List.generate(5, (_) => GlobalKey<NavigatorState>());
Navigator(
  key: _navKeys[index],
  onGenerateRoute: (settings) => ...,
);
```

## ♻️ State Restoration (Optional)
- เก็บ currentIndex + opened tabs ใน `AuthSessionService` หรือ simple local storage
- On app resume → restore index ถ้าไม่ conflict กับ deeplink launch

## 📦 DI & Binding
`home_binding.dart`:
```dart
class HomeBinding extends Bindings {
  @override
  void dependencies() {
    Get.put(HomeController(
      session: GetIt.I<AuthSessionService>(),
      analytics: GetIt.I<AnalyticsService>(),
      deeplink: GetIt.I<DeeplinkService>(),
      notificationRepo: GetIt.I<NotificationRepository>(),
    ));
  }
}
```
> ไม่มีการ new service ซ้ำ ให้ resolve จาก get_it เท่านั้น

## 🧱 Controller Outline
```dart
class HomeController extends GetxController {
  HomeController({
    required this.session,
    required this.analytics,
    required this.deeplink,
    required this.notificationRepo,
  });
  final AuthSessionService session;
  final AnalyticsService analytics;
  final DeeplinkService deeplink;
  final NotificationRepository notificationRepo;

  final _currentIndex = 0.obs;
  int get currentIndex => _currentIndex.value;
  set currentIndex(int v) => _currentIndex.value = v;

  final Set<int> _initedTabs = {0};
  final tabs = <int, Widget>{};
  final badgeMap = <int, int>{}.obs;

  int get effectiveIndex => currentIndex; // ปรับถ้ามี central action

  @override
  void onInit() {
    super.onInit();
    _initTab(0);
    _listenBadges();
    _listenDeeplink();
  }

  void onTapNav(int index) async {
    if (index == 2 && _isCentralAction()) { _openCentralAction(); return; }
    if (_requireAuth(index) && !await _ensureAuth()) return;
    if (currentIndex == index) {
      _onReselect(index);
      return;
    }
    final from = currentIndex;
    currentIndex = index;
    _initTab(index);
    analytics.logTabChange(from: from, to: index);
  }

  void _initTab(int index) {
    if (_initedTabs.contains(index)) return;
    // สร้าง/resolve page widget เฉพาะครั้งแรก
    _initedTabs.add(index);
  }

  void _onReselect(int index) {
    analytics.logTabReselect(index: index);
    // Optional: pop to root / scroll to top
  }

  bool _requireAuth(int index) => index == 3 || index == 4;
  Future<bool> _ensureAuth() async { /* ตามตัวอย่าง guard ด้านบน */ return true; }
  bool _isCentralAction() => false; // toggle ถ้าใช้ central action
  void _openCentralAction() { /* open sheet */ }

  void _listenBadges() {
    notificationRepo.countStream.listen((c) => badgeMap[3] = c);
  }
  void _listenDeeplink() {
    deeplink.stream.listen(_handleDeeplinkUri);
  }
  void _handleDeeplinkUri(Uri uri) { /* parse + switch tab */ }
}
```

## 🧪 Testing Strategy (เฉพาะ Home)
| Test | รายละเอียด |
|------|------------|
| initial_tab | เริ่มต้นต้องเป็น index 0 และ init แค่ tab 0 |
| lazy_init | เลือก tab 1 แล้วจึงสร้าง tab 1 ครั้งเดียว |
| auth_guard_block | เลือก tab 4 ขณะไม่ login → ไม่เปลี่ยน index |
| badge_update | push count stream → badgeMap[3] อัปเดต |
| deeplink_switch | deeplink ไป tab=search → index เปลี่ยนถูก |

## 🔔 Badge Handling
- เก็บ badge ต่อ tab ใน observable map (int→count)
- UI ตัดสินใจแสดง (dot กับ number) จาก threshold ( > 99 = 99+ )

## ♿ Accessibility
- แต่ละปุ่มใส่ tooltip / semanticLabel → ใช้ l10n keys: `nav.home`, `nav.search`, ...
- Contrast icon vs background ≥ 3:1

## 📝 Analytics Interface (ตัวอย่าง)
```dart
abstract class AnalyticsService {
  void logTabChange({required int from, required int to});
  void logTabReselect({required int index});
}
```

## 🚫 Anti-Patterns
- ใส่ business logic (repository call) ใน widget ของ tab โดยไม่ผ่าน controller/usecase
- rebuild ทั้ง Scaffold ทุกครั้งเพราะใช้ `GetBuilder` แบบรวม (ให้ใช้ Rx/Obx เฉพาะส่วนจำเป็น)
- new service ใน controller โดยตรง แทน resolve ผ่าน DI
- ปล่อยให้ deeplink เปลี่ยน tab แล้วไม่ guard auth

## ✅ Checklists ก่อน Merge Feature
- [ ] ใช้ IndexedStack + lazy init
- [ ] Guard auth tabs (profile/notifications) ทำงานถูกต้อง
- [ ] Analytics event ส่งเมื่อเปลี่ยน tab/reseselect
- [ ] Deeplink ไป tab/search พร้อม query ทำงานได้
- [ ] Badge notifications อัปเดตจาก stream
- [ ] ไม่มีการ new service ซ้ำ (ทุก dependency มาจาก get_it)
- [ ] i18n key ครบสำหรับ label ปุ่มทุกภาษา

จบ — ปรับปรุง/เพิ่ม tab ใหม่ให้แก้ที่ไฟล์นี้เท่านั้นเพื่อความสอดคล้อง
