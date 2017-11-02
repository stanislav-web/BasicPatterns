# Приемы объектно-ориентированного программирования. Шаблоны проектирования (далее Паттерны)

## Обзор и обсуждение часто используемых паттернов, таких как  Factory Method, Abstract Factory, DI

**`Замечание: учащиеся должны быть ознакомлены с основами ООП и UML`**

### 1.0 Предисловие
Все чаще и чаще слышно от разработчиков и в статьях, что шаблоны проектирования никому не нужны. 
Мол, они появились во времена «цветения» UML, RUP, CASE систем и прочих чересчур «сложных» инструментов, подходов и практик. 
А сейчас самое важное — это код рабочий написать, да побыстрее. 
На умные толстые книжки ни у кого нет времени, разве что для прохождения собеседования.
### 1.1 Зачем нужны паттерны?
Паттерны — это один из инструментов разработчика, который помогает ему **сэкономить время** и сделать более **качественное решение**. 
Как и любой другой инструмент, в одних руках он может принести много пользы, а в других — один только вред. 
Паттерны — это не законченное архитектурное решение, которое можно напрямую преобразовать в исходный или машинный код. 
Это описание подхода к решению проблемы, который можно применять в разных ситуациях.
На данном этапе мы рассмотрим базовые реализации таких паттернов как Factory Method, Abstract Factory и Dependency Injection.
### 2.1 Factory Method (Фабричный метод) он же Виртуальный Конструктор.
**Фабричный метод** — это *"порождающий паттерн"* проектирования, который определяет общий интерфейс для создания объектов в суперклассе (на схеме ниже "Product"), 
позволяя своим подклассам использовать общий для реализации интерфейс. Фабричный метод инкапсулирует логику создания объекта.
Ключевой особенностью этого паттерна является то, что это просто **"метод"**. 
#### Статический Фабричный метод
![StaticFactoryMethod](/images/static-factory.jpg)

*стандартная диаграмма реализации. Обобщенный конструктор - статическая фабрика*
##### Рассмотрим на примере
Допустим, мы занимаемся разработкой проекта "DHL Express" и у нас стала задача использовать новый вид доставки грузов, например по морю. 
Как мы можем это реализовать?
```php
   <?php
   /**
    * Class Truck 
    * Грузовики
    */
   class Trucks {
        public function deliver() {
                return 'Delivered by Truck';
        }       
   }
   
   /**
    * Class Ship
    * Корабли
    */   
   class Ship {
        public function deliver() {
            return 'Delivered by Truck';
        }       
   }
   
   $trucks = (new Trucks())->deliver();
   $ship = (new Ship())->deliver();
   
```
На первый взгляд покажется, что все понятно. Мы сделали наш транспорт и доставили груз..
Но, как же наша компания будет контролировать свой транспорт?
В серьезной компании транспорт должен быть под контролем.
Давайте ниже рассмотрим пример реализации "Фабричным методом" и то, как он может помочь нам с решением этой проблемы:

```php
<?php
/**
 * Class LogisticCompany
 * Класс реализующий в себе фабричный метод (абстракция не обязательна)
 * Поведение транспорта будет отличаться своей реализацией в подклассах (Truck, Ship) 
 * но интерфейс у них будет общий LogisticFactory
 */
abstract class LogisticCompany
{
    private static $loaded = [];

    /**
     * Фабричный метод
     */
    public static function addTransport(string $transportType) : LogisticCompany {

        if(true === isset(self::$loaded[$transportType])) {
            throw new \Exception($transportType.' уже загружен и ждет отправки!');
        }

        $transport = new $transportType;
        self::$loaded[$transportType] = $transport;

        return $transport;
    }

    abstract public function delivery();
}

/**
 * Class Truck
 * Грузовик
 * Доставка грузовиком будет отличаться от корабля, но над ними будет общий контроль компании
 */
class Truck extends LogisticCompany {
    public function delivery() {
        return 'Delivery by Truck';
    }
}

/**
 * Class Ship
 * Корабль
 * Доставка кораблем будет отличаться от грузовика, но над ними будет общий контроль компании
 */
class Ship extends LogisticCompany {
    public function delivery() {
        return 'Delivery by Ship';
    }
}

$truck = LogisticCompany::addTransport('Truck');
$truck->delivery();
$ship = LogisticCompany::addTransport('Ship');
$ship->delivery();
```

В примере выше, статический метод addTransport регистрирует наш транспорт в компании 
и делает проверку, на предмет загрузки. И теперь компания знает, какой транспорт будет отправлен.
![FactoryMethod](/images/factorymethod.png)
#### Классическая фабрика
![FactoryMethodClassic](/images/classic-factory.jpg)

*стандартная диаграмма реализации. Классическая фабрика*
Тот же принцип : определяет общий интерфейс для создания объектов в суперклассе, 
но теперь в классическом стиле

```php
<?php 

/**
 * Interface IDelivery
 * Описываем общее поведение нашего транспорта (интерфейс обязателен)
 * Поведение будут отличаться реализацией, но интерфейс у них будет общий
 */
interface IDelivery {
    public function delivery();
}

/**
 * Class Truck
 * Грузовик
 * Доставка кораблем будет отличаться от грузовика, но над ними будет общий контроль компании
 */
class Truck implements IDelivery {
    public function delivery() {
        return 'Delivery by Truck';
    }
}

/**
 * Class Ship
 * Корабль
 * Доставка кораблем будет отличаться от грузовика, но над ними будет общий контроль компании
 */
class Ship implements IDelivery {
    public function delivery() {
        return 'Delivery by Ship';
    }
}

/**
 * Class LogisticCompanyFactory
 * Класс делегирующий интерфейс с фабричным методом
 */
abstract class LogisticCompanyFactory {
    abstract protected function addTransport(string $transportType) : IDelivery;
}

/**
 * Class LogisticCompany
 * Класс реализующий в себе фабричный метод LogisticCompanyFactory
 * Поведение транспорта будет отличаться своей реализацией в подклассах (Truck, Ship) 
 * но интерфейс у них будет общий IDelivery
 * Фабрика взаимодействует не с конкретными объектами, а их интерфейсом
 */
class LogisticCompany extends LogisticCompanyFactory {
    
    private $loaded = [];
    
    public function addTransport(string $transportType) : IDelivery {

        if(true === isset($this->loaded[$transportType])) {
            throw new \Exception($transportType.' уже загружен и ждет отправки!');
        }

        $transport = new $transportType;
        $this->loaded[$transportType] = $transport;

        return $transport;
    }
}

$logisticCompany = new LogisticCompany();
$truck = $logisticCompany->addTransport('Truck');
$truck->delivery();
$ship = $logisticCompany->addTransport('Ship');
$ship->delivery();
```
Как видите , фабричный метод оперирует внешним интерфейсом IDelivery в отличии от статического,
но поведение обоих подходов одинаковое.
Фабричный метод является как бы основой для "Абстрактной Фабрики", "Cтроителя" и "Прототипа". 
В разработке часто именно так и получается, сперва реализуют фабричный метод, а по мере усложнения кода выбирают во что именно его преобразовать. 
### 3.1 Abstract Factory (Абстрактная фабрика).
**Абстрактная фабрика** — это также *"порождающий паттерн"* проектирования, предоставляет интерфейс для создания семейств взаимосвязанных или взаимозависимых объектов, не специфицируя их конкретных классов. 
Это фабрика фабрик. То есть фабрика, группирующая индивидуальные, но взаимосвязанные/взаимозависимые фабрики без указания для них конкретных классов.
![AbstractFactory](/images/abstract-factory.jpg)

*стандартная диаграмма реализации. На схеме Client - это наше приложение, которое реализует данный паттерн*
##### Рассмотрим на примере
Вернёмся к нашей компании "DHL Express". Вы уже реализовали новый вид доставки, и гарантировали компании, что ваша программа сможет контроллировать транспорт.
Вы также легко можете расширить программу добавив в него новый способ доставки.. Но тут случилась ситуация,
что вот транспорт не вечный, и его постоянно нужно ремонтировать , заказывать новые детали, налаживать диалоги с различными поставщиками запчастей.
Компания предлагает вам что то придумать с этим, ведь если траспорт неожиданно выйдет из строя, мы не успеем вовремя доставить его обратно в строй.

**Решение**: Вам известно уже , что разработкой определенного вида запчастей занимаются различные фабрики (в прямом смысле)
и нет смысла в случае поломки, заказывать новый транспорт. Поэтому вам нужно настроить диалоги с поставщиками.
В вашем интерфейсе с реализацией может помочь Абстрактная Фабрика:
```php
<?php
### Опрелеляем для начала, что необходимо нашему транспорту в случае поломки

/**
 * Interface ITransmission
 * Определяем интерфейсом, что нам нужны будут элементы трансмиссии
 */
interface ITransmission
{
    public function releaseTransmission();
}

/**
 * Interface IEngine
 * Определяем интерфейсом, что нам также нужны будут двигатели
 */
interface IEngine
{
    public function releaseEngine();
}


### Когда мы уже знаем в общем плане что нам нужно для ремонта, мы готовы составить список запчастей для конеретного транспорта

/**
 * class TruckTransmission
 * Реализовываем наш план по заказу трансмиссии для грузовиков
 */
class TruckTransmission implements ITransmission
{
    public function releaseTransmission() {
        return 'Released trucks transmission';
    }
}

/**
 * class ShipTransmission
 * Реализовываем наш план по заказу трансмиссии для кораблей
 */
class ShipTransmission implements ITransmission
{
    public function releaseTransmission() {
        return 'Released ships transmission';
    }
}

/**
 * class TruckEngine
 * Реализовываем наш план по заказу двигателя для грузовиков
 */
class TruckEngine implements IEngine
{
    public function releaseEngine() {
        return 'Released truck engine';
    }
}

/**
 * class ShipEngine
 * Реализовываем наш план по заказу двигателя для кораблей
 */
class ShipEngine implements IEngine
{
    public function releaseEngine() {
        return 'Released ships engine';
    }
}


### Далее создадим фабрики, которые будут отпускать нам запчасти,
### например, запчасти на авто мы можем заказать у VAG (Volkswagen Audi Group), а кораблями занимается SudMash

class VAGFactory {
    public function releaseTrucksEngine(string $type): IEngine {
        return new $type;
    }
    public function releaseTrucksTransmission(string $type): ITransmission {
        return new $type;
    }
}

class SudMashFactory {
    
    public function releaseShipsEngine(string $type): IEngine {
        return new $type;
    }
    public function releaseShipsTransmission(string $type): ITransmission {
        return new $type;
    }
}

### Как видно из примера, у этого паттерна очень много общего с Фабричным методом, 
### а точнее, он и является его не отъемлемой частью `release*Engine` и `release*Transmission` опираются на интерфейсы запчастей*
### Выглядит это так: у нас сломалась трансмиссия на грузовике, нас оповестил об этом менеджер компании, и нам нужно сделать заказа трансмиссии

$vagFactory = new VAGFactory();
$truckEngine = $vagFactory->releaseTrucksEngine('TruckEngine')->releaseEngine();

### Беда с коробкой передач
$truckTransmission = $vagFactory->releaseTrucksTransmission('TruckTransmission')->releaseTransmission();

### Корабль вышел из строя, нужны срочно запчасти! отправляемся на SudMash
$sudMashFactory = new SudMashFactory();
$shipTransmission = $sudMashFactory->releaseShipsTransmission('ShipTransmission')->releaseTransmission();
$shipEngine = $sudMashFactory->releaseShipsEngine('ShipEngine')->releaseEngine();
?>
```
Единственный минус, этого паттерна - это его сложность.
В данном случае сложно добавить поддержку нового вида запчастей для одной из фабрик.
Для этого необходимо:
1. создать интерфейс новой детали , например IGlass
2. имплементировать его для определенного вида транспорта, например TruckGlass
3. создать фабричный метод для реализации, например releaseTrucksGlass

На примере выше, это кажется очень простым, но реальные задачи отличаются от примеров, 
то как организовывать подход к реализации - зависит от Вас. 
Что касается примеров, то в действительности первым делом нам нужно наладить поставки запчастей, а уже потом организовывать доставку. 
Этот паттерн не является "серебряной пулей",н о его подход может повысить надежность вашего приложения. Например на стадии тестирования, вам достаточно будет протестировать вашу фабрику, чем отдельные классы реализации

```php
class Test {
    function testVagFactory() {
        $vagFactory = new VAGFactory();
        $truckEngine = $vagFactory->releaseTrucksEngine('TruckEngine');
        
        assertTrue('$truckEngine is Engine')
        assertTrue('$truckEngine->releaseEngine() is Released');
    }
}

```
### 4.1 Dependency Injection  -> DI  (Внедрение зависимости).
![DI](/images/di.png)

**Dependency Injection (внедрение зависимостей)** — это процесс предоставления внешней зависимости программному компоненту, который позволяют писать слабосвязный код, создавать слабосвязанную структуру.
Не путать с **type hints**, паттерн оперирует объектами и интерфейсами а не типами данных.
Ранее мы обсуждали Фабричный Метод и Абстрактную Фабрику...Так вот фабричный метод реализует это простой паттерн. 
Теперь давайте в этом методе создадим зависимость от интерфейса IDelivery, чтобы наша компания могла сделать осмотр траспорта который вот вот отправиться.
```php
class LogisticCompany extends LogisticCompanyFactory {
    
    # ....
        
    public function addTransport(IDelivery $transport) : IDelivery {
        # ... некоторая логика с объектом $transport (тех осмотр)
        
        return $transport;
    }
    
    # ....
}
```
В данном случае, метод принимает аргумент - любой объект который соответветствует интерфейсу IDelivery.
Таким образом мы внедрили **Dependency Injection** в нашу фабрику.
#### Типы внедрений
Есть три основных способа внедрить ваши зависимости в класс: 
##### через конструктор
```php
<?php 
class Alfa {}

class Beta {
    public function __construct(Alfa $alfa){/** .... */}
}
$beta = new Beta(new Alfa);
?>
```
##### через сеттер функцию
```php
<?php 
class Alfa {}

class Beta {
    public function __construct(){}
    public function setAlfa(Alfa $alfa) {/** .... */}
}
$beta = new Beta();
$beta->setAlfa(new Alfa());
?>
```
##### через свойство (не в PHP)
```javascript
class Alfa():

class Beta:
    alfa = Alfa()
    def function __init__(self):
        pass
    
    def get_alfa(self):
        return self.alfa    
}
beta = Beta();
beta.get_alfa()
?>
```
На базовых примерах можно усомниться, в полезности такого подхода,но представьте ситуацию, что в вашей транспортной компании
случилась беда, в связи с высокой нагрузкой на базу данных, в данный момент прием и выдача накладных закрыта.
Это означает что фирма несет убытки и все перевозки остановлены, бухгалтерия не справляется с накладными, так как не хватает рук
для их оформления...
![DI](/images/dberror.jpg)

Такая проблема - не редкость, и обычно **за ранее** это нужно предусматривать...
Чем мы как разработчики можем помочь?
- мы можем заняться рефакторингом базы данных... но она у нас ооочень большая, а время деньги
- мы можем на время подключить другую базу данных, что в итоге исправит ситуацию
```php
<?php
class FailedDB {
    public function connect() {/****/}
    public function getData() {/****/}
    public function setData() {/****/}
}

class SuccessDB {
    public function connect() {/****/}
    public function getData() {/****/}
    public function setData() {/****/}
}

class InvoicesCargo {
    
    private $db;
    public function getDb() {
        $this->db = new SuccessDB(); // заменим на рабочую
    }
}
?>
```
Но если у нас система огромная, и в конечном итоге она вся держится на одной базе данных, которая уже не работает.
Нам придется потрошить все классы которые подключены к не работающей базе. (( 
И не понятно что будет быстрее рефакторинг базы или структуры?
Конечно, в такой ситуации у вас мало выбора и вам так или иначе придется рефакторить.
Проблема в том что ваши системные компоненты жестко связаны друг с другом
С **Dependency Injection** вам будет проще управлять вашими компонентами
```php
<?php
/**
 * Interface DB
 * Создадим общий интерфейс для наших баз данных
 */
interface DB {
    public function connect();
    public function getData();
    public function setData();
}
class FailedDB implements DB {
    public function connect() {/****/}
    public function getData() {/****/}
    public function setData() {/****/}
}

class SuccessDB implements DB {
    public function connect() {/****/}
    public function getData() {/****/}
    public function setData() {/****/}
}

/**
 * class InvoicesCargo
 * Класс реализующий Dependency Injection через setDB метод (можно через конструктор)
 */
class InvoicesCargo {
    private $db;
    public function setDb(DB $db) {
        $this->db = $db;
    }
}

$db = new SuccessDB();  // меняем с new FailedDB();
$invoicesCargo = (new InvoicesCargo())->setDb($db);
?>
```
Таким образом вам не нужно будет углубляться в детали реализации методов управления базой данных, и потрошить все ваши компоненты.
Вам достаточно будет заменить класс управление другим классом и передать (инкапсулировать логику реализации БД) в компоненты.
(DIC как способ управления зависимостями на уровне компонентов не в рамках этого урока)
#### Какие еще приимущества есть у DI ?
- сокращение объема связующего кода. Одним из самых больших плюсов DI является возможность значительного сокращения объема кода, который должен быть написан для связывания вместе различных компонентов приложения. Зачастую этот код очень прост - при создании зависимости должен создаваться новый экземпляр соответствующего объекта.
- упрощенная конфигурация приложения. За счет применения DI процесс конфигурирования приложения значительно упрощается. 
- улучшенная возможность тестирования. Когда классы проектируются для DI, становится возможной простая замена зависимостей.

**Dependency Injection** — является хорошим тоном организации связей между объектами.
Паттерн является не отьемлемой частью пятого принципа объектно ориентированного проектирования SOLI:D Dependency Inversion Principe.
Наследование - является прямой противоположностью.

### 4. Полезные ссылки
1. [Шпаргалка по паттернам](/cheatsheet.pdf)
2. [Архитектура корпоративных программных приложений. М. Фаулер](http://www.ooart.ru/uploads/book/arhitektura_korporativnyh_programmnyh_prilozhenij_fauler_m.pdf)
2. [Как два программиста хлеб пекли](https://habrahabr.ru/post/153225/)
