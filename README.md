# TrustEDU
### Кейс реализован в рамках UMIRhack ll
**Презентация команды:** [Miss-Click.pdf](Miss-Click.pdf).
**Продакшн-версия проекта**: **https://miss-click.ru**

TrustEDU - это платформа для проверки подлинности дипломов и безопасного обмена подтверждением образования.
Платформа выступаем первоисточником информации, и должна внедрятся как ВУЗам так и Работодателям

- проверка диплома по номеру и ФИО;
- роли с разными сценариями работы: студент, вуз, администратор, HR;
- публичный API для интеграции проверки дипломов в HR-системы;
- выпуск и отзыв QR-токенов для быстрого подтверждения диплома;
- защита данных диплома (шифрование номера, шифрование персональных полей, подпись).

## Оглавление

1. [Стек](#stack)
2. [Безопасность](#security)
3. [Быстрый запуск (с сабмодулями)](#quick-start)
4. [Структура проекта (кратко)](#project-structure)
5. [Список участников и роли](#team)
6. [Ссылка на сайт](#site)



**Примеры импортируемых файлов из корня проекта:**

- [CSV Файл](ForTest.csv)
- [Excel Файл](ForTest.xlsx)

<a id="stack"></a>

## 1. Стек

- Frontend: React 19, TypeScript, Vite, Mantine UI, Redux Toolkit (RTK Query), React Router.
- Backend: NestJS 11, TypeScript, Prisma 7, PostgreSQL 15, Swagger.
- Безопасность и криптография: JWT, bcrypt, AES-256-CBC, RSA (SHA-256).
- Инфраструктура: Docker, Docker Compose, Git submodules.

<a id="security"></a>

## 2. Безопасность

- Шифрование персональных данных диплома: ФИО и регистрационный номер хранятся в зашифрованном виде (AES-256-CBC, симметричный ключ `DIPLOMA_SYMMETRIC_KEY`).
- Защита номера диплома: для поиска в БД используется SHA-256 хеш номера, а не открытое значение.
- Цифровая подпись диплома: при создании диплома формируется payload и подписывается приватным RSA-ключом вуза.
- Проверка подлинности по ключам: при запросе диплома подпись проверяется публичным ключом вуза (подпись, созданная приватным ключом, должна валидироваться публичным).
- Контроль доступа: глобальные guard-ы JWT + roles (`JwtAccessGuard`, `RolesGuard`) ограничивают доступ к защищенным endpoint-ам по ролям.
- Защита от перебора и перегрузки:
  - глобальный throttling (`ThrottlerGuard`) по умолчанию 10 запросов в минуту;
  - отдельный лимит для блока дипломов (`@Throttle`) 15 запросов в минуту;
  - защита поиска диплома: после 5 неудачных попыток с одного IP блокировка на 1 минуту.
- Безопасная сессия: access/refresh токены передаются в `HttpOnly` cookies, refresh-токен хранится в БД в виде bcrypt-хеша.
- Валидация входных данных: `ValidationPipe` с `whitelist` и `forbidNonWhitelisted` отбрасывает лишние поля и снижает риск некорректных/вредоносных payload.

<a id="quick-start"></a>

## 3. Быстрый запуск (с сабмодулями)

Сайт уже развернут и доступен на: **https://miss-click.ru**

### 3.1 Клонирование и инициализация сабмодулей

```bash
git clone --recurse-submodules <repo-url>
cd umirhack-ll
```

Если репозиторий уже клонирован без сабмодулей:

```bash
git submodule update --init --recursive
```

Обновить сабмодули до последних коммитов ветки `main`:

```bash
git submodule sync --recursive
git submodule update --remote --recursive --merge
```

Проверить отслеживаемые ветки сабмодулей:

```bash
git config -f .gitmodules --get-regexp "^submodule\..*\.branch$"
```

### 3.2 Запуск backend (NestJS + Prisma + PostgreSQL)

Требования:

- Node.js 20+
- npm 10+
- Docker + Docker Compose

```bash
cd backend
cp .env.example .env
docker compose up -d
npm install
npx prisma migrate dev
npm run db:seed
npm run start:dev
```

После копирования `.env.example` в `.env` обновите значения для локального запуска (хостовая машина + локальный Docker Postgres):

```env
DATABASE_URL=postgresql://admin:admin@localhost:5432/backend_db
POSTGRES_USER=admin
POSTGRES_PASSWORD=admin
POSTGRES_DB=backend_db
COOKIE_SECURE=false
COOKIE_SAMESITE=lax
```

Backend будет доступен на `http://localhost:3000`, Swagger - на `http://localhost:3000/docs`.

### 3.3 Запуск frontend (React + Vite)

```bash
cd frontend
cp .env.example .env
npm install
npm run dev
```

Frontend будет доступен на `http://localhost:5173`.

### 3.4 Mobile

Мобильный клиент подключен как отдельный сабмодуль `mobile`.

<a id="project-structure"></a>

## 4. Структура проекта (кратко)

Проект состоит из трех сабмодулей:

- `frontend` - веб-приложение.
- `backend` - Backend проект.
- `mobile` - мобильный клиент.

### Frontend

Используется архитектура FSD

### Backend

Ключевые модули:

- `auth` - регистрация, вход, refresh/logout, cookie-based JWT;
- `users` - пользователи, роли, верификация заявок от вузов; 
- `diplomas` - создание, поиск, отзыв, QR-токены дипломов;
- `public-api-servise` - публичные endpoints для работодателей:
- `crypto` - подпись и проверка криптографических данных;
- `prisma` - доступ к БД и модель данных.

<a id="team"></a>

## 5. Список участников и роли

Ниже список сформирован по фактической активности в репозиториях (по истории коммитов):

| Участник          | Роль в проекте        |
| ----------------- | --------------------- |
| [Артем Кирюшин](https://github.com/Lyaguh21)     | Team Lead, Fullstack, |
| [Георгий Щербинин](https://github.com/Georgiy5)  | Frontend              |
| [Алиса Завеса](https://github.com/lisazavesa)      | Backend Developer     |
| [Гайдин Даниил](https://github.com/Treew666)     | Backend Developer     |
| [Светличный Никита](https://github.com/bybuss) | Mobile Developer      |

<p align="center">
  <img src="MissClickTeam.png"/>
</p>

<a id="site"></a>

## 6. Ссылка на сайт и приложение

Продакшн-версия проекта: **https://miss-click.ru**
Мобильное приложение: **https://www.rustore.ru/catalog/app/bob.colbaskin.umir_hack_2**
