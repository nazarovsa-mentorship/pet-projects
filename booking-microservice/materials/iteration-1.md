# Итерация 1: Создание микросервиса

# Цель

Сформировать навыки:
- Создание solution и структуры проектов
- Работы с конфигурациями приложения
- Подключение и конфигурация Serilog
- Написание docker-compose для dotnet-приложений

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

При разработке приложений на .net придерживаются следующего паттерна именования сборок  
**Organization.(Domain.).ServiceName.Assembly**

- **Organization** - Название организации
- **Domain** - Название домена. Может быть опциональным, если домена нет. Например, только один сервис.
- **ServiceName** - Имя сервиса
- **Assembly** - Имя сборки, в зависимости от используемых подходов.

Исполняемую сборку сервиса заказа банковских карт можно было бы назвать так: `Bank.Cards.Orders.Host`

В рамках обучения ты будешь использовать следующие типы сборок сборки:

- `.AppServices` - Сборка, где интеграционный слой взаимодействует с предметной областью приложения
- `.Api.Contracts` - Сборка с публичными синхронными контрактами приложения
- `.Domain` - Сборка содержащая описание предметной области, бизнес-логику прилождения
- `.Domain.Contracts` - Сборка содержащая контракты предметной области
- `.Persistence` - Сборка с описанием слоя доступа к БД
- `.Host` - Исполняемая сборка, в которой хостится api
- `.Migrator` - Исполняемая сборка, которая применяет миграции

В ходе работы ты будешь использовать общий код, который в организациях обычно поставляется в виде сборок, но может храниться и в репозитории проекта. Такой код ты будешь помещать в сборку `.Common`.

В каждой организации структура проекта может быть организована по-разному. Поэтому в будущем подстраивайся под то, что используется на работе.

Полную информацию о правилах именования ты найдешь [здесь](https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/naming-guidelines?redirectedfrom=MSDN).

## Сборки и их типы

[Assemblies in .NET](https://learn.microsoft.com/en-us/dotnet/standard/assembly/)

# Generic Host в ASP.NET Core

Generic Host в ASP.NET Core представляет собой систему настройки и запуска приложений. Он заменяет WebHostBuilder, используемый в предыдущих версиях ASP.NET Core, и предлагает гибкий подход к конфигурации приложения.

Подход minimal api вне контекста проекта. Ты можешь прочитать о нем [здесь](https://learn.microsoft.com/en-us/aspnet/core/tutorials/min-web-api?view=aspnetcore-9.0&tabs=visual-studio).

## Что такое Generic Host?

Generic Host (или .NET Generic Host) - это объект, который инкапсулирует ресурсы приложения:
- Конфигурацию
- Внедрение зависимостей (DI)
- Логирование
- И другие кросс-функциональные аспекты приложения

Ключевое преимущество Generic Host в том, что он подходит не только для веб-приложений, но и для других типов приложений .NET.

## Структура с использованием HostBuilder и Startup

В большинстве проектов ASP.NET Core ты встретишь два ключевых класса для настройки приложения:

1. **Класс HostBuilderFactory** (или Program.cs с методами настройки) - отвечает за создание и конфигурацию хоста приложения.
2. **Класс Startup** - отвечает за настройку сервисов и middleware приложения.

### Пример организации HostBuilderFactory

```csharp
// Примерная структура класса HostBuilderFactory
public static class HostBuilderFactory
{
    public static IHost BuildHost(string[] args)
    {
        return Host.CreateDefaultBuilder(args)
            .ConfigureWebHostDefaults(webBuilder =>
            {
                webBuilder.UseStartup<Startup>();
            })
            .UseSerilog() // Подключение Serilog
            .Build();
    }
}
```

### Пример организации Startup

```csharp
public class Startup
{
    private readonly IConfiguration _configuration;

    public Startup(IConfiguration configuration)
    {
        _configuration = configuration;
    }

    // Настройка сервисов
    public void ConfigureServices(IServiceCollection services)
    {
        // Регистрация сервисов в DI-контейнере
        services.AddControllers();
        
        // Добавление Swagger
        services.AddSwaggerGen();
        
        // Другие сервисы...
    }

    // Настройка middleware
    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
        if (env.IsDevelopment())
        {
            app.UseDeveloperExceptionPage();
            app.UseSwagger();
            app.UseSwaggerUI();
        }
        
        app.UseRouting();
        app.UseEndpoints(endpoints =>
        {
            endpoints.MapControllers();
        });
        
        // Другие middleware...
    }
}
```

## Взаимодействие HostBuilderFactory и Startup

Взаимосвязь между этими классами следующая:
1. HostBuilderFactory создает и настраивает хост приложения
2. В процессе настройки веб-хоста указывается класс Startup
3. При запуске приложения сначала вызывается метод ConfigureServices для регистрации служб
4. Затем вызывается метод Configure для настройки конвейера HTTP-запросов

Такое разделение обязанностей делает код более поддерживаемым и тестируемым.

# Конфигурация приложения

[Configuration in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/configuration/?view=aspnetcore-8.0)

## Работа с конфигурациями в ASP.NET Core

ASP.NET Core имеет гибкую систему конфигурации, которая позволяет управлять настройками приложения из различных источников.

### Источники конфигурации

ASP.NET Core может загружать конфигурацию из множества источников:

1. **Файлы JSON** (appsettings.json, appsettings.{Environment}.json)
2. **Переменные окружения**
3. **Аргументы командной строки**
4. **Azure Key Vault**
5. **User secrets** (для разработки)
6. **Другие пользовательские источники**

### Порядок загрузки конфигураций

Порядок имеет значение! Последующие источники перезаписывают предыдущие:

1. appsettings.json
2. appsettings.{Environment}.json
3. User secrets (только в Development)
4. Переменные окружения
5. Аргументы командной строки

### Доступ к конфигурации

Существует несколько способов получить доступ к конфигурации:

1. **Через DI и IConfiguration**:
```csharp
public class SomeService
{
    private readonly string _connectionString;
    
    public SomeService(IConfiguration configuration)
    {
        _connectionString = configuration.GetConnectionString("DefaultConnection");
    }
}
```

2. **Через строго типизированные настройки**:
```csharp
// Класс настроек
public class MyOptions
{
    public string Option1 { get; set; }
    public int Option2 { get; set; }
}

// Регистрация в Startup.ConfigureServices
services.Configure<MyOptions>(Configuration.GetSection("MyOptions"));

// Использование в сервисе
public class MyService
{
    private readonly MyOptions _options;
    
    public MyService(IOptions<MyOptions> options)
    {
        _options = options.Value;
    }
}
```

### Работа с разными окружениями

Для разных окружений (Development, Staging, Production) можно создавать отдельные файлы конфигурации:

- appsettings.json - общие настройки
- appsettings.Development.json - настройки для разработки
- appsettings.Production.json - настройки для продакшена

Текущее окружение определяется через переменную окружения `ASPNETCORE_ENVIRONMENT`.

### Пример структуры файлов конфигурации

**appsettings.json** (базовые настройки):
```json
{
  "ConnectionStrings": {
    "DefaultConnection": "базовая-строка-подключения"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information"
    }
  },
  "AllowedHosts": "*"
}
```

**appsettings.Development.json** (настройки для разработки):
```json
{
  "ConnectionStrings": {
    "DefaultConnection": "строка-подключения-для-разработки"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Debug",
      "Microsoft": "Information" // Переопределяем поведение логирования для разработки
    }
  }
}
```

**appsettings.Production.json** (настройки для продакшена):
```json
{
  "ConnectionStrings": {
    "DefaultConnection": "строка-подключения-для-продакшена"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Warning",
      "Microsoft": "Error" // Переопределяем поведение логирования для продакшена
    }
  }
}
```

# Swagger

Swagger (OpenAPI) — это спецификация и набор инструментов для описания, создания, использования и визуализации REST API. В ASP.NET Core используется Swashbuckle.AspNetCore для интеграции Swagger.

## Зачем нужен Swagger?

1. **Документация API** — автоматически генерирует документацию по вашим API-маршрутам
2. **Интерактивный интерфейс** — позволяет тестировать API прямо из браузера
3. **Генерация клиентов** — возможность автоматически создавать клиентские библиотеки для вашего API

## Подключение Swagger в ASP.NET Core проект

Подключение Swagger требует следующих шагов:

1. **Установка пакетов NuGet**:
```bash
dotnet add package Swashbuckle.AspNetCore
```

2. **Регистрация сервисов в методе ConfigureServices класса Startup**:
```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllers();
    
    // Регистрация Swagger
    services.AddSwaggerGen(c =>
    {
        c.SwaggerDoc("v1", new OpenApiInfo 
        { 
            Title = "Booking Service API", 
            Version = "v1",
            Description = "API для сервиса бронирования"
        });
    });
}
```

3. **Настройка Swagger в конвейере HTTP в методе Configure**:
```csharp
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    // ...
    
    // Добавление Swagger middleware
    app.UseSwagger();
    app.UseSwaggerUI(c =>
    {
        c.SwaggerEndpoint("/swagger/v1/swagger.json", "Booking Service API V1");
    });
    
    // ...
}
```

## Настройка Swagger в зависимости от окружения

Часто Swagger нужен только в среде разработки. Чтобы контролировать его наличие в разных окружениях, можно:

1. **Использовать условие на базе окружения**:
```csharp
if (env.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}
```

2. **Использовать параметр из конфигурации**:
```csharp
// В appsettings.json
{
    "SwaggerEnabled": true
}

// В Startup.Configure
if (Configuration.GetValue<bool>("SwaggerEnabled"))
{
    app.UseSwagger();
    app.UseSwaggerUI();
}
```

Это позволяет отключить Swagger для production-среды через конфигурацию.

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

## Подробная конфигурация Serilog в ASP.NET Core

Serilog предоставляет мощные возможности для структурированного логирования в .NET приложениях. Давайте рассмотрим более детальную настройку.

### Подключение Serilog с помощью кода

Помимо конфигурации через appsettings.json, Serilog можно настроить программно:

```csharp
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .UseSerilog((hostingContext, loggerConfiguration) => 
        {
            loggerConfiguration
                .ReadFrom.Configuration(hostingContext.Configuration)
                .Enrich.FromLogContext()
                .Enrich.WithMachineName()
                .Enrich.WithThreadId()
                .WriteTo.Console(
                    outputTemplate: "[{Timestamp:HH:mm:ss} {Level:u3}] {Message:lj}{NewLine}{Exception}"
                )
                .WriteTo.File(
                    path: "logs/app-.log", 
                    rollingInterval: RollingInterval.Day,
                    outputTemplate: "[{Timestamp:yyyy-MM-dd HH:mm:ss.fff} {Level:u3}] {Message:lj}{NewLine}{Exception}"
                );
        })
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.UseStartup<Startup>();
        });
```

### Использование разных выводов (sinks) в зависимости от окружения

Можно настроить разные способы вывода логов в зависимости от среды выполнения:

```csharp
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .UseSerilog((hostingContext, loggerConfiguration) => 
        {
            loggerConfiguration
                .ReadFrom.Configuration(hostingContext.Configuration)
                .Enrich.FromLogContext();
                
            // Разная настройка для разных окружений
            if (hostingContext.HostingEnvironment.IsDevelopment())
            {
                // В режиме разработки логируем в консоль с подробной информацией
                loggerConfiguration.WriteTo.Console(
                    outputTemplate: "[{Timestamp:HH:mm:ss} {Level:u3}] {SourceContext} {Message:lj}{NewLine}{Exception}"
                );
            }
            else
            {
                // В продакшене логируем в файл и в консоль с более краткой информацией
                loggerConfiguration
                    .WriteTo.Console()
                    .WriteTo.File(
                        path: "/var/logs/booking-service-bookings/app-.log", 
                        rollingInterval: RollingInterval.Day,
                        shared: true
                    );
            }
        });
```

### Использование контекста логирования

Serilog поддерживает обогащение логов контекстной информацией:

```csharp
// В контроллере или сервисе
using (LogContext.PushProperty("BookingId", bookingId))
{
    _logger.LogInformation("Обработка бронирования");
    
    // Все логи внутри этого блока будут содержать свойство BookingId
    ProcessBooking(booking);
}
```

### Фильтрация логов

Можно настроить фильтрацию логов по разным критериям:

```csharp
loggerConfiguration
    .MinimumLevel.Debug()
    .MinimumLevel.Override("Microsoft", LogEventLevel.Information)
    .MinimumLevel.Override("System", LogEventLevel.Warning)
    .MinimumLevel.Override("Microsoft.AspNetCore.Authentication", LogEventLevel.Information)
    .Filter.ByExcluding(c => c.Properties.ContainsKey("HealthCheck") && c.Properties["HealthCheck"].ToString() == "true");
```

### Структурированное логирование

Одно из главных преимуществ Serilog — структурированное логирование:

```csharp
// Вместо интерполяции строк
_logger.LogInformation("Создано бронирование {BookingId} для пользователя {UserId}", booking.Id, booking.UserId);

// А не так (это хуже для производительности и анализа)
_logger.LogInformation($"Создано бронирование {booking.Id} для пользователя {booking.UserId}");
```

### Настройка форматирования логов

Можно настроить форматирование вывода логов:

```csharp
.WriteTo.Console(
    outputTemplate: "[{Timestamp:HH:mm:ss} {Level:u3}] {SourceContext:l} {Message:lj}{NewLine}{Exception}",
    theme: AnsiConsoleTheme.Code
)
```



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
