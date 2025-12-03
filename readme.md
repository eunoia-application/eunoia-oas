# 🍃 Eunoia OpenAPI Contracts

> Digital Garden API Contracts - `Design First`, `Validated`

[![GitHub Actions](https://github.com/eunoia-application/eunoia-oas/actions/workflows/deploy.yml/badge.svg)](https://github.com/eunoia-application/eunoia-oas/actions)
[![GitHub Packages](https://img.shields.io/badge/GitHub-Packages-blue)](https://github.com/eunoia-application/eunoia-oas/packages)
[![Java 17](https://img.shields.io/badge/Java-17-red.svg)](https://openjdk.org/projects/jdk/17/)
[![OpenAPI 3.0](https://img.shields.io/badge/OpenAPI-3.0-green.svg)](https://swagger.io/specification/)

Централизованное хранилище OpenAPI контрактов для проекта Eunoia — цифрового сада для прекрасного мышления.

## 📁 Структура проекта
```text
eunoia-oas/  
├── shared-contract/ # Общие типы и компоненты  
├── notes-contract/ # Контракты заметок (REST + Kafka)  
├── tags-contract/ # Контракты тегов  
├── auth-contract/ # Контракты аутентификации  
├── .github/workflows/ # CI/CD пайплайны  
└── README.md
```

## 🏗️ Модули

| Модуль | Описание | Версия |
|--------|----------|--------|
| **shared-contract** | Общие схемы (UUID, DateTime, Error), security схемы | `1.0.0` |
| **notes-contract** | API заметок + события Kafka для микросервисов | `1.0.0` |
| **tags-contract** | API тегов для категоризации заметок | `1.0.0` |
| **auth-contract** | API аутентификации и управления пользователями | `1.0.0` |

## 🚀 Быстрый старт

### 1. Клонирование проекта
```bash
git clone https://github.com/eunoia-application/eunoia-oas.git
cd eunoia-oas
```

### 2. Использование в проектах
Добавьте в ваш pom.xml:
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
        <artifactId>notes-contract</artifactId>
        <version>1.0.0</version>
    </dependency>
    <dependency>
        <groupId>com.eunoia.application</groupId>
        <artifactId>shared-contract</artifactId>
        <version>1.0.0</version>
    </dependency>
</dependencies>
```
### Структура модуля

```text
notes-contract/
├── pom.xml                          # Конфигурация Maven
└── src/main/resources/openapi/
    ├── notes-api.yaml              # REST API спецификация
    └── notes-events.yaml           # Kafka события
```

