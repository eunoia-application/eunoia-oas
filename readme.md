# 🍃 Eunoia OpenAPI Contracts

> Knowledge Garden API Contracts - `Design First`, `Validated`

[![GitHub Actions](https://github.com/eunoia-application/eunoia-oas/actions/workflows/deploy.yml/badge.svg)](https://github.com/eunoia-application/eunoia-oas/actions)
[![GitHub Packages](https://img.shields.io/badge/GitHub-Packages-blue)](https://github.com/eunoia-application/eunoia-oas/packages)
[![Java 17](https://img.shields.io/badge/Java-17-red.svg)](https://openjdk.org/projects/jdk/17/)
[![OpenAPI 3.0](https://img.shields.io/badge/OpenAPI-3.0-green.svg)](https://swagger.io/specification/)
[![npm](https://img.shields.io/badge/npm-@eunoia--application-blue.svg)](https://github.com/eunoia-application/eunoia-oas/packages)

Централизованное хранилище OpenAPI контрактов проекта Eunoia — Knowledge Garden для «выращивания» графа знаний (первая вертикаль — изучение языков).

## 📁 Структура проекта
```text
eunoia-oas/
├── shared-contract/   # Общие типы и компоненты
├── auth-contract/     # Контракты аутентификации и жизненного цикла аккаунта
├── user-contract/     # Контракты профиля пользователя
├── api-types/         # npm-пакет с TS-типами для фронта (генерируется)
├── .github/workflows/ # CI/CD пайплайны
├── pom.xml            # Parent + aggregator (lockstep <revision>)
├── redocly.yaml       # Конфиг OpenAPI-линта
└── readme.md
```

> Контракты заметок/тегов удалены как рудименты прежней идеи (проект развёрнут на Knowledge Garden).

## 🏗️ Дистрибуция

Один источник правды (YAML-спеки) публикуется в двух форматах:

- **Backend** — 3 Maven JAR с YAML внутри (по модулю на bounded context). Бэк
  генерит Spring-интерфейсы через `openapi-generator-maven-plugin`.
- **Frontend** — один npm-пакет `@eunoia-application/api-types` с готовыми `.d.ts` на
  весь REST API. Кодогенерации на стороне фронта не требуется. См. [`FRONTEND.md`](FRONTEND.md).

### Maven-модули (backend)

| Модуль | Maven Artifact | Описание | Версия |
|--------|----------------|----------|--------|
| **shared-contract** | `com.eunoia.application:shared-contract` | Общие схемы (UUID, ISO8601DateTime, ErrorResponse), общие responses | `2.0.0` |
| **auth-contract** | `com.eunoia.application:auth-contract` | API аутентификации и жизненного цикла аккаунта (Design B — только identity) | `2.0.0` |
| **user-contract** | `com.eunoia.application:user-contract` | API профиля пользователя (данные, аватар, настройки, экспорт) | `2.0.0` |

> Версии — lockstep: все модули публикуются одной версией (единый `<revision>` в корневом `pom.xml`).

### npm-пакет (frontend)

| Пакет | Содержимое | Версия |
|-------|------------|--------|
| **`@eunoia-application/api-types`** | TypeScript-типы (`components`/`paths`/`operations`) + bundled `openapi.json` на весь REST API | `2.0.0` |

## 🚀 Быстрый старт

### 1. Клонирование проекта
```bash
git clone https://github.com/eunoia-application/eunoia-oas.git
cd eunoia-oas
```

### 2. Использование в проектах (backend)
Добавьте в ваш [`pom.xml`](pom.xml):
```xml
<repositories>
    <repository>
        <id>github</id>
        <name>GitHub Packages</name>
        <url>https://maven.pkg.github.com/eunoia-application/eunoia-oas</url>
    </repository>
</repositories>

<dependencies>
    <dependency>
        <groupId>com.eunoia.application</groupId>
        <artifactId>auth-contract</artifactId>
        <version>2.0.0</version>
    </dependency>
    <dependency>
        <groupId>com.eunoia.application</groupId>
        <artifactId>user-contract</artifactId>
        <version>2.0.0</version>
    </dependency>
    <dependency>
        <groupId>com.eunoia.application</groupId>
        <artifactId>shared-contract</artifactId>
        <version>2.0.0</version>
    </dependency>
</dependencies>
```

### 2.1 Использование в React + TypeScript проектах (frontend)

Фронтенд ставит один пакет с готовыми типами:

```bash
npm install -D @eunoia-application/api-types
```

`.npmrc` (только scoped-registry — глобальный `registry=` НЕ добавлять):

```ini
@eunoia-application:registry=https://npm.pkg.github.com
//npm.pkg.github.com/:_authToken=${GITHUB_TOKEN}
```

```typescript
import type { components } from '@eunoia-application/api-types';
type UserProfile = components['schemas']['UserProfile'];
```

Подробная документация для фронтенда: [`FRONTEND.md`](FRONTEND.md)

### 3. Сборка проекта

Корневой `pom.xml` — parent + aggregator, реактор сам ставит `shared-contract` первым.

```bash
# Сборка всех модулей (parent + 3)
mvn clean install

# Один модуль + его зависимости
mvn -pl user-contract -am clean install

# npm-пакет с TS-типами для фронта
cd api-types && npm install && npm run build
```

### Версионирование (lockstep)

Единый источник версии — свойство `<revision>` в корневом [`pom.xml`](pom.xml).
Меняешь его → едут все 3 Maven-артефакта **и** npm `@eunoia-application/api-types`
(его версия деривится из `<revision>` скриптом `api-types/scripts/build-types.mjs`).

Semver: additive = minor, breaking = major, доки = patch. Релиз:
правишь `<revision>` → `mvn clean install` + `cd api-types && npm run build` →
коммит → запуск deploy-workflow.

### 4. Деплой через GitHub Actions

Проект использует GitHub Actions для деплоя в GitHub Packages (lockstep — весь трейн
одной версией):

1. Перейдите в раздел **Actions** в репозитории
2. Выберите workflow **🚀 Deploy Contracts**
3. Нажмите **Run workflow**

Один прогон публикует parent + 3 Maven JAR и npm-пакет `@eunoia-application/api-types`.
npm-publish идемпотентен: если такая версия уже в registry — шаг пропускается (нужен
bump `<revision>`).

## 🔧 Структура модуля

```text
user-contract/
├── pom.xml                          # Тонкий pom (наследует parent)
└── src/main/resources/openapi/
    └── user-api.yaml                # REST API спецификация
```

> Модуль с Kafka-событиями кладёт рядом `<module>-events.yaml` — этот файл остаётся
> в backend-JAR и в npm-бандл типов НЕ попадает (фронту события не нужны).

## 🔄 CI/CD

- [`deploy.yml`](.github/workflows/deploy.yml) — сборка и публикация в GitHub Packages
  (Maven JAR + npm) по `workflow_dispatch`.
- [`ci-checks.yml`](.github/workflows/ci-checks.yml) — проверки на PR:
  - **contracts** — bundle → `redocly lint` → `tsc` typecheck сгенерированных `.d.ts`
  - **maven** — `mvn clean install` (parent + 3 модуля)
  - **breaking-changes** — `oasdiff` base↔head, гейт на major-бамп `<revision>`

Все спецификации следуют стандарту OpenAPI 3.0 и линтуются в собранном виде
(`api-types/dist/openapi.json`) через [`redocly.yaml`](redocly.yaml).
