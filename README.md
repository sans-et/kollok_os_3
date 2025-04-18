### Часть 1: Паттерны проектирования поведения

#### 1. Паттерн "Команда" (Command)

**Суть паттерна:**  
Команда — это объектно-ориентированная реализация вызова функции. Паттерн инкапсулирует запрос в виде объекта, позволяя параметризовать клиенты различными запросами, ставить их в очередь или поддерживать отмену операций.

**Практические примеры:**  
1. **UI и Undo/Redo:**  
   - *Проблема:* В графическом редакторе нужно реализовать отмену последнего действия.  
   - *Решение:* Каждая операция (рисование линии, изменение цвета) представляется объектом-командой с методами `execute()` и `undo()`. История команд хранится в стеке.

2. **Управление устройствами:**  
   - *Проблема:* Пультам дистанционного управления нужно контролировать разные приборы (телевизор, кондиционер).  
   - *Решение:* Для каждого действия (включить/выключить) создается отдельная команда, а пульт работает с абстрактным интерфейсом Command.

3. **Транзакции:**  
   - *Проблема:* В банковской системе необходимо обеспечить атомарность операций.  
   - *Решение:* Перевод средств оформляется как команда, которая либо выполняется полностью, либо откатывается при ошибке.

**Принципы проектирования:**  
- *Инкапсуляция:* Детали выполнения операции скрыты внутри команды.  
- *Разделение ответственности:* Инициатор команды (Invoker) не знает, как выполняется операция.  
- *Гибкость:* Новые команды добавляются без изменения существующего кода.

**Многопоточность:**  
Команды удобны для параллельного выполнения, но требуют:  
- Синхронизации при доступе к общим ресурсам  
- Механизмов для гарантии порядка выполнения (например, очереди задач)

---

#### 2. Паттерн "Стратегия" (Strategy)

**Основная идея:**  
Определяет семейство алгоритмов, инкапсулирует каждый из них и делает их взаимозаменяемыми. Позволяет модифицировать поведение системы во время выполнения.

**Примеры из практики:**  
1. **Платежные системы:**  
   - *Проблема:* Поддержка разных способов оплаты (карта, PayPal, криптовалюта).  
   - *Решение:* Каждый платежный метод реализует интерфейс PaymentStrategy с методом `pay()`.

2. **Навигация:**  
   - *Проблема:* Построение маршрутов для разных видов транспорта (авто, пешком, общественный транспорт).  
   - *Решение:* Алгоритмы маршрутизации инкапсулированы в классах CarRouteStrategy, WalkingStrategy и т.д.

3. **Сортировка данных:**  
   - *Проблема:* Выбор оптимального алгоритма сортировки в зависимости от типа данных.  
   - *Решение:* Реализации QuickSort, MergeSort и BubbleSort подчиняются общему интерфейсу SortStrategy.

**Ключевые аспекты:**  
- *Изоляция:* Каждая стратегия независима и может тестироваться отдельно.  
- *Динамическое поведение:* Алгоритмы можно переключать "на лету".  
- *Расширяемость:* Добавление новой стратегии не требует изменения контекста.

**Параллелизм:**  
- Стратегии без состояния (stateless) безопасны для многопоточной среды  
- Для stateful-стратегий требуется синхронизация доступа к данным

---

#### 3. Паттерн "Шаблонный метод" (Template Method)

**Суть подхода:**  
Задает "скелет" алгоритма в базовом классе, позволяя подклассам переопределять отдельные шаги без изменения структуры алгоритма.

**Реальные кейсы:**  
1. **Генерация документов:**  
   - *Проблема:* Единый процесс создания отчетов с разными форматами вывода (HTML, PDF).  
   - *Решение:* Базовый класс определяет шаги (сбор данных → форматирование → экспорт), а подклассы реализуют специфичные этапы.

2. **Игровые уровни:**  
   - *Проблема:* Стандартная последовательность инициации уровня с разными настройками сложности.  
   - *Решение:* Родительский класс Level задает структуру (init() → spawnEnemies() → setRewards()), дочерние классы уточняют детали.

3. **Тестирование:**  
   - *Проблема:* Унификация процесса тестирования для разных модулей.  
   - *Решение:* Абстрактный TestCase определяет шаблон (setup() → runTest() → tearDown()).

**Принципы:**  
- *Контроль структуры:* Базовый класс диктует порядок операций  
- *Гибкость реализации:* Подклассы могут варьировать конкретные шаги  
- *Минимизация дублирования:* Общий код вынесен в родительский класс

**Многопоточные аспекты:**  
- Шаблонные методы могут выполняться параллельно  
- Критические секции требуют аккуратного проектирования hook-методов  
- Идеально подходит для реализации "рабочих потоков" (worker threads)

---

### Часть 2: Основы архитектуры и параллелизм

**Архитектура ПО — кратко:**  
Архитектурный дизайн — это фундаментальная организация системы, воплощенная в ее компонентах, их отношениях между собой и окружением, а также принципах, определяющих проектирование и эволюцию системы.

**Ключевые характеристики:**  
- Компонентная декомпозиция  
- Четкие контракты взаимодействия  
- Учет нефункциональных требований (производительность, надежность)

**Многопоточность в архитектуре:**  
Параллельные вычисления влияют на архитектурные решения через:  
1. **Модели исполнения:**  
   - Пул потоков vs асинхронный ввод-вывод  
   - Реактивные паттерны (Event Loop)

2. **Проблемы синхронизации:**  
   - Блокировки vs безблокирующие алгоритмы  
   - Управление состоянием (shared vs isolated)

3. **Распределенные системы:**  
   - Очереди сообщений (Kafka, RabbitMQ)  
   - Шардинг данных для минимизации конфликтов

**Пример архитектурного решения:**  
Микросервисная архитектура с:  
- Stateless-сервисами для горизонтального масштабирования  
- Event Sourcing для согласованности данных  
- Circuit Breaker для устойчивости  

---

### Итог  
В работе рассмотрены три поведенческих паттерна с анализом их структуры, примеров применения и особенностей работы в многопоточной среде. Отдельно разобрано влияние параллелизма на архитектурные решения, включая типичные проблемы и подходы к их решению.
