<h1 align="center">EzSNI</h1>

<p align="center"><i>by MacanDev</i></p>

<p align="center">
  <b>An all-in-one, fully offline DPI-bypass &amp; SNI-spoofing toolkit — written in Go.</b><br>
  Local proxy · SNI fragmentation · fake-packet desync · CDN fronting · SPlus LiveKit tunnel · scanners · Xray tester · WinDivert manager
</p>

<p align="center">
  <img alt="Go" src="https://img.shields.io/badge/Go-1.22%2B-00ADD8">
  <img alt="Platform" src="https://img.shields.io/badge/OS-Windows%20%7C%20Linux%20%7C%20macOS-444">
  <img alt="UI" src="https://img.shields.io/badge/UI-EN%20%2F%20%D9%81%D8%A7%D8%B1%D8%B3%DB%8C-2fe6c8">
  <img alt="Offline" src="https://img.shields.io/badge/assets-100%25%20offline-34d399">
</p>

<p align="center">
  <a href="#english">English</a> &nbsp;•&nbsp; <a href="#فارسی">فارسی 🇮🇷</a><br>
  Made by <b>MacanDev</b> · Telegram <a href="https://t.me/EzAccess1">@EzAccess1</a>
</p>

---

<a name="english"></a>

## English

A Go rewrite and major expansion of a Python SNI-spoofing / DPI-bypass tool, delivered as a **local web control panel** that binds to loopback. Everything is **self-contained and offline** — no fonts, scripts, or styles are fetched from any CDN, which matters for a censorship-circumvention tool. A live console streams status from every action over Server-Sent Events.

> ⚠️ **Use only where it is lawful for you to do so.** This is a network-diagnostics and circumvention toolkit; you are responsible for how you use it.

### Table of contents

- [Features](#features)
- [Quick start](#quick-start)
- [Building](#building)
- [Run-time flags](#run-time-flags)
- [The control panel](#the-control-panel)
- [DPI bypass](#dpi-bypass)
- [Xray tester](#xray-tester)
- [WinDivert](#windivert)
- [SPlus tunnel](#splus-tunnel)
- [Scope &amp; safety](#scope--safety)
- [Project layout](#project-layout)
- [Testing](#testing)
- [Credits &amp; license](#credits--license)

### Features

| Area | What you get |
|------|--------------|
| **SNI Tunnel** | Triple-mode local TCP proxy — *transparent* (terminates TLS upstream with a spoofed SNI), *passthrough* (raw TCP for clients with their own TLS), or *CDN fronting* (front SNI + rewritten `Host` header). Accepts **multiple SNIs (one per line)**, rotated per connection. Inline relay test, port check, and LAN sharing. |
| **DPI bypass** | SNI-aware ClientHello fragmentation, plus fake-ClientHello injection with `wrong_checksum` / `wrong_seq` desync and uTLS-style fingerprint presets. |
| **SPlus Tunnel** | SOCKS5 over a SoroushPlus voice call's LiveKit data channel (port of [`theermia/SPlusTunnel`](https://github.com/theermia/SPlusTunnel)). Optional **username/password auth** for LAN-shared SOCKS. |
| **Psiphon** | Embedded Psiphon device tunnel that dials out **through an upstream proxy this app exposes**, then offers local SOCKS5/HTTP for the whole device *(build with `-tags psiphon`)*. |
| **Xray** | Test a `vless://`/`vmess://` link (direct or via the spoofed-SNI proxy), **mass-check many configs** by ping/relay delay, **run xray on the device** as a local SOCKS proxy, and **detect/download** the xray binary. |
| **Scanners** | Single SNI check, relay timing, mass SNI scan (with a built-in captcha-domain list), a Cloudflare IP sweeper, and a **CDN-fronting edge tester** that ranks edge IPs by ping with one-click connect. |
| **URI Parser** | Decode `vless://` / `vmess://` links and push host/port/SNI straight into the SNI Tunnel. |
| **WinDivert** | One-click install / start / remove of the WinDivert kernel-driver service (Windows + Administrator). |
| **Bilingual UI** | Toggle **English ⇄ Persian (فارسی)**, with full right-to-left layout. |

### Quick start

```sh
# requires Go 1.22+
go build -o ezsni .
./ezsni
```

The panel opens at `http://127.0.0.1:8765/`. Use `-addr` to change the bind address and `-open=false` to skip auto-opening the browser. Everything except the SPlus tunnel works in this default build with **zero external dependencies**.

### Building

**Default (offline, no deps):**

```sh
go build -o ezsni .
```

**With the SPlus LiveKit tunnel:** the LiveKit transport sits behind a build tag so the default build stays dependency-free and compiles offline.

```sh
go get github.com/livekit/server-sdk-go/v2@latest
go build -tags livekit -o ezsni .
```

Without `-tags livekit` the SPlus tab still loads, but **Start tunnel** returns these instructions instead of connecting. A recent SDK may require Go 1.23+; if `go get` asks for a toolchain upgrade, install a newer Go or pin a compatible SDK version (e.g. `@v2.16.3`). The three SDK touch points are isolated in `internal/splus/transport_livekit.go`.

**With the embedded Psiphon tunnel:** likewise gated behind a build tag.

```sh
go get github.com/Psiphon-Labs/psiphon-tunnel-core/ClientLibrary/clientlib@latest
go build -tags psiphon -o ezsni .
```

Without `-tags psiphon` the Psiphon tab loads but **Start** returns build instructions; the engine is isolated in `internal/psiphon/psiphon_real.go`. A real Psiphon deployment also needs valid Psiphon config values (propagation/sponsor IDs) supplied via the `PSIPHON_CONFIG` environment variable.

**Cross-compile for Windows:**

```sh
GOOS=windows GOARCH=amd64 go build -o ezsni.exe .
```

### Run-time flags

```
-addr 127.0.0.1:8765   address to listen on
-open=false            do not auto-open the browser

DPI-evasion defaults (also settable per-start in the SNI Tunnel tab):
-enable-fragment       split the real ClientHello (off by default)
-sni-chunk    3        SNI bytes per write; 0 = whole host (hcaptcha.com → hca, ptc, ha., com)
-fragment-delay 500ms  delay between split ClientHello writes
-mode none             bypass mode: none | wrong_checksum | wrong_seq
-utls firefox          fake-ClientHello fingerprint preset (run -h to list)
-fake-repeat  1        number of fake ClientHello injections
-fake-delay   2ms      delay after fake injection before real traffic
-ack-timeout  2s       max wait for the server response after injection
```

Run `ezsni -h` to see the full `-utls` preset list.

### The control panel

Nine tabs: **SNI Tunnel**, **SPlus Tunnel**, **Xray Test**, **SNI Scan**, **Mass SNI Scan**, **IP Scanner**, **URI Parser**, **WinDivert**, **About**. A header button switches the whole UI between English and Persian.

### DPI bypass

Applied to the first ClientHello a connection sends upstream — in **passthrough** mode that is the client's relayed ClientHello, in **transparent** mode the proxy's own. Two independent layers:

**Fragmentation** (`-enable-fragment`, `-sni-chunk`, `-fragment-delay`) splits the real ClientHello across several TCP writes so the SNI is broken between segments. With `-sni-chunk 3`, `hcaptcha.com` goes out as `hca`, `ptc`, `ha.`, `com`; `0` keeps the host whole. This is pure userspace, works on every OS, and is the **reliable** bypass — verified end-to-end in `internal/proxy/proxy_test.go`.

**Fake injection** (`-mode`, `-utls`, `-fake-repeat`, `-fake-delay`, `-ack-timeout`) crafts a benign fake ClientHello as a raw TCP segment that the real server rejects but a DPI still inspects, desynchronising it:

- `wrong_checksum` — the segment carries a deliberately invalid TCP checksum.
- `wrong_seq` — the sequence number is pushed out of the receive window.
- `-utls` shapes the fake hello after `firefox`, `chrome`, `safari`, `ios`, `android`, `edge`, `360`, `qq`, `randomized`, `random`, or `none`.

The segment crafting (checksum, both corruption modes, the SNI parser and fragmenter) is unit-tested. **Sending** the raw segment needs OS privileges and is implemented per platform: **Linux** via raw sockets (root / `CAP_NET_RAW`), **Windows** via WinDivert (place `WinDivert.dll` next to the app and run as Administrator). When injection is unavailable the proxy logs it and **continues with fragmentation**, so a mode selection never breaks the connection.

> The raw-send paths are compiled for their targets but follow the documented WinDivert v2 / Linux raw-socket APIs; verify them on real hardware.

### CDN fronting (important)

CDN fronting in the SNI Tunnel presents an **allowed front domain as the TLS SNI** while rewriting the HTTP `Host:` header to the real target on the same CDN. This is only the *obfuscation layer*: a reachable edge IP with a passing ping/relay test means the CDN edge accepts your TLS — it does **not** by itself give you a working tunnel to the open internet.

To actually move traffic you need a **backend protocol behind the CDN**, and that is what the Xray and Psiphon tabs are for:

- Put a `vless://`/`vmess://`/`trojan` server behind the CDN (e.g. a WS+TLS config whose `host`/`sni` is the front domain), then **run Xray on this device** with that config — optionally routing its outbound through the SNI/CDN-fronting proxy on the chosen edge IP.
- Or start one of this app's proxies and run **Psiphon** through it as the upstream.

So the flow is: device → Xray/Psiphon (real proxy protocol) → CDN edge (fronted by the SNI layer) → your server → internet. Host-header fronting alone, with just an edge IP and no backend protocol, has nothing to tunnel — which is the expected behaviour when "ping and relay are OK" but nothing flows.

### Xray tester

Set the xray path (or **Detect** / **Download** the binary), then:

- **Single test** — paste a link and run a real HTTPS request. *Direct* connects straight to the config server; unchecking it routes through the local SNI proxy (start that proxy in PASSTHROUGH first). If the proxy isn't listening the test fails fast; if Xray errors, its log is surfaced (no bare "EOF").
- **Mass URI check** — paste many configs (one per line) and rank them by ping (TCP connect) and relay delay (TLS) to each config's own server. No xray needed; **Use** sends a row to the runner.
- **Run xray on this device** — launch xray as a persistent local SOCKS5 your whole device can point at (bind `0.0.0.0` to share on the LAN).

### Psiphon device tunnel

Builds only with `-tags psiphon` (see Building). It runs the embedded Psiphon tunnel configured with `UpstreamProxyUrl` set to a proxy this app exposes (the SNI Tunnel, or the SPlus SOCKS), and provides local SOCKS5/HTTP proxies for the device to use. Without the build tag the tab returns build instructions; the engine and Psiphon dependency are isolated in `internal/psiphon/psiphon_real.go` and can't be compiled or verified in the default offline build.

### WinDivert

Windows only. Place `WinDivert64.sys` (and the matching `WinDivert.dll`/`.sys` for your architecture) next to the executable or set the folder path in the tab, run the app **as Administrator**, then *Install &amp; start* creates and starts the `WinDivert` kernel service; *Stop &amp; remove* deletes it. On other systems the tab only reports status.

### SPlus tunnel

The tunnel reuses the LiveKit room of a live SoroushPlus voice call:

1. Start a voice call in **SoroushPlus Messenger** (web) between the two machines.
2. DevTools (`F12`) → **Network** → filter **WS** → find the socket to `k.splus.ir:8446`.
3. Copy its `access_token` query parameter — that is the token.
4. Start the **server** side first, then the **client**. Each side uses its own token.

The **client** role opens a local SOCKS5 listener (default `0.0.0.0:1080`):

```sh
curl --socks5-hostname 127.0.0.1:1080 https://ifconfig.me
```

### Scope &amp; safety

The proxy intentionally skips upstream TLS certificate verification (`InsecureSkipVerify`) — that is inherent to SNI spoofing, where the presented certificate won't match the fake SNI. The original tool's kernel packet-**capture** path is not reproduced; what is ported is the user-space feature set plus a WinDivert driver-service manager. Use only where lawful.

### Project layout

```
.
├── main.go                         # launches the local web control panel
└── internal/
    ├── protocol/                   # SPlus wire frames (C/A/D/X) + tests
    ├── sni/                        # URI parser, SNI/relay/mass scan, Cloudflare scan + tests
    ├── proxy/                      # dual-mode (transparent/passthrough) TCP proxy + e2e frag test
    ├── desync/                     # DPI evasion: SNI fragmentation + fake-segment crafting (+ tests)
    │   ├── desync.go               #   parser, fragmenter, fake hello & segment builders, conn wrapper
    │   ├── raw_linux.go            #   raw-socket sender   (build tag: linux, needs root)
    │   ├── raw_windows.go          #   WinDivert sender    (build tag: windows, needs admin + WinDivert.dll)
    │   └── raw_other.go            #   stub for other platforms
    ├── splus/                      # SPlus tunnel: relay, SOCKS5, transport (+ livekit build tag)
    ├── xray/                       # xray/v2ray connection tester (stdlib SOCKS5 client)
    ├── windivert/                  # WinDivert driver-service manager (sc create/start/delete)
    ├── netutil/                    # TCP port check + LAN address enumeration
    ├── logbus/                     # log fan-out for the SSE console
    └── server/                     # HTTP routes + embedded bilingual SPA (web/index.html)
```

### Testing

```sh
go test ./...            # protocol, URI parsing, tunnel e2e, SNI parse/fragment, segment checksums
go test ./... -race      # all of the above under the race detector
```

### Credits &amp; license

Made by **MacanDev** — Telegram channel [@EzAccess1](https://t.me/EzAccess1).
SPlus tunnel ported from [`theermia/SPlusTunnel`](https://github.com/theermia/SPlusTunnel).

Provided as-is, for lawful network diagnostics and censorship circumvention. No warranty.

---

<a name="فارسی"></a>

## فارسی

بازنویسی و توسعهٔ گستردهٔ یک ابزار پایتونیِ جعل SNI و عبور از DPI، که این بار به‌صورت یک **پنل کنترل وب محلی** روی لوپ‌بک ارائه شده است. همه‌چیز **مستقل و آفلاین** است — هیچ فونت، اسکریپت یا استایلی از CDN گرفته نمی‌شود؛ این برای یک ابزار عبور از فیلترینگ اهمیت دارد. یک کنسول زنده هم وضعیت هر اقدام را با SSE پخش می‌کند.

> ⚠️ **فقط جایی استفاده کنید که برایتان قانونی است.** این یک جعبه‌ابزار عیب‌یابی شبکه و عبور از فیلترینگ است و مسئولیت نحوهٔ استفاده با خودتان است.

### فهرست

- [امکانات](#امکانات)
- [شروع سریع](#شروع-سریع)
- [ساخت پروژه](#ساخت-پروژه)
- [پرچم‌های اجرا](#پرچمهای-اجرا)
- [پنل کنترل](#پنل-کنترل)
- [عبور از DPI](#عبور-از-dpi)
- [تستر Xray](#تستر-xray)
- [WinDivert‏](#windivert-fa)
- [تونل SPlus](#تونل-splus)
- [دامنه و ایمنی](#دامنه-و-ایمنی)
- [اعتبار و مجوز](#اعتبار-و-مجوز)

### امکانات

| بخش | چه چیزی به‌دست می‌آورید |
|------|--------------------------|
| **تونل SNI** | پروکسی محلی TCP سه‌حالته — *شفاف* (خاتمهٔ TLS با SNI جعلی)، *عبوری* (TCP خام برای کلاینت‌هایی که TLS خودشان را دارند) یا *CDN فرانتینگ* (SNI فرانت + بازنویسی هدر `Host`). از **چند SNI (هر خط یک مورد)** که به‌ازای هر اتصال چرخش می‌یابند پشتیبانی می‌کند. همراه با تست رله، بررسی پورت و اشتراک شبکه. |
| **عبور از DPI** | تکه‌تکه‌سازی ClientHello مبتنی بر SNI، به‌علاوهٔ تزریق ClientHello جعلی با حالت‌های `wrong_checksum` / `wrong_seq` و پریست‌های اثرانگشت به سبک uTLS. |
| **تونل SPlus** | SOCKS5 روی کانال دادهٔ LiveKit یک تماس صوتی سروش‌پلاس (پورت‌شدهٔ [`theermia/SPlusTunnel`](https://github.com/theermia/SPlusTunnel)). همراه با **احراز هویت نام‌کاربری/رمز** اختیاری برای SOCKS اشتراکی در شبکه. |
| **سایفون** | تونل دستگاه سایفونِ تعبیه‌شده که خروجش را **از طریق یک پروکسی بالادست که همین برنامه ارائه می‌دهد** برقرار می‌کند، سپس SOCKS5/HTTP محلی برای کل دستگاه فراهم می‌کند *(ساخت با `-tags psiphon`)*. |
| **Xray** | تست لینک `vless://`/`vmess://` (مستقیم یا از طریق پروکسی جعل‌SNI)، **بررسی گروهی چندین کانفیگ** بر اساس پینگ/تأخیر رله، **اجرای xray روی دستگاه** به‌عنوان پروکسی SOCKS محلی، و **تشخیص/دانلود** فایل اجرایی xray. |
| **اسکنرها** | بررسی تکی SNI، زمان‌سنجی رله، اسکن گروهی SNI (با لیست داخلی دامنه‌های کپچا)، جاروی IP کلودفلر و یک **تستر اِج CDN فرانتینگ** که IPهای اِج را بر اساس پینگ مرتب کرده و با یک کلیک متصل می‌شود. |
| **تجزیه‌گر URI** | لینک‌های `vless://`/`vmess://` را تجزیه کرده و هاست/پورت/SNI را مستقیم به تونل SNI بفرستید. |
| **WinDivert** | نصب / راه‌اندازی / حذف یک‌کلیکی سرویس درایور هسته‌ای WinDivert (ویندوز + دسترسی مدیر). |
| **رابط دوزبانه** | جابه‌جایی **انگلیسی ⇄ فارسی**، با چیدمان کامل راست‌به‌چپ. |

### شروع سریع

```sh
# به Go نسخهٔ ۱.۲۲ یا بالاتر نیاز است
go build -o ezsni .
./ezsni
```

پنل روی `http://127.0.0.1:8765/` باز می‌شود. با `-addr` آدرس را عوض کنید و با `-open=false` از باز شدن خودکار مرورگر جلوگیری کنید. در این بیلد پیش‌فرض، همه‌چیز جز تونل SPlus **بدون هیچ وابستگی خارجی** کار می‌کند.

### ساخت پروژه

**پیش‌فرض (آفلاین، بدون وابستگی):**

```sh
go build -o ezsni .
```

**همراه با تونل LiveKit (SPlus):** ترانسپورت LiveKit پشت یک build tag قرار دارد تا بیلد پیش‌فرض بدون وابستگی و آفلاین بماند.

```sh
go get github.com/livekit/server-sdk-go/v2@latest
go build -tags livekit -o ezsni .
```

بدون `-tags livekit` تب SPlus باز می‌شود اما **شروع تونل** به‌جای اتصال، همین دستورها را برمی‌گرداند. SDK جدید ممکن است به Go نسخهٔ ۱.۲۳ به بالا نیاز داشته باشد؛ اگر `go get` درخواست ارتقای toolchain داد، Go جدیدتری نصب کنید یا نسخهٔ سازگاری از SDK را پین کنید (مثلاً `@v2.16.3`). سه نقطهٔ تماس با SDK در فایل `internal/splus/transport_livekit.go` ایزوله شده‌اند.

**کامپایل برای ویندوز:**

```sh
GOOS=windows GOARCH=amd64 go build -o ezsni.exe .
```

### پرچم‌های اجرا

```
-addr 127.0.0.1:8765   آدرس شنونده
-open=false            مرورگر را خودکار باز نکن

پیش‌فرض‌های عبور از DPI (در تب «تونل SNI» هم قابل تنظیم‌اند):
-enable-fragment       تکه‌تکه‌کردن ClientHello واقعی (پیش‌فرض: خاموش)
-sni-chunk    3        بایت SNI در هر نوشتن؛ ۰ = کل هاست (hcaptcha.com → hca, ptc, ha., com)
-fragment-delay 500ms  تأخیر بین نوشتن‌های تکه‌شده
-mode none             حالت عبور: none | wrong_checksum | wrong_seq
-utls firefox          پریست اثرانگشت ClientHello جعلی (برای فهرست کامل: -h)
-fake-repeat  1        تعداد تزریق ClientHello جعلی
-fake-delay   2ms      تأخیر پس از تزریق جعلی پیش از ترافیک واقعی
-ack-timeout  2s       حداکثر انتظار برای پاسخ سرور پس از تزریق
```

برای دیدن فهرست کامل پریست‌های `-utls` دستور `ezsni -h` را اجرا کنید.

### پنل کنترل

نُه تب: **تونل SNI**، **تونل SPlus**، **تست Xray**، **اسکن SNI**، **اسکن گروهی SNI**، **اسکنر IP**، **تجزیه‌گر URI**، **WinDivert** و **درباره**. یک دکمه در هدر کل رابط را میان فارسی و انگلیسی جابه‌جا می‌کند.

### عبور از DPI

روی نخستین ClientHello که یک اتصال به بالادست می‌فرستد اعمال می‌شود — در حالت **عبوری** همان ClientHello رله‌شدهٔ کلاینت، و در حالت **شفاف** ClientHello خودِ پروکسی. دو لایهٔ مستقل:

**تکه‌تکه‌سازی** (`-enable-fragment`، `-sni-chunk`، `-fragment-delay`) ClientHello واقعی را میان چند نوشتن TCP تقسیم می‌کند تا SNI بین قطعه‌ها بشکند. با `-sni-chunk 3`، عبارت `hcaptcha.com` به‌صورت `hca`، `ptc`، `ha.`، `com` ارسال می‌شود؛ مقدار `0` هاست را کامل نگه می‌دارد. این روش کاملاً کاربری است، روی هر سیستم‌عاملی کار می‌کند و **روش مطمئن** است — و در `internal/proxy/proxy_test.go` به‌صورت سرتاسری تست شده است.

**تزریق جعلی** (`-mode`، `-utls`، `-fake-repeat`، `-fake-delay`، `-ack-timeout`) یک ClientHello جعلیِ بی‌ضرر را به‌شکل یک سگمنت خام TCP می‌سازد که سرور واقعی ردش می‌کند ولی DPI همچنان بازرسی‌اش می‌کند و از همگامی خارج می‌شود:

- `wrong_checksum` — سگمنت یک checksum عمداً نامعتبر دارد.
- `wrong_seq` — شمارهٔ توالی به بیرون از پنجرهٔ دریافت هل داده می‌شود.
- `-utls` شکل ClientHello جعلی را شبیه `firefox`، `chrome`، `safari`، `ios`، `android`، `edge`، `360`، `qq`، `randomized`، `random` یا `none` می‌کند.

ساخت سگمنت (checksum، هر دو حالت خراب‌سازی، تجزیه‌گر و تکه‌کنندهٔ SNI) واحدتست شده است. **ارسال** سگمنت خام به دسترسی سیستم‌عامل نیاز دارد و برای هر پلتفرم جداگانه پیاده شده: **لینوکس** با raw socket (نیازمند root / `CAP_NET_RAW`)، **ویندوز** با WinDivert (فایل `WinDivert.dll` را کنار برنامه بگذارید و به‌عنوان Administrator اجرا کنید). وقتی تزریق در دسترس نباشد، پروکسی آن را ثبت می‌کند و **با تکه‌تکه‌سازی ادامه می‌دهد**؛ بنابراین انتخاب یک حالت هرگز اتصال را نمی‌شکند.

> مسیرهای ارسال خام برای پلتفرم‌های هدف کامپایل می‌شوند و از APIهای مستند WinDivert v2 و raw socket لینوکس پیروی می‌کنند؛ روی سخت‌افزار واقعی صحت‌سنجی کنید.

### تستر Xray

`xray` (یا `v2ray`) را نصب کنید و در `PATH` یا کنار فایل اجرایی قرار دهید. سپس:

۱. پروکسی **تونل SNI** را در حالت **عبوری (PASSTHROUGH)** اجرا کنید — پورت شنونده‌اش برابر «پورت پروکسی محلی» در تب Xray، و IP/پورت مقصدش روی سرور واقعی شما.
۲. در تب Xray یک لینک `vless://`/`vmess://` وارد کرده و تست را اجرا کنید.

اکس‌ری خروجی خود را به پروکسی محلی شما اشاره می‌دهد و یک درخواست واقعی HTTPS را از آن عبور می‌دهد. اگر پروکسی روی آن پورت در حال شنیدن نباشد، تست سریع و با پیامی روشن شکست می‌خورد؛ و اگر خود اکس‌ری خطا بدهد، لاگش نمایش داده می‌شود (دیگر «EOF» خشک‌وخالی نیست).

<a name="windivert-fa"></a>
### WinDivert

فقط ویندوز. فایل `WinDivert64.sys` (و `WinDivert.dll`/`.sys` متناسب با معماری) را کنار فایل اجرایی بگذارید یا مسیر پوشه را در تب وارد کنید، برنامه را **به‌عنوان Administrator** اجرا کنید؛ سپس «نصب و راه‌اندازی» سرویس هسته‌ای `WinDivert` را می‌سازد و استارت می‌زند و «توقف و حذف» آن را پاک می‌کند. روی سیستم‌های دیگر این تب فقط وضعیت را گزارش می‌دهد.

### تونل SPlus

تونل از اتاق LiveKit یک تماس صوتی فعال سروش‌پلاس استفادهٔ مجدد می‌کند:

۱. در **پیام‌رسان سروش‌پلاس** (نسخهٔ وب) میان دو دستگاه یک تماس صوتی برقرار کنید.
۲. ابزار توسعه‌دهنده (`F12`) → **Network** → فیلتر **WS** → اتصال به `k.splus.ir:8446` را بیابید.
۳. پارامتر `access_token` آن را کپی کنید — همان توکن است.
۴. ابتدا سمت **سرور** و سپس **کلاینت** را شروع کنید. هر طرف توکن خودش را دارد.

نقش **کلاینت** یک شنوندهٔ SOCKS5 محلی باز می‌کند (پیش‌فرض `0.0.0.0:1080`):

```sh
curl --socks5-hostname 127.0.0.1:1080 https://ifconfig.me
```

### دامنه و ایمنی

پروکسی عمداً تأیید گواهی TLS بالادست را نادیده می‌گیرد (`InsecureSkipVerify`) — این ذاتی جعل SNI است، جایی که گواهی ارائه‌شده با SNI جعلی مطابقت ندارد. مسیر **کپچر** بستهٔ هسته‌ای ابزار اصلی بازتولید نشده؛ آنچه پورت شده مجموعهٔ امکانات فضای کاربر به‌علاوهٔ یک مدیر سرویس درایور WinDivert است. فقط جایی که قانونی است استفاده کنید.

### اعتبار و مجوز

ساخت: **MacanDev** — کانال تلگرام [@EzAccess1](https://t.me/EzAccess1).
تونل SPlus پورت‌شده از [`theermia/SPlusTunnel`](https://github.com/theermia/SPlusTunnel).

به‌همان‌صورت که هست، برای عیب‌یابی قانونی شبکه و عبور از فیلترینگ ارائه می‌شود. بدون هیچ ضمانتی.
