# Reels автомат постлолт — суулгах заавар

Carousel-ийн систем хэвээрээ ажиллана. Энэ бол **тусдаа**, зэрэгцээ ажиллах
урсгал: `manifest-reels.json`-ыг уншиж, өдөр бүр 18:00-д (UB) тухайн өдрийн
reel-ийг Instagram болон Facebook дээр тавина.

Яагаад тусдаа вэ: Instagram reel-ийг `media_type=REELS` + `video_url`-ээр,
Facebook нь `video_reels` (эсвэл `/videos`) endpoint-оор авдаг. Carousel-ийн
зурган урсгалтай нэг workflow-д хийвэл ажиллаж байгаа системийг эвдэх эрсдэлтэй.

## 1. Workflow импортлох

1. n8n → **Workflows** → баруун дээд **⋯** → **Import from File**
2. `n8n-autopost-reels.json` файлыг сонго
3. **Save**

Одоо байгаа "Level Hunter - Auto-post Carousel (IG + FB)" workflow-д юу ч
өөрчлөгдөхгүй — энэ бол шинэ, тусдаа workflow.

## 2. Config node бөглөх

Carousel-ийн workflow дээрх **яг ижил утгууд** (тэндээсээ хуулж болно):

| Талбар | Утга |
|---|---|
| `graphVersion` | `v25.0` |
| `igUserId` | Instagram business account ID |
| `pageId` | Facebook Page ID |
| `accessToken` | Long-lived Page access token |
| `manifestUrl` | (аль хэдийн бөглөгдсөн — `manifest-reels.json`) |

## 3. Туршиж үзэх

**Test (manual)** товчоор гараар ажиллуулна. Тухайн өдрийн огноотой reel
manifest дотор байхгүй бол `skipped: true` гэж хариулна — энэ хэвийн.

Одоо туршихыг хүсвэл `manifest-reels.json` доторх `date`-ийг өнөөдрийн огноо
болгож түр өөрчлөөд push хийж, дараа нь буцааж болно.

## 4. Идэвхжүүлэх

Баруун дээд **Active** toggle-ыг асаа. Дараа нь өдөр бүр 18:00-д (UB)
manifest доторх тухайн өдрийн reel автоматаар тавигдана.

## Шинэ reel нэмэх

1. MP4-ээ `2026-09/reels/<id>/reel.mp4` болгож энэ repo руу push хий
2. `manifest-reels.json`-д мөр нэмэ:

```json
{
  "id": "02-statistik",
  "title": "Статистик — Сахилгын зөрүү",
  "date": "2026-09-12",
  "time": "18:00",
  "ig": true,
  "fb": true,
  "status": "queued",
  "type": "reel",
  "video": "https://raw.githubusercontent.com/monkherdenett-eng/levelhunter-carousel/main/2026-09/reels/02-statistik/reel.mp4",
  "videoCdn": "https://cdn.jsdelivr.net/gh/monkherdenett-eng/levelhunter-carousel@main/2026-09/reels/02-statistik/reel.mp4",
  "caption": "…"
}
```

**Санамж:** `videoCdn` (jsDelivr) хаягийг заавал бөглө. `raw.githubusercontent`
нь MP4-ийг `application/octet-stream` гэж өгдөг тул Meta-гийн API татахаас
өмнө татгалзаж мэднэ; jsDelivr яг ижил файлыг `video/mp4` гэж зөв өгдөг.
Workflow нь `videoCdn` байвал түүнийг, байхгүй бол `video`-г ашиглана.

## Хязгаарлалт

- **Давхардлаас хамгаалалт огноогоор**: нэг өдөрт нэг reel. Хэрэв нэг өдөр
  хоёр удаа ажиллуулбал хоёр удаа тавигдана (carousel-ийнх ч мөн адил).
- **FB Reels эрх**: `video_reels` endpoint ажиллахгүй бол автоматаар ердийн
  видео пост (`/videos`) болж тавигдана. Гүйцэтгэлийн үр дүнд `fbFallback`
  гэсэн мөр гарвал тэр тохиолдол болсон гэсэн үг.
- **Видеоны шаардлага** (Instagram Reels): 3–90 секунд, 9:16, H.264 + AAC,
  1080×1920 — энэ repo дахь reel бүгд эдгээрийг хангасан.
