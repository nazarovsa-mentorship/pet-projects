# Недели 1 - 2: Создание микросервиса

# TODO

- Добавить раздел про swagger
- Добавить раздел про конфигурацию приложения (либо записать видео)

# Цель

Сформировать навыки:
- Создание solution и структуры проектов
- Написание docker-compose для dotnet-приложений
- Работы с конфигурациями приложения
- Подключение и конфигурация Serilog

# Что такое серверное приложение и для чего оно нужно

[Клиент-серверная архитектура в картинках](https://habr.com/ru/articles/495698/#server)

# Структура проекта

## Структура каталогов проекта

- корневой каталог - sln, readme.md и прочие обслуживающие файлы
- src - Каталог для исполняемых сборок и библиотек 
- tests - Каталог для сборок тестов

```
/MyGreatProject
    MyGreatProject.sln
    Readme.md
    /src
        /MyGreatProject.Library/
            MyGreatProject.Library.csproj и другие файлы сборки библиотеки
        /MyGreatProject.Console/
            MyGreatProject.Console.csproj и другие файлы исполняемой сборки
    /tests
        /MyGreatProject.Library.UnitTests/
            MyGreatProject.LibraryUnitTests.csproj и другие файлы тестовой сборки
```

## Сборки проекта

### Правила именования

При разработке приложений на .net придерживаются следующего паттерна именования сборок  
**Organization.(Domain.).ServiceName.Assembly**

- **Organization** - Название организации
- **Domain** - Название домена. Может быть опциональным, если домена нет. Например, только один сервис.
- **ServiceName** - Имя сервиса
- **Assembly** - Имя сборки, в зависимости от используемых подходов.

Исполняемую сборку сервиса заказа банковских карт можно было бы назвать так: `Bank.Cards.Orders.Host`

В рамках обучения мы будем использовать следующие типы сборок сборки:

- `.AppServices` - Сборка, где интеграционный слой взаимодействует с предметной областью приложения
- `.Api.Contracts` - Сборка с публичными синхронными контрактами приложения
- `.Domain` - Сборка содержащая описание предметной области, бизнес-логику прилождения
- `.Domain.Contracts` - Сборка содержащая контракты предметной области
- `.Persistence` - Сборка с описанием слоя доступа к БД
- `.Host` - Исполняемая сборка, в которой хостится api
- `.Migrator` - Исполняемая сборка, которая применяет миграции

В ходе работы мы будем использовать общий код, который в организациях обычно поставляется в виде сборок, но может храниться и в репозитории проекта. Такой код мы будем помещать в сборку `.Common`.

В каждой организации структура проекта может быть организована по-разному. Поэтому в будущем подстраивайся под то, что используется на работе.

Полную информацию о правилах именования ты найдешь [здесь](https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/naming-guidelines?redirectedfrom=MSDN).

## Сборки и их типы

[Assemblies in .NET](https://learn.microsoft.com/en-us/dotnet/standard/assembly/)

# Docker и docker-compose

[Что такое докер](https://www.youtube.com/watch?v=aZTL2zRmOnA)

**Docker - это инструмент для "удобного" запуска приложений в изолированной среде**. Также он используется при запуске приложений на проде. Например, внутри kubernetes.  
В нашем проекте docker-compose позволяет запустить всю инфраструктуру в контейнерах и построить взаимодействия между ними. Не придется вручную поднимать брокер, БД и разработанные мной сервисы. Ты сможешь сделать это одной командой. По ходу ты будешь добавлять разработанные тобой исполняемые сборки в файл docker-compose.yml для запуска. Это позволит освоить необходимый минимум.

## Dockerfile

Из видео выше ты узнал, что **при работе с docker существует три основных сущности dockerfile - образ (image) - контейнер**.  
Dockerfile нужен для описания команд, необходимых для функционирования контейнера. На стадии формирования образа эти команды исполняются и формируют сам образ.  
Разберем на примере файла для сервиса "Каталог".
Этот файл использует [multi-stage build](https://docs.docker.com/build/building/multi-stage/). В нем используются 2 базовых образа:
- `mcr.microsoft.com/dotnet/sdk:8.0` - Образ для сборки, содержащий dotnet sdk.
- `mcr.microsoft.com/dotnet/aspnet:8.0` - Образ со средой исполнения. Легковесен за счет отсутсвия dotnet sdk.

Во время сборки dockerfile'а docker создает временный (сборочный) контейнер на основе директив `FROM` и выполняет в нем описанные команды.

```dockerfile
# Используем образ и даем ему алиас base
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
# Устанавливаем директорию сборочного контейнера в /app . Следующие команды будут выполняться относительно ее.
WORKDIR /app
# Делаем порт 80 открытым для взаимодействия внутри сети докера
EXPOSE 80
# Делаем порт 443 открытым для взаимодействия внутри сети докера
EXPOSE 443

# Используем образ и даем ему алиас build
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
# Устанавливаем директорию сборочного контейнера в /src
# Следующие команды будут выполняться относительно ее
WORKDIR /src
# Копируем содержимое каталога с кодовой базой. Пути в dockerfile указываются относительно корневого каталога, в котором лежит Solution
# Так как мы будем собирать dockerfile из docker-compose, который будет исполняться из корневого каталога, следующая команда скопирует его содержимое в /src сборочного контейнера
COPY . .

# Запускаем команду на восстановление nuget-пакетов проекта BookingService.Catalog.Host
RUN dotnet restore "src/BookingService.Catalog.Host/BookingService.Catalog.Host.csproj"
# Устанавливаем директорию сборочного контейнера в /src/src/BookingService.Catalog.Host
# /src/src возникает из-за того, что выше мы скопировали содержимое корневого каталога с Solution в папку /src временного контейнера 
WORKDIR "/src/src/BookingService.Catalog.Host"
# Выполняем сборку проекта BookingService.Catalog.Host
RUN dotnet build "BookingService.Catalog.Host.csproj" -c Release -o /app/build

# Используем результаты выполнения шага build и даем ему алиас publish
FROM build AS publish
# Выполняем публикацию исполняемых файлов и зависимостей проекта в папку /app/publish временного контейнера 
RUN dotnet publish "BookingService.Catalog.Host.csproj" -c Release -o /app/publish

# Используем результаты выполнения шага base и даем ему алиас final. 
# Это будет означать, что при сборке контейнера из образа будет использоваться образ mcr.microsoft.com/dotnet/aspnet:8.0
FROM base AS final

# Устанавливаем директорию сборочного контейнера в /app
# Следующие команды будут выполняться относительно ее
WORKDIR /app
# Копируем содержимое каталога /app/publish из шага publish в /app
# . соответствует текущей директории
COPY --from=publish /app/publish .
# Устанавливаем входную команду контейнера после сборки из полученного образа
# По сути запускает dotnet приложение
ENTRYPOINT ["dotnet", "BookingService.Catalog.Host.dll"]
```

## docker-compose

**docker-compose - это метод оркестрации приложений** для удобного запуска. Обычно он используется для локальной разработки, но иногда и для прода.
Нужно освоить основные команды и описание сервисов в yml файле. Разберем на примере сервиса "Каталог" с сборкой образа локально.

```yml
# Версия файла docker-compose
version: "3"

# Сервисы - контейнеры, которые будут запущены в рамках этого файла
services:
  # Описание сервиса
  booking-service_catalog-host:
    # Имя контейнера после запуска
    container_name: catalog-host
    # Имя образа после сборки
    image: booking-service_catalog-hots:latest
    # Указание, что нужно собрать образ из dockerfile
    # Если используется готовый образ, то эта секция будет отсутсвовать, а в image будет указано имя образа
    # Например, как в docker-compose.yml, который выдается в задании
    build:
      # Контекст - директория, относительно которой будет выполняться сборка
      # . - текущая директория, из которой выполняются команды docker-compose: сборка, запуск, остановка
      # Поэтому запускаем команды docker-compose up, docker-compose down и т.д. в директории, в которой располагается docker-compose.yml
      context: .
      # Путь относительно context к dockerfile собираемого приложения
      dockerfile: src/BookingService.Catalog.Host/Dockerfile
    # Порты, которые необходимо смапить на корневую операционную систему. То есть, на твой компьютер
    ports:
      # Порт контейрнера 8080 будет доступен на порту 8000 твоей операционной системы 
      - "8000:8080"
    # Монтирование каталогов или файлов корневой операционной системы в контейнер
    volumes:
      # Монтирует файл корневой операционной системы ./docker-compose-mount/BookingService.Catalog.Host/appsettings.json в файл контейнера /app/appsettings.json
      - ./docker-compose-mount/BookingService.Catalog.Host/appsettings.json:/app/appsettings.json
    # Выставляет переменные окружнения. Слева ключ, справа значение.
    environment:
      ASPNETCORE_ENVIRONMENT: Production
    # Зависимости сервиса в docker-compose от других сервисов
    # Сервис booking-service_catalog-host будет ожидать выполнения всех условий и запустится только после
    depends_on:
      booking-service_catalog-db:
        condition: service_healthy
      booking-service_catalog-migrations:
        condition: service_completed_successfully
      booking-service_rabbitmq:
        condition: service_healthy
```

При реализации задания тебе понадобятся секции `build` и `environment` и ключ `container_name`.

## Команды, которые нужно знать:
- docker build
- docker run
- docker-compose build
- docker-compose up
- docker-compose stop
- docker-compose down

# Конфигурация приложения

[Configuration in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/configuration/?view=aspnetcore-8.0)

## Среды окружения

При стандартном подходе работы с конфигурациями ASP.NET приложение ориентируется на исполняемую среду. По умолчанию есть среды `Development` и `Production`. Переопределить среду исполнения можно через переменную окружения `ASPNETCORE_ENVIRONMENT`. Можно создать столько сред, сколько требуется. Например, отдельную среду для запуска в docker-compose. Тогда мы сможем создать отдельный файл конфигурации по паттерну `appsettings.ENV.json` и переопределить в нем значения для этой среды.

Например, в файле `appsettings.json` по умолчанию включен swagger
```json
{
    "SwaggerEnabled": true,
    "CleanUpInterval": 1000
}
```
Тогда в файле `appsettings.DockerCompose.json` мы можем переопределить это значение.
```json
{
    "SwaggerEnabled": false
}
```

При запуске приложения с средой окружения `DockerCompose` ASP.NET прочитает файл `appsettings.DockerCompose.json`, и его приоритет будет выше, чем `appsettings.json`. Таким образом значение `SwaggerEnabled` в приложении будет `false`. При этом значение `CleanUpInterval` будет взято из `appsettings.json`. Грубо говоря, происходит объединение файлов с приоретизацией значений на базе исполняемой среды.

# Swagger

О Swagger прочитай [здесь](https://learn.microsoft.com/en-us/aspnet/core/tutorials/web-api-help-pages-using-swagger?view=aspnetcore-8.0).  
О подключении Swagger прочитай [здесь](https://learn.microsoft.com/en-us/aspnet/core/tutorials/getting-started-with-swashbuckle?view=aspnetcore-8.0&tabs=visual-studio).

# Логирование и Serilog

Логирование обязательная функциональность каждого приложения. Логирование помогает диагностировать проблемы в запущенном ПО.

Наиболее популярная библиотека для логирования Serilog.

## Уровни логирования

Об уровнях логирования можно прочитать здесь: [Logging in .NET Core and ASP.NET Core: Log level](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/logging/?view=aspnetcore-8.0#log-level)

## Подключение Serilog в ASP.NET приложение

Для того, чтобы подключить Serilog в приложение используются nuget-пакеты `Serilog.Extensions.Hosting` и `Serilog.Settings.Configuration`. Для базового подключения достаточно вызвать следующий код на инстансе `IHostBuilder`.
```csharp
.UseSerilog((ctx, logBuilder) =>
{
    // Указывает, что конфигурировать логгер нужно из секции конфигурации "Serilog"
    logBuilder.ReadFrom.Configuration(ctx.Configuration);
});
```

## Конфигурация Serilog

Конфигурация Serilog в файле appsettings.json выглядит следующим образом.
```json
{
  "Serilog": {
    // Указываются минимальные уровни логирования
    "MinimumLevel": {
      // Глобальный уровень
      "Default": "Debug",
      // Переопределения уровней для namespace
      "Override": {
        "Microsoft": "Information",
        "Microsoft.AspNetCore": "Error",
        "System.Net.Http.HttpClient": "Warning"
      }
    }
  },
  // Здесь указываются параметры выхода логов
  "WriteTo": [
    {
      // Пишем в консоль
      "Name": "Console",
      "Args": {
        "outputTemplate": "[{Level:u3}] {Timestamp:MM-dd HH:mm:ss} {TraceId} {SourceContext:l} {Message:lj}{NewLine}{Exception}"
      }
    },
    {
      // Пишем в файл
      "Name": "File",
      "Args": {
        "path": "/var/log/service-a-.log",
        "outputTemplate": "[{Level:u3}] {Timestamp:dd-MM-yyyy HH:mm:ss} {TraceId} {SourceContext:l} {Message:lj}{NewLine}{Exception}",
        "fileSizeLimitBytes": 104857600,
        "rollingInterval": "Day",
        "rollOnFileSizeLimit": true
      }
    }
  ]
}
```

Для подключения Serilog нужно подключить nuget-пакет `Serilog`, а для вывода в консоль и файл - `Serilog.Sinks.Console`, `Serilog.Sinks.File` соответсвенно.