# DNS Hızlı — Okubeni

Popüler, bilinen herkese açık DNS sunucularını (Cloudflare, Google, Quad9, AdGuard…) tek listede toplayan ve
**tek dokunuşla** bağlanan, sade ve akışkan (Frutiger Aero) bir Android uygulaması. Telefonda yerel bir VPN arayüzü
açar; yalnızca DNS sorguları seçilen sunucuya gider, diğer tüm trafik normal yolundan devam eder. Kullanıcı kendi DNS'ini
ekleyemez (bilinçli: yalnızca bilinen sunucular). Sorgular varsayılan olarak **şifreli** (DoT/DoH) gider. **12 dil**: Türkçe,
English, Deutsch, Español, Français, Português, Русский, Italiano, Bahasa Indonesia, Tiếng Việt, Polski, Українська. **12 tema.**

- **Dil / araçlar:** Kotlin 2.3 · AGP 8.13.2 · Gradle 8.13 · minSdk 26 (Android 8.0) · hedef/derleme SDK 36 · **dış bağımlılık yok.**
- **Özellikler:** `OZELLIKLER.md` (110 özellik, hangisinin nasıl doğrulandığıyla birlikte).
- **Gizlilik politikası şablonu:** `GIZLILIK.md` · **Mağaza metni:** `store/magaza_metni.md`.

> **Dürüst durum özeti.** Çekirdek mantık (DNS yönlendirici, önbellek, yedek sunucu, ağ kuralları, model ve tüm arayüz sahnesi)
> bilgisayarda **1296 otomatik kontrolle** sınandı ve geçti (her dil ayrı ayrı; şifreli DNS yerel bir TLS sunucusuna karşı).
> Android'e özgü kısım (VPN servisi/tüneli, bildirim, karo, widget, açılışta bağlanma, erişilebilirlik sağlayıcısı, etkinlik)
> yazıldı; kaynaklar gerçek **aapt2** ile bağlandı ve tüm kod **Kotlin 2.3.21** ile Android 36 API'sine karşı hatasız derlendi;
> ama **bir telefonda ya da emülatörde çalıştırılmadı.** Tam Gradle derlemesi (AGP/R8) bu ortamda çalıştırılamadı (Google
> sunucularına erişim yok); GitHub'daki bulut derlemesi bunu yapar (aşağıda). Önizleme görselleri cihaz ekran görüntüsü değil, aynı çizim kodunun bilgisayarda üretilmiş halidir.
> Yani: ilk denemede küçük hatalar çıkması normaldir; aşağıdaki "İlk cihaz denemesi" listesi tam bunun için.

## 0. Bilgisayarsız: telefondan APK (GitHub)

Proje `.github/workflows/apk.yml` ile gelir; GitHub derlemeyi bulutta yapar.

1. GitHub uygulamasında ya da tarayıcıda (telefonda Chrome → "Masaüstü sitesi" daha rahattır) **boş** bir depo aç.
2. **Add file → Upload files** ile **yalnızca `dns-hizli.zip`** dosyasını yükle (açma; dosyaları tek tek yüklersen klasörler
   kaybolur, önceki derleme hatasının en olası nedeni buydu). Commit.
3. **Add file → Create new file**; ad kutusuna tam olarak `.github/workflows/apk.yml` yaz, içine zip'teki aynı adlı dosyanın
   içeriğini yapıştır, commit. (GitHub, zip'in içindeki iş akışını okuyamaz; bu dosya depoda ayrıca bulunmalıdır.)
4. **Actions** sekmesi → "APK derle" → **Run workflow** (commit sonrası kendiliğinden de başlar). 5–10 dk sürer.
5. Bitince çalıştırmaya gir → en altta **Artifacts → dns-hizli-apk** indir → içindeki `app-debug.apk`'yı aç ve kur
   ("Bilinmeyen kaynaklar" izni istenir).

Kırmızı ✗ çıkarsa: çalıştırmaya gir → kırmızı adıma dokun → son 30 satırı kopyala. İş akışı Türkçe açıklamalı hata yazar
("Zip içinde settings.gradle.kts yok", "Klasör yapısı bozuk" gibi). Debug APK yalnızca deneme içindir; Play'e `bundleRelease`
(imzalı .aab) gider, bkz. bölüm 4.

## 1. Derleme ve çalıştırma (bilgisayarla)

1. **Android Studio**'nun güncel sürümünü kur. `File → Open` ile bu klasörü aç. İlk senkronizasyon internetten Gradle 8.13,
   Android Gradle Plugin ve Kotlin eklentisini indirir. Studio eksik SDK parçalarını ("Android SDK Platform 36",
   "Build-Tools 35.0.0") kendisi önerir: **Install** de.
2. Telefonda *Geliştirici seçenekleri → USB hata ayıklama* açık olsun; ▶ **Run** ile yükle.
3. Komut satırı: `./gradlew assembleDebug` (APK: `app/build/outputs/apk/debug/`), `./gradlew installDebug` (bağlı cihaza yükler).
   JDK 17+ gerekir (Studio'nun gömülü JDK'sı yeter). `gradlew` çalışmazsa (sarmalayıcı jar'ı burada çalıştırılamadı) kurulu
   bir Gradle ile `gradle wrapper --gradle-version 8.13` komutu onu yeniden üretir; Studio de kendisi onarır.

Bir sürüm sayısı Studio'da "güncelle" diye çıkarsa sorun yok: tüm sürümler tek yerde — kök `build.gradle.kts` (AGP, Kotlin),
`gradle/wrapper/gradle-wrapper.properties` (Gradle).

## 2. Testler (Android gerekmez)

```
./run-tests.sh            # hepsi: çekirdek, çizim, model, arayüz, rastgele kullanım (~10 sn; derleme ~1 dk)
./run-tests.sh core ui    # seçili gruplar: core secure gfx model ui monkey
./run-tests.sh alloc      # kare başına bellek ayırma raporu
./run-tests.sh previews   # tasarım önizlemesi PNG'leri (previews/)
```

Gerekenler: JDK 17+ ve `kotlinc` (kotlinlang.org/docs/command-line.html). Testler gerçek UDP soketleri üzerinden sahte bir DNS
sunucusuyla yönlendiriciyi uçtan uca dener; arayüzü ise ekransız bir çizim yüzeyine çizip erişilebilirlik etiketleri üzerinden
"siyah kutu" olarak kullanır. `selftest/MonkeyTest.kt` yüzlerce rastgele dokunuş/ayar/boyut dizisiyle çökme arar.

`tools/catalog_check.py` katalogdaki her sunucuya **gerçek** bir DNS sorgusu (düz UDP ve DoT) gönderip adreslerin ve TLS
sertifika adlarının hâlâ doğru olduğunu kontrol eder (`python3 tools/catalog_check.py`). Katalog değiştikçe ve her yayından önce çalıştır. Araç, ağın DNS trafiğini yakalayıp
yakalamadığını da denetler ve yakalıyorsa sonuçlara güvenmemeni söyler (bu sandbox'ta tam böyleydi: AdGuard bile reklam alan
adını engellemedi; bu yüzden katalog adresleri **burada canlı doğrulanamadı**, sen kendi ağında çalıştır).

## 3. İlk cihaz denemesi kontrol listesi

Android kısmı hiç çalıştırılmadığı için ilk telefon denemesinde sırayla şunlara bak. Takılırsan `adb logcat` ve uygulamanın
**Ayarlar → Tanılama günlüğü** ekranı nedeni genellikle gösterir.

1. Uygulama açılıyor mu, arka plan gökyüzü rengi ve küre çiziliyor mu? (Açılmazsa: logcat'te `AndroidRuntime`.)
2. Küreye dokun → ilk açılış bilgilendirmesi → "Anladım, devam et" → Android'in VPN izni penceresi → "Tamam" → küre dolup **Bağlı**.
3. Gerçekten kullanılıyor mu? Tarayıcıda `https://one.one.one.one/help/` aç (Cloudflare seçiliyse "Connected to 1.1.1.1: Yes"
   demeli; şifreleme DoT ise "Using DNS over TLS (DoT): Yes", DoH ise "…HTTPS (DoH): Yes") ya da `dnsleaktest.com` ("Extended test")
   dene. Listeden başka bir sunucuya dokunup sonucun değiştiğini gör. Ayrıntı panelinde "Şifreli DNS: DoT" yazmalı.
4. Tarayıcıda *Güvenli DNS* (Chrome → Ayarlar → Gizlilik ve güvenlik → Güvenli DNS kullan) ve Android *Özel DNS* ayarının
   "Otomatik/Kapalı" olduğundan emin ol; yoksa sistem sorguları bu yönlendirmeyi atlayabilir (yardım sayfasında da yazar).
5. Bildirimdeki **Kes** düğmesi, hızlı ayarlar kutucuğu (düzenle → "DNS Hızlı"), uygulama simgesine uzun basınca çıkan kısayollar.
6. Wi-Fi ↔ mobil veri geçişi ve uçak modu aç/kapa: bağlantı toparlıyor mu? Ayarlardan "Wi-Fi sunucusu" farklı seçilince geçişte
   sunucu değişiyor mu?
7. Bağlıyken başka bir VPN uygulaması başlat: bu uygulama "sistem tarafından kapatıldı" göstermeli (ya da bağlanamazken
   "Başka bir VPN açık olabilir").
8. Ayarlar → "Telefon açılınca bağlan" açık + telefonu yeniden başlat; ayrıca uygulamayı **zorla durdur** ve sonra aç.
9. TalkBack açıkken ana ekranı ve ayarları gez: küre, satırlar, kaydırıcı ve anahtarlar okunuyor mu, kaydırma çalışıyor mu?
10. Pil/bellek: bir gün bağlı bırak; *Ayarlar → Pil → Uygulama kullanımı*. (Beklenti: boştayken neredeyse sıfır.)
11. Gece teması, tema geçişi animasyonu, düşük güç modu, büyük yazı boyutu (sistem ayarından) ve yatay/tablet görünümü.
13. 3. tur: Ayarlar → Gizlilik → "Bağlantıyı test et" (DNS çalışıyor, DNSSEC, reklam engelleme sonuçları); ağı kesip yeniden
    bağlan (5 sn / 30 sn / 2 dk'da dener); Şifrelemeyi "Sıkı" yap ve şifresiz bir ağda dene; bildirimde "15 dk duraklat" ve geri sayım;
    "Kesmeden önce sor"; telefonu yeniden başlat (açılışta bağlanma, ağ henüz hazır değilse yeniden denemeli); duvar kâğıdı teması
    (Android 12+); 4 yeni dilin her birinde ana ekran ve ayarlar taşmıyor mu?
12. Yeni olanlar: Ayarlar → Gizlilik → Şifreleme "DoH" ve "Kapalı" seçip yeniden bağlan (bildirimde protokol değişmeli);
    küreye uzun bas → Duraklat 5 dk (5 dk sonra kendiliğinden bağlanmalı, ekran kapalıyken de); ana ekrana widget ekle;
    Ayarlar → Dil → English; Android 13+'ta sistem Ayarlar → Uygulamalar → DNS Hızlı → Dil.

## 4. Google Play'e yayın

Aşağıdakiler Ekim 2026'da resmî Google belgelerinden okunarak yazıldı; kurallar değişebilir, Play Console'daki güncel metin esastır.
Hukuki tavsiye değildir.

1. **Uygulama kimliği.** `app/build.gradle.kts` üstündeki `val appId = "app.dnshizli"` satırını kendi benzersiz kimliğinle
   değiştir (ör. `com.seninadin.dnshizli`; `com.example.*` olmaz). Yalnızca bu satır değişir: kod paketi aynı kalır, kısayollar
   kimliği oradan alır. Yayından sonra kimlik **değiştirilemez**.
2. **Hedef API.** Play, 31 Ağustos 2026'dan beri yeni uygulama ve güncellemelerde **API 36** (Android 16) ister
   (uzatma talebi 1 Kasım 2026'ya kadar). Proje 36'ya ayarlı; her yıl bir üst sürüme çıkman gerekecek.
3. **İmza.** Bir kez yükleme anahtarı üret ve **yedekle** (kaybedersen güncelleme yayınlayamazsın):
   ```
   keytool -genkeypair -v -keystore dnshizli-upload.jks -alias upload -keyalg RSA -keysize 2048 -validity 10000
   ```
   `keystore.properties.example` dosyasını `keystore.properties` olarak kopyalayıp doldur (git'e girmez). Play Console'da
   *Play Uygulama İmzalama* açık kalsın.
4. **Paket.** `./gradlew bundleRelease` → `app/build/outputs/bundle/release/app-release.aab`. Her yüklemede `versionCode`'u artır
   (`app/build.gradle.kts`).
5. **Kişisel geliştirici hesabı** (13 Kasım 2023'ten sonra açıldıysa) üretime çıkmadan önce **en az 12 test kullanıcısıyla
   kesintisiz 14 gün kapalı test** ister; kuruluş hesaplarında bu şart yoktur.
6. **Play Console → Uygulama içeriği** (hepsi zorunlu):
   - **Gizlilik politikası** URL'si: `GIZLILIK.md` şablonunu doldurup herkese açık bir adrese koy (GitHub Pages, kendi siten…).
   - **Reklamlar:** Yok. **Uygulama erişimi:** kısıtlama yok. **Hedef kitle:** çocuklara yönelik değil. İçerik derecelendirme anketi.
   - **Veri güvenliği:** Uygulama kendi adına veri toplamaz/kaydetmez. Ancak DNS sorgularının (alan adları) kullanıcının seçtiği
     DNS sağlayıcısına gittiğini göz önünde bulundur; bunu forma ve politikaya açıkça yazmak en güvenli yoldur (formu nasıl
     işaretleyeceğin senin kararındır).
   - **VPN hizmeti beyanı.** Politika, `VpnService`'i yalnızca VPN'i çekirdek işlevi olan uygulamalara (ve sayılan istisnalara)
     izin verir, mağaza sayfasında kullanımını belgelemeni, uygulama içinde **açık bilgilendirme + onay** göstermeni (ilk açılış
     bilgilendirmesi bunu karşılar) ve formda kısa bir kullanım videosu (≤ 90 sn) ile bilgilendirme videosu vermeni ister. Bu
     uygulamada VPN çekirdek işlevdir: *yerel* VPN arayüzü yalnızca DNS yönlendirmek içindir. Politika ayrıca verinin
     **şifrelenmesini** ister: sorgular varsayılan olarak DoT/DoH ile şifreli gider; kullanıcı ayardan "Kapalı" seçerse düz
     DNS'e döner (ve bu uygulamada açıkça yazar). Formda durumu olduğu gibi anlat.
   - **Önplan hizmeti beyanı** (hedef Android 14+ olduğu için): hizmet türü `systemExempted`; Android belgeleri bunu VPN
     uygulamaları (Ayarlar → Ağ ve internet → VPN üzerinden kurulanlar) için açıkça izinli sayar. Formda tür/kullanım için
     VPN ile ilgili seçeneği seç; video: uygulamayı aç → küreye dokun → izni ver → bağlı + bildirim.
7. **Mağaza girişi:** `store/magaza_metni.md` (kısa/uzun açıklama; VPN kullanımı belgelendi). Görseller: `store/ikon-512.png`,
   `store/one-cikan-gorsel-1024x500.png`. Ekran görüntüsünü **kendi telefonundan** al; `previews/` içindekiler çizim önizlemesidir.

## 5. Bilinen sınırlar (dürüstçe)

- **Şifreli DNS** yalnızca bilgisayarda, yerel bir TLS sunucusuna karşı sınandı; gerçek sağlayıcılarla ilk cihaz denemesinde bak.
  Sunucu adı (DoT/DoH ana bilgisayarı) bir kez sağlayıcının **düz** DNS'iyle çözülür (korumalı soketle); bu tek sorgu şifresizdir.
  Şifre "Kapalı" iken bazı ağlar 53. porttaki trafiği kendi sunucusuna yönlendirebilir.
- **Uygulama ile sistem arasında DNS taşıma IPv4 üzerinden**; TCP/53 yok (büyük cevaplar artık IP parçalamayla iletilir). IPv6
  trafiği engellenmez. IPv6-only/DNS64 ağlarda herkese açık DNS'ler DNS64 yapmadığı için sorun olabilir.
- **Duraklatma alarmı yaklaşıktır** (Android 12+'da "kesin alarm" izni istenmez): servis çalışırken zamanlayıcı kendi içinde, süreç
  ölmüşse sistem alarmı devreder; ikisi de en fazla birkaç dakika kayabilir. Arka plandan servis başlatma reddedilirse
  kullanıcı bildirimle uygulamayı açmaya çağrılır.
- **Yeni derleme eski derlemenin üzerine kurulur** (sabit debug anahtarı `app/debug.keystore`), ama daha önce bu projenin ESKİ
  GitHub derlemesini yüklediysen (rastgele anahtarla imzalıydı) **bir kez** kaldırıp yeniden kurman gerekir. Bu anahtar herkese
  açıktır; Play Store için kendi `keystore.properties` anahtarını kullan.
- **Tek VPN kuralı.** Android aynı anda yalnızca bir VPN'e izin verir; başka VPN açıkken bu uygulama bağlanamaz.
- **Atlayanlar.** Android'in belirli bir sunucu adına sabitlenmiş "Özel DNS"i (uygulama bağlanırken bunu fark edip uyarır) ve
  tarayıcıların/uygulamaların kendi DoH'u bu yönlendirmeyi kullanmaz.
- **Engellenen sayacı tahminidir**: yalnızca 0.0.0.0/:: ya da standart "engellendi" kodlu cevaplar sayılır; bazı sağlayıcılar
  başka biçimde engeller.
- **Önbellek boyutu** bağlanırken uygulanır (ayarı değiştirince bir sonraki bağlantıda). **Ağ kuralı** ağ değişiminde ve kullanıcı
  sunucu seçmeden başlayan bağlanmalarda (açılış, kutucuk) uygulanır; listeden elle seçim her zaman kuralın önündedir.
- **Diller:** 12 dil elle çevrildi ama anadili konuşanlarca gözden geçirilmedi. **Klavye/harici donanımla gezinti** yok (dokunmatik + TalkBack var).
- Çizim kodu sistemin standart bileşenlerini kullanmadığı için metin seçme gibi sistem davranışları yoktur; bunun karşılığı
  küçük bellek ve tam tasarım kontrolüdür.

## 6. Geliştirme önerileri (öncelik sırasıyla)

1. **Gerçek cihazda deneme** (bölüm 3) ve çevirileri anadili konuşanlara okutmak.
2. **Başka diller** (Arapça/Farsça için sağdan sola düzen gerekir; Japonca/Çince için yazı tipi denetimi).
3. **TCP/53 ve IPv6 DNS adresi.** Tünelde küçük bir TCP durum makinesi ve IPv6 adres/yol; "çok büyük cevap" ve DNS64 sınırlarını kaldırır.
4. **Uygulama bazlı ayırma** (`addAllowedApplication` / `addDisallowedApplication`): hangi uygulamalar bu DNS'i kullansın.
   Güçlü ama arayüzü karmaşıklaştırır; "karmaşıklıktan kaçın" ilkesine karşı bir seçimdir.
5. **Katalog güncelleme.** Sunucu listesini uygulama güncellemesi beklemeden yenilemek (imzalı küçük bir JSON). Ağ erişimi ve
   gizlilik dengesi getirir; basit tutmak için her sürümde `tools/catalog_check.py` çalıştırıp katalogu elle güncellemek yeterli olabilir.
6. **Gelir (isteğe bağlı).** Reklamsız kalıp tek seferlik "destekçi" satın alımı (Play Billing; bir bağımlılık ekler) en uyumlu yol.
   Gelir beklentisi vaat etmiyorum; bu tür araçlarda indirme/gelir genellikle düşüktür.
7. **Cihaz testi otomasyonu.** Tüneli, bildirimi ve karoyu enstrümantasyon testleriyle (UiAutomator) sına; yayından önce Play
   Console'un *ön başlatma raporu* ve gerçek cihazlarla elle deneme.
8. **AGP 9 / yerleşik Kotlin göçü.** AGP 8.13 son 8.x hattıdır; Studio "yükselt" önerdiğinde resmî göç kılavuzunu izle.
9. **DoH/3 (QUIC) ve ODoH**: bağımlılık gerektirir; şimdilik DoT/DoH yeterli.

## 7. Nasıl genişletilir?

| İstek | Nerede |
|-------|--------|
| Sunucu ekle/çıkar | `core/Catalog.kt` (tek satır); sonra `python3 tools/catalog_check.py` |
| Tema ekle | `core/Themes.kt` (`Themes.all` listesine bir `AeroTheme`); kontrast testleri otomatik kapsar |
| Yeni ayar | `core/Settings.kt` (alan + kodlama) → `ui/Pages.kt` `SettingsPage` (bir satır) → metin: `core/Lang.kt` (`Tx`) + her `core/Tx*.kt` |
| Metin | `core/Lang.kt` içinde `Tx`'e soyut alan ekle; derleyici her dilde doldurmanı ister |
| Yeni dil | `core/Lang.kt` `Lang` listesine ekle, `TxEn.kt`'yi kopyalayıp çevir, `res/values-xx/strings.xml`, `res/xml/locales_config.xml` |
| Servis davranışı | `platform/DnsVpnService.kt` (komutlar tek iş parçacığında sıralanır; iptal numarası `cancelGen`) |
| Yeni ekran | `ui/Pages.kt` (`ScrollPage`'den türet) + `Scene.kt` gezinti |

## 8. Klasör yapısı

```
app/src/main/java/app/dnshizli/
  core/       saf Kotlin: DNS yönlendirici/önbellek/paketler, katalog, ayarlar, tema ve çizim sanatı (Android'siz test edilir)
  ui/         saf Kotlin: model + sahne (kendi çizdiğimiz arayüz: düğümler, sayfalar, paneller, animasyon)
  platform/   Android: VpnService + tünel, ağ izleme, bildirim, hızlı ayar karosu, açılış alıcısı, etkinlik, SceneView/AndroidGfx
app/src/main/res/   simgeler (uyarlanabilir + temalı), tema, kısayollar
selftest/     JVM testleri (1296 kontrol), önizleme üreticisi
tools/        catalog_check.py (canlı katalog doğrulaması)
store/        Play mağaza metni ve görselleri
previews/     tasarım önizlemeleri (PNG)
```

Mimari neden böyle: arayüz Android bileşenleriyle değil, kendi küçük sahne kitaplığımızla çizilir. Böylece (1) AndroidX/Compose
bağımlılığı olmaz, APK ve bellek küçük kalır, (2) tüm arayüz mantığı bilgisayarda ekransız test edilebilir, (3) Frutiger Aero cam/sıvı
görünüm her cihazda birebir aynıdır. Bedeli: erişilebilirlik ve geri hareketi gibi sistem entegrasyonlarını biz bağlarız (bağlandı,
ama cihazda sınanmadı — yukarıdaki kontrol listesi).

## 9. Sorun giderme

- *Senkronizasyon "Could not resolve …"*: internet/proxy ayarını kontrol et; ilk senkronizasyon indirme yapar.
- *"SDK location not found"*: Studio `local.properties` oluşturur; komut satırında `ANDROID_HOME` ayarla.
- *"Failed to find Build Tools / Platform 36"*: SDK Manager'dan yükle.
- *Bağlanamadı: "VPN arayüzü açılamadı"*: başka bir VPN açık ya da sistem izni geri alınmış; Ayarlar → "Sistem VPN ayarları".
- *Bağlı görünüyor ama etkisi yok*: yukarıda "Atlayanlar"a bak (Özel DNS / tarayıcı güvenli DNS).
