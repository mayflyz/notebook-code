# TypeScript介绍

JS语言由Mozilla创造，最初主要是为了解决页面中的逻辑交互问题，它和HTML（负责页面内容）、CSS（负责页面布局和样式）共同组成了Web页面/应用开发的基础。

大型的应用工程一般会涉及较复杂的代码以及较多的团队协作，对语言的规范性，模块的复用性、扩展性以及相关的开发工具都提出了更高的要求。

Microsoft在JS的基础上，创建了TS语言，并在2014年正式发布了1.0版本。TS主要从以下几个方面做了进一步的增强：

* 引入了**类型系统**，并提供了类型检查以及类型自动推导能力，可以进行编译时错误检查，有效的提升了代码的规范性以及错误检测范围和效率。
* 在类型系统基础上，引入了**声明文件**（Declaration Files）来管理接口或其他自定义类型。声明文件一般是以d.ts的形式来定义模块中的接口，这些接口和具体的实现做了相应的分离，有助于各模块之间的分工协作。另外，TS通过接口，泛型（Generics）等相关特性的支持，进一步增强了设计复杂的框架所需的扩展以及复用能力。

TS在兼容JS生态方面也做了较好的平衡，TS应用通过相应编译器可以编译出纯JS应用，可以在标准的JS引擎上运行。

### 1、基础类型

#### 布尔值

最基本的数据类型就是简单的true/false值， boolean

```ts
let isDone: boolean = false;
```

#### 数字

TypeScript里的所有数字都是浮点数，型是 number。可支持二进制、八进制、十进制、十六进制

```ts
let decLiteral: number = 6;
let hexLiteral: number = 0xf00d;
let binaryLiteral: number = 0b1010;
let octalLiteral: number = 0o744;
```

#### 字符串

文本数据类型使用`string`表示，可以使用双引号（ "）或单引号（'）表示字符串。

可以使用模版字符串，它可以定义多行文本和内嵌表达式。使用反引号包围（\`），并且以${ expr }这种形式嵌入表达式

```ts
let name: string = "bob";
name = 'smith';

let name: string = `Gene`;
let age: number = 37;
let sentence: string = `Hello, my name is ${ name }.

I'll be ${ age + 1 } years old next month.`;
```

#### 数组

定义数组方式：

* 元素类型后面接上 \[]，表示由此类型元素组成的一个数组
* 使用数组泛型，Array

```ts
let list: number[] = [1, 2, 3];

let list: Array<number> = [1, 2, 3];
```

#### 元组Tuple

元组类型允许表示一个已知元素数量和类型的数组，各元素的类型不必相同。

```ts
// Declare a tuple type
let x: [string, number];
// Initialize it
x = ['hello', 10]; // OK
// Initialize it incorrectly
x = [10, 'hello']; // Error
```

#### 枚举

枚举类型可以为一组数值赋予友好的名字。

```ts
enum Color {Red, Green, Blue}
let c: Color = Color.Green;

//默认情况下，从0开始为元素编号。可以指定成员的数值或者全部都采用手动赋值
enum Color {Red = 1, Green = 2, Blue = 4}
let c: Color = Color.Green;
```

#### Any

想要为那些在编程阶段还不清楚类型的变量指定一个类型。使用 any类型来标记这些变量

```ts
let notSure: any = 4;
notSure = "maybe a string instead";
notSure = false; // okay, definitely a boolean
```

`Object`类型的变量只是允许你给它赋任意值——但是却不能够在它上面调用任意的方法，即便它真的有这些方法

```ts
let notSure: any = 4;
notSure.ifItExists(); // okay, ifItExists might exist at runtime
notSure.toFixed(); // okay, toFixed exists (but the compiler doesn't check)

let prettySure: Object = 4;
prettySure.toFixed(); // Error: Property 'toFixed' doesn't exist on type 'Object'.
```

#### Void

void表示没有任何类型。声明一个`void`类型的变量没有什么大用，因为你只能为它赋予`undefined`和`null`：

#### Null 和 Undefined

默认情况下null和undefined是所有类型的子类型。

#### Never

never类型表示的是那些永不存在的值的类型。never类型是任何类型的子类型

#### Object

`object`表示非原始类型，即除number，string，boolean，symbol，null或undefined之外的类型。 使用object类型，就可以更好的表示像Object.create这样的API。

```ts
declare function create(o: object | null): void;

create({ prop: 0 }); // OK
create(null); // OK
```

#### 类型断言

类型没有运行时的影响，只是在编译阶段起作用。 类型断言有两种形式。 其一是“尖括号”语法，另一个为as语法。

```ts
let someValue: any = "this is a string";

let strLength: number = (<string>someValue).length;

let strLength: number = (someValue as string).length;
```

### 2、变量声明

#### var 声明

一直以来我们都是通过var关键字定义JavaScript变量。

var声明可以在包含它的函数，模块，命名空间或全局作用域内部任何位置被访问。可能会引发一些错误。

```ts
function f(shouldInitialize: boolean) {
    if (shouldInitialize) {
        var x = 10;
    }

    return x;
}

f(true);  // returns '10'
f(false); // returns 'undefined'
```

#### let 声明

当用let声明一个变量，使用的是词法作用域或块作用域。

```ts
function f(input: boolean) {
    let a = 100;

    if (input) {
        // Still okay to reference 'a'
        let b = a + 1;
        return b;
    }

    // Error: 'b' doesn't exist here
    return b;
}
```

#### const 声明

const声明被赋值后不能再改变。它们引用的值是不可变的。实际上const变量的内部状态是可修改的.

```ts
const numLivesForCat = 9;
const kitty = {
    name: "Aurora",
    numLives: numLivesForCat,
}

// Error
kitty = {
    name: "Danielle",
    numLives: numLivesForCat
};

// all "okay"
kitty.name = "Rory";
kitty.name = "Kitty";
kitty.name = "Cat";
kitty.numLives--;
```

#### 解构

* 解构数组

```ts
let input = [1, 2];
let [first, second] = input;
console.log(first); // outputs 1
console.log(second); // outputs 2
```

可以在数组里使用...语法创建剩余变量

```ts
let [first, ...rest] = [1, 2, 3, 4];
console.log(first); // outputs 1
console.log(rest); // outputs [ 2, 3, 4 ]
```

* 对象解构

```ts
let o = {
    a: "foo",
    b: 12,
    c: "bar"
};
let { a, b } = o;
```

* 函数声明 解构也能用于函数声明。

```ts
type C = { a: string, b?: number }
function f({ a, b }: C): void {
    // ...
}
```

### 3、接口

TypeScript的核心原则之一是对值所具有的结构进行类型检查。在TypeScript里，接口的作用就是为这些类型命名和为你的代码或第三方代码定义契约。

#### 接口初探

```ts
function printLabel(labelledObj: { label: string }) {
  console.log(labelledObj.label);
}

let myObj = { size: 10, label: "Size 10 Object" };
printLabel(myObj);
```

类型检查器会查看`printLabel`的调用。 `printLabel`有一个参数，并要求这个对象参数有一个名为`label`类型为`string`的属性。 我们传入的对象参数实际上会包含很多属性，但是编译器只会检查那些必需的属性是否存在，并且其类型是否匹配。

```ts
interface LabelledValue {
  label: string;
}

function printLabel(labelledObj: LabelledValue) {
  console.log(labelledObj.label);
}

let myObj = {size: 10, label: "Size 10 Object"};
printLabel(myObj);
```

#### 可选属性

接口里的属性不全都是必需的。可选属性在应用“option bags”模式时很常用，即给函数传入的参数对象中只有部分属性赋值了。

```ts
interface SquareConfig {
  color?: string;
  width?: number;
}

function createSquare(config: SquareConfig): {color: string; area: number} {
  let newSquare = {color: "white", area: 100};
  if (config.color) {
    newSquare.color = config.color;
  }
  if (config.width) {
    newSquare.area = config.width * config.width;
  }
  return newSquare;
}

let mySquare = createSquare({color: "black"});
```

可选属性名字定义的后面加一个`?`符号。好处：

* 可以对可能存在的属性进行预定义
* 可以捕获引用了不存在的属性时的错误

#### 只读属性

只能在对象刚刚创建的时候修改其值，用 readonly来指定只读属性

```ts
interface Point {
    readonly x: number;
    readonly y: number;
}
//赋值后， x和y再也不能被改变了。
let p1: Point = { x: 10, y: 20 };
p1.x = 5; // error!
```

#### 函数类型

为了使用接口表示函数类型，我们需要给接口定义一个调用签名。是一个只有参数列表和返回值类型的函数定义。参数列表里的每个参数都需要名字和类型。

```ts
interface SearchFunc {
  (source: string, subString: string): boolean;
}

// 定义后，可以像使用其它接口一样使用这个函数类型的接口。
let mySearch: SearchFunc;
mySearch = function(source: string, subString: string) {
  let result = source.search(subString);
  return result > -1;
}
```

#### 可索引的类型

可索引类型具有一个索引签名，它描述了对象索引的类型，还有相应的索引返回值类型。 TypeScript支持两种索引签名：字符串和数字。数字索引的返回值必须是字符串索引返回值类型的子类型。

字符串索引签名能够很好的描述dictionary模式，并且它们也会确保所有属性与其返回值类型相匹配。

```ts
interface StringArray {
  [index: number]: string;
}

let myArray: StringArray;
myArray = ["Bob", "Fred"];

let myStr: string = myArray[0];
```

#### 类类型

```ts
interface ClockInterface {
    currentTime: Date;
}

class Clock implements ClockInterface {
    currentTime: Date;
    constructor(h: number, m: number) { }
}
```

接口描述了类的公共部分，而不是公共和私有两部分。

#### 继承接口

和类一样，接口也可以相互继承。一个接口可以继承多个接口，创建出多个接口的合成接口。

```ts
interface Shape {
    color: string;
}

interface PenStroke {
    penWidth: number;
}

interface Square extends Shape, PenStroke {
    sideLength: number;
}

let square = <Square>{};
square.color = "blue";
square.sideLength = 10;
square.penWidth = 5.0;
```

#### 接口继承类

当接口继承了一个类类型时，它会继承类的成员但不包括其实现。接口同样会继承到类的private和protected成员。

### 4、类

传统的JavaScript程序使用函数和基于原型的继承来创建可重用的组件。从ECMAScript 6开始，JavaScript程序员将能够使用基于类的面向对象的方式。 TypeScript支持类。

```ts
class Greeter {
    greeting: string;
    constructor(message: string) {
        this.greeting = message;
    }
    greet() {
        return "Hello, " + this.greeting;
    }
}

let greeter = new Greeter("world");
```

#### 继承

允许使用继承来扩展现有的类。

```ts
class Animal {
    name: string;
    constructor(theName: string) { this.name = theName; }
    move(distanceInMeters: number = 0) {
        console.log(`${this.name} moved ${distanceInMeters}m.`);
    }
}

class Snake extends Animal {
    constructor(name: string) { super(name); }
    move(distanceInMeters = 5) {
        console.log("Slithering...");
        super.move(distanceInMeters);
    }
}

class Horse extends Animal {
    constructor(name: string) { super(name); }
    move(distanceInMeters = 45) {
        console.log("Galloping...");
        super.move(distanceInMeters);
    }
}

let sam = new Snake("Sammy the Python");
let tom: Animal = new Horse("Tommy the Palomino");

sam.move();
tom.move(34);
```

#### 公共，私有与受保护的修饰符

* 默认为 public
* private修饰符：不能在声明它的类的外部访问
* protected修饰符：成员在派生类中仍然可以访问
* readonly修饰符：只读属性必须在声明时或构造函数里被初始化

#### 存取器

TypeScript支持通过getters/setters来截取对对象成员的访问。能帮助你有效的控制对对象成员的访问。

```ts
let passcode = "secret passcode";

class Employee {
    private _fullName: string;

    get fullName(): string {
        return this._fullName;
    }

    set fullName(newName: string) {
        if (passcode && passcode == "secret passcode") {
            this._fullName = newName;
        }
        else {
            console.log("Error: Unauthorized update of employee!");
        }
    }
}

let employee = new Employee();
employee.fullName = "Bob Smith";
if (employee.fullName) {
    alert(employee.fullName);
}
```

#### 静态属性

静态成员，存在于类本身上面而不是类的实例上。

#### 抽象类

抽象类做为其它派生类的基类使用。一般不会直接被实例化。抽象类可以包含成员的实现细节。

```ts
abstract class Animal {
    abstract makeSound(): void;
    move(): void {
        console.log('roaming the earch...');
    }
}
```

抽象类中的抽象方法不包含具体实现并且必须在派生类中实现。

### 5、函数

函数是JavaScript应用程序的基础。 它帮助你实现抽象层，模拟类，信息隐藏和模块。

#### 函数定义

TypeScript函数可以创建有名字的函数和匿名函数。在JavaScript里，函数可以使用函数体外部的变量。 当函数这么做时，我们说它‘捕获’了这些变量。

```ts
// Named function
function add(x, y) {
    return x + y;
}

// Anonymous function
let myAdd = function(x, y) { return x + y; };

let z = 100;
function addToZ(x, y) {
    return x + y + z;
}
```

#### 函数类型

TypeScript能够根据返回语句自动推断出返回值类型，因此我们通常省略它。

```ts
function add(x: number, y: number): number {
    return x + y;
}

let myAdd = function(x: number, y: number): number { return x + y; };
```

#### 可选参数和默认参数

TypeScript里传递给一个函数的参数个数必须与函数期望的参数个数一致。可以在参数名旁使用`?`实现可选参数的功能，可选参数必须跟在必须参数后面。

```ts
function buildName(firstName: string, lastName?: string) {
    if (lastName)
        return firstName + " " + lastName;
    else
        return firstName;
}

let result1 = buildName("Bob");  // works correctly now
let result2 = buildName("Bob", "Adams", "Sr.");  // error, too many parameters
let result3 = buildName("Bob", "Adams");  // ah, just right
```

默认初始化值的参数，指当没有传递这个参数或传递的值是undefined时为参数提供一个默认值。

```ts
function buildName(firstName: string, lastName = "Smith") {
    return firstName + " " + lastName;
}
```

#### 剩余参数

在TypeScript里，你可以把所有参数收集到一个变量里：

```ts
function buildName(firstName: string, ...restOfName: string[]) {
  return firstName + " " + restOfName.join(" ");
}

let employeeName = buildName("Joseph", "Samuel", "Lucas", "MacKinzie");
```

剩余参数会被当做个数不限的可选参数。

#### this

JavaScript里，this的值在函数被调用的时候才会指定。

#### 重载

方法是为同一个函数提供多个函数类型定义来进行函数重载。 编译器会根据这个列表去处理函数的调用。

为了让编译器能够选择正确的检查类型，typescript会按顺序查找重载列表，尝试使用第一个重载定义。 如果匹配的话就使用这个。 因此，在定义重载的时候，一定要把最精确的定义放在最前面。

```ts
let suits = ["hearts", "spades", "clubs", "diamonds"];

function pickCard(x: {suit: string; card: number; }[]): number;
function pickCard(x: number): {suit: string; card: number; };
function pickCard(x): any {
    // Check to see if we're working with an object/array
    // if so, they gave us the deck and we'll pick the card
    if (typeof x == "object") {
        let pickedCard = Math.floor(Math.random() * x.length);
        return pickedCard;
    }
    // Otherwise just let them pick the card
    else if (typeof x == "number") {
        let pickedSuit = Math.floor(x / 13);
        return { suit: suits[pickedSuit], card: x % 13 };
    }
}

let myDeck = [{ suit: "diamonds", card: 2 }, { suit: "spades", card: 10 }, { suit: "hearts", card: 4 }];
let pickedCard1 = myDeck[pickCard(myDeck)];
alert("card: " + pickedCard1.card + " of " + pickedCard1.suit);

let pickedCard2 = pickCard(15);
alert("card: " + pickedCard2.card + " of " + pickedCard2.suit);
```

### 6、泛型

#### 定义泛型

泛型是一种特殊的变量，只用于表示类型而不是值。

```ts
function identity<T>(arg: T): T {
    return arg;
}
//使用方式：
// 1、传入所有的参数，包含类型参数
let output = identity<string>("myString");  // type of output will be 'string'

// 2、利用了类型推论 -- 即编译器会根据传入的参数自动地帮助我们确定T的类型
let output = identity("myString");  // type of output will be 'string'
```

#### 使用泛型变量

编译器要求你在函数体必须正确的使用这个通用的类型。就是说必须把这些参数当做是任意或所有类型。

```ts
function loggingIdentity<T>(arg: Array<T>): Array<T> {
    console.log(arg.length);  // Array has a .length, so no more error
    return arg;
}
```

#### 泛型类型

泛型函数的类型与非泛型函数的类型没什么不同，只是有一个类型参数在最前面，像函数声明一样：

```ts
function identity<T>(arg: T): T {
    return arg;
}

let myIdentity: <T>(arg: T) => T = identity;
```

也可以使用不同的泛型参数名，还可以使用带有调用签名的对象字面量来定义泛型函数

```ts
function identity<T>(arg: T): T {
    return arg;
}

let myIdentity: <U>(arg: U) => U = identity;

let myIdentity: {<T>(arg: T): T} = identity;
```

#### 泛型类

泛型类使用（ <>）括起泛型类型，跟在类名后面。

```ts
class GenericNumber<T> {
    zeroValue: T;
    add: (x: T, y: T) => T;
}

let myGenericNumber = new GenericNumber<number>();
myGenericNumber.zeroValue = 0;
myGenericNumber.add = function(x, y) { return x + y; };
```

#### 泛型约束

我们定义一个接口来描述约束条件。

```ts
interface Lengthwise {
    length: number;
}

function loggingIdentity<T extends Lengthwise>(arg: T): T {
    console.log(arg.length);  // Now we know it has a .length property, so no more error
    return arg;
}
```

### 7、枚举

使用枚举我们可以定义一些带名字的常量。TypeScript支持数字的和基于字符串的枚举。

#### 数字枚举

```ts
enum Direction {
    Up = 1,
    Down,
    Left,
    Right
}
```

#### 字符串枚举

在一个字符串枚举里，每个成员都必须用字符串字面量，或另外一个字符串枚举成员进行初始化。

```ts
enum Direction {
    Up = "UP",
    Down = "DOWN",
    Left = "LEFT",
    Right = "RIGHT",
}
```

#### 异构枚举

从技术的角度来说，枚举可以混合字符串和数字成员，除非你真的想要利用JavaScript运行时的行为，否则我们不建议这样做。

```ts
enum BooleanLikeHeterogeneousEnum {
    No = 0,
    Yes = "YES",
}
```

#### 计算的和常量成员

每个枚举成员都带有一个值，它可以是常量或计算出来的。 当满足如下条件时，枚举成员被当作是常量：

* 枚举的第一个成员且没有初始化器，这种情况下它被赋予值 0
* 不带有初始化器且它之前的枚举成员是一个数字常量，当前枚举成员的值为它上一个枚举成员的值加1。
* 枚举成员使用常量枚举表达式初始化。常数枚举表达式在编译阶段求值。常量枚举表达式：
  * 一个枚举表达式字面量（主要是字符串字面量或数字字面量）
  * 一个对之前定义的常量枚举成员的引用（可以是在不同的枚举类型中定义的）
  * 带括号的常量枚举表达式
  * 一元运算符 +, -, \~其中之一应用在了常量枚举表达式
  * 常量枚举表达式做为二元运算符 +, -, \*, /, %, <<, >>, >>>, &, |, ^的操作对象。

```ts
enum FileAccess {
    // constant members
    None,
    Read    = 1 << 1,
    Write   = 1 << 2,
    ReadWrite  = Read | Write,
    // computed member
    G = "123".length
}
```

#### 联合枚举与枚举成员的类型

字面量枚举成员是指不带有初始值的常量枚举成员，或者是值被初始化为:

* 任何字符串字面量（例如： "foo"， "bar"， "baz"）
* 任何数字字面量（例如： 1, 100）
* 应用了一元 -符号的数字字面量（例如： -1, -100）

#### 运行时的枚举

枚举是在运行时真正存在的对象。

#### const枚举

常量枚举通过在枚举上使用 const修饰符来定义。常量枚举成员在使用的地方会被内联进来。因此常量枚举不允许包含计算成员。

```ts
const enum Enum {
    A = 1,
    B = A * 2
}
```

### 8、类型推论

类型是在哪里如何被推断的。

#### 基础

TypeScript里，在有些没有明确指出类型的地方，类型推论会帮助提供类型。大多数情况下，类型推论是直截了当的。

```ts
let x = 3;
```

#### 最佳通用类型

当需要从几个表达式中推断类型时候，会使用这些表达式的类型来推断出一个最合适的通用类型。

```ts
let x = [0, 1, null];
```

计算通用类型算法会考虑所有的候选类型，并给出一个兼容所有候选类型的类型。当候选类型不能使用的时候我们需要明确的指出类型。

#### 上下文类型

按上下文归类会发生在表达式的类型与所处的位置相关时。

```ts
//TypeScript类型检查器使用Window.onmousedown函数的类型来推断右边函数表达式的类型。 
window.onmousedown = function(mouseEvent) {
    console.log(mouseEvent.button);  //<- Error
};
```

如果上下文类型表达式包含了明确的类型信息，上下文的类型被忽略。

```ts
window.onmousedown = function(mouseEvent: any) {
    console.log(mouseEvent.button);  //<- Now, no error is given
};
```

### 9、迭代器和生成器

#### 可迭代性

当一个对象实现了`Symbol.iterator`属性时，我们认为它是可迭代的。一些内置的类型如 Array，Map，Set，String，Int32Array，Uint32Array等都支持。

* `for..of` 语句 & `for..in`语句 for..of会遍历可迭代的对象，调用对象上的Symbol.iterator方法。 for..in迭代的是对象的键的列表，而for..of则迭代对象的键对应的值。

```ts
let someArray = [1, "string", false];

for (let entry of someArray) {
    console.log(entry); // 1, "string", false
}
```

### 10、模块

模块在其自身的作用域里执行，而不是在全局作用域里； 模块是自声明的；两个模块之间的关系是通过在文件级别上使用imports和exports建立的。

#### 导出

* 导出声明: 任何声明（比如变量，函数，类，类型别名或接口）都能够通过添加export关键字来导出。
* 导出语句: 对导出的部分重命名
* 重新导出：只导出那个模块的部分内容或者一个模块可以包裹多个模块，并把他们导出的内容联合在一起通过语法：export \* from "module"。

```ts
//导出声明
export const numberRegexp = /^[0-9]+$/;

export interface StringValidator {
    isAcceptable(s: string): boolean;
}

//导出语句
class ZipCodeValidator implements StringValidator {
    isAcceptable(s: string) {
        return s.length === 5 && numberRegexp.test(s);
    }
}
export { ZipCodeValidator };
export { ZipCodeValidator as mainValidator };
```

#### 导入

导入一个模块中的某个导出内容，可以对导入内容重命名

```ts
import { ZipCodeValidator as ZCV } from "./ZipCodeValidator";
let myValidator = new ZCV();
```

将整个模块导入到一个变量，并通过它来访问模块的导出部分

```ts
import * as validator from "./ZipCodeValidator";
let myValidator = new validator.ZipCodeValidator();
```

#### 默认导出

每个模块都可以有一个default导出。 默认导出使用 default关键字标记；并且一个模块只能够有一个default导出。

```ts
const numberRegexp = /^[0-9]+$/;

export default function (s: string) {
    return s.length === 5 && numberRegexp.test(s);
}
```

`default`导出也可以是一个值

#### export = 和 import = require()

`export =`语法定义一个模块的导出对象。 这里的对象一词指的是类，接口，命名空间，函数或枚举。 若使用`export =`导出一个模块，则必须使用TypeScript的特定语法`import module = require("module")`来导入此模块。

```ts
//ZipCodeValidator.ts
let numberRegexp = /^[0-9]+$/;
class ZipCodeValidator {
    isAcceptable(s: string) {
        return s.length === 5 && numberRegexp.test(s);
    }
}
export = ZipCodeValidator;

//Test.ts
import zip = require("./ZipCodeValidator");
```

#### 可选的模块加载和其它高级加载场景

编译器会检测是否每个模块都会在生成的JavaScript中用到。 如果一个模块标识符只在类型注解部分使用，并且完全没有在表达式中使用时，就不会生成 require这个模块的代码。

模块加载器会被动态调用（通过 require），利用了省略引用的优化，所以模块只在被需要时加载。

为了确保类型安全性，我们可以使用typeof关键字。

#### 创建模块结构指导

尽可能地在顶层导出

> 如果仅导出单个 class 或 function，使用 export default 如果要导出多个对象，把它们放在顶层里导出 明确地列出导入的名字

```ts
export default class SomeType {
  constructor() { ... }
}
```

#### 使用重新导出进行扩展

在扩展一个模块的功能，推荐的方案是 不要去改变原来的对象，而是导出一个新的实体来提供新的功能。

#### 模块里不要使用命名空间

在组织方面，命名空间对于在全局作用域内对逻辑上相关的对象和类型进行分组是很便利的。

模块本身已经存在于文件系统之中，这是必须的。必须通过路径和文件名找到它们，这已经提供了一种逻辑上的组织形式。

### 11、模块解析

模块解析是指编译器在查找导入模块内容时所遵循的流程。首先，编译器会尝试定位表示导入模块的文件。 其次，如果上面的解析失败了并且模块名是非相对的，编译器会尝试定位一个外部模块声明。最后，如果编译器还是不能解析这个模块，它会记录一个错误。

#### 相对 vs. 非相对模块导入

相对导入是以/，./或../开头的。 所有其它形式的导入被当作非相对的。

```ts
//相对导入
import Entry from "./components/Entry";
import { DefaultHeaders } from "../constants/http";
import "/mod";
//非相对导入
import * as $ from "jQuery";
import { Component } from "@angular/core";
```

相对导入在解析时是相对于导入它的文件，并且不能解析为一个外部模块声明。 非相对模块的导入可以相对于baseUrl或通过下文会讲到的路径映射来进行解析。可以被解析成`外部模块生命`

#### 模块解析策略

共有两种可用的模块解析策略：Node和Classic。

Classic策略以前是TypeScript默认的解析策略。现在，它存在的理由主要是为了向后兼容。 Node策略试图在运行时模仿Node.js模块解析机制。

#### 附加的模块解析标记

有时工程源码结构与输出结构不同。通常是要经过一系统的构建步骤最后生成输出。

TypeScript编译器有一些额外的标记用来通知编译器在源码编译成最终输出的过程中都发生了哪个转换。

`Base URL`要求在运行时模块都被放到了一个文件夹里。设置baseUrl来告诉编译器到哪里去查找模块。 所有非相对模块导入都会被当做相对于 baseUrl。

### 12、声明合并

“声明合并”是指编译器将针对同一个名字的两个独立声明合并为单一声明。合并后的声明同时拥有原先两个声明的特性。 任何数量的声明都可被合并；不局限于两个声明。

#### 基础概念

TypeScript中的声明会创建以下三种实体之一：命名空间，类型或值。 创建命名空间的声明会新建一个命名空间，它包含了用（.）符号来访问时使用的名字。创建类型的声明是：用声明的模型创建一个类型并绑定到给定的名字上。 最后，创建值的声明会创建在JavaScript输出中看到的值。

#### 合并接口

从根本上说，合并的机制是把双方的成员放到一个同名的接口里。

接口的非函数的成员应该是唯一的。如果它们不是唯一的，那么它们必须是相同的类型。如果两个接口中同时声明了同名的非函数成员且它们的类型不同，则编译器会报错。

对于函数成员，每个同名函数声明都会被当成这个函数的一个重载。

#### 合并命名空间

对于命名空间的合并，模块导出的同名接口进行合并，构成单一命名空间内含合并后的接口。如果当前已经存在给定名字的命名空间，那么后来的命名空间的导出成员会被加到已经存在的那个模块里。

```ts
namespace Animals {
    export class Zebra { }
}

namespace Animals {
    export interface Legged { numberOfLegs: number; }
    export class Dog { }
}
```

等同于：

```ts
namespace Animals {
    export interface Legged { numberOfLegs: number; }

    export class Zebra { }
    export class Dog { }
}
```

#### 命名空间与类和函数和枚举类型合并

命名空间可以与其它类型的声明进行合并。 只要命名空间的定义符合将要合并类型的定义。合并结果包含两者的声明类型。

**合并命名空间和类** 合并结果是一个类并带有一个内部类。

```ts
class Album {
    label: Album.AlbumLabel;
}
namespace Album {
    export class AlbumLabel { }
}

```

**命名空间可以用来扩展枚举型：**

```ts
enum Color {
    red = 1,
    green = 2,
    blue = 4
}

namespace Color {
    export function mixColor(colorName: string) {
        if (colorName == "yellow") {
            return Color.red + Color.green;
        }
        else if (colorName == "white") {
            return Color.red + Color.green + Color.blue;
        }
        else if (colorName == "magenta") {
            return Color.red + Color.blue;
        }
        else if (colorName == "cyan") {
            return Color.green + Color.blue;
        }
    }
}
```

### 13、装饰器

若要启用实验性的装饰器特性，你必须在命令行或tsconfig.json里启用experimentalDecorators编译器选项：

```shell
tsc --target ES5 --experimentalDecorators
```

```json
//tsconfig.json:
{
    "compilerOptions": {
        "target": "ES5",
        "experimentalDecorators": true
    }
}
```

#### 装饰器

装饰器是一种特殊类型的声明，它能够被附加到类声明，方法，访问符，属性或参数上。装饰器使用 `@expression`这种形式，`expression`求值后必须为一个函数，它会在运行时被调用，被装饰的声明信息做为参数传入。

#### 装饰器组合

多个装饰器可以同时应用到一个声明上:

* 书写在同一行上：
* 书写在多行上

```ts
@f @g x

@f
@g
x
```

在TypeScript里，当多个装饰器应用在一个声明上时会进行如下步骤的操作：

* 由上至下依次对装饰器表达式求值。
* 求值的结果会被当作函数，由下至上依次调用。

#### 装饰器求值

类中不同声明上的装饰器将按以下规定的顺序应用：

* 参数装饰器，然后依次是方法装饰器，访问符装饰器，或属性装饰器应用到每个实例成员。
* 参数装饰器，然后依次是方法装饰器，访问符装饰器，或属性装饰器应用到每个静态成员。
* 参数装饰器应用到构造函数。
* 类装饰器应用到类。

#### 类装饰器

类装饰器在类声明之前被声明（紧靠着类声明）。类装饰器应用于类构造函数，可以用来监视，修改或替换类定义。

类装饰器表达式会在运行时当作函数被调用，类的构造函数作为其唯一的参数。

#### 方法装饰器

方法装饰器声明在一个方法的声明之前（紧靠着方法声明）。 它会被应用到方法的 属性描述符上，可以用来监视，修改或者替换方法定义。

方法装饰器表达式会在运行时当作函数被调用，传入下列3个参数：

* 对于静态成员来说是类的构造函数，对于实例成员是类的原型对象。
* 成员的名字。
* 成员的属性描述符。

#### 访问器装饰器

访问器装饰器声明在一个访问器的声明之前（紧靠着访问器声明）。 访问器装饰器应用于访问器的 属性描述符并且可以用来监视，修改或替换一个访问器的定义

访问器装饰器表达式会在运行时当作函数被调用，传入下列3个参数：

* 对于静态成员来说是类的构造函数，对于实例成员是类的原型对象。
* 成员的名字。
* 成员的属性描述符。

#### 属性装饰器

属性装饰器声明在一个属性声明之前（紧靠着属性声明）。

属性装饰器表达式会在运行时当作函数被调用，传入下列2个参数：

* 对于静态成员来说是类的构造函数，对于实例成员是类的原型对象。
* 成员的名字

#### 参数装饰器

参数装饰器声明在一个参数声明之前（紧靠着参数声明）。 参数装饰器应用于类构造函数或方法声明。

参数装饰器表达式会在运行时当作函数被调用，传入下列3个参数：

* 对于静态成员来说是类的构造函数，对于实例成员是类的原型对象。
* 成员的名字。
* 参数在函数参数列表中的索引。
