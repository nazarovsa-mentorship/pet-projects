План проекта [OBSOLETE]
===

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