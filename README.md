# LAPDHQ Integrity Bodycam System

LAPDHQ Integrity Bodycam System (LAPDHQ-IBS) — FiveM polis ekibi için tasarlanmış, sicil numarası ile oturum açma, kayıt başlat/durdur, kayıtların listelenmesi ve oynatılması özellikleri sunan masaüstü bodycam uygulamasıdır. Uygulama OBS Studio üzerinden obs-websocket ile entegre çalışarak güvenilir, GPU hızlandırmalı ve yüksek kaliteli video kaydı sağlar.

Kısa link: .gg/lapdhq

---

## Hedef
- Polis ekipleri için güvenilir, merkezi metadata yönetimi ile bodycam kayıt sistemi.
- Kolay kurulum: Electron tabanlı client + (opsiyonel) Node API server.
- Hızlı MVP: OBS Studio + obs-websocket entegrasyonu (local OBS’yi kontrol ederek kayıt).

---

## Öne Çıkan Özellikler (MVP)
- Sicil (badge) numarası ile her açılışta oturum açma
- OBS üzerinden start / stop recording
- Kayıt meta verilerini backend’e kaydetme (kullanıcı, zaman, not, etiketler)
- Pencere/ekran/audio cihazı seçimi (OBS üzerinden)
- Basit oynatma (kaydı açma) ve kayıt listesini filtreleme
- Ayarlar: OBS host/port, kayıt yolu, video formatı, dosya adlandırma şablonu
- Tema: mavi / siyah sade görsel tasarım

---

## Mimari (yüksek seviye)
- Client: Electron + React (TypeScript önerilir)
  - Electron main: pencere yönetimi, preload IPC köprüsü
  - Renderer: UI, obs-websocket client, ayarlar ekranı, playback
- OBS: Kullanıcının cihazında kurulu OBS Studio + obs-websocket eklentisi
- Server (opsiyonel): Node.js + Express — kullanıcı ve kayıt metadata’sı için REST API
- Depolama:
  - Video dosyaları: local disk (başlangıç) veya S3/remote storage
  - Metadata: PostgreSQL (veya SQLite / diğer)

---

## Güvenlik & Gizlilik
- Sicil numarası tek başına güçlü bir kimlik doğrulama yöntemi değildir. MVP aşamasında opsiyonel PIN veya token tabanlı kimlik doğrulamaya eklemenizi şiddetle öneririz.
- Video dosyaları hassas içerik olabilir — dosya erişimini kısıtlayın, disk şifreleme veya özel erişim kontrolleri uygulayın.
- Sunucu-istemci iletişimi TLS (HTTPS) üzerinden olmalı.
- Her kullanılma / işlem için audit log (kimin hangi kayda eriştiği, ne zaman kayıt başlattığı/durdurduğu) tutulmalı.

---

## Kurulum (geliştirme)
Aşağıdaki adımlar geliştirici makinasında projeyi çalıştırmak içindir. (İleride paketleyip installer oluşturulacaktır.)

1. Reponun klonlanması
   - git clone git@github.com:kaiwashere0/lapdhqbodycam.git
   - cd lapdhqbodycam

2. Client (Electron + React)
   - cd client
   - npm install
   - Not: development sırasında renderer için ayrı bir dev server (React CRA veya Vite) kullanılabilir. package.json içindeki script'ler örnek olarak konulmuştur.

3. Server (opsiyonel)
   - cd server
   - npm install
   - npm start
   - API default: http://localhost:3001

4. OBS kurulumu
   - OBS Studio'yu kurun (https://obsproject.com/)
   - obs-websocket eklentisini kurun (https://github.com/obsproject/obs-websocket)
   - obs-websocket ayarlarında port (default 4455) ve parola gerekiyorsa ayarlayın.

---

## Konfigürasyon (varsayılanlar)
- OBS WebSocket:
  - Host: localhost
  - Port: 4455
  - Password: (opsiyonel, ayarlama yapıldıysa girilmeli)
- Kayıt dosya adı şablonu:
  - LAPDHQ_{badge}_{YYYYMMDD}_{HHMMSS}.mp4
- Kayıt klasörü:
  - client/config veya settings ekranından ayarlanır; varsayılan ./videos/

Örnek .env (server için):
- PORT=3001
- DATABASE_URL=postgres://user:pass@localhost:5432/lapdhq
- STORAGE_PATH=./videos

---

## API Endpoints (örnek)
- POST /auth/login { "badge_number": "12345" } -> { token, user }
- GET /recordings?badge=12345 -> [recording]
- POST /recordings { metadata } -> create recording meta
- GET /recordings/:id -> recording meta (veya presigned URL)

(Not: Bu endpoints başlangıç içindir; auth JWT ve erişim kontrolü eklenmelidir.)

---

## Veri Modeli (örnek)
users:
- id, badge_number, display_name, created_at

recordings:
- id, user_id, file_name, file_path, started_at, stopped_at, duration_seconds, size_bytes, notes, tags, created_at

Örnek SQL schema dosyası repo içinde /db/schema.sql olarak bulunmaktadır.

---

## OBS Entegrasyonu - Notlar
- OBS'yi kontrol etmek için obs-websocket kullanıyoruz. (Client'ta obs-websocket-js veya benzeri client library kullanılacak.)
- Temel akış:
  1. Kullanıcı OBS bağlantı bilgilerini girer.
  2. Uygulama obs-websocket ile bağlanır.
  3. Kullanıcı “Start Recording” butonuna basınca obs üzerinden StartRecording çağrılır.
  4. StopRecording çağrıldığında uygulama OBS'den kaydın yolunu/metasını alır (OBS yapılandırmasına göre).
  5. Metadata server’a gönderilir; video dosyası disk üzerinde durur veya upload edilir.

Problemler & dikkat:
- OBS ayarları (Recording path) kullanıcı tarafında yapılandırılmış olmalı; uygulama bu path’i okumaya çalışacaktır.
- Farklı OBS sürümleri / obs-websocket versiyonları farklı message/endpoint setleri kullanabilir — uyumluluk kontrolü gereklidir.

---

## UI / UX (tasarım)
- Tema: Mavi (#1f6feb) ve koyu-siyah (#0b0f14) — sade, okunaklı arayüz
- Ekranlar:
  - Login (sicil)
  - Dashboard (kayıt listesi + yeni kayıt)
  - Record Panel (kaydet, kaynak seçimi, audio cihazları)
  - Settings (OBS host/port, kayıt yolu, kalite)
  - Playback (player + metadata)
- Kayıt sırasında belirgin kırmızı "RECORDING" göstergesi

---

## Geliştirme Yol Haritası (ilk sprintler)
1. Electron + React skeleton (UI + login)
2. Basit server (in-memory) ile auth ve metadata endpoint'leri
3. obs-websocket bağlantısı + start/stop demo
4. Kayıt metadata’sının server’a kaydı
5. Playback ekranı ve dosya yönetimi
6. Installer (electron-builder) ve Windows hedef paketleme

---

## Katkıda Bulunma
- Lütfen feature istekleri/buglar için Issues açın.
- PR'ler branch bazlı olmalı; `init/obs-integration` gibi feature branch’ler üzerine PR açın.
- Kod stilinde TypeScript/ESLint + Prettier kullanımı tercih edilir (ileriki commitlerde eklenecek).

---

## Lisans
- MIT License (varsayılan). Lisans dosyası LICENSE.md olarak eklenecektir.

---

## İletişim / Destek
- Proje sahibi: kaiwashere0
- Discord / .gg/lapdhq (tanımlandığında paylaşılacak)
