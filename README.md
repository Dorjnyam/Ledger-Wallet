# General Ledger & Wallet Service

> Давхар бичилтийн нягтлан бодох бүртгэлийн зарчим дээр суурилсан, Go хэл дээр бичигдсэн, PostgreSQL ашигласан санхүүгийн ерөнхий дэвтэр систем.

![Go](https://img.shields.io/badge/Go-1.22+-00ADD8?style=for-the-badge&logo=go&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16+-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)

---

## 📋 Агуулга

- [Төслийн Тухай](#-төслийн-тухай)
- [Архитектур](#-архитектур)
- [Технологийн Стек](#-технологийн-стек)
- [Эхлүүлэх Заавар](#-эхлүүлэх-заавар)
- [API Лавлагаа](#-api-лавлагаа)
- [Нягтлан Бодох Бүртгэлийн Ойлголтууд](#-нягтлан-бодох-бүртгэлийн-ойлголтууд)
- [Дизайн Шийдвэрүүд](#-дизайн-шийдвэрүүд)
- [Тестүүд](#-тестүүд)
- [Төслийн Бүтэц](#-төслийн-бүтэц)
- [Баримтжуулалт](#-баримтжуулалт)

---

## 🎯 Төслийн Тухай

Энэхүү систем нь **давхар бичилтийн нягтлан бодох бүртгэл** (double-entry bookkeeping) дээр суурилсан **ерөнхий дэвтэр** (general ledger) бөгөөд дээр нь **түрийвч** (wallet) давхаргыг бүтээсэн.

### Гол Зарчим

| Зарчим | Тайлбар |
|---|---|
| 🔒 **Өөрчлөгдөшгүй байдал** | Бүртгэлийн бичилт (entry) хэзээ ч засагдах, устгагдахгүй — зөвхөн буцаалт хийнэ |
| ⚖️ **Дебит = Кредит** | Бүх гүйлгээнд нийт дебит нь нийт кредиттэй тэнцүү байх ёстой |
| 🧮 **Тооцоолсон үлдэгдэл** | Түрийвчний үлдэгдэл нь бичилтүүдийн нийлбэрээс тооцогдоно, тусдаа хадгалагддаггүй |
| 🔑 **Идемпотент үйлдлүүд** | Ижил idempotency key-тэй давтагдсан хүсэлт нь давхар гүйлгээ үүсгэхгүй |
| 🔐 **Зэрэгцээ аюулгүй байдал** | `SELECT … FOR UPDATE` мөр түвшний түгжээгээр race condition-оос сэргийлнэ |

### Юу Хийж Чадах Вэ?

- ✅ 5 төрлийн данс үүсгэх (Хөрөнгө, Өр төлбөр, Өмч, Орлого, Зардал)
- ✅ Давхар бичилтийн гүйлгээ бүртгэх
- ✅ Түрийвч рүү мөнгө оруулах (deposit)
- ✅ Түрийвчнээс мөнгө гаргах (withdraw)
- ✅ Түрийвч хооронд шилжүүлэг хийх (transfer)
- ✅ Одоогийн болон түүхэн үлдэгдэл лавлах
- ✅ Гүйлгээний түүх харах

---

## 🏗 Архитектур

### Системийн Бүтэц

```mermaid
graph TD
    Client["🖥️ HTTP Клиент<br/>(curl, Postman, frontend)"] -->|"REST JSON<br/>:8080"| API["🌐 API Давхарга<br/>(chi router + handlers)"]

    subgraph "Go Аппликейшн"
        API -->|"түрийвч үйлдлүүд"| WS["💰 Түрийвч Үйлчилгээ<br/>(deposit, withdraw, transfer)"]
        API -->|"гүйлгээ & данс"| LS["📒 Дэвтэр Үйлчилгээ<br/>(PostTransaction, Balance)"]
        WS -->|"давхар бичилтийн<br/>гүйлгээ үүсгэнэ"| LS
        LS -->|"SQL query"| Store["🗄️ Хадгалалтын Давхарга<br/>(pgx/v5 raw SQL)"]
    end

    Store -->|"pgx connection pool"| DB[("🐘 PostgreSQL")]
```

### Давхаргын Хамаарал

```mermaid
graph BT
    Domain["📦 domain<br/>(цэвэр Go төрлүүд)"] 
    Store["🗄️ store<br/>(PostgreSQL persistence)"] --> Domain
    Ledger["📒 ledger<br/>(давхар бичилтийн цөм)"] --> Domain
    Ledger --> Store
    Wallet["💰 wallet<br/>(түрийвч үйлдлүүд)"] --> Domain
    Wallet --> Ledger
    API["🌐 api<br/>(HTTP handlers)"] --> Domain
    API --> Ledger
    API --> Wallet
    Main["🚀 cmd/server<br/>(эхлэл цэг)"] --> API
```

> **Хамаарлын дүрэм:** Хамаарал нь зөвхөн **доошоо** чиглэнэ. `domain` нь юунаас ч хамаарахгүй. `store` нь зөвхөн `domain`-оос хамаарна. Дээшээ хамаарал хэзээ ч байхгүй.

### Өгөгдлийн Сангийн Схем

```mermaid
erDiagram
    ACCOUNTS ||--o{ ENTRIES : "олон бичилттэй"
    TRANSACTIONS ||--|{ ENTRIES : "агуулна"

    ACCOUNTS {
        uuid id PK
        text name "Дансны нэр"
        text type "asset | liability | equity | revenue | expense"
        char3 currency "ISO 4217 (жнь: USD, MNT)"
        timestamptz created_at
    }

    TRANSACTIONS {
        uuid id PK
        text idempotency_key UK "Давтагдашгүй түлхүүр"
        text description "Тайлбар"
        timestamptz posted_at
        timestamptz created_at
    }

    ENTRIES {
        uuid id PK
        uuid transaction_id FK
        uuid account_id FK
        text direction "debit | credit"
        bigint amount "Бага нэгж (цент), үргэлж > 0"
        timestamptz created_at
    }
```

---

## 🛠 Технологийн Стек

| Технологи | Сонголт | Яагаад |
|---|---|---|
| **Хэл** | Go 1.22+ | Даалгаврын шаардлага; concurrency-д хүчтэй |
| **Өгөгдлийн сан** | PostgreSQL 16+ | Даалгаврын шаардлага; ACID гүйлгээ, row-level locking |
| **DB драйвер** | `pgx/v5` | Цэвэр Go, өндөр гүйцэтгэл, native PostgreSQL төрлүүд |
| **HTTP router** | `chi` | stdlib-тэй нийцтэй, middleware дэмжлэг, хялбар |
| **Миграц** | `golang-migrate` | SQL файл суурилсан, шидгүй, хяналттай |
| **Мөнгөний төрөл** | `int64` бага нэгж (цент) | `float64` **хэзээ ч** ашиглахгүй — IEEE 754 бөөрөнхийлөлт санхүүд зөвшөөрөгдөхгүй |
| **Контейнер** | Docker + Docker Compose | Нэг команд-аар ажиллуулах |

---

## 🚀 Эхлүүлэх Заавар

### Шаардлагатай Хэрэгслүүд

| Хэрэгсэл | Хамгийн Бага Хувилбар |
|---|---|
| Docker | 24+ |
| Docker Compose | v2+ |
| Go *(хөгжүүлэлтэнд)* | 1.22+ |

### Docker Compose-оор Ажиллуулах (Зөвлөмж)

```bash
# 1. Репозиторийг clone хийх
git clone <repo-url>
cd ledger-wallet

# 2. Бүгдийг эхлүүлэх (PostgreSQL + Go сервис)
docker compose up --build

# 3. Логоос "Server started on :8080" гарч ирэхийг хүлээх

# 4. API-г шалгах
curl http://localhost:8080/v1/accounts
```

**Ингээд л болоо.** Docker Compose нь:
- PostgreSQL-г эхлүүлнэ
- Өгөгдлийн сангийн миграцийг ажиллуулна
- Go сервисийг бүтээж, эхлүүлнэ

### Гараар Ажиллуулах (Docker-гүйгээр)

```bash
# 1. PostgreSQL суулгаж, өгөгдлийн сан үүсгэх
createdb ledger

# 2. Орчны хувьсагчдыг тохируулах
export DATABASE_URL="postgres://postgres:postgres@localhost:5432/ledger?sslmode=disable"
export SERVER_PORT="8080"

# 3. Бүтээж ажиллуулах
go build -o ledger-wallet ./cmd/server
./ledger-wallet
```

---

## 📡 API Лавлагаа

**Үндсэн URL:** `http://localhost:8080/v1`

### Дансны Эндпойнтууд

| Арга | Зам | Тайлбар |
|---|---|---|
| `POST` | `/v1/accounts` | Шинэ данс үүсгэх |
| `GET` | `/v1/accounts/:id` | Дансны мэдээлэл харах |

#### Данс Үүсгэх

```bash
curl -X POST http://localhost:8080/v1/accounts \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Бэлэн мөнгө",
    "type": "asset",
    "currency": "MNT"
  }'
```

**Хариу (201 Created):**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "name": "Бэлэн мөнгө",
  "type": "asset",
  "currency": "MNT",
  "created_at": "2026-08-20T10:00:00Z"
}
```

### Гүйлгээний Эндпойнт

| Арга | Зам | Тайлбар |
|---|---|---|
| `POST` | `/v1/transactions` | Давхар бичилтийн гүйлгээ бүртгэх |

```bash
curl -X POST http://localhost:8080/v1/transactions \
  -H "Content-Type: application/json" \
  -d '{
    "idempotency_key": "txn-001",
    "description": "Анхны орлого",
    "entries": [
      {"account_id": "<cash-uuid>", "direction": "debit", "amount": 100000},
      {"account_id": "<wallet-uuid>", "direction": "credit", "amount": 100000}
    ]
  }'
```

> **Тэмдэглэл:** `amount` нь **бага нэгжээр** (цент) тэмдэглэгдэнэ. `100000` = ₮1,000.00

### Үлдэгдлийн Эндпойнтууд

| Арга | Зам | Тайлбар |
|---|---|---|
| `GET` | `/v1/accounts/:id/balance` | Одоогийн үлдэгдэл |
| `GET` | `/v1/accounts/:id/balance?at=<ISO8601>` | Түүхэн үлдэгдэл (тодорхой хугацааны) |
| `GET` | `/v1/accounts/:id/transactions` | Гүйлгээний түүх |

```bash
# Одоогийн үлдэгдэл
curl http://localhost:8080/v1/accounts/<account-uuid>/balance

# 2026-08-15 өдрийн үлдэгдэл (түүхэн)
curl "http://localhost:8080/v1/accounts/<account-uuid>/balance?at=2026-08-15T12:00:00Z"
```

### Түрийвчний Эндпойнтууд

| Арга | Зам | Тайлбар |
|---|---|---|
| `POST` | `/v1/wallets/deposit` | Түрийвч рүү мөнгө оруулах |
| `POST` | `/v1/wallets/withdraw` | Түрийвчнээс мөнгө гаргах |
| `POST` | `/v1/wallets/transfer` | Түрийвч хооронд шилжүүлэх |

#### Мөнгө Оруулах (Deposit)

```bash
curl -X POST http://localhost:8080/v1/wallets/deposit \
  -H "Content-Type: application/json" \
  -d '{
    "wallet_id": "<wallet-uuid>",
    "amount": 100000,
    "idempotency_key": "dep-001",
    "description": "Цалингийн орлого"
  }'
```

#### Мөнгө Гаргах (Withdraw)

```bash
curl -X POST http://localhost:8080/v1/wallets/withdraw \
  -H "Content-Type: application/json" \
  -d '{
    "wallet_id": "<wallet-uuid>",
    "amount": 50000,
    "idempotency_key": "wd-001",
    "description": "ATM-ээс авсан"
  }'
```

#### Шилжүүлэг Хийх (Transfer)

```bash
curl -X POST http://localhost:8080/v1/wallets/transfer \
  -H "Content-Type: application/json" \
  -d '{
    "source_wallet_id": "<alice-uuid>",
    "destination_wallet_id": "<bob-uuid>",
    "amount": 25000,
    "idempotency_key": "xfer-001",
    "description": "Бат-д шилжүүлэг"
  }'
```

### Алдааны Хариуны Формат

Бүх алдаа нь нэгдсэн JSON бүтэцтэй:

```json
{
  "error": {
    "code": "INSUFFICIENT_BALANCE",
    "message": "Түрийвчний үлдэгдэл 50000 нь 99999 гаргахад хүрэлцэхгүй байна"
  }
}
```

| Алдааны Код | HTTP Статус | Утга |
|---|---|---|
| `INVALID_REQUEST` | 400 | Буруу формат эсвэл шаардлагатай талбар дутуу |
| `NOT_FOUND` | 404 | Данс эсвэл гүйлгээ олдсонгүй |
| `DUPLICATE_IDEMPOTENCY_KEY` | 409 | Идемпотент түлхүүр давтагдсан; өмнөх хариуг буцаана |
| `UNBALANCED_TRANSACTION` | 422 | Нийт дебит ≠ нийт кредит |
| `INSUFFICIENT_BALANCE` | 422 | Түрийвчний үлдэгдэл хүрэлцэхгүй |
| `INTERNAL_ERROR` | 500 | Серверийн дотоод алдаа |

---

## 📚 Нягтлан Бодох Бүртгэлийн Ойлголтууд

### Таван Төрлийн Данс

Давхар бичилтийн нягтлан бодох бүртгэлд данс бүр дараах 5 төрлийн аль нэгэнд хамаарна:

| Дансны Төрөл | Хэвийн Үлдэгдэл | Дебитээр | Кредитээр | Жишээ |
|---|---|---|---|---|
| **Хөрөнгө** (Asset) | Дебит | Нэмэгдэнэ ↑ | Багасна ↓ | Бэлэн мөнгө, Банкны данс |
| **Өр Төлбөр** (Liability) | Кредит | Багасна ↓ | Нэмэгдэнэ ↑ | Зээл, **Хэрэглэгчийн түрийвч** |
| **Өмч** (Equity) | Кредит | Багасна ↓ | Нэмэгдэнэ ↑ | Эзэмшигчийн хөрөнгө |
| **Орлого** (Revenue) | Кредит | Багасна ↓ | Нэмэгдэнэ ↑ | Борлуулалт, Шимтгэл |
| **Зардал** (Expense) | Дебит | Нэмэгдэнэ ↑ | Багасна ↓ | Түрээс, Цалин |

### Яагаад Түрийвч нь Өр Төлбөрийн Данс Вэ?

**Платформын** өнцгөөс харвал, хэрэглэгчийн түрийвчний үлдэгдэл нь платформ тухайн хэрэглэгчид **өгөх өртэй** мөнгийг илэрхийлнэ. Тиймээс түрийвч нь **Өр Төлбөр** (Liability) данс юм.

### Нягтлан Бодох Бүртгэлийн Тэгшитгэл

```
Хөрөнгө = Өр Төлбөр + Өмч
```

Бүх зөв давхар бичилтийн гүйлгээ нь энэ тэгшитгэлийг хадгална. `Нийт Дебит = Нийт Кредит` байгаа тохиолдолд тэгшитгэл **хэзээ ч** алдагдахгүй.

### Үйлдэл Бүрийн Дебит/Кредит Зураглал

#### 💰 Мөнгө Оруулах (Deposit)

Платформ бэлэн мөнгө хүлээн авч, хэрэглэгчид өр үүсгэнэ.

```
Дебит:  Бэлэн мөнгө (Хөрөнгө)     +₮1,000  ← Платформ мөнгөтэй болов
Кредит: Түрийвч:Бат (Өр төлбөр)     +₮1,000  ← Платформ Бат-д өртэй болов
```

```mermaid
graph LR
    Cash["🏦 Бэлэн мөнгө<br/>(Хөрөнгө)<br/>ДЕБИТ +₮1,000"] --> TX["📄 Гүйлгээ"]
    TX --> Wallet["👛 Түрийвч:Бат<br/>(Өр төлбөр)<br/>КРЕДИТ +₮1,000"]
```

#### 💸 Мөнгө Гаргах (Withdrawal)

Платформ бэлэн мөнгөө шилжүүлж, өрөө бууруулна.

```
Дебит:  Түрийвч:Бат (Өр төлбөр)    -₮500   ← Платформын өр багасав
Кредит: Бэлэн мөнгө (Хөрөнгө)     -₮500   ← Платформын мөнгө багасав
```

```mermaid
graph LR
    Wallet["👛 Түрийвч:Бат<br/>(Өр төлбөр)<br/>ДЕБИТ -₮500"] --> TX["📄 Гүйлгээ"]
    TX --> Cash["🏦 Бэлэн мөнгө<br/>(Хөрөнгө)<br/>КРЕДИТ -₮500"]
```

#### 🔄 Шилжүүлэг (Transfer)

Платформын нийт өр өөрчлөгдөхгүй — зөвхөн нэг хэрэглэгчээс нөгөөд шилжинэ.

```
Дебит:  Түрийвч:Бат (Өр төлбөр)    -₮250   ← Бат-ын үлдэгдэл буурна
Кредит: Түрийвч:Сарнай (Өр төлбөр) +₮250   ← Сарнайн үлдэгдэл нэмэгдэнэ
```

```mermaid
graph LR
    WA["👛 Түрийвч:Бат<br/>(Өр төлбөр)<br/>ДЕБИТ -₮250"] --> TX["📄 Гүйлгээ"]
    TX --> WB["👛 Түрийвч:Сарнай<br/>(Өр төлбөр)<br/>КРЕДИТ +₮250"]
```

### Үлдэгдлийн Тооцооллын Томъёо

Үлдэгдэл нь **хэзээ ч** тусдаа хадгалагддаггүй — бичилтүүдээс тооцоологдоно:

```sql
-- Хөрөнгө, Зардал дансууд (хэвийн үлдэгдэл = дебит):
үлдэгдэл = SUM(дебит дүн) - SUM(кредит дүн)

-- Өр төлбөр, Өмч, Орлого дансууд (хэвийн үлдэгдэл = кредит):
үлдэгдэл = SUM(кредит дүн) - SUM(дебит дүн)
```

**Түүхэн үлдэгдэл** тооцохдоо бичилтүүдийг `posted_at <= хүссэн_хугацаа` гэж шүүнэ.

---

## 🧠 Дизайн Шийдвэрүүд

### 1. Өөрчлөгдөшгүй Байдал (Immutability)

| Арга | Хэрэгжүүлэлт |
|---|---|
| **Өгөгдлийн сангийн түвшинд** | `REVOKE UPDATE, DELETE ON entries FROM PUBLIC` — миграц дотор |
| **Кодын түвшинд** | `entries` хүснэгтэд UPDATE/DELETE SQL бичигдээгүй |
| **Залруулга** | Буцаалтын гүйлгээ (reversal transaction) — дебит/кредитийг эсрэгээр бичнэ |

> Энэ нь зөвхөн "дүрмээр" биш, **архитектураар** хэрэгжсэн. Ямар ч SQL клиент `entries` хүснэгтийг өөрчлөх боломжгүй.

### 2. Зэрэгцээ Аюулгүй Байдал (Concurrency)

**Сонгосон арга:** `SELECT … FOR UPDATE` мөр түвшний түгжээ.

```mermaid
sequenceDiagram
    participant W1 as 🔵 Хүсэлт 1
    participant W2 as 🔴 Хүсэлт 2
    participant DB as 🐘 PostgreSQL

    W1->>DB: BEGIN
    W2->>DB: BEGIN
    W1->>DB: SELECT ... FOR UPDATE (мөрийг түгжинэ)
    Note right of W2: Хүсэлт 2 ХҮЛЭЭЖ байна
    W1->>DB: Үлдэгдэл хангалттай → INSERT entries
    W1->>DB: COMMIT (түгжээ суллагдана)
    W2->>DB: SELECT ... FOR UPDATE (одоо үргэлжилнэ)
    W2->>DB: Үлдэгдэл хүрэлцэхгүй → ROLLBACK
    Note right of W2: Хүсэлт 2 зөв татгалзагдлаа ✅
```

**Яагаад энэ аргыг сонгов:**
- Энгийн, таамаглах боломжтой
- Зөвхөн хэрэгтэй мөрүүдийг түгжинэ
- Шалгалт дээр тайлбарлахад хялбар

### 3. Идемпотент Байдал (Idempotency)

**Сонгосон арга:** Өгөгдлийн сангийн `UNIQUE` хязгаарлалт `transactions.idempotency_key` дээр.

```
Хүсэлт 1: {idempotency_key: "dep-001"} → 201 Created (шинэ гүйлгээ)
Хүсэлт 2: {idempotency_key: "dep-001"} → 200 OK (өмнөх хариуг буцаана)
```

Redis шаардлагагүй. Өгөгдлийн санд хадгалагдсан тул сервер дахин ачааллахад ч алдагдахгүй.

### 4. Мөнгөний Төлөөлөл

| Сонголт | Шийдвэр |
|---|---|
| `float64` | ❌ **ХЭЗЭЭ Ч ХЭРЭГЛЭХГҮЙ** — IEEE 754 бөөрөнхийлөлт |
| `shopspring/decimal` | ❌ Нэмэлт dependency, нэг валютад хэт их |
| **`int64` бага нэгж (цент)** | ✅ Яг тооцоо, dependency-гүй, хурдан |

`100000` = ₮1,000.00. API-ийн хил дээр хөрвүүлж, хэрэглэгчид "₮1,000.00" гэж харуулна.

---

## 🧪 Тестүүд

### Тест Ажиллуулах

```bash
# Бүх тестийг ажиллуулах
go test ./...

# Race detector-тэй ажиллуулах
go test -race ./...

# Интеграцийн тест (ажиллаж буй PostgreSQL шаардлагатай)
go test -tags=integration ./...
```

### Тестийн Матриц — Инвариант Баталгаа

| Тест | Инвариант | Хүлээгдэж буй Үр Дүн |
|---|---|---|
| Тэнцвэргүй гүйлгээ | Дебит ≠ Кредит → татгалзагдана | 422, бичилт үүсэхгүй |
| Тэнцвэртэй гүйлгээ | Дебит = Кредит → хүлээн авагдана | 201, бичилтүүд хадгалагдана |
| Хэтрэх гаргалга | Үлдэгдлээс их гаргах → татгалзагдана | 422, бичилт үүсэхгүй |
| Идемпотент давталт | Ижил key → давхар гүйлгээгүй | 200, бичилтийн тоо өөрчлөгдөхгүй |
| Зэрэгцээ гаргалга | 100 goroutine, ₮50 үлдэгдэлтэй → яг 50 нь амжилттай | Эцсийн үлдэгдэл = ₮0 |
| Түүхэн үлдэгдэл | T1, T2 цэгүүд дээр бичилт → T1-д лавлах | Зөвхөн T1 бичилтүүд тоологдоно |
| Өөрчлөгдөшгүй байдал | Entry UPDATE оролдлого | Өгөгдлийн сан татгалзана |

### Гарын Авлагын Демо Дараалал

`docker compose up`-ийн дараа:

```bash
# 1. Бэлэн мөнгөний данс үүсгэх (Хөрөнгө)
curl -s -X POST http://localhost:8080/v1/accounts \
  -H "Content-Type: application/json" \
  -d '{"name":"Бэлэн мөнгө","type":"asset","currency":"MNT"}' | jq

# 2. Бат-ын түрийвч үүсгэх (Өр төлбөр)
curl -s -X POST http://localhost:8080/v1/accounts \
  -H "Content-Type: application/json" \
  -d '{"name":"Түрийвч:Бат","type":"liability","currency":"MNT"}' | jq

# 3. ₮100,000 оруулах
curl -s -X POST http://localhost:8080/v1/wallets/deposit \
  -H "Content-Type: application/json" \
  -d '{"wallet_id":"<bat-uuid>","amount":10000000,"idempotency_key":"demo-1"}' | jq

# 4. Үлдэгдэл шалгах (= 10000000)
curl -s http://localhost:8080/v1/accounts/<bat-uuid>/balance | jq

# 5. Хэтрэх гаргалга (→ 422)
curl -s -X POST http://localhost:8080/v1/wallets/withdraw \
  -H "Content-Type: application/json" \
  -d '{"wallet_id":"<bat-uuid>","amount":99999999,"idempotency_key":"demo-fail"}' | jq

# 6. Давтагдсан idempotency key (→ өмнөх хариу)
curl -s -X POST http://localhost:8080/v1/wallets/deposit \
  -H "Content-Type: application/json" \
  -d '{"wallet_id":"<bat-uuid>","amount":10000000,"idempotency_key":"demo-1"}' | jq
```

---

## 📁 Төслийн Бүтэц

```
ledger-wallet/
├── cmd/
│   └── server/
│       └── main.go                   # Эхлэл цэг — dependency injection
├── internal/
│   ├── domain/                       # Цэвэр Go төрлүүд (гадаад хамааралгүй)
│   │   ├── account.go                # Account, AccountType
│   │   ├── transaction.go            # Transaction, Entry
│   │   └── money.go                  # Amount (int64 бага нэгж)
│   ├── store/                        # PostgreSQL persistence (pgx/v5)
│   │   ├── store.go                  # Store struct, connection pool
│   │   ├── account_store.go          # Данс CRUD
│   │   ├── transaction_store.go      # Гүйлгээ + бичилт INSERT
│   │   └── balance_store.go          # Үлдэгдлийн query-нүүд
│   ├── ledger/                       # Давхар бичилтийн цөм
│   │   ├── service.go                # PostTransaction, GetBalance
│   │   └── service_test.go           # Инвариант тестүүд
│   ├── wallet/                       # Түрийвч үйлдлүүд
│   │   ├── service.go                # Deposit, Withdraw, Transfer
│   │   └── service_test.go           # Хэтрэлт, идемпотент, зэрэгцээ тестүүд
│   └── api/                          # HTTP handlers
│       ├── router.go                 # chi router тохиргоо
│       ├── accounts_handler.go       # /v1/accounts эндпойнтууд
│       ├── transactions_handler.go   # /v1/transactions эндпойнт
│       ├── wallet_handler.go         # /v1/wallets/* эндпойнтууд
│       └── balance_handler.go        # /v1/accounts/:id/balance
├── migrations/
│   ├── 000001_create_accounts.up.sql
│   ├── 000001_create_accounts.down.sql
│   ├── 000002_create_transactions.up.sql
│   ├── 000002_create_transactions.down.sql
│   ├── 000003_create_entries.up.sql
│   └── 000003_create_entries.down.sql
├── docs/                             # Баримтжуулалт (SDLC фазаар)
├── docker-compose.yml
├── Dockerfile
├── go.mod
├── go.sum
└── README.md                         # ← Та одоо энэ файлыг уншиж байна
```

---

## 📖 Баримтжуулалт

Төслийн `docs/` хавтас нь **Програм Хангамж Хөгжүүлэлтийн Амьдралын Мөчлөг** (SDLC)-ийн фазуудаар бүтэцлэгдсэн:

```
docs/
├── 00-project-charter.md                 # Төслийн хүрээ, хязгаарлалтууд
├── ledger-wallet-project-manual.md       # Мастер лавлагаа
├── roadmap.md                            # Хувилбарын зорилтууд
├── v2-backlog.md                         # Ирээдүйн боломжууд
├── architecture-evaluation.md            # Техникийн шийдвэрийн бүртгэл
├── learning.md                           # Сурсан зүйлс & асуултуудын бүртгэл
│
├── phase-1-discovery/                    # Домайн & оролцогчид
│   ├── vision.md
│   ├── stakeholders.md
│   └── assumptions.md
├── phase-2-requirements/                 # Функциональ & чанарын шаардлагууд
│   ├── functional-requirements.md
│   └── non-functional-requirements.md
├── phase-3-analysis/                     # Өгөгдлийн загварчлал
│   ├── data-model.md
│   └── use-cases.md
├── phase-4-design/                       # Архитектур & API
│   ├── architecture.md
│   ├── api-spec.md
│   └── accounting-concepts.md
├── phase-5-development/                  # Хөгжүүлэлтийн төлөвлөгөө
│   ├── v1-plan.md
│   └── sprint-1.md
├── phase-6-testing/                      # Тестийн стратеги
│   ├── test-plan.md
│   └── bug-tracker.md
└── phase-7-deployment/                   # Байршуулалт
    ├── environment-setup.md
    └── deployment-runbook.md
```

---

## 📄 Лиценз

MIT License — дэлгэрэнгүйг [LICENSE](LICENSE) файлаас үзнэ үү.

---

<p align="center">
  <b>Дебет = Кредит тэнцвэрийг хэзээ ч зөрчдөггүй жижиг систем нь заримдаа алдаа гаргадаг олон функцтэй системээс хавьгүй илүү үнэ цэнэтэй.</b>
</p>
