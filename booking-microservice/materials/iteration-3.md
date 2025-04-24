# Итерация 3: Создание бизнес-логики приложения

# Цель

Сформировать понимание:
- Отличия анемичной и богатой модели разработки бизнес-логики.
- Назначение unit-тестирования.

Сформировать навыки:
- Реализации бизнес-логики приложения с использованием богатой модели.
- Реализации минимального набора unit-тестов на разработанную бизнес-логику.

# Абстракция даты и времени

При разработке бизнес-логики приходится работать с датой и временем. При этом использовать свойства `DateTime.UtcNow`, `DateTimeOffset.UtcNow` в явном виде нельзя потому что они делают код недетерминированным и лишают разработчика возможности полноценно тестировать логику, зависящую от времени.

Для того, чтобы исправить это, следует использовать абстракцию даты и времени. Это интерфейс с методами, предоставляющими доступ к текущей дате и времени. Использование интерфейса позволяет заменять реализацию при тестировании приложения и явно управлять значениями, получаемыми в коде.

За основу возьмем следующий контракт и его реализацию:
```csharp
public interface ICurrentDateTimeProvider 
{
    /// <summary>
    /// Локальное время
    /// </summary>
    public DateTimeOffset Now { get; }

    /// <summary>
    /// Время в UTC
    /// </summary>
    public DateTimeOffset UtcNow { get; }
}

public class DefaultCurrentDateTimeProvider : ICurrentDateTimeProvider
{
    public DateTimeOffset Local => DateTimeOffset.Now.ToLocalTime();

    public DateTimeOffset Utc => DateTimeOffset.UtcNow;
}
```

Далее необходимо зарегистрировать реализацию в DI контейнере, а в код внедрять `ICurrentDateTimeProvider` для получения текущей даты и времени.  

## DateTimeOffset

`DateTimeOffset` в отличии `DateTime` представляет дату с часовым поясом, что позволит без лишних действий обрабатывать дату и время, поступающие в API при работе клиентов в разных часовых поясах.

# Модели предметной области

Как бы там ни было, мы создаем программное обеспечение для решения реальной задачи. А "реальная задача" существует в некой предметной области. 

Давай дадим определение:

> Предметная область - часть реального мира, подлежащая изучению с целью организации управления и, в конечном счете, автоматизации.

Мы создаем приложения для управления и автоматизации реальных процессов. И центральной идеей создания приложения является реализация бизнес-задачи, которая упростит жизнь потребителям или решит их проблемы.

Предметная область может быть большой. Например, система управления доставкой поставок на склад хранения товаров. Управление поставками - это и есть предметная область в нее входят все сущности и бизнес-процессы, необходимые для выполнения задач бизнеса и пользователей. В то же время сама поставка, заявка на поставку или грузо-место на складе является моделью предметной области. То

Для описания предметной область в программировании существуют модели, описывающие объекты, процессы или реакции на какое-либо событие.

## Анемичная модель предметной области

> **Анемичная модель предметной области** - это подход к разработке ПО, в котором для отображения бизнес-логики используются DTO-объекты, которые не содержат бизнес-логики, а бизнес-логика реализуется на сервисном слое.

Анемичная модель не отражает предметную область по причине того, что объекты в анемичной модели, по своей сути являются DTO (Data transfer object). 

Подход с использованием анемичной моделей часто используют в разработке. И эта модель почти на 100 процентов совпадает по структуре с моделью базы данных. Логика управления состоянием этой модели располагается в других частях приложения, других слоях. Иногда этот слой называют "слоем бизнес логики". Он может быть реализован с применением сервисов (классов в которых содержатся методы по валидации и изменению модели) либо в классах обработчиках (Handlers), когда каждый обработчик отвечает за 1 действие (Подробнее в теме про CQRS)

Проблема такого подхода в том, что с течением времени логика приложения разрастается, и начинает размазываться тонким слоем по всему решению. Поддерживать такой код становится достаточно сложно и увеличивается вероятность появления ошибок.

Давай посмотрим на анемичную модель, описывающую самолет перед взлетом:

```csharp
public class Plane
{
  public long Id { get; set; }

  public string AircraftNumber { get; set; }

  public int MaxSeatsCount { get; set; }

  public long[] PassengerIds { get; set; }

  public long PilotId { get; set; }

  public bool FlightStarted { get; set; }
}
```

Как можно заметить эта модель лишена какой-либо логики. Она содержит свойства, и они являются открытыми для чтения и для изменения из любой точки приложения. Мы можем получить данные из хранилища, поместить их в такую модель, и начать с ней работать (изменять, редактировать) на слое бизнес логики. В целом, почему бы и нет. 

Давай посмотрим, как будет выглядеть код сервиса содержащего методы создания самолета, запуска в полет, смены пилота и добавления пассажира в анемичной модели:

```csharp
public class PlanesService
{
  private readonly IPlanesRepository _planeRepository; // <-- Репозиторий для доступа к данным

  private static readonly Regex AircraftNumberValidationRegex =
    new Regex(@"^[A-Z]{2}-\d{4}$", RegexOptions.Compiled); // <-- Регулярное выражение для валидации

  public PlanesService(IPlanesRepository planeRepository)
  {
    _planeRepository = planeRepository;
  }

  public async Task<long> CreatePlane(string aircraftNumber, int maxSeatsCount, long pilotId,
    params long[] passengerIds) // <-- Метод создания нового самолета
  {
    ArgumentException
      .ThrowIfNullOrWhiteSpace(
        aircraftNumber); // <-- Начиная с этой строки, и до создания самолета выполняем валидацию входных данных

    if (maxSeatsCount < 1)
    {
      throw new ArgumentException("Max seats count should be greater than 1");
    }

    if (passengerIds.Length > maxSeatsCount)
    {
      throw new InvalidOperationException("Passengers count greater than max seats count");
    }

    if (!AircraftNumberValidationRegex.IsMatch(aircraftNumber)) // << -- Проверяем, что значение соответсвует шаблону
    {
      throw new ArgumentException("Invalid aircraft number format", nameof(aircraftNumber));
    }

    var plane = new Plane // <-- Создаем экземпляр самолета
    {
      AircraftNumber = aircraftNumber,
      MaxSeatsCount = maxSeatsCount,
      PilotId = pilotId,
      PassengerIds = passengerIds,
      FlightStarted = false,
    };

    return await _planeRepository.Create(plane); // <-- Сохраняем самолет в БД
  }

  public async Task StartFlight(long planeId) // <-- Метод начала полета
  {
    var plane = await _planeRepository.GetById(planeId); // <-- Извлекаем самолет из БД
    if (plane.FlightStarted) // <-- Проверяем, что полет еще не начался
    {
      throw new InvalidOperationException("Failed to start flight: flight already started");
    }

    plane.FlightStarted = true; // <-- Меняем состояние самолета: полет начался

    await _planeRepository.Update(plane); // <-- Сохраняем измененное состояние самолета в БД
  }

  public async Task ChangePilot(long planeId, Person pilot) // <-- Метод смены пилота
  {
    var plane = await _planeRepository.GetById(planeId); // <-- Извлекаем самолет из БД
    if (plane.FlightStarted) // <-- Проверяем, что полет еще не начался
    {
      throw new InvalidOperationException("Failed to change pilot: flight already started");
    }

    plane.PilotId = pilot.Id; // <-- Меняем состояние самолета: пилот изменен

    await _planeRepository.Update(plane); // <-- Сохраняем измененное состояние самолета в БД
  }

  public async Task AddPassenger(long planeId, Person person) // <-- Метод добавления пассажира
  {
    ArgumentNullException.ThrowIfNull(person);

    var plane = await _planeRepository.GetById(planeId); // <-- Извлекаем самолет из БД
    if (plane.FlightStarted) // <-- Если полет начался, бросаем исключение, так как нельзя добавить пассажира  
    {
      throw new InvalidOperationException("Failed to add passenger: flight already started");
    }

    if (plane.PassengerIds.Length ==
        plane.MaxSeatsCount) // <-- Если свободных мест нет, бросаем исключение, так как нельзя добавить пассажира       
    {
      throw new InvalidOperationException("Failed to add passenger: not enough free seats");
    }

    if (plane.PassengerIds.Any(x => x == person.Id)) // <-- Если пассажир уже добавлен, бросаем исключение      
    {
      throw new InvalidOperationException("Failed to add passenger: already added");
    }

    plane.PassengerIds = plane.PassengerIds.Concat(new[] { person.Id })
      .ToArray(); // <-- Меняем состояние самолета: добавляем id пассажира

    await _planeRepository.Update(plane); // <-- Сохраняем измененное состояние самолета в БД
  }
}

public interface IPlanesRepository
{
  Task<long> Create(Plane plane);

  Task<Plane> GetById(long id);

  Task Update(Plane plane);
}
```

Каждый метод сервиса выполняет 4 функции:
- Извлекает данные из БД (кроме метода создания)
- Выполняет проверки бизнес-правил
- Меняет состояние самолета
- Сохраняет измененное состояние самолета в БД

Обрати внимание, что вся логика проверки бизнес-правил находится в методах сервиса. Это приводит к тому, что
**код валидации и выполнения бизнес-правил невозможно переиспользовать**. В случае такой необходимости, придется продублировать его в другой части приложения, либо завести отдельный класс валидатор. **Это вызывает "размазывание" бизнес-логики по слоям приложения, что приводит к сложности поддержки и тестирования кода.**
Например, ты добавил один и тот же по смене пилота в два разных класса. В следующий раз, чтобы поменять логику, тебе придется внести изменения в оба. А еще нужно исправить два теста. Фактически, это приводит к увеличению времени х2.

Пока программа небольшая и состоит из 2-3 сервисов сложности не заметны. Но когда проект растет, и количество сервисов переваливает за десятки, то вносить изменения становится очень трудозатрантно.

Так же стоит отметить возможность тестирования кода. **Когда появляются отдельные валидаторы, или проверки на валидность данных проходят непосредственно в коде сервиса, становтися сложно писать Unit-тесты.** Например, ты не можешь отдельно проверить логику валидации номера борта (`AircraftNumber`) в методе создания самолета (`CreatePlane`). Тебе придется выполнить метод `CreatePlane`, что потребует передать все данные и создать заглушки для всех зависимостей сервиса из конструктора. И это всё, чтобы сделать одну проверку.

**Плюсом такого подхода является его простота:**
- Одна модель может использоваться как модель БД так и модель бизнес логики
- Нет необходимости создавать отдельные мапперы
- Класс модели маленький, содержит только необходимые свойства

## Богатая модель предметной области

> **Богатая модель предметной области** - это подход к разработке ПО, в котором для отображения предметной области используются классы, которые содержат в себе бизнес-логику для управления состоянием объекта.

Понятие "богатая модель" больше подходит к названию - "модель предметной области". Эта модель содержит в себе не только поля и свойства реального объекта, но и методы по управлению (изменению) этими свойствами (изменению инварианта). Другими словами, бизнес логика по управлению состоянием объекта предметной области сокрыта в самой модели предметной области.

Богатая модель далеко не всегда (почти никогда) не сходится на 100 процентов с тем, как она будет храниться в каком-то хранилище (например БД). Потому что проектирование такой системы отталкивается не от способа хранения, и не от API, а от потребностей бизнеса и предметной области. А вся инфраструктура уже выбирается после по мере необходимости.

# Агрегат

> Агрегат - это корневой объект, описывающий модель предметной области.

Как правило агрегатом выбирают корневой объект, через который меняют состояние системы. На основе нашего примера, это - самолет (`Plane`).   
Агрегат содержит характеризующие его поля, а также может содержать сущности и ValueObject'ы.

# Сущность(Entity)

> Сущность - это объект, описывающий предметную модель и имеющий идентификатор. 

Так как сущность имеет идентификатор, она хранится в БД и может быть использована при построении других агрегатов. В примере с самолетом, сущностью является физическое лицо (`Person`), которое может быть и пилотом и пассажиром.

# ValueObject

> ValueObject - это объект, описывающий значение в предметной области.

ValueObject не имеет идентификатора. Поэтому он существует только в рамках агрегата или сущности. 

ValueObject хранит значение и выполняет необходимую валидацию при его создании. В примере с самолетом, бортовой номер (`AircraftNumber`) является ValueObject.

Типичные ValueObject: email и номер телефона.

# Пример

Теперь ты в общих чертах знаешь, что такое агрегат и ValueObject. Давай рассмотрим модель самолета в "богатой" модели на примере кода, чтобы лучше понять суть подхода.

```csharp
public class PlanesService
{
  private readonly IPlanesRepository _planeRepository; // <-- Репозиторий для доступа к данным

  public PlanesService(IPlanesRepository planeRepository)
  {
    _planeRepository = planeRepository;
  }

  public async Task<long> CreatePlane(string aircraftNumber, int maxSeatsCount, Person person,
    params Person[] passengers) // <-- Метод создания нового самолета
  {
    // Создаем экземпляр самолета. Все валидации выполняются внутри
    var plane = Plane.Initialize(aircraftNumber, person, maxSeatsCount, passengers);

    return await _planeRepository.Create(plane); // <-- Сохраняем самолет в БД
  }

  public async Task StartFlight(long planeId) // <-- Метод начала полета
  {
    var plane = await _planeRepository.GetById(planeId); // <-- Извлекаем самолет из БД

    // Отправляем самолет в полет. Валидация и изменение состояния происходят внуктри метода
    plane.StartFlight();

    await _planeRepository.Update(plane); // <-- Сохраняем измененное состояние самолета в БД
  }

  public async Task ChangePilot(long planeId, Person pilot) // <-- Метод смены пилота
  {
    var plane = await _planeRepository.GetById(planeId); // <-- Извлекаем самолет из БД

    // Меняем пилота. Валидация и изменение состояния происходят внуктри метода
    plane.ChangePilot(pilot);

    await _planeRepository.Update(plane); // <-- Сохраняем измененное состояние самолета в БД
  }

  public async Task AddPassenger(long planeId, Person person) // <-- Метод добавления пассажира
  {
    var plane = await _planeRepository.GetById(planeId); // <-- Извлекаем самолет из БД

    // Добавляем пассажира. Валидация и изменение состояния происходят внуктри метода
    plane.AddPassenger(person);

    await _planeRepository.Update(plane); // <-- Сохраняем измененное состояние самолета в БД
  }
}

// <summary>
/// Модель самолета
/// </summary>
public class Plane
{
  private readonly List<Person> _passengers = new();

  public long Id { get; }

  public AircraftNumber AircraftNumber { get; }

  public Person Pilot { get; private set; }

  public int MaxSeatsCount { get; }

  public bool FlightStarted { get; private set; }

  public IReadOnlyCollection<Person> Passengers => _passengers.AsReadOnly();

  private Plane(
    in AircraftNumber aircraftNumber,
    Person pilot,
    int maxSeatsCount,
    params Person[] passengers)
  {
    AircraftNumber = aircraftNumber;
    Pilot = pilot;
    MaxSeatsCount = maxSeatsCount;
    _passengers.AddRange(passengers);
  }

  public static Plane Initialize(
    string aircraftNumber,
    Person pilot,
    int maxPassengersCount,
    params Person[] passengers)
  {
    if (maxPassengersCount < 1)
    {
      throw new ArgumentException("Max seats count should be greater than 1");
    }

    if (passengers.Length > maxPassengersCount)
    {
      throw new InvalidOperationException("Passengers count greater than max seats count");
    }

    var aircraftNumberValueObject = new AircraftNumber(aircraftNumber);

    return new Plane(in aircraftNumberValueObject, pilot, maxPassengersCount, passengers);
  }

  public void StartFlight() // <-- Метод начала полета
  {
    if (FlightStarted) // <-- Если полет начался, бросаем исключение, так как нельзя начать полет два раза
    {
      throw new InvalidOperationException("Failed to start flight: flight already started");
    }

    FlightStarted = true; // <-- Изменяем состояние самолета: полет начался
  }

  public void ChangePilot(Person pilot) // <-- Метод изменения пилота
  {
    if (FlightStarted) // <-- Если полет начался, бросаем исключение, так как нельзя поменять пилота
    {
      throw new InvalidOperationException("Failed to change pilot: flight already started");
    }

    Pilot = pilot; // <-- Изменяем состояние самолета: меняем пилота
  }

  public void AddPassenger(Person person) // <-- Метод добавления пассажира
  {
    ArgumentNullException.ThrowIfNull(person);

    if (FlightStarted) // <-- Если полет начался, бросаем исключение, так как нельзя добавить пассажира  
    {
      throw new InvalidOperationException("Failed to add passenger: flight already started");
    }

    if (_passengers.Count ==
        MaxSeatsCount) // <-- Если свободных мест нет, бросаем исключение, так как нельзя добавить пассажира       
    {
      throw new InvalidOperationException("Failed to add passenger: not enough free seats");
    }

    if (_passengers.Any(x => x.Id == person.Id)) // <-- Если пассажир уже добавлен, бросаем исключение      
    {
      throw new InvalidOperationException("Failed to add passenger: already added");
    }

    _passengers.Add(person);
  }
}

/// <summary>
/// Бортовой номер самолета. Имеет длину 7 символов и формат "RA-1234"
/// </summary>
public readonly record struct AircraftNumber
{
  private static readonly Regex ValidationRegex =
    new Regex(@"^[A-Z]{2}-\d{4}$", RegexOptions.Compiled); // <-- Регулярное выражение для валидации

  public AircraftNumber(string value)
  {
    ArgumentException.ThrowIfNullOrWhiteSpace(value); // <-- Проверяем, что значение не пустое

    if (!ValidationRegex.IsMatch(value)) // << -- Проверяем, что значение соответсвует шаблону
    {
      throw new ArgumentException("Invalid aircraft number format", nameof(value));
    }

    Value = value;
  }

  public string Value { get; }
}

/// <summary>
/// Физическое лицо
/// </summary>
public class Person
{
  public long Id { get; }

  public string FullName { get; }

  public Person(long id, string fullName)
  {
    ArgumentException.ThrowIfNullOrWhiteSpace(fullName); // <-- Проверяем, что ФИО не пустое. Для примера валидация упрощена.

    Id = id;
    FullName = fullName;
  }
}
```

Теперь каждый метод сервиса выполняет 3 функции:
- Извлекает данные из БД (кроме метода создания)
- Вызывает метод класса Plane, который выполняет валидацию и меняет состояние самолета
- Сохраняет измененное состояние самолета в БД

**Теперь бизнес-логика по взаимодействию с самолетом инкапсулирована в классе `Plane`**, и он предоставляет API в виде методов: `Initialize`, `StartFlight`, `ChangePilot` и `AddPassenger`. Каждый метод выполняет проверки и изменение состояния самолета. Поэтому теперь сервис (`PlanesService`) выполняет только интеграционыне функции: чтение из БД, вызов соответсвующего метода экземпляра класса `Plane` и сохранение изменений в БД. Код сервиса стал проще, а в коде класса `Plane` мы сконцентрировались на написании только бизнес-логики.

> Благодаря тому, что бизнес-логика взаимодействия с самолетом инкапсулирована в одном классе, ты можешь переиспользовать ее в разных частях приложения, используя класс `Plane`.

Обрати внимание на ValueObject `AircraftNumer`. Он содержит в конструкторе валидацию переданного строкового значения. Это позволяет не выполнять дополнительные проверки перед созданием объекта-значения. Теперь для создания валидного бортового номера (`AircraftNumber`) достаточно написать следующий код:
```csharp
var aircraftNumber = new AircraftNumber("RA-1111");
```

В случае неуспешной валидации, конструктор бросит исключение.

Что стало с тестированием?  
> Чтобы протестировать код бизнес-логики больше не нужно создавать экземпляр сервиса (`PlanesService`). Достаточно тестировать класс `Plane`. А это упрощает код, потому что больше не нужно создавать заглушки для зависимостей.  

**Плюсы богатой модели:**
- Возможность переиспользовать логику ValueObject'ов и агрегатов
- Простота тестирования ValueObject'ов и агрегатов; Тестируем только бизнес-логику, а не интеграции

Основным минусом богатой модели по сравнению с анемичной является повышенная сложность "входа".  

## Материалы для изучения
[Блеск и нищета модели предметной области](https://habr.com/ru/companies/jugru/articles/503868/)

# Unit-тесты

**Unit-тесты** - это инструмент контроля качества кода. Юнит-тесты позволяют автоматически проверять бизнес-логику при внесении изменений на наличие ошибок и тем самым экономят время разработчика при разработке новых фич. Разумеется, сначала нужно потратить время на их написание. 

Более официальное определение из книги Принципы юнит-тестирования Владимира Хорикова:
Unit-тестом называют автоматизированный тест, который проверяет правильность работы небольшого фрагмента кода (также называемого юнитом) и делает это быстро. 

**Unit** - это модуль. Под модулем обычно понимают небольшой независимый блок кода. Например, класс. Самолет из предыдущего раздела можно считать юнитом. 

**Ты будешь разрабатывать тесты с использованием фреймворка xUnit.**

## Структура unit-теста: Патерн AAA

При разработке тестов ты будешь использовать паттерн AAA.  
В нем каждый тест разбивается на 3 части:
- Arrange (подготовка)
- Act (действие)
- Assert (проверка)

Для примера рассмотрим класс `Calcucator` с методом вычисления суммы двух чисел:
```csharp
public class Calculator
{
  public double Sum(double first, double second)
  {
    return first + second;
  }
}
```

Вот как будет выглядеть тест для класса `Caclucator` по паттерну AAA:
```csharp
public class CalculatorTests // <-- Класс контейнер для тестов
{
  [Fact] // <-- Атрибут xunit, обозначающий тест
  public void Sum_of_two_numbers() // <-- Название юнит-теста
  {
    // Arragne
    double first = 10;
    double second = 20l
    var calculator = new Calculator(); // <-- Секция подготовки

    // Act
    double result = calculator.Sum(first, second); // <-- Секция действия

    // Assert
    Assert.Equal(30, result); // <-- Секция проверки
  }
}
```

Паттерн ААА предоставляет простую единообразную структуру для всех тестов в проекте. Единобразие - главное преимущество паттерна: освоив его, ты сможешь легко прочитать и понять любой тест.  
Структура выглядит так:
- В секции подготовки тестируемая система (system under test, SUT) и ее зависимости приводятся в нужное начальное состояние
- В секции действия вызываются методы SUT, передаются подготовленные зависимости и сохраняется выходное значение (если оно есть)
- В секции проверки проверяется результат, который может быть представлен возвращаемым значением или итоговым состоянием тестируемой системы.

## Необходимое количество unit-тестов

Тестирование кода измеряют метрикой покрытия тестов. Она измеряется в процентах и показывает сколько процентов кода приложения покрыто unit-тестами. То есть проверяется в ходе выполнения реализованных unit-тестов. Эта метрика может вычисляться по-разному. Ознакомься с темой подробнее в первой главе книги Владимира Хорикова из раздела [Материалы для изучения](#материалы-для-изучения-1). 

Нет единой точки зрения насколько высоким должен быть показатель покрытия кода. Здравый смысл подсказывает, что процент покрытия тестами наиболее критичной для бизнеса функциональности должен стремиться к 100%. Но не всегда можно достичь это в виду отсутвия ресурсов и времени. При обучении мы будем руководствоваться следующим правилом:
> Каждый метод бизнес-логики должен иметь минимум один тест на положительный сценарий и на отрицательный.

**Положительный сценарий** - это сценарий выполнения логики, который приводит к ожидаемому результату.  
**Негативный сценарий** - это сценарий выполнения логики, который не приводит к ожидаемому результату.

То есть на каждый метод класса бизнес-логики мы пишем два теста. Например, рассмотрим на примере новой модели самолета: При создании указывается количество мест, которое не может быть меньше 1 и больше 300. Класс содержит метод бронирования места `TakeSeat`, который проверяет, наличие свободных мест и уменьшает его на 1, если места доступны.

```csharp
public class Plane 
{
  private int _seatsCount; // <-- Общее количество мест в самолете

  private int _freeSeatsCount; // <-- Количество свободных мест в самолете

  public const int MaxSeats = 300; // <-- Константа указывающая на максимальное количество мест в самолете

  public int SeatsCount => _seatsCount; // <-- Свойство для доступа к приватному полю только на чтение

  public int FreeSeatsCount => _freeSeatsCount; // <-- Свойство для доступа к приватному полю только на чтение

  public Plane(int seatCount) // <-- Создание экземпляра класса самолет с указанием количесива мест
  {
    if(seatCount < 1 || seatCount > MaxSeats) // <-- Выполняем проверку допустимых значений
    {
      throw new ArgumentException("Seats count should be greater than 0 and less than max seats", nameof(seatCount));
    }

    _seatsCount = seatCount; // <-- Устанавливаем общее количество мест
    _freeSeatsCount = seatCount; // <-- При создании мамолета все места свободны
  }

  public void TakeSeat() // <-- Метод занятия места
  { 
    if(_freeSeats == 0) // <-- Проверяем количество свободных мест
    {
      throw new InvalidOperationException("No free seats left");
    }

    _freeSeats--; // <-- Уменьшаем количество свободных мест
  }
}
```

Теперь давай напишем тесты на конструктор и метод `TakeSeat` с использованием паттерна ААА:
```csharp
public class PlaneTests
{
  [Fact]
  public void Ctor_valid_seat_count_creates_instance_with_correct_free_and_total_seats_count() // <-- Тест позитивного сценария конструктора
  {
      // Arrange
      var seats = 250;

      // Act
      var plane = new Plane(seats);

      // Assert
      Assert.Equal(seats, plane.FreeSeatsCount); // <-- проверяем корректность выполнения логики конструктора
      Assert.Equal(seats, plane.SeatsCount); // <-- проверяем корректность выполнения логики конструктора
  }

  [Fact]
  public void Ctor_negaive_seat_count_throws_AE() // <-- Тест негативного сценария конструктора
  {
      // Arrange
      var seats = -1;

      // Act & Assert <-- Так как мы проверяем выброс исключеня стадии Act и Assert сливаются
      Assert.Throws<ArgumentException>(() => new Plane(seats)) // <-- корректность выполнения логики конструктора при невалидном значениее
  }

  [Fact]
  public void Ctor_seat_count_greater_than_max_count_throws_AE() // <-- Тест негативного сценария конструктора
  {
      // Arrange
      var seats = Plane.MaxSeats + 1;

      // Act & Assert <-- Так как мы проверяем выброс исключеня стадии Act и Assert сливаются
      Assert.Throws<ArgumentException>(() => new Plane(seats)) // <-- корректность выполнения логики конструктора при невалидном значениее
  }

  [Fact]
  public void TakeSeat_seats_available_reduces_free_seat_count_by_one() // <-- Тест позитивного сценария метода TakeSeat
  {
      // Arrange
      var seats = 200;
      var plane = new Plane(seats);

      // Act 
      plane.TakeSeat();

      // Assert
      Assert.Equal(seats - 1, plane.FreeSeats);
  }

  [Fact]
  public void TakeSeat_seats_not_available_throws_IOE() // <-- Тест негативного сценария метода TakeSeat
  {
      // Arrange
      var seats = 1;
      var plane = new Plane(seats);
      plane.TakeSeat(); // <-- Снижаем количество свободных мест до 0

      // Созданием самолета и вызовом метода TakeSeat мы привели систему в тестируемое состояние, когда количество свободных мест равно нулю

      // Act && Assert
      Assert.Throws<InvalidOperationException>(() => plane.TakeSeat()); // <-- Выполняем негативный сценарий
  }
}
```

Данные тесты покрывают 1 позитивный + 1 негативный сценарий конструктора и 1 позитивный + 2 негативных сценария метода `TakeSeat`. 

Обрати внимание на наименование тестов:
- Название класса `PlaneTests` говорит о том, что тесты будут преимущественно тестировать класс `Plane`
- Название тестового метода следует следующему паттерну: Название тестируемого метода - условия начала теста - ожидаемый результат.

Например, метод `Ctor_valid_seat_count_creates_instance_with_correct_free_and_total_seats_count`:
- `Ctor` говорит о том, что тестируется конструктор класса `Plane` (об этом говорит название класса `PlaneTests`)
- `valid_seat_count` говорит о том, что в аргумент передается валидное значение seatCount
- `creates_instance_with_correct_free_and_total_seats_count` говорит о том, что на выходе ожидается экземпляр класса с корректным значением свободных и общих мест

`Ctor_valid_seat_count_creates_instance_with_correct_free_and_total_seats_count` выглядит громоздко можно попытаться сократить название метода, если это не повлечет потерю смысла. Например, `Ctor_valid_seat_count_creates_instance_with_correct_seats_count`

Дополнительно рассмотрим метод `TakeSeat_seats_available_reduces_free_seat_count_by_one`:
- `TakeSeat` говорит нам о том, что тестируется метод `TakeSeat` класса `Plane` (об этом говорит название класса `PlaneTests`)
- `seats_available` говорит о том, что в начале тестового сценария в самолете есть свободные места
- `reduces_free_seat_count_by_one` говорит о том, что в результате выполнения теста количество свободных мест должно уменьшиться на 1

Теперь ты знаком с общими принципами написания тестов и их наименованием. Сначала тебе будет сложно выбирать наименования, но со временем ты освоишься.

## Материалы для изучения

Владимир Хориков: Принципы unit-тестирования - Главы 1, 2, 3

