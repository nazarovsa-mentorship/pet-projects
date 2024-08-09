# План проекта

# Недели 1 - 2: Создание микросервиса

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

# Недели 3 - 4: Создание API и абстракций бизнес-логики приложения

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
9.  Создать реализацию интерфейса `IBookingsQueries`, `BookingsQueries`, в том же каталоге, что интерфейс, без реализации методов (в теле методов `throw new NotImplementedException()`).
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

# Недели 5 - 6: Создание и тестирование бизнес-логики приложения

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
2. Создать каталог `Bookings` в сборке `BookingService.Booking.Domain.Contracts`
3. Переместить созданные ранее enums, относящиеся к `BookingData` из сборки `BookingService.Booking.Api.Contracts` в каталог `Bookings` сборки `BookingService.Booking.Domain.Contracts`
4. Добавить зависимость от сборки `BookingService.Booking.Domain.Contracts` в `BookingService.Booking.Api.Contracts`
5. Создать каталог `Bookings` в сборке `BookingService.Booking.Domain`
   - Создать класс `BookingAggregate` в каталоге `Bookings` сбокри `BookingService.Booking.Domain`
   - Реализовать в `BookingAggregate` поля необходимые для выполнения бизнес-логики (см. требования в общем репозитории)
   - Реализовать API класса `BookingAggregate` для обслуживания бизнес логики
     - Приватный конструктор, создающий новый экземпляр `BookingAggregate`
     - Статический метод `Initialize`, возвращающий `BookingAggregate` и принимающий аргументы необходимые для создания нового экземпляра класса. Должен проверять входные аргументы на валидность и возвращать результат вызова приватный конструктора.
     - Метод `Confirm`, который будет переводить бронирование в статус "Подтверждено"
     - Метод `Cancel`, кторый будет переводить бронирование в статус "Отменено"
6. Создать в solution каталог `_Tests` через IDE
7. Создать проект тестов `BookingService.Booking.Domain.UnitTests` в solution-каталоге `_Tests`. Проект должен находиться в каталоге файловой системы `tests`
8. Создать файл `BookingAggregateTests.cs` с классом `BookingAgregateTests` в `BookingService.Booking.Domain.UnitTests`
9. Реализовать Unit-тесты на бизнес-логику `BookingAggregate` в классе `BookingAgregateTests`. 
   - Метод `Initialize` должен быть покрыт тестами проверяющими валидацию входных параметров и создания экземпляра класса
   - Публичные методы бизнес-логики класса должны быть покрыты тестами, которые проверяют минимум 2 сценария: положительный и отрицательный
10. 

## Критерии оценки

1. Реализован класс `BookingAggregate`, содержащий бизнес-логику, указанную в требованиях пет-проекта (см. требования в общем репозитории)
	- Содержит методы `Initialize`, `Confirm` и `Cancel`
	- Выполняется валидация входных параметров в публичных методах
	- Методы реализуют корректные переходы по статусам (см. требования в общем репозитории)
2. Реализованы unit-тесты на разработанную бизнес-логику
	- Минимум один положительный и один отрицательный сценарий на каждый публичный метод `BookingAggreage`

## Материалы для изучения

[Блеск и нищета модели предметной области](https://habr.com/ru/companies/jugru/articles/503868/)

# Недели 7 - 8: 

# Недели 9 - 10

# Недели 11 - 12

# Недели 13 - 14

# Недели 15 - 16
