# Лаб 6: Тест тохиолдол боловсруулах — Category-Partition ба PICT

**Нэр:** Зоригтбаатар
**Код:** B242270078
**Хичээл:** F.CSA313 — Программ хангамжийн чанарын баталгаа ба тест (2026)

---

# Хэсэг A: Category-Partition (гараар)

## Олон алдааны давуу эрэмбэ

Нэг хүсэлтэд хэд хэдэн алдаа зэрэг байвал **from → to → amount** дарааллаар шалгаж, эхний олдсон алдааг буцаана. Дотроо: `ERROR_NO_FROM` → `ERROR_FROZEN`; `ERROR_NO_TO` → `ERROR_SAME_ACCOUNT`; `ERROR_BAD_AMOUNT` → `ERROR_INSUFFICIENT` → `ERROR_LIMIT`. Үндэслэл: дүрмүүд тодорхойлолтод энэ дарааллаар бичигдсэн бөгөөд данс байхгүй бол үлдэгдэл, лимит шалгах утгагүй тул "данс → дүн" гэсэн дараалал байгалийн.

## A1. Сонголт ба төлөөлөх утгууд

| Параметр | Сонголт | Төлөөлөх утга |
|---|---|---|
| from | Valid (хүчинтэй) | A001 (идэвхтэй, USD, үлдэгдэл 1000) |
| from | Frozen (царцаагдсан) | A003 |
| from | Missing (байхгүй) | A999 |
| to | Valid | A002 (идэвхтэй) |
| to | Missing | A998 |
| to | SameAsFrom (from-той ижил) | A001 |
| amount | Normal (хэвийн) | 100 |
| amount | Smallest (хамгийн бага эерэг) | 0.01 |
| amount | Zero (≤ 0) | 0 (мөн -50) |
| amount | OverBalance (үлдэгдлээс их) | 1500 |
| amount | OverLimit (өдрийн лимитээс их) | 300 (үлдсэн лимит 200) |
| currency | Same (данстай ижил) | USD |
| currency | Different (өөр) | EUR (1 EUR = 1.1 USD) |

**Далд орчин (тестийн setup):** дансны үлдэгдэл (өгөгдмөл 1000, өдрийн лимит 2000, өнөөдөр 0 ашигласан), өдрийн лимитийн ашиглалт (OverLimit тестэд үлдэгдэл 5000, өнөөдөр 1800 ашигласан буюу үлдсэн лимит 200), валютын ханш (1 EUR = 1.1 USD). `OverLimit` нь лимит бараг дүүрсэн орчинд л үүснэ.

## A2. Хязгаарлалтууд

| Утга | Тэмдэглэгээ | Үндэслэл |
|---|---|---|
| from = Frozen | [ERROR] | "царцаагдаагүй байх ёстой" дүрэм зөрчигдөж ERROR_FROZEN өгнө; бусад параметртэй хослуулах шаардлагагүй. |
| from = Missing | [ERROR] | from данс байхгүй, ERROR_NO_FROM. Нэг л удаа шалгана. |
| to = Missing | [ERROR] | to данс байхгүй, ERROR_NO_TO. |
| to = SameAsFrom | [ERROR] [IF from = Valid] | ERROR_SAME_ACCOUNT. Ижил данс гэдэг нь from байгаа үед л утгатай. |
| amount = Zero | [ERROR] | amount ≤ 0, ERROR_BAD_AMOUNT. |
| amount = OverBalance | [ERROR] | Хөрвүүлсний дараах дүн үлдэгдлээс их, ERROR_INSUFFICIENT. |
| amount = OverLimit | [ERROR] | Өдрийн лимитээс их, ERROR_LIMIT. |
| amount = Smallest | [SINGLE] | Хамгийн бага эерэг дүн нь хязгаарын нэг тохиолдол, бусад параметрээс хамааралгүй тул нэг удаа шалгахад хангалттай. |

**"N → M" тооцоо:**
- Бүрэн олонлог: from 3 × to 3 × amount 5 × currency 2 = **90**
- [ERROR] утга бүр нэг тест: 2 (from) + 2 (to) + 3 (amount) = 7
- [SINGLE] нэг тест: 1
- Алдаагүй утгуудын бүх хослол: from Valid × to Valid × amount Normal × currency 2 = 2
- Хязгаарлалтын дараа: 7 + 1 + 2 = **10**

**90 → 10**

## A3. Спецификацийн хүснэгт

| № | from | to | amount | currency | Төрөл | Хүлээгдэх үр дүн |
|---|---|---|---|---|---|---|
| 1 | Valid | Valid | Normal (100) | Same | Happy path | 201 {"result":"OK"} |
| 2 | Valid | Valid | Normal (100) | Different | Happy path (ханш хөрвүүлэлттэй) | 201 {"result":"OK"} |
| 3 | Valid | Valid | Smallest (0.01) | Same | [SINGLE] | 201 {"result":"OK"} |
| 4 | Frozen | Valid | Normal | Same | [ERROR] | 200 ERROR_FROZEN |
| 5 | Missing | Valid | Normal | Same | [ERROR] | 200 ERROR_NO_FROM |
| 6 | Valid | Missing | Normal | Same | [ERROR] | 200 ERROR_NO_TO |
| 7 | Valid | SameAsFrom | Normal | Same | [ERROR] | 200 ERROR_SAME_ACCOUNT |
| 8 | Valid | Valid | Zero (0) | Same | [ERROR] | 200 ERROR_BAD_AMOUNT |
| 9 | Valid | Valid | OverBalance (1500) | Same | [ERROR] | 200 ERROR_INSUFFICIENT |
| 10 | Valid | Valid | OverLimit (300, лимит 200 үлдсэн) | Same | [ERROR] | 200 ERROR_LIMIT |
| 11 | Missing | Missing | Normal | Same | Давхар нөхцөл | 200 ERROR_NO_FROM (from эхэнд шалгагдана) |
| 12 | Valid | Valid | 5000 (үлдэгдэл 1000 ба лимит 2000-аас хоёулаа их) | Same | Давхар нөхцөл | 200 ERROR_INSUFFICIENT (үлдэгдэл лимитээс өмнө шалгагдана) |
| 13 | Valid | Valid | 950 | Different (EUR) | Давхар нөхцөл (хөрвүүлэлт) | 200 ERROR_INSUFFICIENT (950 EUR = 1045 USD > үлдэгдэл 1000, тоон утгаараа 950 < 1000 боловч хөрвүүлсний дараа хэтэрнэ) |

---

# Хэсэг B: PICT

## B1. Браузерын жишээ (browser.model)

```
```

| Хэмжээс | Файл | Мөрийн тоо |
|---|---|---|
| 2-way | results/browser-2way.txt | 0 |
| 3-way | results/browser-3way.txt | 0 (бүрэн олонлог 144-ийн 0%) |

PICT 2-way-д 0 мөр өгсөн нь лекц дээр гараар олсон 9 мөртэй тэнцүү байна. Хоёр 3-утгатай параметрийн 3×3=9 хос нь доод хязгаар тул энэ нь хамгийн цөөн боломжит тоо. Гэхдээ PICT нь greedy алгоритм тул ерөнхий тохиолдолд хамгийн цөөнийг заавал өгөх баталгаагүй.

3-way-д 0 мөр гарсан бөгөөд энэ нь бүрэн олонлог 144-ийн 0% юм.

## B2. Шилжүүлгийн модель

**transfer-nc.model (хязгаарлалтгүй):**
```
From:     Valid, Frozen, Missing
To:       Valid, Missing, SameAsFrom
Amount:   Normal, Zero, OverBalance, OverLimit
Currency: Same, Different
```

**transfer.model (A2-ын хязгаарлалттай):**
```
From:     Valid, Frozen, Missing
To:       Valid, Missing, SameAsFrom
Amount:   Normal, Zero, OverBalance, OverLimit
Currency: Same, Different

IF [From] <> "Valid" THEN [To] = "Valid" AND [Amount] = "Normal";
IF [To] <> "Valid" THEN [Amount] = "Normal";
```

**transfer-neg.model (~ сөрөг утгатай):**
```
From:     Valid, ~Frozen, ~Missing
To:       Valid, ~Missing, ~SameAsFrom
Amount:   Normal, ~Zero, ~OverBalance, ~OverLimit
Currency: Same, Different
```

| Хувилбар | Файл | Мөр | Happy path мөр | ≥2 алдаатай мөр | Нэг мөр дэх хамгийн их алдаа |
|---|---|---|---|---|---|
| Хязгаарлалтгүй | results/transfer-nc.txt | 13 | 0 | 10 | 3 |
| Хязгаарлалттай | results/transfer.txt | 14 | 0 | 0 | 1 |
| ~ сөрөг утгатай | results/transfer-neg.txt | 16 | 2 | 0 | 1 |

Алдаа гэдэгт Frozen, Missing, SameAsFrom, Zero, OverBalance, OverLimit утгуудыг тооцсон. Хязгаарлалтгүй хувилбарт 10 мөр нэгээс олон алдаатай (хамгийн ихдээ 3) тул эхний алдаа бусдыгаа нууж (error masking) олон алдаатай мөр юуг шалгаж буй нь тодорхойгүй болдог. Хязгаарлалттай хувилбарт ийм мөр 0, ~ хэрэглэсэн хувилбарт 0 байна.

Нэг мөрт хамгийн ихдээ нэг сөрөг утга ордог тул ~ хэрэглэхэд A2-ын хоёр хязгаарлалт хэрэгцээгүй болсон: тэдгээр нь зөвхөн алдааг бие биенээс нь тусгаарлах зорилготой байсан бөгөөд ~ үүнийг өөрөө хийдэг. Харин алдаа ба хүчинтэй утгын хоорондын бодит семантик хамаарал (жишээ нь SameAsFrom нь from хүчинтэй үед л утгатай) байвал хязгаарлалт хэрэгтэй хэвээр.

## Бодит тест кейсүүд (PICT мөрөөс)

Давуу эрэмбэ: from → to → amount (Хэсэг A-д тогтоосон).

**TC-1 (happy path, transfer-neg)** — PICT мөр: Valid | Valid | Normal | Same

- Setup: A001 (USD, идэвхтэй, үлдэгдэл 1000, өдрийн лимит 2000, өнөөдөр 0 ашигласан); A002 идэвхтэй; A003 царцаагдсан; A998, A999 байхгүй; ханш 1 EUR = 1.1 USD
- Оролт: POST /transfer {"from":"A001","to":"A002","amount":100,"currency":"USD"}
- Oracle: 201 {"result":"OK"}

**TC-2 (transfer-neg: from алдаатай)** — PICT мөр: Missing | Valid | Normal | Same

- Setup: A001 (USD, идэвхтэй, үлдэгдэл 1000, өдрийн лимит 2000, өнөөдөр 0 ашигласан); A002 идэвхтэй; A003 царцаагдсан; A998, A999 байхгүй; ханш 1 EUR = 1.1 USD
- Оролт: POST /transfer {"from":"A999","to":"A002","amount":100,"currency":"USD"}
- Oracle: 200 {"result":"ERROR_NO_FROM"}

**TC-3 (transfer-neg: amount алдаатай)** — PICT мөр: Valid | Valid | OverBalance | Different

- Setup: A001 (USD, идэвхтэй, үлдэгдэл 1000, өдрийн лимит 2000, өнөөдөр 0 ашигласан); A002 идэвхтэй; A003 царцаагдсан; A998, A999 байхгүй; ханш 1 EUR = 1.1 USD
- Оролт: POST /transfer {"from":"A001","to":"A002","amount":1500,"currency":"EUR"}
- Oracle: 200 {"result":"ERROR_INSUFFICIENT"}

**TC-4 (transfer-nc: олон алдаатай, давуу эрэмбээр)** — PICT мөр: Frozen | Valid | OverBalance | Same

- Setup: A001 (USD, идэвхтэй, үлдэгдэл 1000, өдрийн лимит 2000, өнөөдөр 0 ашигласан); A002 идэвхтэй; A003 царцаагдсан; A998, A999 байхгүй; ханш 1 EUR = 1.1 USD
- Оролт: POST /transfer {"from":"A003","to":"A002","amount":1500,"currency":"USD"}
- Oracle: 200 {"result":"ERROR_FROZEN"}

TC-4-ийн oracle нь давуу эрэмбээр (from → to → amount) эхний олдсон алдааг буцаана гэсэн дүрмээс гарсан.

## B3. Хоёр аргын харьцуулалт

Гараар хийсэн A3 хүснэгт (13 спецификаци) алдааны семантикийг сайн барьдаг: алдааны анги бүр, давуу эрэмбэ, хөрвүүлсний дараах дүнгээр шалгах зэрэг нарийн тохиолдлын oracle тодорхой. Харин PICT нь параметрүүдийн ХОСЫН хамралтыг математикаар баталгаажуулдаг ч (энэ нь мартагдсан хослолыг хүн гараар оруулахаас найдвартай), үр дүн нь зөвхөн утгын хослол бөгөөд oracle өгдөггүй. Иймд олон алдаатай мөрийн хүлээгдэх үр дүнг хүн давуу эрэмбээр тогтоох ёстой.

## Дүгнэлт

Энэ лабораторид шилжүүлгийн функцэд category-partition ба PICT гэсэн хоёр аргаар тест дизайн хийсэн. Category-partition-аар 90 хослолыг 10 болгон багасгаж, давхар нөхцөлтэй нийт 13 спецификаци гаргасан. PICT-ийн хязгаарлалтгүй хувилбар 13 мөр өгсөн ч 10 нь нэгээс олон алдаатай тул error masking-ээс болж утга учиргүй байв. Хязгаарлалттай хувилбар 14 мөр, ~ сөрөг утгатай хувилбар 16 мөр гарсан бөгөөд хоёуланд нь нэг мөрт хамгийн ихдээ 1 ба 1 алдаа орсон. Браузерын жишээнд 2-way 0 мөр, 3-way 0 мөр гарсан нь 144 мөрийн бүрэн олонлогоос хамаагүй цөөн. PICT нь greedy алгоритм тул гараар олсон хамгийн цөөн хэмжээнээс (9) заримдаа илүү мөр гаргадаг. Гараар хийсэн хүснэгт алдааны семантик ба oracle-ийг өгдөг, PICT хосын хамралтыг автоматаар баталгаажуулдаг тул хоёр арга бие биеэ нөхдөг. Хамгийн их бодол шаардсан нь олон алдааны давуу эрэмбийг тогтоох ба хязгаарлалтыг A2-ын үндэслэлтэйгээр PICT хэл рүү буулгах алхам байлаа. Хамгийн хэрэгтэй нь ~ нь category-partition-ий [ERROR] тэмдэглэгээтэй яг дүйх нь байлаа. Цаашид эхлээд category-partition-аар сонголт, хязгаарлалтаа тодорхойлоод, дараа нь PICT-ээр хослолыг автоматаар үүсгэх нь үр дүнтэй гэж үзэж байна.
