# LAPDHQ Integrity Bodycam System

Kısa açıklama:
LAPDHQ için geliştirilmiş, sicil numarası ile giriş yapılan, kayıt ve oynatma yetenekleri sunan bodycam uygulaması. OBS Studio entegrasyonu (obs-websocket) ile güçlü capture.

İlk adımlar (dev):
- client dizini: Electron + React uygulaması
- server dizini: basit Express API (auth + metadata)
- OBS ile entegrasyon için obs-websocket kullanılır (default host: localhost, port: 4455)

Özellikler (MVP):
- Sicil ile giriş
- OBS üzerinden start/stop recording
- Kayıt meta verilerinin backend'e kaydı

Not: Bu repo başlangıç (skeleton) olarak hazırlanmıştır.
