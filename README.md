# FloatLauncher

Kurulu uygulamaları tarayan, Material 3 arayüzüyle listeleyen ve cihazın **resmi olarak
desteklediği** yollarla kayan/serbest-form pencere deneyimi sunmaya çalışan bir Android
uygulaması.

> **Bu bir klonlama / sanallaştırma / çift alan (dual-space) uygulaması DEĞİLDİR.**
> Hiçbir uygulamanın APK'sını kopyalamaz, verisini sanallaştırmaz ya da ikinci bir
> kopyasını çalıştırmaz. Sadece PackageManager ile listeleme ve resmi Intent/Activity
> API'leriyle normal/serbest-form başlatma yapar.

## Mimari

```
app/
 └─ src/main/java/com/floatlauncher/
    ├─ FloatLauncherApp.kt          → basit el yapımı DI (Application)
    ├─ data/
    │   ├─ model/AppInfo.kt         → uygulama veri modeli
    │   └─ repository/AppRepository.kt
    ├─ ui/
    │   ├─ main/                    → MainActivity, MainViewModel, MainScreen, detay sheet
    │   ├─ permissions/              → izin onboarding ekranı ve durum modeli
    │   ├─ components/                → arama çubuğu, kategori filtresi, grid
    │   ├─ theme/                     → Material 3 koyu tema + mavi vurgu
    │   └─ overlay/                   → kayan panel/balon custom View'ları
    ├─ service/
    │   ├─ FloatingPanelService.kt   → ImGui esintili kayan kontrol paneli (overlay)
    │   ├─ FloatingWindowManager.kt  → seçilen uygulamayı başlatma mantığı
    │   └─ FloatLauncherAccessibilityService.kt → opsiyonel, pencere durumu algılama
    └─ util/
        ├─ PermissionUtils.kt        → izin kontrolü + Ayarlar yönlendirme
        ├─ DeviceCapabilityUtils.kt  → freeform/üretici yeteneği tespiti
        └─ PreferencesManager.kt     → DataStore: favoriler & son kullanılanlar
```

MVVM + Coroutines/Flow + Jetpack Compose (Material 3) + DataStore. Harici DI
framework'ü (Hilt/Koin) kasıtlı olarak eklenmedi; bağımlılıklar `FloatLauncherApp`
içinde açıkça kuruluyor — daha az boilerplate, daha düşük RAM ayak izi.

## Kayan pencere hakkında dürüst bir not

Android, kök (root) olmayan üçüncü taraf bir uygulamanın **başka bir uygulamayı**
kendi çizdiği bir overlay'in içine gömmesine izin vermez. Bu proje bu sınırı aşmaya
çalışmaz; bunun yerine üç gerçek senaryoyu ayırt eder:

1. **Cihaz resmi olarak "freeform window management" destekliyorsa**
   (`PackageManager.FEATURE_FREEFORM_WINDOW_MANAGEMENT`, bazı tabletler,
   Samsung DeX, geliştirici modunda "force resizable activities" açık cihazlar):
   hedef uygulama `ActivityOptions.setLaunchBounds()` ile gerçek anlamda
   serbest-form bir pencerede başlatılır.
2. **Cihaz üreticiye özgü bir çoklu pencere/pop-up özelliğine sahipse**
   (Samsung, Xiaomi/MIUI, Oppo/ColorOS, vivo, vb.): uygulama bunu programatik
   olarak tetikleyemez (genel API yok), bu yüzden kullanıcıyı doğrudan
   üreticinin kendi ayar sayfasına yönlendirir.
3. **Hiçbiri yoksa**: kullanıcıya açıkça bilgi verilir, uygulama normal şekilde
   başlatılır; hiçbir zaman sahte/kırık bir "kayan pencere" taklidi sunulmaz.

Uygulamanın kendi kontrol paneli/balonu ise gerçek anlamda kayandır: bu,
`SYSTEM_ALERT_WINDOW` izniyle çizilen kendi UI'ımız olduğu için resmi ve
tam desteklenen bir Android özelliğidir.

## Gerekli izinler

| İzin | Zorunlu mu? | Amaç |
|---|---|---|
| `SYSTEM_ALERT_WINDOW` (Diğer uygulamaların üzerinde göster) | Evet | Kayan kontrol paneli/balon |
| Bildirim izni (Android 13+) | Evet | Ön plan servisi kalıcı bildirimi |
| Erişilebilirlik servisi | Hayır (opsiyonel) | Sadece pencere durumu algılama; içerik okunmaz |
| `QUERY_ALL_PACKAGES` | Otomatik (manifest) | Kurulu uygulamaları listeleyebilmek |

## Derleme

1. Bu klasörü Android Studio (Koala veya üzeri) ile açın.
2. Studio, Gradle wrapper'ı otomatik olarak indirip senkronize edecektir
   (bu paylaşılan projede wrapper jar'ı dahil edilmemiştir; ilk açılışta
   Studio "Gradle wrapper bulunamadı" uyarısı verirse **File → Sync Project
   with Gradle Files** yeterlidir, ya da terminalde `gradle wrapper` çalıştırın).
3. `minSdk 29` (Android 10), `targetSdk/compileSdk 35` (Android 15/16 çizgisi).
4. Çalıştırmadan önce fiziksel bir cihaz veya API 29+ emülatör seçin.

## Desteklenen üreticiler

Samsung, Xiaomi/Redmi, Oppo, Vivo, Realme ve diğerleri için `DeviceCapabilityUtils`
içinde bilinen üretici listesi ve (varsa) resmi ayar sayfası yönlendirmeleri
tanımlıdır. Bu Intent component'leri üreticiler tarafından resmi olarak
dokümante edilmemiştir ve ROM sürümüne göre değişebilir; bu yüzden her zaman
genel "Uygulama Ayarları" sayfasına düşen bir fallback zinciri olarak
tasarlanmıştır.
