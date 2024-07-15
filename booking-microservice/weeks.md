План проекта [OBSOLETE]
===

# Недели 1 - 2: Создание микросервиса

## Цель

Сформировать навыки создания solution и структуры проектов, запуска микросервиса в docker-compose, работы с конфигурациями и подключения Serilog.

## Задача

- Создать каталог для решения:
	- Скопировать в него файл docker-compose.yml и каталог `docker-compose-mount` из репозитория
	- Создать пустой solution `BookingService.Bookings` для проекта.
	- В каталоге с созданным solution создать каталог `src`. В нем будут располагаться каталоги с сборками из пункта 2.
	- В каталоге с созданным solution создать каталог `tests`. В нем будут располагаться каталоги с сборками тестов
- В созданном solution создать проекты в каталоге `src`: 
	- Консольное приложение: `BookingService.Bookings.Host` - хост приложения
	- Консольное приложение: `BookingService.Bookings.Migrations` - миграции приложения
	- Библиотека классов: `BookingService.Bookings.Api.Contracts` - публичные контракты приложения
	- Библиотека классов: `BookingService.Bookings.AppServices` - сервисный слой
- Добавить `BookingService.Bookings.Host` в docker-compose.yml 
	- Сгенерировать dockerfile
	- Добавить в секцию `services` docker-compose.yml сервис booking-service_bookings-host
	- У сервиса должен быть healthcheck, проверяющий, что контейнер с миграциями успешно завершил выполнение
- Добавить `BookingService.Bookings.Migrations` в docker-compose.yml 
	- Сгенерировать dockerfile
	- Добавить в секцию `services` docker-compose.yml сервис booking-service_bookings-migrations, который будет запускаться один раз
- В сборке `BookingService.Bookings.Host` создать классы `Startup.cs` и `HostBuilderFactory.cs`, который будет содержать статичный метод, возвращающий сконфигурированный хост с использованием [Generic Host](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/host/generic-host?view=aspnetcore-8.0) (Не используем minimal-api)
- Добавить Swagger
- Добавить файл конфигурации `appsettings.Production.json`, который будет использоваться для настройки Production окружения во время запуска приложения в docker контейнере.
- Добавить логирование с Serilog в приложение вызовом `UseSerilog` на HostBuilderFactory. Serilog должен быть сконфигурирован следующим образом:
	- Конфигурироваться из конфигурации приложения (файла appsettings.json)
	- Использовать минимальный уровень логирования по умолчанию `Information` (задается в appsettings.json)
	- В Development окружении выводить логи в [консоль](https://github.com/serilog/serilog-sinks-console)
	- В Production окружении выводить логи в [консоль](https://github.com/serilog/serilog-sinks-console) и [файл](https://github.com/serilog/serilog-sinks-file) в каталоге `/var/logs/booking-service-bookings/`
- В сборке `BookingService.Bookings.Host` в каталоге Controllers создать класс контроллера, `BookingsController`, без реализации.

## Критерии оценки

1. Микросервис собирается и запускается как в IDE, так и в docker-compose
2. Микросервис предоставляет функциональность Swagger
3. При старте сервиса логи пишутся в консоль и файл

## Материалы для изучения

[Configuration in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/configuration/?view=aspnetcore-8.0)  
[Serilog ASP.NET Core github repo](https://github.com/serilog/serilog-aspnetcore)  
[.NET Generic Host in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/host/generic-host?view=aspnetcore-8.0) - Использование HostBuilder  
[Docker Compose Overview](https://docs.docker.com/compose/)  

# Недели 3 - 4: Создание  API и абстракций бизнес-логики приложения

## Цель

Сформировать навыки реализации API слоя приложения, обработки исключений в asp.net приложениях.

## Задача

- В сборке `BookingService.Bookings.Api.Contracts` создать каталог `Bookings/Dtos` и реализовать в нем DTO-класс `BookingData` . `BookingData` должен содержать все поля сущности "Бронирование" из описания задания.
- В сборке `BookingService.Bookings.Api.Contracts` cоздать каталог `Bookings/Commands`, в котором создать команды:
	- `CreateBookingCommand` - команда на создание бронирования
	- `CancelBookingCommand` - команда на отмену бронирования
- В сборке `BookingService.Bookings.Api.Contracts` cоздать каталог `Bookings/Queries`, в котором создать запросы:
	- `GetBookingByIdQuery` - запрос на получение бронирования по идентификатору
	- `GetBookingStatusByIdQuery` - запрос на получение статуса бронирования по идентификатору
	- `GetBookingsByFilterQuery` - запрос на получение бронирований по фильтру 
- В сборке `BookingService.Bookings.AppServices` создать каталог `Bookings/Commands`, в котором создать интерфейс `IBookingCommandHandler`с контрактами бизнес-логики, обрабатывающей команды. Методы интерфейса:
	- Обработчик `CreateBookingCommand`, возвращающий идентификатор созданного бронирования.
	- Обработчик `CancelBookingCommand`.
- Создать реализацию интерфейса `IBookingCommandHandler`, `BookingCommandHandler`, в том же каталоге, что интерфейс, без реализации методов (в теле методов `throw new NotImplementedException()`).
- Зарегистрировать `BookingCommandHandler` как реализацию `IBookingCommandHandler` с уровнем жизни Scoped
- В сборке `BookingService.Bookings.AppServices` создать каталог `Bookings/Queries`, в котором создать интерфейс  `IBookingQueryHandler` с контрактами бизнес-логики, обрабатывающей запросы. Методы интерфейса:
	- Обработчик `GetBookingByIdQuery`, возвращающий `BookingData`
	- Обработчик `GetBookingStatusByIdQuery`, возвращающий статус бронирования
	- Обработчик `GetBookingsByFilterQuery`, возвращающий `BookingData[]`
- Создать реализацию интерфейса `IBookingQueryHandler`, `BookingQueryHandler`, в том же каталоге, что интерфейс, без реализации методов (в теле методов `throw new NotImplementedException()`).
- Зарегистрировать `BookingQueryHandler` как реализацию `IBookingQueryHandler` с уровнем жизни Scoped.
- Внедрить `IBookingCommandsHandler` в конструктор контроллера и присвоить приватному полю `_commandHandler`. 
- Внедрить `IBookingQueryHandler` в конструктор контроллера и присвоить приватному полю `_queryHandler`.
- Реализовать методы синхронного API для управления бронированием в `BookingsController` согласно заданию. Методы контроллера должны вызывать соответствующие методы `IBookingCommandHander` и `IBookingQueryHandler`:
	- Метод создания должен возвращать идентификатор созданного бронирования
	- Методы, возвращающие одно "Бронирование", должны возвращать `Task<BookingData>` или массив `Task<BookingData[]>`
	- Методы, возвращающие несколько представлений "Бронирование", должны возвращать массив `Task<BookingData[]`
	- Метод отмены не возвращает ничего (Task), в случае ошибки возвращает описание в формате ProblemDetails (см. пункт 6.4)
- Создать исключение `BusinessRuleException`, которое будет порождаться на сервисном слое (Command- и Query- handlers) в случае, если не выполнены проверки бизнес-логики. Например, дата начала больше даты окончания, сущность в неправильном статусе, не указаны обязательные поля для выполнения операции, etc.
- Создать исключение `ValidationException` с дополнительным свойством `public string Errors[] { get; }`, которое будет порождаться на сервисном слое (Command- и Query- handlers) в случае неуспешных проверок валидации входящих команд и запросов. В свойство `Errors[]` будут записываться ошибки валидации.
- Реализовать ответ сервера в случае возникновения исключений по формату [[Problem Details]]. 
	- Выполнить маппинг исключения `BusinessRuleException` на http статус 400. `BusinessRuleException.Message` записывать в Title, `BusinessRuleException.StackTrace` записывать в Details. 
	- Выполнить маппинг исключения `BusinessRuleException` на http статус 402.  `ValidationException.Message` записывать в Title, `ValidationException.Errors` соединять разделителем `Environment.NewLine` в одну строку и записывать в Details. 

## Критерии оценки

1. Созданы классы для DTO, команд и запросов.  
2. В `BookingsController` внедрены `IBookingsCommandHandler` и `IBookingsQueriesHandler`.  
3. В `BookingsController` созданы методы синхронного API из задания. Методы вызывают соответсвующие методы `IBookingsCommandHandler` и `IBookingsQueriesHandler`.  
4. Подключена обработка исключений с ProblemDetails.  
5. Создано исключение `BusinessRuleException`. Выполнен маппинг `BusinessRuleException` на ProblemDetails.  

## Материалы для изучения

[Handling Web API Exceptions with ProblemDetails middleware](https://andrewlock.net/handling-web-api-exceptions-with-problemdetails-middleware/)
[Metanit: Создание контроллера](https://metanit.com/sharp/aspnet5/23.2.php)

# Недели 5 - 6: Создание и тестирование бизнес-логики приложения

## Цель

Сформировать навыки реализации бизнес-логики приложения и написания unit-тестов.

## Задача

- В сборке `BookingService.Bookings.AppServices` в каталоге `Bookings` создать сущность "Бронирование", `Booking`.
- Подключаем в `BookingService.Bookings.AppServices` nuget-пакет `BookingService.Catalog.Api.Contracts` для синхронных взаимодействий с сервисом "Каталог". См. `Приложение 1 Подключение клиента микросервиса "Каталог"`
- Создать валидаторы команд, `CreateBookingCommand` и `CancelBookingCommand`, с использованием FluentValidation. Валидация должна выолняться в начале метода обрабатывающего команду путем создания нового экземпляра валидатора `new CommandValidator();` В случае неуспешной валидации порождать исключение `ValidationException` и записывать ошибки из результата валидации в поле `Errors`.
	- `CreateBookingCommandValidator` должен проверять, что идентификатор пользователя и ресурса не равны 0, дата окончания бронирования больше даты начала бронирования.
	- `CancelBookingCommandValidator` должен проверять, что идентификатор бронирования не равен 0.
- Создать сборку `BookingsService.Bookings.AppServices.UnitTests` типа unit-tests с использованием xUnit в каталоге tests. В ней будут реализовываться тесты.
- В сборке `BookingsService.Bookings.AppServices.UnitTests` написать тесты на созданные классы валидаторов.
- Реализовать методы `BookingCommandHandler`. Класс должен взаимодействовать с микросервисом "Каталог" через внедрение `IBookingJobsController` и работать с слоем доступа к данным через репозиторий `IBookingRepository`.
- В сборке `BookingsService.Bookings.AppServices.UnitTests` написать тесты на методы `BookingCommandHandler` с использованием [[xUnit]]: Минимальный набор тестов должен включать в себя один позитивный и один негативный сценарии. Зависимости должны быть замоканы с использованием [[NSubstitute]]. Успешная проверка должна включать проверку вызова зависимостей (`IBookingRepository`) с верными параметрами.
- Реализовать методы `BookingQueryHandler`. Класс может использовать абстракции прямого доступа к данным без репозитория.
- Создать интерфейс репозитория, `IBookingRepository`, и его in-memory реализацию, `InMemoryBookingRepository`. Методы:
	- Получение бронирования по идентификатору
	- Добавление бронирования
	- Создание бронирования
	- Получение всех бронирований. Данный метод будет использоваться для реализации методов `BookingsQueriesHandler` с Linq для фильтрации. 
- Зарегистрировать `InMemoryBookingRepository` как реализацию `IBookingRepository` с уровнем жизни Scoped

## Критерии оценки

1. Реализована сущность `Booking`
2. Реализован интерфейс `IBookingCommandsHandler` согласно требованиям
3. Все публичные методы `BookingCommandsHandler` покрыты тестами
4. Реализован интерфейс `IBookingQueryHandler` согласно требованиям
5. Создан интерфейс `IBookingRepository`, и его in-memory реализация
6. Реализованы валидаторы для команд
7. Валидаторы для команд покрыты тестами

### Приложения

#### Приложение 1: Подключение клиента микросервиса "Каталог"

1. Создаем класс опций `CatalogServiceOptions` с полем `public string BasePath { get; set; }` для указания базового пути к микросервису "Каталог". 
2. В методе `ConfigureServices` класса `Startup` в сборки `BookingService.Bookings.Host` добавляем следующий код
```csharp
		// Конфигурируем CatalogServiceOptions из IConfiguration
	    services.Configure<CatalogServiceOptions>(Configuration.GetSection(nameof(CatalogServiceOptions)));
	    // Добавляем клиент RestEase для интерфейса `IBookingJobsController`. Теперь можно внедрять IBookingJobsController в сервисы для синхронного взаимодействия с сервисом "Каталог"
	    services.AddScoped(ctx => RestClient.For<IBookingJobsController>(ctx.GetRequiredService<IOptions<CatalogServiceOptions>>().Value.BasePath));
```
1. Для взаимодействия с микросервисом "Каталог" при запуске приложения из IDE добавляем в appsettings.json секцию 
```json
	  "CatalogServiceOptions": {
		  "BasePath": "http://localhost:8000"
	  }
```
1. Для взаимодействия с микросервисом "Каталог" при запуске приложения в docker-compose добавляем в appsettings.Production.json секцию
```json 
	"CatalogServiceOptions": {
		  "BasePath": "http://catalog-host:8000"
	  }
```
## Материалы для изучения

[Martin Fowler: CQRS](https://martinfowler.com/bliki/CQRS.html)   
[Microservices.io: CQRS](https://microservices.io/patterns/data/cqrs.html)   
[NSubstitute: github.io](https://nsubstitute.github.io)   
[Metanit: Паттерн 'Репозиторий' в ASP.NET](https://metanit.com/sharp/articles/mvc/11.php)  
[RestEase](https://github.com/canton7/RestEase)  
[FluentValidation](https://docs.fluentvalidation.net/en/latest/)

# Недели 9 - 10

# Недели 11 - 12

# Недели 13 - 14

# Недели 15 - 16