<a id="top"></a>
<div align="center">

<img src="docs/banner.svg" alt="V2RayEz — DPI Bypass & CDN Toolkit" width="100%">

<br>

[![Version](https://img.shields.io/badge/version-4.4.5-2dd4bf?style=for-the-badge)](#)
[![Go](https://img.shields.io/badge/Go-1.22+-00ADD8?style=for-the-badge&logo=go&logoColor=white)](https://go.dev)
[![Platform](https://img.shields.io/badge/Windows%20·%20macOS%20·%20Linux-1c2330?style=for-the-badge)](#)
[![Telegram](https://img.shields.io/badge/@EzAccess1-229ED9?style=for-the-badge&logo=telegram&logoColor=white)](https://t.me/EzAccess1)

**A single-binary local web panel to beat DPI filtering and work with CDN-fronted proxies.**

**🌐 Language / زبان —** [**English**](#english) · [**فارسی**](#persian)

</div>

> [!WARNING]
> **For education, testing, and research only.** You are responsible for obeying your
> local laws and the terms of any service you use it with (Google, Cloudflare, …).
> Provided **as is**, with no warranty.

---

<a id="english"></a>

## 🇬🇧 English

V2RayEz is a **single-binary local web control panel**. You run one executable, a clean
dashboard opens in your browser (or its own app window), and *everything* — an
SNI-spoofing tunnel, xray/sing-box engines, scanners, a config library, and a
Google-fronted relay — lives behind that one page. No installer, no hidden services,
no telemetry. It's Go with an embedded UI, so the whole app is one portable file.

<div align="center">
<img src="docs/screenshots/dashboard.png" alt="V2RayEz dashboard" width="92%">
</div>

### ✨ Features

| | Feature | What it does |
|---|---|---|
| ≋ | **SNI Tunnel** | Local TCP proxy that does the TLS handshake with a *fake* SNI while connecting to the real endpoint, with optional DPI-desync (fragmentation, fake packets, uTLS fingerprints). |
| ⚙ | **XRay Core** | Detect/download **xray** & **sing-box**; run any config as a local **SOCKS5** proxy or a system-wide **TUN VPN**. Start/stop, flip TUN, and **Set/Clear the system proxy** from the header. |
| ☷ | **Config Library** | Groups, subscriptions, the built-in SNI/Spoof list, paste & drag-drop, a **full structured v2ray editor**, QR sharing, **bulk relay + speed tests**, right-click menu, shift/multi-select — all auto-saved. |
| ◎ | **Scanners** | SNI scan, Mass SNI, **Clean IP Scanner**, CDN Edge test, CDN Configs builder, **Mass URI** tester, and a rebuilt **Site Scanner** (live progress, Cloudflare-first, Connect/Save). |
| ◈ | **Google Tunnel (Fastly)** | Domain-fronted relay through a **Google Apps Script** + **Cloudflare Worker** you deploy. Both scripts are generated in-app — no external repo involved. |
| ☁ | **Extras** | Cloudflare Worker maker, WinDivert management, Psiphon & SPlus tunnels. |
| 🎨 | **Modern UI** | Light & dark themes, readable fonts, fully fluid layout (phone → ultra-wide), English & Persian. |

### 🚀 Quick start

```bash
# run a release build
./v2rayez            # Linux / macOS
v2rayez.exe          # Windows
```

Listens on `0.0.0.0:8765` by default and opens the dashboard. If it doesn't, visit
**http://127.0.0.1:8765**.

<details>
<summary><b>Build from source</b> (Go 1.22+)</summary>

```bash
git clone <your-fork-url> v2rayez
cd v2rayez
go build -o v2rayez .
./v2rayez

# cross-compile
GOOS=windows GOARCH=amd64 go build -o v2rayez.exe .
GOOS=darwin  GOARCH=arm64 go build -o v2rayez-mac .
GOOS=linux   GOARCH=amd64 go build -o v2rayez .

# optional transports behind build tags
go build -tags "livekit psiphon" -o v2rayez .
```
</details>

<details>
<summary><b>Command-line flags</b></summary>

| Flag | Default | Meaning |
| --- | --- | --- |
| `-addr` | `0.0.0.0:8765` | Address the panel listens on |
| `-open` | `true` | Open the dashboard on start |
| `-window` | `true` | Use a dedicated app window if available |
| `-minimize` | `true` | Minimize the console window on Windows |

```bash
./v2rayez -addr 127.0.0.1:8765 -open=false -window=false   # local only
```
</details>

### 🧭 How it works

<div align="center">
<img src="docs/architecture.svg" alt="V2RayEz architecture" width="96%">
</div>

Most basic DPI blocks by reading the **SNI** sent in the clear during the TLS
handshake. V2RayEz connects to the real server but writes a *permitted* hostname into
that field, so the filter sees an allowed domain while your real session continues.
The **Google Tunnel** goes further: your traffic is domain-fronted to a Google edge,
relayed by a Google Apps Script you deploy, and fetched by your Cloudflare Worker — so
the network only ever sees ordinary Google traffic.

### 📚 Using the tabs

<details>
<summary><b>Config Library</b> (default tab)</summary>

- Make **groups**; add configs by pasting `vless:// vmess:// trojan:// ss://`, dragging a
  `.txt`, importing a **subscription URL**, **Load SNI Configs**, or **Get latest**.
- **Connect** (green) runs a config through the engine chosen in XRay Core.
- **Right-click** a row or selection → Connect / Test / Speed test / Share / Edit / Copy /
  Delete. `Ctrl/⌘-click` + `Shift-click` to multi-select, `Ctrl/⌘-A` all, `Enter` connects.
- **Edit** opens a structured editor (address, port, UUID/password, cipher, transport,
  path, host header, security, SNI, fingerprint, ALPN, Reality keys) with a live preview.
</details>

<details>
<summary><b>XRay Core · SNI Tunnel · Google Tunnel</b></summary>

- **XRay Core** — detect/download engines, connect a config, pick **xray (SOCKS5)** or
  **sing-box (TUN VPN)**, Start. TUN / Start-Stop / system-proxy controls live in the header.
- **SNI Tunnel** — listen port + target + one or more **fake SNI**, a mode and optional
  desync, plus relay test, single-config test, and your **Saved SNI list**.
- **Google Tunnel (Fastly)** — ① generate `worker.js` + `Code.gs` in-app, ② deploy them to
  Cloudflare & Google Apps Script and copy the Deployment ID, ③ paste it + auth key, pick a
  **front preset**, Start, ④ point your proxy at `127.0.0.1:8086` and install the CA
  (**.pem** or **.crt** for one-click Windows install).
</details>

### ⚠️ Notes & limits

- **TUN** and **WinDivert** need administrator / root.
- **System proxy** uses native tooling per OS (Windows registry + WinINET, macOS
  `networksetup`, GNOME `gsettings`).
- The **Google Tunnel** relays request/response only (no WebSocket/streaming) and needs a
  relay *you* deploy; "Fastly" fronting only works with a Fastly-hosted relay — the Google
  presets are the supported path.

### 🙌 Credits

Built by **MacanDev** · [@EzAccess1](https://t.me/EzAccess1). The Google Tunnel concept is
inspired by the MasterHttpRelay / mhr-cfw approach; V2RayEz ships its own clean-room Go
implementation and its own generated scripts.

<div align="right"><a href="#top">↑ back to top</a></div>

---

<a id="persian"></a>

<div dir="rtl" align="right">

## 🇮🇷 فارسی

V2RayEz یک **پنل کنترل وب محلیِ تک‌فایلی** است. یک فایل اجرایی را اجرا می‌کنید، یک داشبورد
تمیز در مرورگر (یا پنجرهٔ اختصاصی خودش) باز می‌شود و *همه‌چیز* — تونل اسپوف SNI، موتورهای
xray/sing-box، اسکنرها، کتابخانهٔ کانفیگ و رلهٔ فرانت‌شده با گوگل — پشت همان یک صفحه است.
بدون نصب‌کننده، بدون سرویس مخفی و بدون تله‌متری. با Go و رابط تعبیه‌شده نوشته شده، پس کل
برنامه یک فایل قابل‌حمل است.

<div align="center">
<img src="docs/screenshots/dashboard.png" alt=" داشبورد V2RayEz" width="92%">
</div>

### ✨ ویژگی‌ها

| | ویژگی | توضیح |
|---|---|---|
| ≋ | **تونل SNI** | پروکسی TCP محلی که دست‌دادن TLS را با SNI *جعلی* انجام می‌دهد در حالی که به مقصد واقعی وصل می‌شود، با ترفندهای اختیاری ضد‌DPI (تکه‌تکه‌کردن، پکت جعلی، اثرانگشت uTLS). |
| ⚙ | **XRay Core** | یافتن/دانلود **xray** و **sing-box**؛ اجرای هر کانفیگ به‌صورت پروکسی **SOCKS5** محلی یا **VPN سراسری TUN**. شروع/توقف، تغییر به TUN و **تنظیم/پاک‌کردن پروکسی سیستم** از نوار بالا. |
| ☷ | **کتابخانهٔ کانفیگ** | گروه‌ها، اشتراک‌ها، لیست SNI/اسپوف داخلی، چسباندن و کشیدن‌ورها، **ویرایشگر کامل و ساختاریافتهٔ v2ray**، اشتراک QR، **تست‌های دسته‌ای رله + سرعت**، منوی راست‌کلیک و انتخاب چندتایی — همه خودکار ذخیره می‌شوند. |
| ◎ | **اسکنرها** | اسکن SNI، اسکن گروهی، **اسکنر IP تمیز**، تست اِج CDN، سازندهٔ CDN Configs، تستر **انبوه URI** و **اسکنر سایت** بازسازی‌شده (پیشرفت زنده، کلودفلر در صدر، اتصال/ذخیره). |
| ◈ | **تونل گوگل (Fastly)** | رلهٔ دامین‌فرانتینگ از طریق **Google Apps Script** و **Cloudflare Worker** که خودتان دیپلوی می‌کنید. هر دو اسکریپت داخل برنامه ساخته می‌شوند — هیچ مخزن بیرونی دخیل نیست. |
| ☁ | **امکانات دیگر** | سازندهٔ Cloudflare Worker، مدیریت WinDivert، تونل‌های Psiphon و SPlus. |
| 🎨 | **رابط مدرن** | تم روشن و تیره، فونت‌های خوانا، چیدمان کاملاً واکنش‌گرا (موبایل تا اولترا‌واید)، انگلیسی و فارسی. |

### 🚀 شروع سریع

```bash
# اجرای نسخهٔ آماده
./v2rayez            # لینوکس / مک
v2rayez.exe          # ویندوز
```

به‌صورت پیش‌فرض روی `0.0.0.0:8765` گوش می‌دهد و داشبورد را باز می‌کند. اگر باز نشد،
**http://127.0.0.1:8765** را باز کنید.

<details>
<summary><b>ساخت از سورس</b> (Go 1.22+)</summary>

```bash
git clone <your-fork-url> v2rayez
cd v2rayez
go build -o v2rayez .
./v2rayez

# کامپایل برای پلتفرم‌های دیگر
GOOS=windows GOARCH=amd64 go build -o v2rayez.exe .
GOOS=darwin  GOARCH=arm64 go build -o v2rayez-mac .
GOOS=linux   GOARCH=amd64 go build -o v2rayez .

# ترنسپورت‌های اختیاری پشت build tag
go build -tags "livekit psiphon" -o v2rayez .
```
</details>

<details>
<summary><b>آرگومان‌های خط فرمان</b></summary>

| آرگومان | پیش‌فرض | توضیح |
| --- | --- | --- |
| `-addr` | `0.0.0.0:8765` | آدرسی که پنل روی آن گوش می‌دهد |
| `-open` | `true` | باز کردن داشبورد هنگام شروع |
| `-window` | `true` | استفاده از پنجرهٔ اختصاصی در صورت وجود |
| `-minimize` | `true` | کوچک‌کردن کنسول در ویندوز |

```bash
./v2rayez -addr 127.0.0.1:8765 -open=false -window=false   # فقط محلی
```
</details>

### 🧭 چطور کار می‌کند

<div align="center">
<img src="docs/architecture.svg" alt="معماری V2RayEz" width="96%">
</div>

بیشتر DPIهای ساده با خواندن **SNI** که هنگام دست‌دادن TLS آشکار فرستاده می‌شود مسدود
می‌کنند. V2RayEz به سرور واقعی وصل می‌شود اما یک نام میزبان *مجاز* در آن فیلد می‌نویسد، پس
فیلتر یک دامنهٔ مجاز می‌بیند در حالی که نشست واقعی ادامه می‌یابد. **تونل گوگل** یک گام جلوتر
است: ترافیک شما به اِج گوگل فرانت می‌شود، یک Google Apps Script که خودتان دیپلوی می‌کنید
آن را رله می‌کند و Cloudflare Worker شما سایت واقعی را می‌گیرد — پس شبکه فقط ترافیک عادی
گوگل را می‌بیند.

### 📚 کار با تب‌ها

<details>
<summary><b>کتابخانهٔ کانفیگ</b> (تب پیش‌فرض)</summary>

- **گروه** بسازید؛ با چسباندن `vless:// vmess:// trojan:// ss://`، کشیدن یک `.txt`، ایمپورت
  **لینک اشتراک**، **بارگذاری کانفیگ‌های SNI** یا **دریافت آخرین** کانفیگ اضافه کنید.
- **اتصال** (سبز) کانفیگ را با موتور انتخاب‌شده در XRay Core اجرا می‌کند.
- **راست‌کلیک** روی ردیف یا انتخاب → اتصال / تست / تست سرعت / اشتراک / ویرایش / کپی / حذف.
  با `Ctrl/⌘-کلیک` و `Shift-کلیک` چندتایی، `Ctrl/⌘-A` همه، `Enter` اتصال.
- **ویرایش** یک ویرایشگر ساختاریافته باز می‌کند (آدرس، پورت، UUID/رمز، رمزنگاری، ترنسپورت،
  مسیر، هدر Host، امنیت، SNI، اثرانگشت، ALPN، کلیدهای Reality) با پیش‌نمایش زنده.
</details>

<details>
<summary><b>XRay Core · تونل SNI · تونل گوگل</b></summary>

- **XRay Core** — موتورها را پیدا/دانلود کنید، یک کانفیگ متصل کنید، **xray (SOCKS5)** یا
  **sing-box (TUN)** را انتخاب و شروع کنید. کنترل‌های TUN/شروع‌توقف/پروکسی‌سیستم در نوار بالا هستند.
- **تونل SNI** — پورت گوش‌دادن + مقصد + یک یا چند **SNI جعلی**، یک حالت و desync اختیاری،
  به‌همراه تست رله، تست تک‌کانفیگ و **لیست SNI ذخیره‌شده**.
- **تونل گوگل (Fastly)** — ① `worker.js` و `Code.gs` را داخل برنامه بسازید، ② آن‌ها را روی
  Cloudflare و Google Apps Script دیپلوی و Deployment ID را کپی کنید، ③ آن را با کلید احراز
  بچسبانید، یک **پیش‌تنظیم فرانت** انتخاب و شروع کنید، ④ پروکسی را روی `127.0.0.1:8086`
  بگذارید و گواهی را نصب کنید (**.pem** یا **.crt** برای نصب یک‌کلیکی در ویندوز).
</details>

### ⚠️ نکته‌ها و محدودیت‌ها

- **TUN** و **WinDivert** به ادمین/روت نیاز دارند.
- **پروکسی سیستم** از ابزار بومی هر سیستم‌عامل استفاده می‌کند (رجیستری + WinINET ویندوز،
  `networksetup` مک، `gsettings` گنوم).
- **تونل گوگل** فقط درخواست/پاسخ را رله می‌کند (بدون WebSocket/استریم) و به رله‌ای که
  *خودتان* دیپلوی می‌کنید نیاز دارد؛ فرانت «Fastly» فقط با رلهٔ روی Fastly کار می‌کند —
  مسیر پشتیبانی‌شده پیش‌تنظیم‌های گوگل است.

### 🙌 سازندگان

ساختهٔ **MacanDev** · [@EzAccess1](https://t.me/EzAccess1). ایدهٔ تونل گوگل از رویکرد
MasterHttpRelay / mhr-cfw الهام گرفته شده؛ V2RayEz پیاده‌سازی مستقل و تمیز Go خودش و
اسکریپت‌های تولیدی خودش را دارد.

</div>

<div align="center"><sub>© MacanDev · V2RayEz v4.4.5</sub></div>
