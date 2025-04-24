# План проекта

# Итерация 1: Создание микросервиса

## Цель

Сформировать навыки:
- Создание solution и структуры проектов
- Написание docker-compose для dotnet-приложений
- Работы с конфигурациями приложения
- Подключение и конфигурация Serilog

## Подготовка

1. Создать приватный репозиторий на github.
2. Выдать права на просмотр для аккаунта nazarovsa.
3. Основной ветвью будет main.
4. Добавить в main пустой файл README.md

Разработка каждой двухнедельной итерации должна вестись в отдельной ветке и вливаться в main по завершению итерации. 

## Задача

1. Создать пустой solution `BookingService.Booking`:
	- Скопировать файл docker-compose.yml и каталог `docker-compose-mount` из репозитория в каталог с созданным solution (с файлом `BookingService.Booking.sln`)
	- Создать каталог `src` в каталоге с созданным solution. В нем будут располагаться каталоги с сборками из пункта 2.
	- Создать каталог `tests` в каталоге с созданным solution . В нем будут располагаться каталоги с сборками тестов.
2. Создать проекты в каталоге `src` в созданном solution: 
	- Консольное приложение: `BookingService.Booking.Host` - хост приложения
	- Библиотека классов: `BookingService.Booking.Api.Contracts` - публичные контракты приложения
	- Библиотека классов: `BookingService.Booking.AppServices` - сервисный слой
	- В сборку `BookingService.Booking.Host` добавить ссылку на `BookingService.Booking.AppServices` и `BookingService.Booking.Api.Contracts`
3. Заменить значение `Microsoft.NET.Sdk` на `Microsoft.NET.Sdk.Web` в значении `Sdk` тэга `Project` в файле `BookingService.Booking.Host.csproj`.
4. Добавить `BookingService.Booking.Host` в docker-compose.yml 
	- Сгенерировать dockerfile средствами IDE
	- Добавить в секцию `services` docker-compose.yml сервис `booking-service_bookings-host`. Секция должна содержать:
		- Ключ `container_name` в именем контейнера `bookings-host`
		- Раздел `build` для сборки образа проекта `BookingService.Booking.Host` с ключами `context` и `dockerfile`
5. Создать классы `Startup.cs` и `HostBuilderFactory.cs` в сборке `BookingService.Booking.Host`
   	- `Startup` должен содержать настройку сервиса
	- `HostBuilderFactory` должен содержать статичный метод, возвращающий сконфигурированный хост с использованием [Generic Host](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/host/generic-host?view=aspnetcore-8.0) (Не используем minimal-api) и класса `Startup`
6. Добавить Swagger
7. Добавить файлы конфигурации:
    - `appsettings.json` - общий файл конфигурации.
    - `appsettings.Development.json` - файл конфигурации для настройки Development окружения во время запуска приложения в ide.
    - `appsettings.Production.json` - файл конфигурации для настройки Production окружения во время запуска приложения в docker-compose.
8. Добавить логирование с Serilog в приложение вызовом `UseSerilog` на HostBuilderFactory. Serilog должен быть сконфигурирован следующим образом:
	- Конфигурироваться из конфигурации приложения (файла appsettings.json)
	- Использовать минимальный уровень логирования по умолчанию `Information` (задается в appsettings.json)
	- В Development окружении выводить логи в [консоль](https://github.com/serilog/serilog-sinks-console)
	- В Production окружении выводить логи в [консоль](https://github.com/serilog/serilog-sinks-console) и [файл](https://github.com/serilog/serilog-sinks-file) в каталоге `/var/logs/booking-service-bookings/`
9.  В сборке `BookingService.Booking.Host` в каталоге Controllers создать класс контроллера, `BookingsController`, без реализации.

## Критерии оценки

1. Микросервис собирается и запускается как в IDE, так и в docker-compose
2. Микросервис предоставляет функциональность Swagger
3. При старте сервиса логи пишутся в консоль и файл

## Материалы для изучения

[Configuration in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/configuration/?view=aspnetcore-8.0)  
[Serilog ASP.NET Core github repo](https://github.com/serilog/serilog-aspnetcore)  
[.NET Generic Host in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/host/generic-host?view=aspnetcore-8.0) - Использование HostBuilder  
[Docker Compose Overview](https://docs.docker.com/compose/)  
[Git aliases](https://git-scm.com/book/en/v2/Git-Basics-Git-Aliases)

# Итерация 2: Создание API и абстракций бизнес-логики приложения

## Цель

- Познакомить с слоёной архитектурой
- Сформировать понимание DI
- Сформировать навыки:
  - Реализации API слоя приложения
  - Реализации абстракций сервисного слоя приложения
  - Обработки исключений по формату Problem Details в ASP.NET приложениях

## Задача

1. В сборке `BookingService.Booking.Api.Contracts` создать каталог `Bookings/Dtos`.
2. Реализовать DTO-класс `BookingData` в каталоге, созданном в пункте 1. `BookingData` должен содержать все поля сущности "Бронирование" из описания задания.
3. Создать каталог `Bookings/Requests` в сборке `BookingService.Booking.Api.Contracts`, в котором создать запросы:
	- `CreateBookingRequest` - запрос на создание бронирования
	- `GetBookingsByFilterRequest` - запрос на получение бронирований по фильтру 
4. Создать статический класс `WebRoutes` в сборке `BookingService.Booking.Api.Contracts` в корневом каталоге `Bookings`. Он будет содержать значения путей для контроллера. Класс должен содержать константы для базового пути контроллера, а также всех методов:
   - `BasePath` - Базовый путь - `api/bookings`.
   - `Create` - Путь для создания бронирования, `BasePath + "/create"`.
   - `GetById` - Путь для получения бронирования по id, `BasePath + "/{id}"`.
   - `Cancel` - Путь для отмены бронирования по id, `BasePath + "/{id}/cancel"`.
   - `GetByFilter` - Путь для получения списка бронирований по фильтру, `BasePath + "/by-filter"`.
   - `GetStatusById` - Возвращает статус бронирования по идентификатору,  `BasePath + "{id}/status`.
5. Создать каталог `Bookings` в сборке `BookingService.Booking.AppServices`.
6. Создать интерфейс `IBookingsService` с контрактами бизнес-логики, обрабатывающей команды в каталоге `Bookings` сборки `BookingService.Booking.AppServices`. Методы интерфейса должны отражать методы контроллера, но вместо типов запроса `*Request` принимать целевой набор значений.
   - `Create` - Принимает значения из запроса `CreateBookingRequest`. Возвращает идентификатор.
   - `GetById` - Принимает идентификатор. Возвращает `BookingData`.
   - `Cancel` - Принимает идентификатор.
7. Создать реализацию интерфейса `IBookingsService`, `BookingService`, в том же каталоге, что интерфейс, без реализации методов (в теле методов `throw new NotImplementedException()`).
8. Cоздать интерфейс  `IBookingsQueries` с контрактами бизнес-логики, обрабатывающей запросы на выборку данных в каталоге `Bookings` сборки `BookingService.Booking.AppServices`. Методы интерфейса:
	- Обработчик `GetByFilter`, возвращающий `BookingData[]`.
	- Обработчик `GetStatusById`, возвращающий статус бронирования.
9.  Создать реализацию интерфейса `IBookingsQueries`, `BookingQueries`, в том же каталоге, что интерфейс, без реализации методов (в теле методов `throw new NotImplementedException()`).
10. Создать статический класс `ServiceCollectionExtensions` в каталоге `Bookings`  сборки `BookingService.Booking.AppServices`. Создать в нем статический метод расширения для `IServiceCollection` с именем `AddAppServices`.
11. Добавить код, регистрирующий `BookingsService` как реализацию `IBookingsService` с уровнем жизни Scoped в DI, в метод `AddAppServices`.
12. Добавить код, регистрирующий `BookingsQueries` как реализацию `IBookingsQueries` с уровнем жизни Scoped в DI, в метод `AddAppServices`.
13. Вызвать метод `AddAppServices` в классе `Startup` сборки `BookingService.Booking.Host`.
14. Внедрить `IBookingsService` в конструктор контроллера и присвоить приватному полю `_bookingsService`. 
15. Внедрить `IBookingsQueries` в конструктор контроллера и присвоить приватному полю `_bookingsQueries`.
16. Реализовать методы синхронного API для управления бронированием в `BookingsController` согласно заданию. В качестве путей использовать константы из класса `WebRoutes` сборки `BookingService.Booking.Api.Contracts`. Не использовать строковые литералы. Методы контроллера должны вызывать соответствующие методы `IBookingsService` и `IBookingsQueries`:
	- Метод создания должен возвращать идентификатор созданного бронирования
	- Метод, возвращающий одно "Бронирование", должны возвращать `Task<BookingData>`
	- Метод, возвращающий несколько представлений "Бронирование", должны возвращать массив `Task<BookingData[]`
	- Метод, возвращающий статус, должен возвращать `Task<Enum>`.
	- Метод отмены не возвращает ничего (`Task`), в случае ошибки возвращает описание в формате ProblemDetails (см. пункт 6.4)
17. Создать каталог `Exceptions` в сборке `BookingService.Booking.AppServices`. Создать в нём исключение `ValidationException`, наследующее от `Exception`, которое будет порождаться на сервисном слое в случае неуспешной валидации входящих данных.
18. Создать каталог `Exceptions` в сборке `BookingService.Booking.Domain`. В нём создать исключение `DomainException`, наследующее от `Exception`, которое будет порождаться на уровне домена в случае, если не выполнены проверки бизнес-логики.
19. Реализовать ответ сервера в случае возникновения исключений по формату Problem Details c использованием nuget-пакета `Hellang.Middleware.ProblemDetails`. 
	- Выполнить маппинг исключения `DomainException` на http статус 402. `DomainException.Message` записывать в Title, `DomainException.StackTrace` записывать в Details. 
	- Выполнить маппинг исключения `ValidationException` на http статус 400.  `ValidationException.Message` записывать в Title. 
20. В методе `BookingService.Create` заменить `NotImplementedException` на `ValidationException`.
21. В методе `BookingService.Cancel` заменить `NotImplementedException` на `DomainException`.
22. Вызвать методы контроллера из пунктов 17 и 18 в swagger. Убедиться, что маппинг из пункта 16 работает. 

## Критерии оценки

1. Рализована сборка контрактов: созданы классы для DTO, статический класс с путями контроллера.  
2. В `BookingsController` внедрены `IBookingsService` и `IBookingsQueries`. `BookingsController` использует в качестве путей константы из класса `WebRoutes`.  
3. В `BookingsController` созданы методы синхронного API из задания. Методы вызывают соответсвующие методы `IBookingsService` и `IBookingsQueries`.  
4. Подключена обработка исключений с ProblemDetails.  
5. Созданы исключения `DomainException` и `ValidationException`. Выполнен их маппинг на ответ сервера по формату Problem Details.  

## Материалы для изучения

[Metanit: Создание контроллера](https://metanit.com/sharp/aspnet5/23.2.php)  
[Handling Web API Exceptions with ProblemDetails middleware](https://andrewlock.net/handling-web-api-exceptions-with-problemdetails-middleware/)  

# Итерация 3: Создание и тестирование бизнес-логики приложения

## Цель

- Сформировать общее понимание о богатой модели разработки бизнес-логики
- Сформировать понимание о назначении тестирования unit-тестирования
- Сформировать навыки:
  - Реализации бизнес-логики приложения с использованием богатой модели
  - Реализации минимального набора unit-тестов на разработанную бизнес-логику

## Задача

1. Создать проекты в каталоге `src` в solution:
	- Библиотека классов: `BookingService.Booking.Domain` - сборка для хранения бизнес-логики
	- Библиотека классов: `BookingService.Booking.Domain.Contracts` - сборка для хранения общих компонентов бизнес-логики
2. Создать каталог `Dates` в сборке `BookingService.Booking.AppServices`
	- Создать интерфейс `ICurrentDateTimeProvider` и его реализацию `DefaultCurrentDateTimeProvider` в каталоге `Dates`
	- Зарегистрировать `DefaultCurrentDateTimeProvider` как реализвацию `ICurrentDateTimeProvider` в DI контейнере с уровнем жизни `Singleton` в методе `AddAppServices` из предыдущей недели
3. Создать каталог `Bookings` в сборке `BookingService.Booking.Domain.Contracts`
4. Переместить созданные ранее enums, относящиеся к `BookingData` из сборки `BookingService.Booking.Api.Contracts` в каталог `Bookings` сборки `BookingService.Booking.Domain.Contracts`
5. Добавить зависимость от сборки `BookingService.Booking.Domain.Contracts` в `BookingService.Booking.Api.Contracts`
6. Создать каталог `Bookings` в сборке `BookingService.Booking.Domain`
   - Создать класс `BookingAggregate` в каталоге `Bookings` сбокри `BookingService.Booking.Domain`
   - Реализовать в `BookingAggregate` поля необходимые для выполнения бизнес-логики (см. требования в общем репозитории)
   - Реализовать API класса `BookingAggregate` для обслуживания бизнес логики
     - Приватный конструктор, создающий новый экземпляр `BookingAggregate`
     - Статический метод `Initialize`, возвращающий `BookingAggregate` и принимающий аргументы необходимые для создания нового экземпляра класса. Должен проверять входные аргументы на валидность и возвращать результат вызова приватный конструктора.
     - Метод `Confirm`, который будет переводить бронирование в статус "Подтверждено"
     - Метод `Cancel`, кторый будет переводить бронирование в статус "Отменено"
7. Создать в solution каталог `_Tests` через IDE
8. Создать проект тестов `BookingService.Booking.Domain.UnitTests` в solution-каталоге `_Tests`. Проект должен находиться в каталоге файловой системы `tests`
9.  Создать файл `BookingAggregateTests.cs` с классом `BookingAgregateTests` в `BookingService.Booking.Domain.UnitTests`
10. Реализовать Unit-тесты на бизнес-логику `BookingAggregate` в классе `BookingAgregateTests`. 
   - Метод `Initialize` должен быть покрыт тестами проверяющими валидацию входных параметров и создания экземпляра класса
   - Публичные методы бизнес-логики класса должны быть покрыты тестами, которые проверяют минимум 2 сценария: положительный и отрицательный

## Критерии оценки

1. Реализован класс `BookingAggregate`, содержащий бизнес-логику, указанную в требованиях пет-проекта (см. требования в общем репозитории)
	- Содержит методы `Initialize`, `Confirm` и `Cancel`
	- Выполняется валидация входных параметров в публичных методах
	- Методы реализуют корректные переходы по статусам (см. требования в общем репозитории)
2. Реализованы unit-тесты на разработанную бизнес-логику
	- Минимум один положительный и один отрицательный сценарий на каждый публичный метод `BookingAggreage`

## Материалы для изучения

[Блеск и нищета модели предметной области](https://habr.com/ru/companies/jugru/articles/503868/)  
Владимир Хориков: Принципы unit-тестирования - Глава 1, 2, 3

# Итерация 4: Создание слоя взаимодействия с БД; Библиотека EntityFramework Core

## Цель

Сформировать понимание:
- Назначение паттерна Repository 
- Назначение паттерна Unit of Work

Сформировать навыки:
- Запуск PostgreSQL в docker-compose
- Написание маппинга классов бизнес-логики на БД для EntityFramework Core
- Использование миграций EntityFramework Core
- Реализация сервисного слоя приложения

## Задача

1. Добавить контейнер для БД сервиса `BookingService.Bookings.Host` в `docker-compose`
   - Имя секции: `booking-service_bookings-db`
   - Образ: `postgres:latest`
   - Имя контейнера: `bookings-db`
   - Маппинг портов: `5433:5432`
   - Заполнить переменные окружения для конфигурации PostgreSQL
     - POSTGRES_DB: `booking_service_bookings`
     - POSTGRES_USER: `bookings_admin`
     - POSTGRES_PASSWORD: `admin_bookings`
     - PGDATA: `/data/postgres/`
   - Добавить секцию с healhcheck по аналогии с конфигурацией для `booking-service_catalog-db`
2. Добавить зависимости (секция depends_on) контейнера `booking-service_bookings-host` от `booking-service_bookings-db` и `booking-service_rabbitmq` по аналогии с секцией `booking-service_catalog-host`. 
3. Запустить контейнер `booking-service_bookings-db` с помощью команды `docker compose up -d booking-service_bookings-db` в каталоге с `docker-compose.yml`
4. Создать проекты в каталоге `src` в solution: 
   - Библиотека классов: `BookingService.Booking.Persistence` - сборка доступа к данным
   - Библиотека классов: `BookingService.Booking.AppServices.Contracts` - сборка контрактов сервисного слоя
   - В сборку `BookingService.Booking.Persistence` добавить ссылку на `BookingService.Booking.Domain`
   - В сборку `BookingService.Booking.Host` добавить ссылку на `BookingService.Booking.Persistence`
5. Добавить nuget-пакеты в проект `BookingService.Booking.Persistence`
   - `Microsoft.Extensions.Configuration` версии 8.0.0
   - `Microsoft.Extensions.Configuration.Binder` версии 8.0.0
   - `Microsoft.Extensions.Configuration.Json` версии 8.0.0
   - `Microsoft.EntityFrameworkCore` версии 8.0.8
   - `Microsoft.EntityFrameworkCore.Design` версии 8.0.8
   - `Microsoft.EntityFrameworkCore.Relational` версии 8.0.8
   - `Npgsql.EntityFrameworkCore.PostgreSQL` версии 8.0.4
6. Создать каталог `Configurations` в `BookingService.Booking.Persistence`
7. Создать класс `BookingAggregateConfiguration`, реализующий от `IEntityTypeConfiguration<BookingAggregate>`, в каталоге `Configurations` проекта `BookingService.Booking.Persistence`. В методе `Configure` выполнить маппинг агрегата `BookingAggregate` на базу данных:
   - Идентификатор - колонка `id` типа `bigint`; Первичный ключ таблицы
   - Статус - колонка `status` типа `int`
   - Идентификатор пользователя, - колонка `user_id` типа `bigint`
   - Идентификатор ресурса - колонка `resource_id` типа `bigint`
   - Дата начала бронирования - колонка `start_date` типа `date`
   - Дата окончания бронирования - колонка `end_date` типа `date`
   - Дата и время создания бронирования - колонка `created_at_date_time` типа `timestamptz`
8. Создать класс `BookingsContext`, наследующий от `DbContext`, в проекте `BookingService.Booking.Persistence`. Класс должен содержать `Bookings` типа `DbSet<BookingAggregate>` с публичным сеттером и геттером.
9. Переопределить метод `OnModelCreating` в `BookingsContext`: добавить регистрацию `BookingConfiguration` с помощью метода `ApplyConfiguration`, вызванного на аргументе метода `ModelBuilder`; Не забыть вызвать `base.OnModelCreating(modelBuilder)` в конце переопределенного метода
10. Создать класс `DesignTimeDbContextFactory` в проекте `BookingService.Booking.Persistence` для конфигурации DbContext во времени применения миграций 
11. Создать файл `appsettings.json` в проекте `BookingService.Booking.Persistence`.
```json
{
  "ConnectionStrings": {
    "BookingsContext": "server=localhost;port=5433;database=booking_service_bookings;uid=bookings_admin;pwd=admin_bookings"
  }
}
```
Добавить в файл проекта код, для копирования файла в директорию со сборкой
```xml
    <ItemGroup>
      <None Update="appsettings.json">
        <CopyToOutputDirectory>Always</CopyToOutputDirectory>
      </None>
    </ItemGroup>
```
12. Установить утилиту dotnet-ef. Для этого выполнить команду `dotnet tool install -g dotnet-ef`.
13. Создать инициальную миграцию с именем `InitialMigration` с помощью dotnet-ef в проекте `BookingService.Booking.Persistence`
14. Применить миграцию к локальной БД вызовом команды `dotnet ef database update` в каталоге проекта `BookingService.Booking.Persistence`.
15. Создать интерфейс `IBookingsRepository` в каталоге `Bookings` проекта `BookingService.Booking.Domain`. Интерфейс должен содержать следующие методы:
   - Создание бронирования: принимает `BookingAggregate`, не возвращает ничего
   - Получение бронирования по идентификатору: принимает идентификатор и `CancellationToken`, возвращает `ValueTask<BookingAggregate?>`
   - Обновление бронирования: принимает `BookingAggregate`, не возвращает ничего
16. Создать интерфейс `IUnitOfWork` в проекте `BookingService.Booking.Domain`. Он должен содержать следующие члены:
   - Свойство `public IBookingsRepository BookingsRepository { get; }`
   - Метод `Task CommitAsync(CancellationToken cancellationToken = default)`
17. Создать класс `BookingsRepository`, реализующий `IBookingsRepository`, в проекте `BookingService.Booking.Persistence`
```csharp
public class BookingsRepository : IBookingsRepository
{
  private DbSet<BookingAggregate> _dbSet;

  public BookingsRepository(BookingsContext context)
  {
    _dbSet = context.Bookings;
  }

  public void Create(BookingAggregate aggregate)
  {
    _dbSet.Add(aggregate);
  }

  public ValueTask<BookingAggregate?> GetById(long id, CancellationToken cancellationToken = default)
  {
    return _dbSet.FindAsync(id, cancellationToken);
  }

  public void Update(BookingAggregate aggregate)
  {
    _dbSet.Attach(aggregate);
    _dbSet.Entry(aggregate).State = EntityState.Modified;
  }
}
```
18. Создать класс `UnitOfWork`, реализующий `IUnitOfWork` в проекте `BookingService.Booking.Persistence`.
```csharp
public class UnitOfWork : IUnitOfWork
{
  private readonly BookingsContext _dbContext;

  public IBookingsRepository BookingsRepository { get; }

  public UnitOfWork(BookingsContext dbContext, IBookingsRepository bookingsRepository)
  {
    _dbContext = dbContext;
    BookingsRepository = bookingsRepository;
  }

  public async Task CommitAsync(CancellationToken cancellationToken = default)
  {
    await _dbContext.SaveChangesAsync(cancellationToken);
  }
}
```
19. Создать класс `ServiceCollectionExtensions` в сборке `BookingService.Booking.Persistence`
20. Создать метод расширения `AddPersistence` для регистрации зависимостей в DI:
```csharp
  public static IServiceCollection AddPersistence(this IServiceCollection services, string connectionString)
  {
    if (services == null)
      throw new ArgumentNullException(nameof(services));

    if (string.IsNullOrWhiteSpace(connectionString))
      throw new ArgumentNullException(nameof(connectionString));

    services.AddScoped<IBookingsRepository, BookingsRepository>();
    services.AddScoped<IUnitOfWork, UnitOfWork>();

    services.AddDbContext<BookingsContext>(
      (ctx, context) =>
      {
        context.UseNpgsql(connectionString)
          .UseLoggerFactory(ctx.GetRequiredService<ILoggerFactory>());
      }
    );

    return services;
  }
```
21. Добавить секцию со строками подключения в `appsettings.Development.json` и `appsettings.Production.json` в сборке `BookingService.Booking.Host`
```csharp
"ConnectionStrings": {
   "BookingsContext": "значение_строки_подключения"
}
```
22. Добавить вызов метода `AddPersistence` в метод `ConfigureServices` класса `Startup.cs` сборки `BookingService.Booking.Host`. В качестве аргумента передать строку подключения полученную из конфигурации по имени `BookingsContext` с помощью метода `GetConnectionString`, вызванного на `IConfiguration`.
23. Перенести интерфейс `IBookingQueries` в каталог `Bookings` сборки `BookingService.Booking.AppServices.Contracts`
24. Перенести класс `BookingQueries` в сборку `BookingService.Booking.Persistence`.
25. Перенести регистрацию `BookingQueries` в DI из метода расширения в сборке `BookingService.Booking.AppServices` в метод `AddPersistence` сборки `BookingService.Booking.Persistence`
26. Реализовать класс `BookingsQueries` с использованием `BookingsContext` для получения доступа к БД. При наличии отличных от `null` значений аргументов метода, дополнять `IQueriable` запрос условиями по принципе И.
27. Реализовать класс `BookingsService` с использованием `IUnitOfWork` и его поля `IBookingsRepository` для взаимодействия с БД. Методы изменяющие состояние агрегата сначала должны загрузить его из базы данных, а после внесения изменений сохранить состояние вызовом метода `IBookingsRepository.Update`. Когда все изменения выполнены, необходимо зафиксировать их в БД вызовом метода `IUnitOfWork.CommitAsync`.

## Критерии оценки

- Создан контейнер БД
- Создан `BookingsContext`, конфигурация агрегата `BookingAggregate` и миграция для создания базы данных
  - Поля агрегата смапплены на колонки в БД согласно заданию
  - Конфигурация зарегистрирована в методе `OnModelCreating` контекста
  - Создана миграция `Initial Migration`
- В контейнере БД создана база данных на основании миграции, соответствующая заданию
- Реализован паттерн Unit of Work
- Реализован паттерн Repository для `BookingAggregate`
- Класс `BookingQueries` перенесен в сборку `BookingService.Booking.Persistence`. Его реализация предоставляет возможность получать данные из БД с использованием `BookingsContext` согласно указанной в основном задании логике.
- Реализован метод расширения `AddPersistence` для регистрации зависимостей для доступа к БД в DI. Метод вызван в `ConfigureServices` класса `Startup.cs`
- Реализована логика сервиса `BookignsService` с использованием `UnitOfWork` для доступа и обновления данных.

## Материалы для изучения

[Oracle: Что такое база данных?](https://www.oracle.com/cis/database/what-is-database/)  
Днон П. Смит - EntityFramework в действии: главы 1 - 11  
[Metanit: Управление схемой БД и миграции](https://metanit.com/sharp/entityframeworkcore/2.15.php)  
[Learn EntityFramework Core](https://www.learnentityframeworkcore.com/)  
[Learn EntityFramework Core: EF Core Migrations](https://www.learnentityframeworkcore.com/migrations)  
[ConfigurationExtensions.GetConnectionString(IConfiguration, String) Method](https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.configuration.configurationextensions.getconnectionstring?view=net-8.0)

# Итерация 5: Синхронное межсервисное взаимодействие; Фоновые задачи

## Цель

Сформировать понимание:
- Преимущества и недостатки синхронных взаимодействий

Сформировать навыки:
- Использование RestEase клиента для взаимодействия с другим сервисом
- Реализация фоновых задач с использованием класса `BackgroundService`
- Использование Polly для создания политик повторных вызовов в случае ошибки

## Подготовка

- Проверь, что версии образов в локальном docker-compose для catalog-host и catalog-migrations соответсвуют версиям в docker-compose в этом репозитории

## Задача

1. Создать каталог `Options` в `BookingService.Booking.AppServices`
2. Создать класс `BookingCatalogRestOptions` с полем `BaseAddress` типа `string` в созданном каталоге
3. Добавить в файлы конфигурации `BookingService.BookingHost` секцию `BookingCatalogRestOptions` с следующими значениями `BaseAddress` для разных сред
  - `appsettings.Development.json` - "http://localhost:8000"
  - `appsettings.Production.json` - "http://booking-service_catalog-host:8080"
4. Подключить nuget-пакеты в `BookingService.Booking.AppServices`:
   - `BookingService.Catalog.Api.Contracts`
   - `Microsoft.Extensions.Hosting.Abstractions`
   - `Microsoft.Extensions.Options.ConfigurationExtensions` 
   - `Microsoft.Extensions.Http` 
   - `Microsoft.Extensions.Http.Polly`
5. Добавить в метод `AddAppServices` аргумент типа `IConfiguration`
6. Добавить подключение RestEase клиента для сервиса Catalog в метод `AddAppservices`
  - Сконфигурировать `BookingCatalogRestOptions` на основании секции конфигурации с помощью метода `Configure<T>`
  - Зарегистрировать `HttpClient` для RestEase клиента:
  ```csharp
    services.AddHttpClient(nameof(BookingCatalogRestOptions),
      (ctx, client) =>
      {
        var options = ctx.GetRequiredService<IOptions<BookingCatalogRestOptions>>().Value;
        client.BaseAddress = new Uri(options.BaseAddress);
        client.Timeout = TimeSpan.FromSeconds(90);
      });
  ```
  - Зарегистрировать RestEase клиент для сервиса Catalog:
  ```csharp
    services.AddScoped<IBookingJobsController>(ctx => RestClient.For<IBookingJobsController>(ctx.GetRequiredService<IHttpClientFactory>()
        .CreateClient(nameof(BookingCatalogRestOptions))));
  ```
7. Настроить политику повторных вызовов на основе библиотеки Polly для зарегистрированного http-клиента. В случае неуспешного ответа необходимо выполнить 4 повторf с экспоненциальным возрастанием ожидания между ними: 2, 4, 8 и 16 секунды соответственно. 
8. Добавить свойство `public Guid? CatalogRequestId { get; private set; }` в `BookingAggregate`
9. Добавить метод `SetCatalogRequestId(Guid catalogRequestId)`:
   - Бросает `DomainException`, если `catalogRequestId` имеет значение по умолчанию
   - Устанавливать значение `CatalogRequestId`, если оно null
   - Бросает `DomainException`, если `CatalogRequestId` уже имеет значение 
10.  Выполнить маппинг `CatalogRequestId` в `BookingAggregateConfiguration` на базу данных
11.  Создать миграцию `AddCatalogRequestId`
12.  Обновить БД в контейнере запуском `dotnet ef database update`
13.  Внедрить `IBookingJobsController` в `BookingService`
14.  Расширить логику метода `Create`. 
    - После создания экземпляра `BookingAggregate` создать идентификатор запроса для обращения к сервису Catalog, используя `Guid.NewGuid()`
    - Присвоить созданный идентификатор полю `CatalogRequestId` созданного агрегата
    - Выполнить метод `CreateBookingJob` на `IBookingJobsController` для создания задания на подтверждение ресурса в сервисе `Catalog`
15.  Расширить логику метода `Cancel`
    - Отменить бронирование в сервисе Catalog, если `CatalogRequestId` не равно null. Для простоты считаем, что если значение null, то запрос на бронирование в сервис Catalog не отправлялся, и можно отменить бронирование только в сервисе Booking.
    - Вызвать метод `CancelBookingJob` на `IBookingJobsController` для отмены бронирования в сервисе каталог.
16. Создать каталог `Jobs` в каталоге `Bookings` сборки `BookingService.Booking.AppServices`
17. Создать интерфейс `IBookingsBackgroundServiceHandler` с методом `Handle`, который возвращает `Task` и принимает `CancellationToken`
18. Создать интерфейс `IBookingsBackgroundQueries` в каталоге `Bookings` сборки `BookingService.Booking.Domain`. Интерфейс должен содержать метод `GetConfirmationAwaitingBookings`, получающий количество возвращаемых значений (значение по умолчанию 10) и возвращающий `IReadOnlyCollection<BookingAggregate>`. Агрегаты должны быть отсортированы по возрастанию Id. То есть, метод возвращает n бронирований в статусе "Ожидает подтверждения", отсортированных по возрастанию идентификатора. 
19. Создать класс `BookingsBackgroundQueries`, реализующий `IBookingsBackgroundQueries`, в сборке `BookingService.Booking.Persistence`. Для взаимодействия с БД использовать `BookingsContext`
20. Зарегистрировать `BookingsBackgroundQueries` как реализацию `IBookingsBackgroundQueries` в методе `AddPersistence` сборки `BookingsService.Booking.Persistence`
21. Создать класс `BookingsBackgroundServiceHandler` реализующий `IBookingsBackgroundServiceHandler`. Метод `Handle` должен реализовывать следующую логику
    - Получить из БД 10 агрегатов `Bookings` со статусом `AwaitConfirmation` с использованием интерфейса `IBookingsBackgroundQueries`
    - Для каждого агрегата:
      - Вывести в лог сообщение о том, что у агрегата некорреткное состояние, с уровнем `Warning` и перейти к следущему агрегату
      - Получить статус бронирования ресурса `BookingJobStatus` по `CatalogRequestId` из сервиса Catalog, вызвав метод `GetBookingJobStatusByRequestId` на `IBookingJobsController`
      - Перевести `BookingsAggregate` в состояние "Подтверждено", если полученный статус `Confirmed`
      - Перевести `BookignsAggregate` в состояние "Отменено", если полученный статус `Cancelled`
22. Создать класс `BookingsBackgroundService`, наследующий от `BackgroundService`. Метод ExecuteAsync должен реализовывать следующую логику:
    - Пока не запрошена отмена `stoppingToken`:
    - Создать scope через `ISeviceProvider`
    - Создать экземляр `IBookingsBackgroundServiceHandler` вызовом `scope.ServiceProvider.GetRequiredService<IBookingsBackgroundServiceHandler>()`
    - Вызвать метод `Handle` полученного экземпляра `IBookingsBackgroundServiceHandler`, передав в него `stoppingToken`
    - После успешного вызова поставить работу на ожидание на 5 секунд, через `Task.Delay` 
    - Вышеописанная логика должна быть обернута в блок try. В случае возникновения ошибки необходимо залогировать сообщение с уровнем `Error` и поставить работу на ожидание на 1 минуту, через `Task.Delay`
23. В методе AddAppServices:
    - Зарегистрировать `BookingsBackgroundServiceHandler` как реализацию `IBookingsBackgroundServiceHandler` с временем жизни scoped
    - Зарегистрировать `BookingsBackgroundService` через метод `AddHostedService`

## Дополнительная информация

- В методе создания задачи сервиса каталог предусмотрена проверка идентификатора ресурса, поэтому после расщирения логики создания бронирования, нужно брать идентификатор ресурса из результата ответа метода `api/hotels` сервиса Catalog. Swagger доступен по адресу `http://localhost:8000/swagger`.

## Критерии оценки

- Реализованы синхронные взаимодействия с сервисом Catalog:
  - В методе создания бронирования
  - В методе отмены бронирования
- Реализован BackgroundService, выполняющий проверку статуса бронирования в сервисе каталог и изменяющий состояние агрегата бронирования

## Материалы для изучения

[Dependency injection in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection?view=aspnetcore-8.0)  
[Implement HTTP call retries with exponential backoff with IHttpClientFactory and Polly policies](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/implement-resilient-applications/implement-http-call-retries-exponential-backoff-polly)  
[RestEase github page](https://github.com/canton7/RestEase)  
[OpenApi](https://www.openapis.org)  

# Итерация 6: Асинхронное межсервисное взаимодействие

## Цель

Сформировать понимание:
- Преимущества и недостатки асинхронных взаимодействий

Сформировать наывыки:
- Использование библиотеки Rebus для асинхронного взаимодействия с другим сервисом через RabbitMQ

## Подготовка

- Проверь, что версии образов в локальном docker-compose для catalog-host и catalog-migrations соответсвуют версиям в docker-compose в этом репозитории

## Задача

1. Запустить сервис Catalog и его зависимости через docker compose. Для этого в корневом каталоге решения выполнить команду `docker compose up -d booking-service_catalog-host`
2. Проверить, что сервис Catalog создал exchange и очередь в RabbitMQ
   - Открыть аминистративную панель RabbitMQ по пути `http://localhost:15672`
   - Авторизоваться используя логин admin и пароль admin
   - Проверить наличие exchange с именами `booking-service-catalog-direct`
 и `booking-service-topics` на вкладке `Exchanges`   
   - Проверить наличие queue с именем `booking-service-catalog-queue` на вкладке `Queues and Streams`
   - Проверить, что у очереди существует один Consumer, и она привязана к exchange `booking-service-catalog-direct` по routing key `booking-service-catalog-queue` на странице очереди `booking-service-catalog-queue`
![Queue](rabbitmq-queue-configuration.png)
   - В случае невыполнения одно из условий погасить docker compose командой `docker compose down` и вернуться к пункту 1
3. Подключить nuget-пакеты в сборку `BookingService.Booking.AppServices`
   - `BookingService.Catalog.Async.Api.Contracts` 
   - `Rebus`
   - `Rebus.ServiceProvider`
4. Подключить nuget-пакеты в сборку `BookingService.Booking.Host`
   - `Rebus.RabbitMq`
   - `Rebus.Serilog`
5. Создать класс `RebusRabbitMqOptions` с свойством `string ConnectionString` в сборке `BookingService.Booking.Host`
6. Зарегистрировать `RebusRabbitMqOptions` в DI и смапить на секцию конфигурации с именем `RebusRabbitMqOptions`, вызвав метод `Configure` на `IServiceCollection` в методе `ConfigureServices` класса `Startup.cs`.
7. Добавить секцию `RebusRabbitMqOptions` в appsettings:
   - Строка подключения для `appsettings.Development.json` - `amqp://admin:admin@localhost:5672`
   - Строка подключеня для `appsettings.Production.json` - `amqp://admin:admin@rabbitmq:5672`
8. Сконфигурировать Rebus для работы с RabbitMQ:
   - Подключить Rebus методе `ConfigureServices` класса `Startup`:
```csharp
services.AddRebus((builder, ctx) =>
  builder.Transport(t =>
    t.UseRabbitMq(ctx.GetRequiredService<IOptions<RebusRabbitMqOptions>>().Value.ConnectionString, "booking-service-booking-queue")
      .DefaultQueueOptions(queue => queue.SetDurable(true))
      .ExchangeNames("booking-service-booking-direct", "booking-service-topics"))
    .Serialization(s => s.UseSystemTextJson())
    .Logging(l => l.Serilog())
    .Routing(r => r.TypeBased()));
```
   - Подписаться на событие `BookingJobConfirmed` в методе `Configure` класса `Startup`: `app.ApplicationServices.GetRequiredService<IBus>().Subscribe<BookingJobConfirmed>();` 
   - Подписаться на событие `BookingJobDenied` в методе `Configure` класса `Startup`: `app.ApplicationServices.GetRequiredService<IBus>().Subscribe<BookingJobDenied>();` 
9. Изменить способ взаимодействия сервиса `BookingService` с сервисом Catalog на асинхронный
    - Удалить поле типа `IBookingsJobsController`, которое использовалось для синхронного взаимодействия.
    - Создать поле типа `IBus` и инициализировать его из конструктора, внедрив `IBus` из DI
    - Обновить метод `Create`
      - Создать экземпляр  `CreateBookingJobRequest` по аналогии с `CreateBookingJobComand`, заполнив его соответсвующими значениями`. `EventId` заполнять случайным Guid. 
      - Отправить созданный экземпляр `CreateBookingJobRequest`, вызвав метод `Publish` на `IBus`
      - Удалить код, использующий `IBookingsJobsController` и всё, что с ним связано из метода `Create`
    - Обновить метод `Cancel`
      - Создать экземпляр  `CancelBookingJobByRequestIdRequest` по аналогии с `CancelBookingJobByRequestIdCommand`, заполнив его соответсвующими значениями`. `EventId` заполнять случайным Guid. 
      - Отправить созданный экземпляр `CancelBookingJobByRequestIdRequest`, вызвав метод `Publish` на `IBus`
      - Удалить код, использующий `IBookingsJobsController` и всё, что с ним связано из метода `Cancel`
10. Добавить метод `Task<BookingAggregate?> GetBookingByRequestId(Guid requestId, CancellationToken = default)` в интерфейс `IBookingBackgroundQueries` и реализовать его в `BookingBackgroundQueries`.
11. Изменить способ получения обновлений от сервиса Catalog на асинхронный
    - Создать каталог `EventHandlers` в `BookingService.Booking.AppServices`
    - Создать обработчик для события `BookingJobConfirmed`, которое будет сигнализировать о подтверждении бронирования в сервисе Catalog
      - Создать класс `BookingJobConfirmedEventHandler`, реализующий `IHandleMessages<BookingJobConfirmed>` в каталоге `EventHandlers`
      - В методе `Handle` реализовать логику получения бронирования по `RequestId` из БД с использованием `IBookingBackgroundQueries` и его подтверждения. Если бронирование не найдено по идентификатору запроса из события, выводить в лог сообщение с уровнем `Warning`. 
      - Зарегистрировать `BookingJobConfirmedEventHandler` в DI с помощью метода `AddRebusHandler`, вызванного на `IServiceCollection` в методе `AddAppServices` 
    - Создать обработчик для события `BookingJobDenied`, которое будет сигнализировать о подтверждении бронирования в сервисе Catalog
       - Создать класс `BookingJobDeniedEventHandler`, реализующий `IHandleMessages<BookingJobDenied>` в каталоге `EventHandlers`
      - В методе `Handle` реализовать логику получения бронирования по `RequestId` из БД с использованием `IBookingBackgroundQueries` и его отмены. Если бронирование не найдено по идентификатору запроса из события, выводить в лог сообщение с уровнем `Warning`.  
      - Зарегистрировать `BookingJobDeniedEventHandler` в DI с помощью метода `AddRebusHandler`, вызванного на `IServiceCollection` в методе `AddAppServices`
    - Удалить `BookingsBackgroundService`, каталог `Jobs`, `BookingsBackgroundService`, `IBookingsBackgroundServiceHandler`, `BookingsBackgroundServiceHandler`. Удалить метод `GetConfirmationAwaitingBookings` из `IBookingBackgroundQueries` и его реализацию из `BookingBackgroundQueries`
12. Удалить nuget-пакет `BookingService.Catalog.Api.Contracts` из `BookingService.Booking.AppServices`
    - Удалить `BookingcatalogRestOptions` и секции конфигурации из appsettings.*.json
    - Удалить `Configure` для `BookingCatalogRestOptions` в методе `AddAppServices`
    - Удалить `AddHttpClient` с именем `BookingCataligRestOptions` в методе `AddAppServices`
    - Удалить регистрацию RestEase клиента для `IBookingJobsController` в методе `AddAppServices`  

## Критерии оценки 

- Синхронные взаимодействия заменены на асинхронные
  - Сконфигурирован Rebus
  - Сервис `BookingService.Bookings.Host` отправляет сообщения в RabbitMQ через Rebus при создании и отмене бронирования
  - Сервис `BookingService.Bookings.Host` подписан на события `BookingJobConfirmed` и `BookingJobDenied`
  - Реализованы обработчики для событий `BookingJobConfirmed` и `BookingJobDenied`
  - Удалены зависимости и классы, использовавшиеся для синхронного взаимодействия с сервисом Catalog

# Дополнительные задачи

## Реализовать middleware для логирования входящих запросов

Создать middleware, который будет логировать адрес, body и query входящего запроса. 

Добавить возможность конфигурирования middleware через конфигурацию приложения:
- Управлять уровнем логирования: должна быть возможность через конфигурацию указать с каким уровнем логирования будет выполняться запись в лог.
- Включать и отключать логирование полностью.
- Включать и отключать логирование body и query.

## Добавить кэширование бронирований в памяти

В запросах на получение информации о бронировании сначала проверять наличие записи в кэше:
- При наличии записи возвращать данные на основе значения из кэша.
- При отсутствии записи обращаться в БД и кэшировать бронирование. 

Установить время жизни бронирования в кэше равное 5 минутам. При обращении к записи, продлевать время жизни еще на 5 минут.

При изменении бронирования по результатам операций из сервиса "Каталог" необходимо обновить закешированные данные о бронировании.