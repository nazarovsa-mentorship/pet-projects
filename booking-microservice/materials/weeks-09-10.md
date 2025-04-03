# Недели 9 - 10: Синхронное межсервисное взаимодействие; Фоновые задачи

# TODO: Синхронные межсервисные взаимодействия
Добавить схему аналогичную неделям 13-14, добавить описание Polly

# Цель

Сформировать понимание:
- Преимущества и недостатки синхронных взаимодействий

Сформировать наывыки:
- Использование RestEase клиента для взаимодействия с другим сервисом
- Реализация фоновых задач с использованием класса `BackgroundService`

# Синхронные межсервисные взаимодействия

Все взаимодействия между компонентами системы можно разделить на синхронные и асинхронные. 

Синхронные взаимодействия используются, когда при вызове другого компонента нужно сразу же получить ответ. То есть, вызывающая сторона ждет ответа от вызываемой: либо получает его, либо вызов завершается с ошибкой или по таймауту.

Классический пример синхронного взаимодействия вызов - по http.

# Инструменты упрощающие реализацию синхронных взаимодействий

Для того чтобы упростить реализацию синхронных взаимодействий существуют различные инструменты: OpenApi спецификация API и библиотеки, генерирующие клиентский код. 

## OpenApi

**OpenApi спецификация** - это инструмент для описания синхронного API. Оно описывает методы, агрументы и типы, использующиеся в API. При этом спецификация универсальна и не зависит от языка программирования. 

Swagger использует OpenApi спецификацию для отрисовки визуального интерфейса, позволяющего тебе просто взаимодействовать с API приложения.

## RestEase

**Библиотека RestEase** - это одна из библиотек, позволяющая ускорить разработку синхронных взаимодействий. С помощью специальной разметки создаются интерфейсы описывающие контракт взаимодействия с разрабатываемым сервисом. После разработки сборка с такими контрактами публикуется как nuget-пакет и может быть использована для быстрого подключения взаимодействия с сервисом. Вместо того, чтобы вручную реализовывать клиент для взаимодействия, библиотека создает клиент для вызовов сервиса в несколько строк. После этого разработанный интерфейс можно использовать в коде приложения напрямую или через DI для обращения к другому сервису.

Например, вот интерфейс описывающий контроллер для взаимодействия с сущностью самолета.
```csharp
[BasePath("api/planes")]
public interface IPlanesController 
{
    [Post]
    Task<long> Create([Body] CreatePlaneRequest request, CancellationToken = default);

    [Get("{id}")]
    Task<PlaneData> GetById([Route] long id, CancellationToken = default);
}
```

Контролер сервиса самолетов, должен реализовывать описанный интерфейс, при этом он не обязан делать это явно. То есть, контроллер не обязан наследовать от этого интерфейса. Но в случае, если описанный контракт не будет соответсвовать реализации, при использовании интерфейса на вызывающей стороне возникнут ошибки во время обращения к сервису.

По сути мы создали типизированный интефейс для взаимодействия с контроллером приложения, и дополнительно разметили его атрибутами из библиотеки RestEase для описания того, куда и как нужно обращаться по http.

```csharp
// Указываем базовый путь по отношению к адресу сервиса, на котором располагается контроллер
[BasePath("api/planes")]
public interface IPlanesController 
{
    // Указываем, что метод Create использует HttpPost
    [Post]
    // [Body] Указывает на то, что CreatePlateRequest должен браться из тела запроса. В контроллере будет использоваться атрибут [FromBody]
    Task<long> Create([Body] CreatePlaneRequest request, CancellationToken = default);

    // Указываем, что метод GetById использует HttpGet. Дополнительно указываем шаблонное значение "{id}" в пути. Финальный путь будет "api/planes/{id}", где {id} - параметр, в который будет подставляться Id самолета
    [Get("{id}")]
    // [Route] указывает на то, что long id должен быть заполнен из пути. В контроллере будет использоваться атрибут [FromRoute] 
    Task<PlaneData> GetById([Route] long id, CancellationToken = default);
}
```

# Фоновые задачи

**Фоновые задачи** - это задачи, которые выполняются паралельно с выполнением логики приложения. Стандартым способом выполнения фоновых задач является создание класса наследующего от `BackgroundService`. При этом можно использовать интерфейс `IHostedService`, который реализуется в `BackgroundService`, для самостоятельной реализации логики.

Фоновые задачи могут выполнять логику один раз или же на протяжении всего времени жизни приложения.

## BackgroundService

Рассмотрим пример фонового сервиса, который выводит в лог сообщение о старте приложения.

```csharp
public sealed class LoggingBackgroundService : BackgroundService
{
  private readonly ILogger<LoggingBackgroundService> _logger;

  public LoggingBackgroundService(ILogger<LoggingBackgroundService> logger)
  {
    _logger = logger;
  }
  
  protected override Task ExecuteAsync(CancellationToken stoppingToken)
  {
    _logger.LogInformation("Application started");
    return Task.CompletedTask;
  }
}
```

Этот сервис выводит сообщение с уровнем логирования `Information` во время старта приложения.

Для того, чтобы добавить этот сервис нужно вызвать следующий код на `IServiceCollection`
```csharp
services.AddHostedService<LoggingBackgroundService>();
```

Следующий пример выводит сообщение в лог каждые 5 секунд
```csharp
public sealed class LoggingBackgroundService : BackgroundService
{
  private readonly ILogger<LoggingBackgroundService> _logger;

  public LoggingBackgroundService(ILogger<LoggingBackgroundService> logger)
  {
    _logger = logger;
  }

  protected override async Task ExecuteAsync(CancellationToken stoppingToken)
  {
    while (!stoppingToken.IsCancellationRequested)
    {
      _logger.LogInformation("Application started");
      await Task.Delay(TimeSpan.FromSeconds(5), stoppingToken);
    }
  }
}
```

Чтобы добиться повторяющейся логики, она зациклена в бесконечном цикле, который будет остановлен, в случае отмены `stoppingToken`.  
`Task.Delay()` позволяет управлять интервалом повторения логики, за счет остановки выполнения потока.

## Внедрение зависимостей в BackgroundService

Так как BackgroundService регистрируется в DI-контейнере как singleton, нельзя внедрять в него зависимости с уровнем жизни меньше: scoped и transient. Для этого необходимо использовать интерфейс `IServiceProvider` предоставляющий доступ к DI контейнеру и создавать scope для сервисов вручную.

Пусть мы хотим использовать `IOptionsSnapshot<T>`, который позволяет получать новые значения опций при каждом внедрении, если включен hot-reload испточника конфигурации. Мы не можем просто внедрить `IOptionsSnapshot<T>` в наш сервис, так как он имеет уровень жизни scoped.


```csharp
public sealed class MyOptions 
{
    public string StringValue { get; set; }
}

public sealed class LoggingBackgroundService : BackgroundService
{
  private readonly IServiceProvider _serviceProvider;
  private readonly ILogger<LoggingBackgroundService> _logger;

  public LoggingBackgroundService(IServiceProvider serviceProvider, ILogger<LoggingBackgroundService> logger)
  {
    _serviceProvider = serviceProvider;
    _logger = logger;
  }

  protected override async Task ExecuteAsync(CancellationToken stoppingToken)
  {
    while (!stoppingToken.IsCancellationRequested)
    {
      // Создаем service scope. Это то же самое, что и scope, который создается при http-вызове к asp.net сервису. В рамках него все scoped сервисы будут создаваться единым экземпляром.
      using(var serviceScope = _serviceProvider.CreateScope())
      {
        // Получаем экземпляер IOptionsSnapshot<MyOptions>.
        var options = serviceScope.ServiceProvider.GetRequiredService<IOptionsSnapshot<MyOptions>>().Value;
        _logger.LogInformation("Current value: {Valuee}", options.StringValue);
      }

      await Task.Delay(TimeSpan.FromSeconds(5), stoppingToken);
    }
  }
}
```

Теперь код создает service scope, в рамках которого каждый раз создается новый сервис `IOptionsSnapshot<MyOptions>`.