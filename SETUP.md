# Level Hunter — IG + FB Auto-post суулгах заавар

Энэ систем нь **өдөр бүр 10:00 (UB цагаар)** ажиллаж, `manifest.json` доторх тухайн өдрийн постыг
**Instagram carousel** болон **Facebook олон зурган пост** болгож автоматаар тавина.

Бүрдэл хэсгүүд:
- 🖼 Зураг hosting: `github.com/monkherdenett-eng/levelhunter-carousel` (JPEG, public)
- 📋 Контент жагсаалт: `.../main/manifest.json` (9 пост, огноо + caption)
- ⚙️ n8n workflow: `n8n-autopost-ig-fb.json`

---

## АЛХАМ 1 — Facebook/IG token + ID авах (чи хийнэ, ~10 мин)

> Michael-ийн Messenger bot (`michael fb ig`) аль хэдийн ижил Meta App дээр ажилладаг тул App + IG холбоос бэлэн. Зөвхөн **постлох эрхтэй token** нэмж авна.

1. **[developers.facebook.com](https://developers.facebook.com/) → Tools → Graph API Explorer** нээ.
2. Баруун дээд талд өөрийн **App**-аа сонго (Messenger bot-ийн App).
3. **Permissions** дээр дараах эрхүүдийг нэм:
   - `pages_show_list`
   - `pages_read_engagement`
   - `pages_manage_posts`
   - `instagram_basic`
   - `instagram_content_publish`  ← хамгийн чухал
   - `business_management`
4. **Generate Access Token** → зөвшөөр. (Өөрийн эзэмшдэг акаунт тул Development mode-д App Review шаардлагагүй.)
5. **PAGE_ID + Page token авах:** Explorer-т `me/accounts` GET дуудна → гарч ирсэн Page-ийн `id` (= **PAGE_ID**) болон `access_token` (= **Page token**)-ыг хуулж ав.
6. **IG_USER_ID авах:** `{PAGE_ID}?fields=instagram_business_account` GET дуудна → `instagram_business_account.id` (= **IG_USER_ID**).
7. **Token-ыг урт хугацаат болгох (60 хоног+):**
   [Access Token Tool](https://developers.facebook.com/tools/accesstoken/) эсвэл
   `GET /oauth/access_token?grant_type=fb_exchange_token&client_id={APP_ID}&client_secret={APP_SECRET}&fb_exchange_token={PAGE_TOKEN}`
   → урт хугацаат token. *(Урт хугацаат USER token-оос авсан PAGE token нь ихэвчлэн хугацаагүй болдог.)*

> Эцэст нь чамд 3 зүйл байх ёстой: **IG_USER_ID**, **PAGE_ID**, **урт хугацаат PAGE ACCESS TOKEN**.

---

## АЛХАМ 2 — n8n-д import хийх (чи хийнэ, 2 мин)

1. [kaizeco.online](https://kaizeco.online/) → n8n нээ.
2. Баруун дээд **⋯ (three dots) → Import from File** → `n8n-autopost-ig-fb.json` сонго.
3. **"Config"** node-ыг нээ, дараах 3 талбарыг бөглө:
   - `igUserId` → чиний **IG_USER_ID**
   - `pageId` → чиний **PAGE_ID**
   - `accessToken` → чиний **урт хугацаат PAGE TOKEN**
   *(`graphVersion` = v25.0, `manifestUrl` аль хэдийн бөглөгдсөн — хэвээр үлдээ.)*
4. **Save**.

---

## АЛХАМ 3 — Тест хийх

- **"Test (manual)"** trigger дээр **Execute workflow** дар.
- Хэрэв өнөөдрийн огноотой пост manifest-д байвал → IG + FB-д тавина. Үр дүнд `{ ig: "...", fb: "..." }` гарна.
- Хэрэв өнөөдөр пост байхгүй бол → `{ skipped: true }`. Тест хийхийн тулд manifest доторх нэг постын `date`-ыг өнөөдрийн огноо болгож түр солиод (repo-д push), дахин ажиллуул.

---

## АЛХАМ 4 — Идэвхжүүлэх

- Дээд баруун талын **Active** toggle-ыг асаа.
- Одоо **өдөр бүр 10:00-д** автоматаар ажиллаж, тухайн өдрийн постыг тавина.
- Огноо/цаг өөрчлөх: **"Daily 10:00 (UB)"** node доторх cron (`0 10 * * *`) → жишээ `0 20 * * *` = 20:00.

---

## Шинэ пост / шинэ сар нэмэх

Дараа сарын carousel хийхэд:
1. Зургуудыг ижил бүтэцтэй OneDrive-д гаргана (Claude хийнэ).
2. Claude PNG→JPEG хөрвүүлж, repo-д push, `manifest.json`-д шинэ постуудыг огноотой нэмнэ.
3. Өөр юу ч солих хэрэггүй — workflow автоматаар шинэ manifest уншина.

---

## Анхаарах зүйл

- **Rate limit:** IG ~25–100 пост/24 цаг. Өдөрт 1 пост тул асуудалгүй.
- **Давхар постлохоос сэргийлэх:** cron өдөрт нэг л удаа ажилладаг тул давхардахгүй. Гараар олон удаа ажиллуулбал давхар тавьж болзошгүй — болгоомжтой.
- **Token хугацаа:** урт хугацаат PAGE token хугацаагүй байх ёстой; хэрэв 60 хоногийн дараа алдаа гарвал Алхам 1.7-г давт.
- **Зөвхөн IG эсвэл зөвхөн FB:** manifest доторх постын `ig` эсвэл `fb`-г `false` болго.
- **Caption засах:** manifest-ийн `caption`-ыг засаад push хий (эсвэл Claude-д хэлээрэй).

---

## Аюулгүй байдал
- Access token бол **нууц** — хэн нэгэнтэй бүү хуваалц, public repo-д бүү тавь. n8n-ийн Config node дотор л байлга.
- manifest.json болон зурагнууд public (энэ нь зүгээр — маркетингийн контент нийтэд зориулагдсан).
