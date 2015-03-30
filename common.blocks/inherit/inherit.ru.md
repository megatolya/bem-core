# inherit

Блок предоставляет функцию, реализующую механизмы классового наследования. Функция позволяет определить базовый класс и наследовать его логику, доопределяя ее новыми методами. 

## Обзор

### Режимы выполнения функции

| Режим | Сигнатура | Тип или возвращаемое значение | Описание |
| ----- | --------- | ----------------------------- | -------- |
| <a href="#runmode-declare">Объявление базового класса</a> | inherit(<br><code>{Object} props</code>, <br><code>[{Object} staticProps]</code>) | <code>{Function}</code> | Служит для создания (декларации), базового класса на основе свойств объекта. |
| <a href="#runmode-extend">Создание производного класса</a> | inherit(<br><code>{Function} </code>&#124;<code> {Array} BaseClass</code>, <br><code>{Object} props</code>, <br><code>[{Object} staticProps]</code>) | <code>{Function}</code> | Позволяет наследовать и доопределять свойства и методы базового класса. |

### Свойства и методы объекта базового класса

| Имя | Тип данных | Описание |
| --- | ---------- | -------- |
| <a href="#fields-__constructor">__constructor</a> | <code>{Function}</code> | Функция, которая будет вызвана в ходе создании экземпляра класса. |

### Специализированные поля объекта производного класса

| Поле | Тип | Описание |
| ---- | --- | -------- |
| <a href="#declfields-__self">__self</a> | <code>{*}</code> | Позволяет получить доступ к статическим свойствам из прототипа. |
| <a href="#declfields-__base">__base</a> | <code>{Function}</code> | Позволяет внутри производного класса использовать методы базового (supercall). |

### Публичные технологии блока

Блок реализован в технологиях:

* `vanilla.js`

## Подробности

Функция позволяет:

* создавать класс по декларации;
* задавать метод-конструктор;
* использовать миксины;
* вызывать методы базовой реализации (super call);
* использовать статические члены функции;
* получать доступ к статическим методам класса из его прототипа.

Блок является основой механизма наследования блоков в `bem-core`. Базовый блок `BEM` наследуется с помощью `inherit` от класса `Emitter` блока `events`:

```js
var BEM = inherit(events.Emitter, /** @lends BEM.prototype */ { ... }, { ... }); 
```


Все остальные блоки наследуются от блока `BEM` (или `BEMDOM` для блоков с DOM-представлением). Иначе говоря, доопределяют декларацию блока `BEM`.

Функция полиморфна и, в зависимости от типа первого аргумента, выполняется в двух режимах:

* `{Object}` – объявление базового класса.
* `{Function}` – создание производного класса на основе базового.

Сигнатуры других аргументов функции зависят от режима выполнения.

###Режимы выполнения функции

<a name="runmode-declare"></a>
#### Объявление базового класса

Режим позволяет объявить базовый класс, передав функции объект со свойствами класса.

Принимаемые аргументы:

* `props` `{Object}` – объект с собственными свойствами базового класса. Обязательный аргумент.
* [`staticProps` `{Object}`] – объект со статическими свойствами базового класса.

Возвращаемое значение: `{Function}`. Полностью сформированная функция-конструктор.

```js
modules.require(['inherit'], function(inherit) {
    var props = {}, // объект свойств базового класса
        baseClass = inherit(props); // базовый класс
provide();
});
```


##### Базовый класс со статическими свойствами

Свойства объекта `staticProps` добавляются как статические к создаваемой функции-конструктору:

```js
var A = inherit(props, {
    callMe : function() {
        console.log('mr.Static');
    }
});

A.callMe(); // mr.Static
```


**NB** Статические методы функции-конструктора выполняются в контексте самой функции. Например, ссылка `this` внутри метода `callMe` будет указывать на функцию `A`.

##### Свойства и методы объекта базового класса

<a name="fields-__constructor"></a>
###### Свойство `__constructor`

Тип: `{Function}`. Объект собственных свойств базового класса может содержать зарезервированное свойство `__constructor` – функцию, которая будет автоматически вызвана при создании экземпляра класса.

Использование `__constructor` позволяет задавать динамические свойства экземпляра класса:

```js
var A = inherit(/** @lends A.prototype */{
    __constructor : function(property) { // конструктор
        this.property = property;
    },

    getProperty : function() {
        return this.property + ' of instanceA';
    }
}),
    aInst = new A('Property');

aInst.getProperty(); // Property of instanceA
```


<a name="runmode-extend"></a>
#### Создание производного класса

Режим позволяет создать производный класс на основании базового класса и объектов статических и собственных свойств.

Принимаемые аргументы:

* `BaseClass` `{Function} | {Array}` – базовый класс. Может быть массивом функций-миксинов. Обязательный аргумент.
* `props` `{Object}` – собственные свойства (добавляются к прототипу). Обязательный аргумент.
* [`staticProps` `{Object}`] – статические свойства.

Если один из объектов содержит свойства, которые уже есть в базовом классе – свойства базового класса будут переопределены. Иначе говоря, в производный класс попадут значения из объекта.

Возвращаемое значение: `{Function}`. Производный класс.

```js
var A = inherit(/** @lends A.prototype */{
    getType : function() {
        return 'A';
    }
},
});

// класс, производный от A
var B = inherit(A, /** @lends B.prototype */{
    getAll : function() { // переопределение + 'super' call
        return this.getType() + 'B';
    }
});

var instanceOfB = new B();
instanceOfB.getType(); // возвращает 'AB'
```


##### Создание производного класса с миксинами

При объявлении производного класса можно указать дополнительный набор функций. Их свойства будут примешаны к создаваемому классу. Для этого первым аргументом `inherit` нужно указать массив, первым элементом которого должен быть базовый класс, а последующими – примешиваемые функции:

```js
Function inherit(
    [
        Function BaseClass,
        Function Mixin,
        Function AnotherMixin,
        ...
    ],
    Object props,
    Object staticProps);
```


##### Специализированные поля объекта производного класса

<a name="declfields-__self"></a>
###### Поле `__self`

Тип: `*`.

Позволяет получить доступ к статическим свойствам функции-конструктора непосредственно из ее прототипа:

```js
var A = inherit(/** @lends A.prototype */{
    getStaticProperty : function() {
        return this.__self.staticMethod; // доступ к статическим методам
    }
}, /** @lends A */ {    
    staticProperty : 'staticA',

    staticMethod : function() {
        return this.staticProperty;
    }
}),
    aInst = new A();
aInst.getStaticProperty(); //staticA
```


<a name="declfields-__base"></a>
###### `__base`

Тип: `{Function}`.

Поле позволяет внутри производного класса вызывать методы базового (supercall). Поле `__base` позволяет вызвать так же статические методы базового класса:

```js
var A = inherit(/** @lends A.prototype */{
    getType : function() {
        return 'A';
    }
}, /** @lends A */ {    
    staticProperty : 'staticA',

    staticMethod : function() {
        return this.staticProperty;
    }
});

// класс, производный от A
var B = inherit(A, /** @lends B.prototype */{
    getType : function() { // переопределение + 'super' call
        return this.__base() + 'B';
    }
}, /** @lends B */ {
    staticMethod : function() { // статическое переопределение + 'super' call
        return this.__base() + ' of staticB';
    }
});

var instanceOfB = new B();

instanceOfB.getType(); // возвращает 'AB'
B.staticMethod(); // возвращает 'staticA of staticB'
```


<a name="extra-examples"></a>
### Дополнительные примеры

В примере используются все основные способы наследования с помощью `inherit`:

```js
// базовый класс
var A = inherit(/** @lends A.prototype */{
    __constructor : function(property) { // конструктор
        this.property = property;
    },

    getProperty : function() {
        return this.property + ' of instanceA';
    },

    getType : function() {
        return 'A';
    },

    getStaticProperty : function() {
        return this.__self.staticMethod; // доступ к статическим свойствам и методам
    }
}, /** @lends A */ {    
    staticProperty : 'staticA',

    staticMethod : function() {
        return this.staticProperty;
    }
});

// класс, производный от A
var B = inherit(A, /** @lends B.prototype */{
    getProperty : function() { // переопределение
        return this.property + ' of instanceB';
    },

    getType : function() { // переопределение + 'super' call
        return this.__base() + 'B';
    }
}, /** @lends B */ {
    staticMethod : function() { // статическое переопределение + 'super' call
        return this.__base() + ' of staticB';
    }
});

// миксин M
var M = inherit({
    getMixedProperty : function() {
        return 'mixed property';
    }
});

// производный класс от A с миксином M
var C = inherit([A, M], {
    getMixedProperty : function() {
        return this.__base() + ' from C';
    }
});

var instanceOfB = new B('property');

instanceOfB.getProperty(); // возвращает 'property of instanceB'
instanceOfB.getType(); // возвращает 'AB'
B.staticMethod(); // возвращает 'staticA of staticB'

var instanceOfC = new C();
instanceOfC.getMixedProperty(); // возвращает 'mixed property from C'
```
