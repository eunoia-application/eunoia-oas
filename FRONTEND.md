# Eunoia API types для фронтенда

Фронт потребляет **один** npm-пакет — `@eunoia-application/api-types`. В нём уже лежат
сгенерённые TypeScript-типы на весь REST API (эндпоинты auth/user + общие схемы).
**Кодогенерации на стороне фронта не нужно** — типы готовы к импорту.

> Почему один пакет, а не 3 по числу Maven-модулей: фронт — одно приложение,
> говорящее со всем API сразу. Склейку спек и генерацию типов делаем один раз при
> публикации (см. `api-types/scripts/build-types.mjs`), а наружу отдаём самодостаточный
> набор типов + bundled `openapi.json`.
>
> Kafka-события (бэкендовые контракты микросервисов) в пакет НЕ входят — фронту не нужны.

---

## 1. Доступ к GitHub Packages

Пакет лежит в GitHub Packages (npm registry), нужен токен с правом `read:packages`.

`.npmrc` в корне фронт-проекта:

```ini
# ТОЛЬКО scope @eunoia-application уходит в GitHub Packages.
# НЕ добавляйте глобальный `registry=` — иначе react/axios/... начнут 404-ить.
@eunoia-application:registry=https://npm.pkg.github.com
//npm.pkg.github.com/:_authToken=${GITHUB_TOKEN}
```

Токен — через переменную окружения (не коммитим):

```bash
export GITHUB_TOKEN=ghp_xxx   # classic PAT с read:packages
```

В CI фронта используйте `secrets.GITHUB_TOKEN` (если фронт в той же org) или
отдельный PAT.

## 2. Установка

```bash
npm install -D @eunoia-application/api-types
```

Это dev-зависимость: пакет содержит только `.d.ts` + `openapi.json`, в рантайм-бандл
ничего не утекает (типы стираются при компиляции).

## 3. Использование типов

Пакет экспортирует стандартные для `openapi-typescript` неймспейсы
`components`, `paths`, `operations`.

```typescript
import type { components, paths } from '@eunoia-application/api-types';

// Схемы — самое нужное:
type AuthResponse = components['schemas']['AuthResponse']; // ответ login/register: токены + user
type AuthUser = components['schemas']['AuthUser'];         // слим-идентичность в ответе auth
type UserProfile = components['schemas']['UserProfile'];   // полный профиль (service-user)
type UserSettings = components['schemas']['UserSettings'];
```

Удобно завести барель-алиасы у себя (`src/shared/api/schema.ts`):

```typescript
import type { components } from '@eunoia-application/api-types';

export type Schemas = components['schemas'];
export type UserProfile = Schemas['UserProfile'];
export type AuthResponse = Schemas['AuthResponse'];
// ...
```

### Типобезопасный запрос/ответ по path

```typescript
import type { paths } from '@eunoia-application/api-types';

type GetMe     = paths['/users/me']['get'];
type MyProfile = GetMe['responses']['200']['content']['application/json'];    // UserProfile

type UpdateMe   = paths['/users/me']['put'];
type UpdateBody = UpdateMe['requestBody']['content']['application/json'];      // UserUpdateRequest
```

### Пример с axios

```typescript
import axios from 'axios';
import type { components } from '@eunoia-application/api-types';

type AuthResponse = components['schemas']['AuthResponse'];
type UserProfile = components['schemas']['UserProfile'];

const api = axios.create({ baseURL: import.meta.env.VITE_API_BASE_URL ?? 'http://localhost:7777/api/v1' });

export async function login(email: string, password: string) {
  const { data } = await api.post<AuthResponse>('/auth/login', { email, password });
  return data;
}

export async function getMyProfile() {
  const { data } = await api.get<UserProfile>('/users/me');
  return data;
}
```

> Пути в типах — как в контрактах (`/auth/*`, `/users/*`), без хоста.
> Базовый URL задаёт axios-клиент (`baseURL` — через gateway, напр. `/api/v1`).
> Публичные auth-эндпоинты (`/auth/login`, `/auth/register`, …) не требуют токена;
> остальные — `Bearer <JWT>`.

## 4. Рантайм-спека (опционально)

Если нужен сам OpenAPI-документ (MSW-моки, рантайм-валидация, Swagger UI):

```typescript
import spec from '@eunoia-application/api-types/openapi.json' with { type: 'json' };
```

Это самодостаточный bundled-спек (shared-схемы уже инлайн, внешних `$ref` нет).

## Версионирование

`@eunoia-application/api-types` версионируется в lockstep с Maven-контрактами
(`2.x` ↔ контракты `2.x`), версия деривится из корневого `pom.xml` `<revision>`.
Pin как обычно:

```json
{ "devDependencies": { "@eunoia-application/api-types": "^2.0.0" } }
```

## Troubleshooting

| Симптом | Причина / фикс |
|---|---|
| `401 Unauthorized` при install | Нет/протух `GITHUB_TOKEN`, либо нет права `read:packages`. |
| `404 Not Found` на `@eunoia-application/api-types` | Пакет ещё не опубликован, либо опечатка в scope в `.npmrc`. |
| `react`/`axios` дают `404` | В `.npmrc` затесался глобальный `registry=https://npm.pkg.github.com` — уберите, оставьте только scoped-строку. |
| Типы не обновились после релиза | Бампнут ли `<revision>`? Переустановите: `npm install @eunoia-application/api-types@latest`. |
