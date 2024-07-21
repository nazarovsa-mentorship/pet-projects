# План проекта

# Недели 1 - 2: Создание микросервиса

## Цель

Сформировать навыки:
- Создание solution и структуры проектов
- Написание docker-compose для dotnet-приложений
- Работы с конфигурациями приложения
- Подключение и конфигурация Serilog

## Задача

1. Создать каталог `booking-bookings` для проекта:
	- Скопировать в него файл docker-compose.yml и каталог `docker-compose-mount` из репозитория
	- Создать пустой solution `BookingService.Bookings` в созданном каталоге.
	- В каталоге с созданным solution создать каталог `src`. В нем будут располагаться каталоги с сборками из пункта 2.
	- В каталоге с созданным solution создать каталог `tests`. В нем будут располагаться каталоги с сборками тестов.
2. В созданном solution создать проекты в каталоге `src`: 
	- Консольное приложение: `BookingService.Bookings.Host` - хост приложения
	- Библиотека классов: `BookingService.Bookings.Api.Contracts` - публичные контракты приложения
	- Библиотека классов: `BookingService.Bookings.AppServices` - сервисный слой
	- В сборку `BookingService.Bookings.Host` добавить ссылку на `BookingService.Bookings.AppServices` и `BookingService.Bookings.Api.Contracts`
3. Добавить `BookingService.Bookings.Host` в docker-compose.yml 
	- Сгенерировать dockerfile средствами IDE
	- Добавить в секцию `services` docker-compose.yml сервис booking-service_bookings-host
4. В сборке `BookingService.Bookings.Host` создать классы `Startup.cs` и `HostBuilderFactory.cs`, который будет содержать статичный метод, возвращающий сконфигурированный хост с использованием [Generic Host](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/host/generic-host?view=aspnetcore-8.0) (Не используем minimal-api)
5. Добавить Swagger
6. Добавить файл конфигурации `appsettings.Production.json`, который будет использоваться для настройки Production окружения во время запуска приложения в docker-compose.
7. Добавить логирование с Serilog в приложение вызовом `UseSerilog` на HostBuilderFactory. Serilog должен быть сконфигурирован следующим образом:
	- Конфигурироваться из конфигурации приложения (файла appsettings.json)
	- Использовать минимальный уровень логирования по умолчанию `Information` (задается в appsettings.json)
	- В Development окружении выводить логи в [консоль](https://github.com/serilog/serilog-sinks-console)
	- В Production окружении выводить логи в [консоль](https://github.com/serilog/serilog-sinks-console) и [файл](https://github.com/serilog/serilog-sinks-file) в каталоге `/var/logs/booking-service-bookings/`
8. В сборке `BookingService.Bookings.Host` в каталоге Controllers создать класс контроллера, `BookingsController`, без реализации.

## Критерии оценки

1. Микросервис собирается и запускается как в IDE, так и в docker-compose
2. Микросервис предоставляет функциональность Swagger
3. При старте сервиса логи пишутся в консоль и файл

## Материалы для изучения

[Configuration in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/configuration/?view=aspnetcore-8.0)  
[Serilog ASP.NET Core github repo](https://github.com/serilog/serilog-aspnetcore)  
[.NET Generic Host in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/host/generic-host?view=aspnetcore-8.0) - Использование HostBuilder  
[Docker Compose Overview](https://docs.docker.com/compose/)  

# Недели 3 - 4: Создание API и абстракций бизнес-логики приложения

# Недели 5 - 6: Создание и тестирование бизнес-логики приложения

# Недели 9 - 10

# Недели 11 - 12

# Недели 13 - 14

# Недели 15 - 16