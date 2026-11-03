# Superchat
A single conversational assistant combining Aviasales (flights), the Shopping Assistant (e-commerce) and CinemaPlus (cinema)

SuperChat üç əsas xidməti bir platformada birləşdirən çoxdilli chatbot layihəsidir. Layihə aşağıdakı funksiyaları dəstəkləyir:

- **Səs ilə idarəetmə:** Danışığın başlanğıcını və sonunu avtomatik aşkarlama (VAD).
- **Şəkil ilə axtarış:** Claude Vision vasitəsilə məhsulun tanınması.
- **Çoxdillilik:** Azərbaycan, İngilis və Rus dilləri. Cavablar istifadəçinin dilində təqdim olunur.

---

## 1. Hazırkı vəziyyət: Real və Mock xidmətlər

| Bölmə | Rejim | Qeyd |
|---|---|---|
| Uçuşlar | **Real** (Aviasales / Travelpayouts) | `.env` faylında `AVIASALES_TOKEN` olduqda avtomatik aktivləşir. Qiymətlər keşlənmiş ən ucuz qiymətlərdir; canlı yer sayı göstərilmir. Rezervasiya əvəzinə Aviasales linki təqdim olunur. |
| Alış-veriş | **Real** (simlir) | Simlirden api key elde etdim |
| Visual search | **Real** (Claude Vision) | `ANTHROPIC_API_KEY` olduqda aktivləşir. Açar olmadıqda yalnız rəngə əsaslanan oflayn rejim işləyir. |
| Səs | Brauzer əsaslı | STT/TTS brauzerin Web Speech API-si, VAD isə `static/vad.js` vasitəsilə işləyir. |

---

## 2. Layihənin inkişaf mərhələləri

### 1. Real API adapterləri
- Aviasales.
- Ümumi e-commerce inteqrasiyası (DummyJSON hazırdır).
- CinemaPlus (fərziyyə əsaslı inteqrasiya).
- Claude Vision.

### 3. Səs və VAD
🎧 **Hands-free rejimi:**
- Danışığın başlanğıcı və sonu avtomatik aşkarlanır.
- Nitq mətnə çevrilərək serverə göndərilir.
- Chatbot-un cavabı səsləndirilir.
- İstifadəçi assistant danışarkən onun sözünü kəsə bilər (barge-in).
- Sistem öz səsini yenidən eşidərək dövr yaratmır.

### 4. Şəkil axtarışı və etibarlılıq balı

Claude Vision maksimum 3 namizəd və hər biri üçün 0–1 aralığında etibarlılıq balı qaytarır.

Chatbot baldan asılı olaraq belə davranır:

- **0.75 və yuxarı:** Nəticəni göstərir və faizi təqdim edir.
- **0.40–0.75:** Nəticəni göstərir və istifadəçidən təsdiq istəyir (Bəli / Xeyr düymələri).
- **İki namizədin balları bir-birinə yaxındırsa:** İstifadəçidən hansını nəzərdə tutduğunu soruşur.
- **0.40-dan aşağı:** Axtarış aparmır və məhsulun nə olduğunu dəqiqləşdirməyi xahiş edir.

Qeyd: Bu bal modelin öz qiymətləndirməsidir, kalibrə olunmuş ehtimal deyil. Hədlər `.env` faylındakı `VS_*` dəyişənləri vasitəsilə tənzimlənə bilər.

### 5. Real API problemlərinin aşkarlanması

- Serverin qaytardığı xəta səbəbi artıq görünür (məsələn, `HTTP 401: invalid x-api-key`).
- `python diagnose.py` diaqnostika əmri əlavə edilib.
- `/api/health` endpoint-i yaradılıb.
- `.env` faylının oxunması möhkəmləndirilib (Windows BOM və `.env.txt` problemləri).
- Server standart olaraq yalnız `127.0.0.1` ünvanında işləyir.
- Şəkil sorğularına limit tətbiq olunur.

---

### `.env` faylının konfiqurasiyası

`.env.example` faylını `.env` adı ilə kopyalayın və müvafiq məlumatları daxil edin.

Əsas dəyişənlər:

```env
AVIASALES_TOKEN=...             # Uçuşlar
ANTHROPIC_API_KEY=...           # Şəkil axtarışı
SUPERCHAT_PRODUCTS=dummyjson    # Alış-veriş
SUPERCHAT_DEBUG=1               # Xətanın real səbəbini çatda göstərir
```

**Qaydalar:**
- `=` işarəsinin ətrafında boşluq və dırnaq işarəsi istifadə etməyin.
- `.env` faylı gizli konfiqurasiya faylıdır.
- Windows Notepad istifadə edirsinizsə, yadda saxlayarkən `Save as type: All files` seçin. Əks halda fayl `.env.txt` adı ilə yarana bilər.
- Dəyişiklik etdikdən sonra serveri yenidən başladın.
- İstehsal mühitində `SUPERCHAT_DEBUG=0` istifadə edin.

---
