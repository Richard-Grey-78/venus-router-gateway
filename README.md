# Venus Router Gateway

Public README-only profile for an internal VPS router stack used by Veritex-adjacent services and other private projects.

This repository intentionally contains no source code, deployment files, environment files, credentials, provider configuration, logs, backups, private URLs, admin access details, or live infrastructure secrets.

## Overview

Venus Router Gateway is an internal routing layer that centralizes access to LLM providers and exposes a controlled OpenAI-compatible API to applications running on the VPS and inside Docker networks.

The deployed stack is designed to let product services call one stable internal endpoint instead of talking directly to many external model providers. It also keeps provider credentials, routing policy, operational controls, and admin workflows outside application repositories.

## What It Does

- Provides an OpenAI-compatible API surface for chat, responses, and model discovery.
- Routes requests by logical profile rather than hard-coding a single provider in each product.
- Supports multiple provider families through a common gateway interface.
- Uses fallback and cooldown behavior when a model, key, or provider is unavailable.
- Separates product-facing inference traffic from administrative router operations.
- Exposes a private admin bridge for status checks, model-pool imports, backups, rollbacks, and smoke tests.
- Bridges Docker-hosted services to a local-only router API without publishing the core router port directly.
- Keeps live provider keys and routing configuration in private server-side environment/configuration files.

## Main Components

### Core Router

The core service runs as an internal LLM gateway. It exposes an OpenAI-compatible API for application workloads:

- `GET /health`
- `GET /v1/models`
- `POST /v1/chat/completions`
- `POST /v1/responses`

Applications can treat the router like an OpenAI-compatible backend while selecting internal profiles such as fast, cheap, premium, legal, reasoning, code, web-search, or product-specific pools.

### Admin Bridge

The admin bridge is a private operational interface around the router. It is used for controlled maintenance tasks such as:

- checking router status;
- viewing available models;
- importing or updating model pools;
- creating and listing backups;
- applying imports;
- rolling back to a previous router state;
- running smoke tests for selected profiles.

The admin bridge is intentionally not documented here with live addresses, credentials, or deployment-specific access details.

### Docker/Internal Proxy

The VPS includes an internal bridge that lets containerized projects reach the router through a private network interface. This keeps product services decoupled from host-only ports and avoids exposing the main router directly to the public internet.

### Reverse Proxy Layer

Nginx is used as the public edge for selected services and as a protected entry point for private admin access. Public documentation omits live admin hostnames, authentication files, certificate paths, and other operational details.

## Routing Model

Instead of binding products to a fixed provider/model pair, the router uses profile-based routing. A profile represents a workload intent, for example:

- low-cost generation;
- premium quality generation;
- legal/analytical reasoning;
- fast responses;
- coding-oriented work;
- web-search-enabled work;
- product-specific tool-use pools.

Each profile can map to an ordered pool of providers and models. If one candidate is rate-limited, unavailable, exhausted, or unhealthy, the router can move to another candidate according to private routing policy.

## Provider Strategy

The internal stack is built to work with several provider families behind one API boundary, including OpenAI-compatible endpoints and other major LLM providers. Provider credentials and exact routing priorities are private and are not part of this public mirror.

## Operational Features

- Health checks for the router and admin bridge.
- Key/provider cooldown handling for degraded providers.
- Smoke tests for profile validation.
- Backup and rollback workflow before risky routing changes.
- Private model-pool import workflow for maintaining available model lists.
- Separation between public product routes, internal inference routes, and admin routes.
- System service deployment suitable for VPS operation.

## Used By

The router stack supports internal automation and AI features across private projects, including Veritex-related services and other VPS-hosted applications that need shared LLM access without duplicating provider integration code.

## Security Boundary

The following materials are intentionally excluded from this repository:

- source code;
- environment files;
- API keys and provider tokens;
- admin credentials;
- nginx secrets and auth files;
- exact live admin URLs;
- router configuration files;
- model priority matrices;
- logs and backups;
- private deployment scripts;
- server-specific operational runbooks.

## Repository Status

This is a public, README-only mirror. It exists to describe the architecture and purpose of the router gateway without exposing private implementation details or sensitive infrastructure data.

---

# Venus Router Gateway — русское описание

Публичное README-only описание внутреннего router stack на VPS, который используется рядом с Veritex и другими приватными проектами.

В этом репозитории намеренно нет исходного кода, файлов деплоя, `.env`, учетных данных, провайдерских конфигов, логов, бэкапов, приватных URL, данных админ-доступа и живых инфраструктурных секретов.

## Обзор

Venus Router Gateway — это внутренняя прослойка маршрутизации для LLM-провайдеров. Она централизует доступ к моделям и дает приложениям единый контролируемый API, совместимый с OpenAI-форматом.

Идея проекта простая: продуктовые сервисы не должны напрямую хранить ключи и интеграции каждого LLM-провайдера. Вместо этого они обращаются к одному стабильному внутреннему шлюзу, а вся логика выбора провайдера, fallback, cooldown, администрирования и ротации ключей остается на стороне router stack.

## Что Делает Проект

- Предоставляет OpenAI-compatible API для chat, responses и списка моделей.
- Маршрутизирует запросы по логическим профилям, а не по жестко заданной модели.
- Поддерживает несколько семейств LLM-провайдеров через единый gateway-интерфейс.
- Использует fallback и cooldown, если модель, ключ или провайдер временно недоступны.
- Разделяет пользовательский inference-трафик и административные операции.
- Имеет приватный admin bridge для статуса, импорта model pool, бэкапов, rollback и smoke-тестов.
- Дает Docker-сервисам внутренний доступ к router API без публикации core router напрямую в интернет.
- Хранит реальные ключи и routing policy только в приватной серверной конфигурации.

## Основные Компоненты

### Core Router

Core Router — это основной внутренний LLM gateway. Он предоставляет API, совместимый с OpenAI-подходом:

- `GET /health`
- `GET /v1/models`
- `POST /v1/chat/completions`
- `POST /v1/responses`

Приложения могут использовать router как единый backend для LLM-запросов и выбирать внутренние профили: fast, cheap, premium, legal, reasoning, code, web-search или продуктовые profile pools.

### Admin Bridge

Admin Bridge — это приватный операционный слой поверх router. Он нужен для обслуживания и контроля:

- проверка статуса router;
- просмотр доступных моделей;
- импорт и обновление model pools;
- создание и просмотр бэкапов;
- применение импортированных изменений;
- rollback на предыдущую конфигурацию;
- smoke-тесты отдельных профилей.

Живые адреса, учетные данные и детали доступа к admin bridge не публикуются.

### Docker/Internal Proxy

На VPS есть внутренний bridge, через который контейнерные сервисы могут обращаться к router через приватный сетевой интерфейс. Это позволяет не раскрывать основной router port наружу и не привязывать приложения к host-only деталям.

### Reverse Proxy Layer

Nginx используется как публичный edge для отдельных сервисов и как защищенная точка входа для приватной админки. В публичном README не раскрываются live hostnames, auth-файлы, пути сертификатов и другие чувствительные операционные детали.

## Модель Маршрутизации

Router работает не по принципу “одно приложение — одна модель”, а через профили задач. Профиль описывает тип нагрузки:

- дешевое поколение текста;
- premium-ответы;
- юридический или аналитический reasoning;
- быстрые ответы;
- coding-задачи;
- web-search сценарии;
- продуктовые pools под конкретные сервисы.

Каждый профиль может указывать на упорядоченный набор провайдеров и моделей. Если один кандидат недоступен, уходит в rate limit, исчерпал квоту или дает ошибку, router может переключиться на следующий вариант по приватной routing policy.

## Стратегия Провайдеров

Внутренняя архитектура рассчитана на работу с разными LLM-провайдерами через единый API boundary. Точные приоритеты моделей, provider keys, fallback-матрицы и правила маршрутизации являются приватной частью инфраструктуры и не публикуются в этом зеркале.

## Операционные Возможности

- Health checks для core router и admin bridge.
- Cooldown для деградирующих ключей, моделей и провайдеров.
- Smoke-тесты профилей перед использованием в продуктах.
- Backup и rollback перед рискованными изменениями маршрутизации.
- Приватный workflow импорта model pools.
- Разделение public product routes, internal inference routes и admin routes.
- Деплой как системный сервис, подходящий для VPS-окружения.

## Где Используется

Router stack поддерживает AI-функции и автоматизацию в приватных проектах, включая Veritex-related сервисы и другие приложения на VPS, которым нужен общий доступ к LLM без дублирования провайдерских интеграций в каждом проекте.

## Граница Безопасности

В этом репозитории намеренно отсутствуют:

- исходный код;
- `.env` и другие environment-файлы;
- API keys и provider tokens;
- admin credentials;
- nginx secrets и auth-файлы;
- точные live admin URLs;
- router configuration files;
- матрицы приоритетов моделей;
- логи и бэкапы;
- приватные deployment scripts;
- server-specific runbooks.

## Статус Репозитория

Это публичное README-only зеркало. Оно нужно, чтобы показать назначение и архитектуру Venus Router Gateway без раскрытия приватной реализации, кода и чувствительных инфраструктурных данных.
