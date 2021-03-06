# Testwork **Fanbox**

## Задание:
> <https://dl.fun-box.ru/qt-ruby.pdf> 

## Level 1

1. Если лямбда вызывается с большим, или меньшим количеством аргументов, чем необходимо, тогда `Ruby` выдает ошибку `ArgumentError`.
Когда лямбда-функция возвращает значение и эта лямбда вызывается внутри метода, то метод будет просто продолжать свое выполнение после окончания работы лямбды, 
пока в конце-концов он сам не вернет свое собственное значение. 
Однако, когда Proc возвращает значение и эта процедура вызывается внутри метода, тогда выполнение этого метода будет остановлено.

2. Операция имеет больший приоритет `&&` чем `and`.

3. (
### Достоинтства:
* Относится к языкам программирования сверхвысокого уровня `VHLL`, то есть обладает высоким уровнем абстракции и предметным подходом в реализации алгоритмов.
* Реализует концептуально чистую объектно-ориентированную парадигму (Все операции выделины в классы).
* Большой набор готовых решений `gem`.
* Возможности языка можно расширить при помощи библиотек, написанных на `C` или `Ruby`.
* Удобный синтаксис.
* Метапрограммирование.
### Недостатки:
* Проигрывает по производительности с `Node.js`, `Java`, `Go`, `Elexir`.
* До выхода `Ruby` 2.0 `private` и `protected` вообще ничем не отличались.
* Отсутствие абстрактных классов.
* Часто самым простым способом выполнить одновременно две вещи является использование потоков в `Ruby`. 
  Они являются внутрипроцессными, встроенными в интерпретатор `Ruby`. 
  Это делает потоки `Ruby` полностью переносимыми, т.е. независимыми от операционной системы. 
  Но в то же время вы точно не получите выгоду от использования родных, нативных потоков.

4. `Ruby on Rails` хорошо подходит для быстро растущих проектов.
   Поэтому часто используется в стартапах.
   Успех этому фреймворку обеспечила стандартизация, превалирование соглашений
   над конфигурацией. Принцип `DRY`. И возможность как создавать очень быстро
   новые части приложения (большой набор генераторов) так и тестировать их. 
   Что в будущем экономит много времени используя `MVC` архетектуру.
   Вцелом фремворк хорошо подходит для `CRUD` приложений.
   Этому способствует: `gem`s, поддержка большинства `DB` через `Active
   Records`  и прочие `ORM`. 
   Включен новый режим `Api`. 
  `Api` режим нужен нетолько для построения `Api`, но и созданию модных `SPA` приложений.
   Где `Rails` выступает как источник данных, для фронт части.
   ### Альтернативы
   * Традиционно для создания `Api` использовались микро фреймворки `Grape`, `Sinatra`.
   * Для создания группы зависимых проектов (экосистемы) используется `Hanami`(`Lotus`)
   * `Trailblaizer` выделяет логику в операции и создает `cell` как альтернатива `helpers` которые не находятся в
     глобальной области видимости. В целом интересный подход но не зрелый, как мне
     кажется. 

5. Проверить доступную память. 
   Вести лог, чтобы выявить замыкания.
   Запустить скрипт с максимально возможными предупреждениями -W[level=2]
   Запустить трассировку и включить предупреждения по максимому```ruby  ruby -w -W2 -r tracer xxx.rb ```
   Попробовать начать трасcировать его с `byebug`.

6. Потери памяти можно выделить на пиковые и растущие.
   Пиковые как правило более выражены и являются недостатками алгоритма и применяемых методов.
   К ним можно отнести чтение больших файлов в память, десерилизация и серелизация больших обьемов данных.

   Растущие же потери данных могут указывать на 2 вещи:
   
* Ошибки в С библиотеках, что реже
  Вызваны работой с указателями, необходимость С работать с памятью.

* Зависание обьектов в памяти из-за сохранившейся сылки на него и невозможностью очистки его `Garbage
  Collector`ом.  Oбъекты хранятся в так называемой куче и пока есть сылки на обьект Garbage
  Collector не может его уничтожить и освободить память.

  Чтобы понять с какой проблемой мы столкнулись необходимо собрать статистику и построить график.
  Так-же надо учитывать, что сбор статистики замедлет работу системы.

  `Graphite` , `statsd` и `Grafana` для вычисления метрик приложения.
  Показателем будет `RSS`

  Иснтрументы пригодные для поска проблем с памятью:
* `monit`
* `inspector` 
* `unicorn worker killer`
* `valgrind` – для поиска утечек в самом ruby и экстеншнах.
* `bleak_house` – инструмент для отслеживания аллокейшнов ruby-объектов. Подходит для `Rails`.
  `Ruby 2.1` предоставляет сброс кучи.
* `get_process_mem` — удобный способ получить информацию о памяти, используемой текущим `Ruby-процессом.
* `memory_profiler` В качестве профилировщика можно использовать также `ruby-prof`
* `derailed_benchmarks` для мониторинга имеющий некоторые замечательные возможности, характерные для `Rails`.

  После выявления проблемных мест, приступаем к лечению.  
  Решение проблемы:
* Начиная с `Ruby 2.0` появилась замечательная возможность делать ленивые перечислители (lazy enumerator). 
  Это позволяет сильно уменьшить потребление памяти при вызове цепочки методов перечислителя. 
* Читать файлы построчно
* Использовать потоковые серилизаторы и десерелизаторы
* Неиспользовать больших массивов.
* Удалять сылки на обьект после его использования. 
  К примеру массовая генерация Символов может серьезно отразится на памяти, так как они будут в ней висеть. 

7. Стандартное использование `JRuby` — это его встраивание в `Java`-приложение для
   поддержки скриптинга и ускорения разработки, что является преимуществом языка
   Ruby перед статическими языками. Также может использоваться для запуска
   `Rails`-приложений на `Java`-платформах.
   `JVM` оптимизируется под поступающие запросы и после прогрева работает быстро.
   `JRuby` таким образом может использовать плюсы `JVM`.
   Однако подобные гибриды имеют небольшие несовместимости, которые надо
   учитывать.

8. `Active Record` достоинства и недостатки(

### `AR` Достоинства:

* Первое. Объектно‐ориентированный подход.
При использовании `Active Record` не нужно вносить в бизнес‐логику куски `SQL` запросов, с `Active Record` можно оперировать только объектами.

* Реализации `Active Record` могут работать через различные драйверы для доступа к `DB`, а также генерировать специфичные для конкретной `DB` запросы, 
  вынося все это с уровня бизнес‐логики. И в данном случае, программисту необходимо только описать что он хочет, а не описывать то, как он хочет.

* Валидация, реализуется механизмы для валидации значений полей. 

* Сохранение происходит в одном месте, что позволяет легко изучить, как это работает.

* `Active Record` поддерживают связи

* Писать код с `Active Record` получается быстро и легко, в том случае, когда свойства объекта прямо соотносятся с колонками в базе данных.

* Active Record дает полный стек средств для работы с базой, что ускоряет
  разработку.

### `AR` Недостатки:

* Модели `Active Record` нарушаю принципы `SOLID`. 
В частности, принцип единой ответственности (`SRP` — «S» в принципах `SOLID`).
Согласно принципу, доменный объект должен иметь только одну зону ответственности, 
то есть только свою бизнес-логику. Вызывая его для сохранения данных, вы добавляете ему дополнительную зону ответственности, 
увеличивая сложность объекта, что усложняет его поддержку.

* `Active Record` требует значительного количества кода, который затем приводит к большому объему памяти, 
используемому для разворачивания экземпляра. Cоздав достаточное количество экземпляров `Active Record`,
которые не собираются сборщиком мусора достаточно быстро, то доступная память может быть полностью использована.

* Экземпляры `Active Record` делают много: валидация, генерация запросов, логика домена и многое другое. 
Да, это упрощает и, возможно, делает код более читаемым, но добавляет сложность, по крайней мере, в фоновом режиме.
Если когда-либо столкнутся с новой глубокой ошибкой `Active Record`, то придется потратить много времени.

* `Active Record` не эффективно генерирует `Sql`.
Это не вина `Active Record`, у `ORM` свои ограничения. 
Для простых запросов он работает нормально.
Хотя по мере того, как запросы становятся более сложными, тем хуже, что `SQL` будет генерировать `Active Record`. 
Просто `ORM` не может создать оптимизированный сложный запрос лучше, чем опытный инженер.

* Чтобы запустить тесты `rspec`, мне придется загружать `Rails`.
Тестировать `Active Record` сложнее из-за нескольких зон ответственности.

### `OHM` Достоинство:
Использует hash и списки `Redis`, для эмуляции реляцционых таблиц и их связей.
Используя преимущества производительности `Redis`, получаем довольно интересную
штуку.

### `OHM` Недостатки:
* `Redis` подходит не для всех данных.
* Недостаточная поддержка библеотеки, имеются баги, которые устраняются.
* Привязка к конкретной `DB`

### `DataMapper` Достоинство:
* `ORM`, который является быстрой, потокобезопасной и многофункциональной.
* `DataMapper` — это программная прослойка, разделяющая объект и `DB`. 
  Его обязанность — пересылать данные между ними и изолировать их друг от друга. 
  При использовании `DataMapper`'а объекты не нуждаются в знании о существовании `DB`. 
  Они не нуждаются в `SQL`-коде, и (естественно) в информации о структуре `DB`. 
  Так как `DataMapper` - это разновидность паттерна `Mapper`, сам объект-`Mapper` неизвестен объекту.

### `DataMapper` Недостатки:
* Разработка этой библеотеки кажется остановилась, повлияла на создание `ROM`
  проекта.

### `ROM` Достоинство:
* Изолировать приложение от сохранения данных.
* Предоставить минимум инфраструктуры для отображения и сохранения.
* Предоставление общих абстракций для компонентов нижнего уровня.
* Обеспечить возможность использовать мощных возможностей `DB`

### `ROM` Недостатки
* Небольшое сообщество которое его поддерживает.

### `Sequel` Достоинство:
* `Sequel` обеспечивает безопасность потоков, объединение пулов и краткую `DSL` для построения `SQL`-запросов и схем таблиц.
* `Sequel` включает в себя комплексный уровень ORM для сопоставления записей с объектами Ruby и обработки связанных записей.
* `Sequel` поддерживает расширенные функции базы данных, такие как подготовленные операторы, связанные переменные,
  точки сохранения, двухфазное комментирование, изоляция транзакций, конфигурации главного / подчиненного устройства 
  и разбиение базы данных.
* В `Sequel` в настоящее время есть адаптеры для `ADO`, `Amalgalite`, `IBM_DB`, `JDBC`, `MySQL`, `Mysql2`, `ODBC`, `Oracle`, `PostgreSQL`, `SQLAnywhere`, `SQLite3` и `TinyTDS`.
* `Hanami model` и `Rom` используют его.
### Sequel Недостатки
* Недостаточная поддержка в `Rails` сообществе.

### Известные мне `ORM`
`OHM`, `DataMapper`, `ROM`, `Sequel`

### Выводы:
Большая разница между стилем `Active Record` и стилем `Data Mapper` заключается в том, 
что стиль `Data Mapper` полностью отделяет ваш домен от уровня сохранения. 
Это означает, что ни один из объектов вашей модели ничего не знает о базе данных.
Если есть возможность проектировать базу данных в соответствии с конвенциями `Rails` - используя `Active Record`. Я сэкономлю много сил и средств.
Если я строю проект над готовой базой или если структура проектируемой DB слишком сложна - использую `Sequel`, 
делая преобразователи данных, строя абстрактную, отвязанную от `DB` объектную модель.

## Level 2

**.1 (**

  Есть 3 пути хранения изображения.

  1. Файловая система
  2. `DB`
  3. Удаленный Storage сервер, например `Amazon S3`.

  Я не буду рассматривать готовое решение, поэтому 3 вариант я не беру.

### Файловая система

  Достоинства:

  * Отлично подходит для небольших и средних проектов.
  Файловая система может отлично раздавать статику `nginx` или `lighttpd`

  * Можем загружать файл через nginx.

  Недостатки:

  * С появлением большого числа файлов появляются проблемы.
  * В некоторых файловых системах мы решаем сложности разбивая на под каталоги.
    Однако. Такой вариант правильный только в устаревших FS типа `FAT` & `NTFS`.
    Время доступа к произвольному файлу в `EXT` пропорционально O(log(N)), и если впихнуть туда каталоги, то будет только хуже и сложнее.
    Там и так дерево, и надстройка, дерево над деревом будет только тормозить.
  * В директориях с большим количеством файлов плохо выполняются операции `ls`, `rm -rf *`
  * Метоинформация хранится в `DB`, а файл в файловой системе. 2 состояния. Сложнее выявить ошибку если файл удалили, а запись существует.
  * Делать бекапы сложнее.

### `DB`

  Достоинства:

  * Данные и файл хранятся в 1 месте, что в случие потери легче заметить.
    И востнановить.
  * Можно делать Шардинг.
  * Легче делать бекапы.
  * Поиск требуемого файла, быстрее.
  * Возможность шифрования файлов
  * Безопасность, если файл был заражен, то его можно выполнить.
    В `DB` же будет лежать мертвым грузом.
    Хотя и в файловой системе его можно сделать не исполняемым.
    И для клиентов угрозы не снимет.

  Недостатки:

  * Nginx лучше работает со статикой.

### Вывод

  Так-ка большинство проблем файловой системы связаны с количеством файлов. То на средних системах выбор очевиден.
  Однако на большом количестве, всеже выбираю `DB`.
  Выбор - `Gridfs в mongodb`

### Библиотека

  `Сarrierwave` - хорошая библиотека с большим сообществом.
  Конкуренты: `Paperclip`, `Dragonfly`, `Refile`
  <https://github.com/carrierwaveuploader/carrierwave-mongoid>
  Можем проверить на вирус новый файл.

### Аватарки

  Аватарки мы ужимаем до требуемого размера(имеется в виду размер рамки)
  Можем также вобще доверится стороннему сервису Gavatar.

### Проверки на nginx

  Самые очевидные - это размер и тип файла.
  Тем самым снимем нагрузку на сервер.

**2. (**

1. Использовать конфигурацию `RSpec` 3 и `Rails`

   Помимо всех его замечательных функций, мы любим `RSpec 3`, потому что,  вы не нуждаетесь rails_helper.rb в каждом файле.
   Каждая спецификация может включать в себя spec_helper.rb или rails_helper.rb, что резко влияет на время выполнения.
   При тестировании `API` (а также моделей и т. д.)
   Вам не нужно загружать всю структуру `Rails`.
   Это приводит к сокращению времени выполнения. Так просто экономить время.

2. Иметь меньше зависимостей

   Драгоценные камни загружаются заранее.
   Обычно не требуется большая их часть (хотя иногда это очень полезно, особенно если вам нужно использовать дополнительные классы).
   Самый простой способ ограничить количество загрузочных камней - это группы.
   Назначение групп состоит в том, что они позволяет вам указывать группы зависимостей, которые будут использоваться в определенных средах, но не в других.
   Разделить Gemfile драгоценные камни на группы (например, разработка, тестирование, производство и т. Д.).
   И вынести в тестирование только необходимые.

3. `Mock` и `Stub`

   `Stubbing` and `Mocking` - отличные возможности, которые мы используем при написании спецификаций.
   Если не нужен метод выполнения, который занимает много времени (например, запрос `API` или что-то еще), мы можем временно установить возвращаемое значение.
   Используя методы эмулирования объектов, вы позволяете им возвращать заданное вами значение.
   В этом случае методы не вызываются, и значение доступно для запроса.
   Главное не переборщить.

4. Настройка без `DB`

   Для многих операций нам не требуется соединение с `DB`.
   Операции `DB` являются одним из самых трудоемких в тестах и в большинстве случаев совершенно не нужны.
   Фактически, за исключением некоторых кейсов, только спецификации контроллеров должны сохранять некоторые объекты в `DB`.
   Также `Data Mapper` протестировать быстрее чем `Active Records`.

5. Использование `factories` и `fixtures`

   Чтобы ускорить набор тестов.
   Уменьшим количество объектов, сохраненных в базе данных.
   С `Factory Girl` и `Faker`

6. Подбор инструментария

   Если нам не нужен большой обьем тестовых возможностей `Rspec`.
   Мы можем воспользоватся `MiniTests`.

7. Использовать новые версии `Ruby`

   `Ruby` работает над повышением производительности.
   Версия 2.1 представила хорошую производительность по сравнению с 2.0, когда дело доходит до сборщика мусора.
   Очевидно, что это сильно влияет на время запуска.
   На основе `isrubyfastyet` `Ruby` 2.1 сохраняет память вашего оборудования, и запросы выполняются быстрее.
   Время загрузки также улучшилось. Это, опять же, сэкономит миллисекунды.
   К примеру сейчас доступен `Ruby` 2.4.3

8. Использование `Spring, `Guard`

   `Spring` добавляется в ваш `Gemfile` по умолчанию из `Rails` 4.1, но можно добавить его также вручную.
   Каждый раз, когда запускаеш рейк-задачи, все приложение перезагружается.
   `Spring` позволяет вам перезагрузить только те классы, которые действительно необходимо перезагрузить.
   Он автоматически обнаруживает изменения в коде.
   `Guard` работает в фоновом режиме, запуская спецификации  на основе файла, который только что изменился.
   Как только вы отредактируете свою модель, `Guard` будет запускать спецификации для этой модели.
   Можно настроить его для запуска всех спецификаций в рамках проекта.
   Это экономит время при работе на локальном компьютере: не нужно постоянно переключаться между консолью и редактором.

9. Параллельное тестировании

   Выполнение теста может занять много времени.
   Наряду с причинами, о которых говорилось выше, есть вероятность, что не используется весь потенциал, который находится на вашей машине.
   По умолчанию все спецификации работают в одном процессе, который неэффективен.
   Можно попытаться ускорить это, добавив `parallel_tests` gem.
   Он настраивает тестовую среду, запуская тест в нескольких потоках или процессорах.
   Он позволяет разделить тесты на группы и работать в одном процессе со своей собственной базой данных.
   Это обычно используется с инструментами непрерывной интеграции, такими как `CircleCI`, но также можно использовать его на своей локальной машине.
   Это ускоряет ваши тесты при наличие мощьного компьютера.

10. Интеграционные тесты с `Capibara`

    Они медленные, но ингода необходимы, их можно ускорить используя вместо `Selenium`, использовать  `Poltergeist` на `PhantomJS`.
    Так-же надо учитывать, что некоторые операции `Capibara` используют таймауты, которые серьезно замедлят тесты.
    Некоторую помощь в этом окажет камень `capybara-slow_finder_errors`.o

## Level 3

### Q1

    https://github.com/Rattt/find_max

### Q2

    https://github.com/Rattt/q2

---
C уважением Мороз И.Н. 
