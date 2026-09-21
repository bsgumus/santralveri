# elektrikverileri.com — Power Plant Generation Dashboard

Plant-level hourly electricity generation from the **EPİAŞ Transparency Platform**: search,
charts, capacity factors and Excel/CSV downloads. A single-page, open-source tool.

🔗 **[elektrikverileri.com](https://elektrikverileri.com)** · English / Turkish interface

> 🇹🇷 **Bu belgenin Türkçesi için: [aşağıya bakın](#turkce).**

---

## 🔒 Privacy: your data is not stored

**In short:** this site has no user database. You do not create an account here, and we do not
keep one. Your EPİAŞ email address and password are **never stored, saved or logged** anywhere;
they are passed to EPİAŞ's own server at the moment you sign in, and nowhere else. No cookies,
no analytics, no third-party services.

| Data | Where it lives | For how long |
|---|---|---|
| Your EPİAŞ email | Your browser tab's memory (+ passed to EPİAŞ at sign-in) | Until you close the tab |
| Your EPİAŞ password | Your browser tab's memory (+ passed to EPİAŞ at sign-in) | Until you close the tab |
| Session key (TGT) | Your browser tab's memory | ~50 minutes |
| Language/theme preference | Your own browser (localStorage) | Until you clear it |
| Plant list & generation summaries | Your own browser (local cache) | Until you clear it |
| **Nothing on any server** | **—** | **—** |

Nothing is kept on any server: no database, no log files. What stays in your browser
(preferences and cache) lives on your device only and is never transmitted. Nothing is shared
or sold.

---

## How it works

```
   Your browser              Cloudflare Worker                EPİAŞ
 ┌──────────────┐   HTTPS   ┌──────────────────┐  HTTPS  ┌──────────────┐
 │ elektrik-    │ ────────▶ │   relay server   │ ──────▶ │ Transparency │
 │ verileri.com │ ◀──────── │ (forwards only)  │ ◀────── │  Platform    │
 └──────────────┘           └──────────────────┘         └──────────────┘
   ▲ all computation          ▲ stores nothing             ▲ source of data
     happens here               (no storage at all)          (your account)
```

**Why is there a relay at all?** Browsers cannot call the EPİAŞ API directly because of
cross-origin (CORS) restrictions. So a very small Cloudflare Worker sits in between and does
nothing but forward the request. Its complete source is in this repository:
[`worker.js`](worker.js) — 122 lines, no dependencies.

**Whose data is it?** Yours. Everyone queries with their own EPİAŞ Transparency account and
their own quota. There is no shared service account, which is precisely why we have no need to
keep your credentials.

---

## What happens to your credentials, step by step

1. You type your email and password into the sign-in screen — they stay **in the page's memory**.
2. Your browser sends them **over HTTPS** to the relay server.
3. The relay passes them **straight on to the EPİAŞ login server** (`giris.epias.com.tr`).
   It **does not write, log or cache them** anywhere.
4. EPİAŞ returns a **session key (TGT)**; the relay hands it back to your browser and forgets it.
5. Every subsequent query uses that session key — your password is never sent again.
6. Closing the tab, or pressing "Sign out", clears everything held in memory.

> **Why is the password kept in tab memory?** The EPİAŞ session key expires after about 50
> minutes. Keeping the password in the tab's memory lets the session renew itself silently if it
> expires mid-query. It is **never written to disk** (no localStorage, no cookies); closing or
> reloading the tab discards it.

---

## What is stored in your browser

Four things, all of which stay **on your device** and are never transmitted. None of them
contain personal information:

| Key | Contents | Purpose |
|---|---|---|
| `evLang` | `tr` or `en` | Your language preference |
| `evTheme` | `light` or `dark` | Your theme preference |
| `evPlants` | Power plant list (public data) | Avoids re-fetching on every visit; refreshed daily |
| `evCf:<plantId>:<year>` | Annual generation totals (public data) | Speeds up the capacity factor table |

**To clear them:** use "delete site data" for this site in your browser settings. On a shared
computer, we recommend using a private/incognito window.

**No cookies are used** — which is also why there is no cookie consent banner.

---

## What the relay server does and does not do

The entire source of `worker.js` is in this repository and can be read line by line.

**It does:**
- `/login` → forwards your email and password to EPİAŞ and returns the session key to your browser.
- `/epias` → forwards requests to exactly **two** EPİAŞ endpoints (plant list, real-time
  generation). Everything else is refused (allowlist).
- Sets `Cache-Control: no-store` on responses.
- Answers only requests coming from this project's own pages.

**It does not:**
- Have **any storage binding at all** — no database, no files, no KV, no R2. There is nowhere
  for it to write.
- **Log** emails, passwords, session keys or query contents.
- Keep user profiles, cookies, sessions or analytics.

**One honest exception — transient processing:** to slow down password-guessing attacks,
`/login` keeps an **IP address and a counter** in the worker's volatile memory for **up to 60
seconds** (to block more than 8 attempts per minute). It is never written to disk, it disappears
by itself, and it is used for nothing else.

---

## Don't take our word for it — verify

1. **All source code is here.** The site is a single file ([`index.html`](index.html)); the
   relay is 122 lines ([`worker.js`](worker.js)). No build step, no dependencies, nothing hidden.
2. **Watch the network traffic.** Press `F12` → **Network**. You will see every request the page
   makes: `licenses.json` (served from this site itself) plus `/login` and `/epias` on the relay.
   **No request goes anywhere else.**
3. **Inspect the storage.** `F12` → **Application** → **Local Storage**: you will find only the
   four keys listed above — no email, no password.
4. **The browser already enforces it.** The page ships a Content Security Policy whose
   `connect-src` permits only our own origin and the relay. Even if the code had a flaw, data
   **technically could not be sent** to any other address.
5. **No third-party code.** Not a single external script, font or stylesheet is loaded. No ad
   network, no Google Analytics, no pixels, no beacons, no error-reporting service.
6. **Self-host it.** You are free to clone this repository and run it on your own domain with
   your own relay; setup steps are in [`KURULUM.md`](KURULUM.md) (in Turkish). A local stand-in
   for the relay is included for development ([`yerel-test-proxy.py`](yerel-test-proxy.py)).

---

## Transparency about infrastructure providers

There is one data-processing point outside our control: the site is hosted on **GitHub Pages**
and the relay runs on **Cloudflare Workers**. Both companies may keep standard connection logs
(IP address, timestamp, requested URL) as part of their infrastructure. We have no access to
those logs, they are not part of this project's code, and they are governed by those companies'
own privacy policies.

Likewise, EPİAŞ may record the queries made with your own account in their systems — exactly as
they would if you used the EPİAŞ website directly.

---

## Security measures

- **HTTPS end to end** — enforced on both the site and the relay. The site's certificate comes
  from Let's Encrypt (renewed automatically by GitHub Pages); the relay's is managed by Cloudflare.
- **Content Security Policy** — the page may only talk to its own origin and the relay.
- **Endpoint allowlist** — the relay forwards to two EPİAŞ endpoints only.
- **Origin allowlist** — the relay answers only this project's pages.
- **Rate limiting** — 8 sign-in attempts per minute, to slow brute-force attempts.
- **Show/hide password** — so you can check what is actually in the field.
- **No storage** — data that is never kept cannot leak.

If you find a security issue, please open an **issue** in this repository (a note that you have
found something is enough to start a private conversation).

---

## Data source and disclaimer

**DATA: EPİAŞ TRANSPARENCY PLATFORM** | All visuals, charts and data presented here are for
information purposes only. Neither Enerji Piyasaları İşletme A.Ş. nor
https://elektrikverileri.com accepts any legal liability for any loss or damage that may arise
from the said data. All content and data presented may be reproduced and used provided that the
source is cited.

Capacity figures come from the EPDK licence list. This is **not an official EPİAŞ/EPDK website**.
The generation figures shown are "real-time generation" data and are not the basis for settlement.

---

## Files in this repository

| File | Description |
|---|---|
| `index.html` | The entire site (single file, no dependencies) |
| `worker.js` | Cloudflare Worker — source of the relay server |
| `licenses.json` | Capacity table generated from the EPDK licence list |
| `KURULUM.md` | Setup guide from scratch, in Turkish (Cloudflare + GitHub Pages + DNS) |
| `yerel-test-proxy.py` | Python stand-in for the relay, for local development |
| `CNAME` | GitHub Pages custom domain setting |

---
---

<a id="turkce"></a>

# 🇹🇷 Türkçe

## elektrikverileri.com — Santral Üretim Paneli

EPİAŞ Şeffaflık Platformu verisiyle **santral bazlı saatlik elektrik üretimi**: arama, grafik,
kapasite faktörü ve Excel/CSV indirme. Tek sayfalık, açık kaynak bir araç.

🔗 **[elektrikverileri.com](https://elektrikverileri.com)** · İngilizce / Türkçe arayüz

## 🔒 Gizlilik: verileriniz saklanmıyor

**Özet:** Bu sitenin bir kullanıcı veritabanı yok. Hesap oluşturmuyorsunuz, biz hesap
tutmuyoruz. EPİAŞ e-posta ve şifreniz **hiçbir yerde saklanmıyor, kaydedilmiyor, loglanmıyor**;
yalnızca giriş anında EPİAŞ'ın kendi sunucusuna iletiliyor. Çerez kullanılmıyor, izleme/analitik
kodu yok, üçüncü taraf hiçbir servise veri gönderilmiyor.

| Veri | Nerede | Ne kadar süreyle |
|---|---|---|
| EPİAŞ e-postanız | Tarayıcı sekmenizin belleği (+ giriş anında EPİAŞ'a iletim) | Sekmeyi kapatana kadar |
| EPİAŞ şifreniz | Tarayıcı sekmenizin belleği (+ giriş anında EPİAŞ'a iletim) | Sekmeyi kapatana kadar |
| Oturum anahtarı (TGT) | Tarayıcı sekmenizin belleği | ~50 dakika |
| Dil/tema tercihiniz | Kendi tarayıcınız (localStorage) | Siz silene kadar |
| Santral listesi & üretim özetleri | Kendi tarayıcınız (localStorage önbelleği) | Siz silene kadar |
| **Sunucu tarafında hiçbiri** | **—** | **—** |

Sunucu tarafında hiçbir veri tutulmuyor: veritabanı yok, kayıt dosyası yok. Tarayıcınızda
kalanlar (tercihler ve önbellek) yalnızca sizin cihazınızdadır ve hiçbir yere gönderilmez.
Hiçbir veri kimseyle paylaşılmıyor, satılmıyor.

## Nasıl çalışıyor?

```
   Tarayıcınız                Cloudflare Worker              EPİAŞ
 ┌──────────────┐   HTTPS   ┌──────────────────┐  HTTPS  ┌──────────────┐
 │ elektrik-    │ ────────▶ │  aracı sunucu    │ ──────▶ │ Şeffaflık    │
 │ verileri.com │ ◀──────── │ (yalnızca iletir)│ ◀────── │ Platformu    │
 └──────────────┘           └──────────────────┘         └──────────────┘
   ▲ tüm hesaplama            ▲ hiçbir şey saklamaz        ▲ verinin kaynağı
     burada yapılır             (depolama yok)                (sizin hesabınız)
```

**Neden bir aracı sunucu var?** Tarayıcılar güvenlik kuralları (CORS) yüzünden EPİAŞ API'sine
doğrudan istek atamıyor. Bu yüzden araya, isteği yalnızca ileten çok küçük bir Cloudflare
Worker konuldu. Worker'ın kaynak kodunun tamamı bu depoda: [`worker.js`](worker.js) — 122
satır, bağımlılığı yok.

**Veriyi kim çekiyor?** Siz. Herkes kendi EPİAŞ Şeffaflık hesabıyla, kendi kotasıyla sorgu
yapar. Bizim bir "servis hesabımız" yok; bu yüzden hesabınızın bilgilerine ihtiyacımız da yok.

## Giriş bilgilerinizin yolculuğu (adım adım)

1. E-posta ve şifrenizi giriş ekranına yazarsınız — bu bilgiler **sayfanın belleğinde** kalır.
2. Tarayıcınız bunları **HTTPS ile** aracı sunucuya gönderir.
3. Aracı sunucu bilgileri **aynen EPİAŞ'ın giriş sunucusuna** (`giris.epias.com.tr`) iletir.
   Kendi tarafında **hiçbir yere yazmaz, loglamaz, önbelleğe almaz**.
4. EPİAŞ bir **oturum anahtarı (TGT)** döner; aracı sunucu bunu tarayıcınıza geçirir ve
   kendi belleğinden düşer.
5. Sonraki tüm veri sorguları bu oturum anahtarıyla yapılır — şifreniz bir daha gönderilmez.
6. Sekmeyi kapattığınızda veya "Çıkış"a bastığınızda bellekteki her şey silinir.

> **Şifreniz neden sekme belleğinde tutuluyor?** EPİAŞ oturum anahtarının ömrü ~50 dakika.
> Uzun bir sorgu sırasında süre dolarsa oturumun sessizce yenilenebilmesi için şifre sekme
> belleğinde tutulur. **Diske yazılmaz** (localStorage/çerez kullanılmaz); sekmeyi kapatmak
> veya sayfayı yenilemek onu siler.

## Tarayıcınızda ne saklanıyor?

Yalnızca **sizin cihazınızda** kalan, kimseye gönderilmeyen dört şey var. Hiçbiri kişisel
bilgi (e-posta/şifre) içermez:

| Anahtar | İçerik | Amaç |
|---|---|---|
| `evLang` | `tr` veya `en` | Dil tercihiniz |
| `evTheme` | `light` veya `dark` | Tema tercihiniz |
| `evPlants` | Santral listesi (kamuya açık veri) | Her açılışta yeniden çekmemek için, günlük |
| `evCf:<santralId>:<yıl>` | Yıllık üretim toplamı (kamuya açık veri) | Kapasite faktörü tablosunu hızlandırmak için |

**Silmek isterseniz:** Tarayıcı ayarlarından bu site için "site verilerini sil" demeniz
yeterli. Ortak kullanılan bir bilgisayardaysanız gizli/özel pencerede kullanmanız önerilir.

**Çerez (cookie) kullanılmıyor.** Bu yüzden çerez onay penceresi de yok.

## Aracı sunucu (Cloudflare Worker) ne yapıyor, ne yapmıyor?

`worker.js` dosyasının tamamı bu depoda, satır satır okunabilir.

**Yapar:**
- `/login` → e-posta ve şifreyi EPİAŞ'a iletir, dönen oturum anahtarını tarayıcınıza verir.
- `/epias` → yalnızca **iki** EPİAŞ ucuna istek iletir: santral listesi ve gerçek zamanlı üretim.
  Başka hiçbir uca izin verilmez (beyaz liste).
- Yanıtlara `Cache-Control: no-store` başlığı ekler (ara sunucularda önbelleklenmesin diye).
- Yalnızca `elektrikverileri.com` / `.org` sayfalarından gelen isteklere yanıt verir.

**Yapmaz:**
- Veritabanı, dosya, KV, R2 veya başka **hiçbir depolama bağlantısı yoktur** — yazacak bir
  yeri yok.
- E-posta, şifre, oturum anahtarı veya sorgu içeriklerini **loglamaz**.
- Kullanıcı profili, çerez, oturum kaydı, analitik tutmaz.

**Dürüst not — geçici olarak işlenen tek şey:** Şifre deneme saldırılarını yavaşlatmak için,
`/login` isteklerinde **IP adresi + sayaç** worker'ın geçici belleğinde **en fazla 60 saniye**
tutulur (dakikada 8 denemeden fazlasını engellemek için). Diske yazılmaz, 60 saniye sonra
kendiliğinden düşer, başka hiçbir amaçla kullanılmaz.

## Doğrulayın — bize güvenmek zorunda değilsiniz

Bu iddiaları kendiniz denetleyebilirsiniz:

1. **Kaynak kodun tamamı burada.** Sitenin kendisi tek dosya ([`index.html`](index.html)),
   aracı sunucu 122 satır ([`worker.js`](worker.js)). Derleme adımı, paket bağımlılığı,
   gizli kod yok.
2. **Ağ trafiğini izleyin.** Tarayıcıda `F12` → **Network** sekmesi. Sayfanın yaptığı tüm
   istekler görünür: `licenses.json` (sitenin kendi dosyası) ve aracı sunucuya giden `/login`
   ile `/epias`. **Başka hiçbir adrese istek gitmez.**
3. **Depolamayı görün.** `F12` → **Application** → **Local Storage**: yukarıdaki dört anahtar
   dışında bir şey bulamazsınız; e-posta/şifre orada yoktur.
4. **Tarayıcı zaten engelliyor.** Sayfada bir Content Security Policy (CSP) tanımlı:
   `connect-src` yalnızca kendi sunucumuza ve aracı sunucuya izin verir. Yani kodda bir açık
   olsa bile, veriler **başka bir adrese teknik olarak gönderilemez**.
5. **Üçüncü taraf kod yok.** Sayfada dışarıdan yüklenen tek bir script, font veya stil dosyası
   yoktur; reklam ağı, Google Analytics, piksel, beacon, hata izleme servisi yoktur.
6. **Kendiniz barındırın.** İsterseniz bu depoyu kopyalayıp kendi alan adınızda ve kendi
   aracı sunucunuzla çalıştırabilirsiniz; kurulum adımları [`KURULUM.md`](KURULUM.md) dosyasında.
   Geliştirme için aracı sunucunun yerel bir taklidi de depoda
   ([`yerel-test-proxy.py`](yerel-test-proxy.py)).

## Barındırma sağlayıcıları hakkında şeffaflık

Sitenin bizim kontrolümüzde olmayan, altyapıdan kaynaklanan tek veri işleme noktası şudur:
Site **GitHub Pages** üzerinde barındırılıyor, aracı sunucu **Cloudflare Workers** üzerinde
çalışıyor. Her iki şirket de kendi altyapılarında standart bağlantı kayıtları (IP adresi,
tarih, istenen adres gibi) tutabilir. Bu kayıtlara bizim erişimimiz yoktur ve bunlar bu
projenin kodunun bir parçası değildir; ilgili şirketlerin kendi gizlilik politikalarına tabidir.

Aynı şekilde EPİAŞ, kendi hesabınızla yaptığınız sorguları kendi sistemlerinde kaydedebilir —
tıpkı EPİAŞ'ın sitesine doğrudan girdiğinizdeki gibi.

## Güvenlik önlemleri

- **Uçtan uca HTTPS** — hem sitede hem aracı sunucuda zorunlu. Site sertifikası Let's Encrypt
  (GitHub Pages tarafından otomatik yenilenir), aracı sunucununki Cloudflare tarafından yönetilir.
- **Content Security Policy** — sayfa yalnızca kendi kaynağı ve aracı sunucu ile konuşabilir.
- **Uç beyaz listesi** — aracı sunucu yalnızca iki EPİAŞ ucuna istek iletir.
- **Origin beyaz listesi** — aracı sunucu yalnızca bu projenin sayfalarına yanıt verir.
- **Hız sınırı** — `/login` için dakikada 8 deneme (kaba kuvvet denemelerini yavaşlatır).
- **Şifre göster/gizle** — alandaki değeri görüp doğrulayabilmeniz için.
- **Depolama yok** — saklanmayan veri, sızdırılamaz.

Bir güvenlik açığı fark ederseniz lütfen depoda bir **issue** açın (açığın ayrıntısını
paylaşmadan önce iletişime geçmeniz yeterlidir).

## Veri kaynağı ve sorumluluk reddi

**VERİ: EPİAŞ ŞEFFAFLIK PLATFORMU** | Burada sunulan tüm görsel, grafik ve veriler
bilgilendirme amaçlı olup söz konusu verilerden kaynaklanabilecek kayıp ve zararlardan dolayı
Enerji Piyasaları İşletme A.Ş. veya https://elektrikverileri.com hukuken sorumluluk kabul
etmez. Sunulan tüm içerik ve veriler kaynak gösterilmek suretiyle çoğaltılabilir ve
kullanılabilir.

Kapasite bilgileri EPDK lisans listesinden alınmıştır. Bu site **resmî bir EPİAŞ/EPDK sitesi
değildir**. Gösterilen üretim verisi "gerçek zamanlı üretim" verisidir; uzlaştırmaya esas
değildir.

## Depodaki dosyalar

| Dosya | Açıklama |
|---|---|
| `index.html` | Sitenin tamamı (tek dosya, bağımlılıksız) |
| `worker.js` | Cloudflare Worker — aracı sunucu kaynak kodu |
| `licenses.json` | EPDK lisans listesinden üretilen kapasite tablosu |
| `KURULUM.md` | Sıfırdan kurulum rehberi (Cloudflare + GitHub Pages + DNS) |
| `yerel-test-proxy.py` | Yerel geliştirme için aracı sunucunun Python taklidi |
| `CNAME` | GitHub Pages alan adı ayarı |
