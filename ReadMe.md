# Weather Proxy API

**Погодный сервис на C# с Redis-кэшем и интерфейсом на Next.js.**

Приложение получает текущую погоду и прогноз у внешнего провайдера, нормализует ответ и повторно использует закэшированные данные. Учебный проект для практики HTTP-интеграций, разделения ответственности и тестирования API.

## Что реализовано

- Текущая погода и прогноз по городу.
- Cache-aside через Redis с отдельными TTL для разных запросов.
- Typed HttpClient, таймауты, retry и circuit breaker.
- Валидация параметров, ограничение частоты запросов, обработка ошибок.
- Swagger, health check, correlation ID и метрики.
- Интерфейс запроса погоды на Next.js.
- Unit-тесты кэша и API-тест с подменой внешнего провайдера.

## Стек

C# / .NET 9, ASP.NET Core Minimal API, Redis, Polly, xUnit, FluentAssertions, Next.js, TypeScript, Docker Compose.

## Запуск через Docker

Требуются Docker с Compose и собственный ключ weatherapi.com.

```sh
cd WeatherProxy
cp .env.example .env
```

В Windows PowerShell вместо `cp` можно использовать `Copy-Item .env.example .env`.

Заполните `WEATHERAPI__APIKEY` в локальном `.env` и запустите:

```sh
docker compose up --build
```

API: <http://localhost:8080>. Swagger: <http://localhost:8080/swagger>. Проверка состояния: <http://localhost:8080/healthz>.

Для frontend из корня репозитория:

```sh
cd frontend
npm ci
npm run dev
```

Предварительно создайте `frontend/.env.local` с `NEXT_PUBLIC_API_BASE_URL=http://localhost:8080/api/v1`.

## Запуск и тесты .NET

Из корня репозитория:

```sh
dotnet restore WeatherProxy/WeatherProxy.sln
dotnet test WeatherProxy/WeatherProxy.sln
```

При локальном запуске вне Docker задайте конфигурацию внешнего API и Redis, затем:

```sh
dotnet run --project WeatherProxy/src/Weather.Api
```

## API

| Запрос | Назначение |
|:--|:--|
| `GET /api/v1/weather/current?city=Helsinki` | Текущая погода |
| `GET /api/v1/weather/forecast?city=Helsinki&days=3` | Прогноз |
| `GET /healthz` | Проверка состояния |

Поле `source` помогает отличить ответ внешнего провайдера от ответа из кэша.

## Ограничения

Доступная длина прогноза зависит от тарифа внешнего провайдера. Существующие тесты в основном проверяют работу кэша. Проверки отказов внешнего API и Redis стоит расширить. CI пока не настроен.

[Конфигурация и технические заметки](docs/configuration.md).
