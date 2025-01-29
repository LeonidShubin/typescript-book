### Classes
Причина, чому важливо мати класи в JavaScript як елемент першого класу, полягає в тому, що:
1. [Класи пропонують корисну структурну абстракцію](./tips/classesAreUseful.md)
1. Забезпечують послідовний спосіб для розробників використовувати класи замість кожного фреймворку (emberjs, reactjs тощо), створюючи власну версію.
1. Об'єктно-орієнтовані розробники вже розуміють класи

Нарешті розробники JavaScript можуть *мати `class`*. Як приклад, ми маємо базовий клас під назвою Point:
```ts
class Point {
    x: number;
    y: number;
    constructor(x: number, y: number) {
        this.x = x;
        this.y = y;
    }
    add(point: Point) {
        return new Point(this.x + point.x, this.y + point.y);
    }
}

var p1 = new Point(0, 10);
var p2 = new Point(10, 20);
var p3 = p1.add(p2); // {x:10,y:30}
```
Цей клас генерує наступний JavaScript, грунтуючись на ES5:
```ts
var Point = (function () {
    function Point(x, y) {
        this.x = x;
        this.y = y;
    }
    Point.prototype.add = function (point) {
        return new Point(this.x + point.x, this.y + point.y);
    };
    return Point;
})();
```
Це досить ідіоматичний традиційний шаблон класу JavaScript, який тепер є мовною конструкцією першого класу.

### Inheritance
Наслідування.

Класи в TypeScript (як і в інших мовах) підтримують *single* наслідування (у класа може бути ліше один батьківський клас) за допомогою ключового слова `extends`, як показано нижче:

```ts
class Point3D extends Point {
    z: number;
    constructor(x: number, y: number, z: number) {
        super(x, y);
        this.z = z;
    }
    add(point: Point3D) {
        var point2D = super.add(point);
        return new Point3D(point2D.x, point2D.y, this.z + point.z);
    }
}
```
Якщо у вашому класі є конструктор, ви *повинні* викликати батьківський конструктор із свого конструктора (TypeScript вкаже вам на це). Це гарантує, що все, що потрібно для існування класу та ініціалізації `this`, буде встановлено. Після виклику `super` ви можете додати будь-які додаткові речі, які ви хочете зробити у вашому конструкторі (тут ми додаємо ще один член `z`).
Зауважте, що ви легко замінюєте батьківські функції-члени (тут ми перевизначаємо `add` ) і все ще використовуємо функціональність суперкласу у своїх членах (використовуючи синтаксис `super.` ).

### Statics
Статичні властивості класу.

Класи TypeScript підтримують статичні властивості, які є спільними для всіх екземплярів класу. Природним місцем для їх розміщення (і доступу) є сам клас, і саме це робить TypeScript:

```ts
class Something {
    static instances = 0;
    constructor() {
        Something.instances++;
    }
}

var s1 = new Something();
var s2 = new Something();
console.log(Something.instances); // 2
```

Ви можете мати як статичні члени, так і статичні функції.

### Access Modifiers
Модифікатори доступу.

TypeScript підтримує модифікатори доступу `public` (публічний),`private` (приватний) та `protected` (захищений), які визначають доступність члена класу , як показано нижче:

| доступні для    | `public` | `protected` | `private` |
|-----------------|----------|-------------|-----------|
| клас            | так      | ні          | так       |
| дочерний клас   | так      | так         | ні        |
| екземпляр класу | так      | ні          | ні        |


Якщо модифікатор доступу не вказано, він неявно відкритий, оскільки це відповідає зручному характеру JavaScript 🌹.

Зверніть увагу, що під час виконання (у згенерованому JS) вони не мають значення, але викличуть помилки під час компіляції, якщо ви використовуєте їх неправильно. Приклад кожного показаний нижче:
```ts
class FooBase {
    public x: number;
    private y: number;
    protected z: number;
}

// EFFECT ON INSTANCES
var foo = new FooBase();
foo.x; // okay
foo.y; // ERROR : private
foo.z; // ERROR : protected

// EFFECT ON CHILD CLASSES
class FooChild extends FooBase {
    constructor() {
      super();
        this.x; // okay
        this.y; // ERROR: private
        this.z; // okay
    }
}
```

Як завжди, ці модифікатори працюють як для властивостей члена, так і для функцій члена.

### Abstract
Абстрактний клас.

`abstract` можна розглядати як модифікатор доступу. Ми представляємо його окремо, тому що, на відміну від згаданих раніше модифікаторів, він може бути як у `class`, так і в будь-якому члені класу. Наявність `abstract` модифікатора в першу чергу означає, що така функціональність *не може бути безпосередньо викликана* , і дочірній клас повинен надавати цю функціональність.

* Ми не можемо створити екземпляр `abstract` **classes**. Замість цього користувач повинен створити якийсь `class`, який наслідує від `abstract class` 

```ts
abstract class FooCommand {}

class BarCommand extends FooCommand {}

const fooCommand: FooCommand = new FooCommand(); // Не можно створити екземпляр (instance) класу
const barCommand = new BarCommand(); // Можно створити екземпляр класу, який є потомком абстрактного класу
```

*  неможливо отримати прямий доступ до`abstract` **members**, тому дочірній клас повинен забезпечувати цю функціональність.

```ts
abstract class FooCommand {
  abstract execute(): string;
}

class BarErrorCommand  extends FooCommand {} // 'BarErrorCommand' повинен реалізувати'execute'.

class BarCommand extends FooCommand {
  execute() {
    return `Command Bar executed`;
  }
}

const barCommand = new BarCommand();

barCommand.execute(); // Command Bar executed
```

### Constructor is optional
Конструктор класу необов'язковий.

Клас не потребує наявності конструктора. наприклад, наступне цілком нормально. Якщо не створити конструктор власноруч, TypeScript зробить конструктор за замовчуванням

```ts
class Foo {}
var foo = new Foo();
```

### Define using constructor
Визначення властивості за допомогою конструктора.

Наявність члена в класі та його ініціалізація, як показано нижче:

```ts
class Foo {
    x: number;
    constructor(x:number) {
        this.x = x;
    }
}
```
є настільки поширеним шаблоном, що TypeScript надає скорочення, де ви можете додати до члена *access modifier*, і він автоматично оголошується в класі та копіюється з конструктора. Тож попередній приклад можна переписати таким чином (зверніть увагу на `public x:numbe`r ):
```ts
class Foo {
    constructor(public x:number) {
    }
}
```

### Property initializer
Ініціалізатор властивості.

Це чудова функція, яка підтримується TypeScript (з ES7). Ви можете ініціалізувати будь-який член класу поза конструктором класу, що корисно для встановлення значення за замовчуванням (зверніть увагу на `number = []` )
```ts
class Foo {
    members = [];  // Initialize directly
    add(x) {
        this.members.push(x);
    }
}
```
