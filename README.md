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
