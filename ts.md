## 基础类型

* 布尔值

  ````js
  let isDone:boolean = false;
  ````

* 数字

  ````js
  let decLiteral:number = 6;
  ````

* 字符串

  ````js
  let name:string = "bob";
  let sentence:string = `Hello,${name}`;
  ````

* 数组

  ````js
  let list:number[] = [1,2,3];
  let list:Array<number> = [1,2,3]; //数组泛型
  ````

* 元祖Tuple

  ````js
  let x: [string,number];
  x = ['hello', 10]; // ok
  x = [1, 'hello']; // error
  // 当访问越界元素，使用联合类型替代
  x[3] = 'word'; // ok
  x[3] = true ; // error，布尔不是(string|number)类型
  ````

* 枚举

  enum类型是对js标准数据类型的补充，

  ````typescript
  enum Color {Red,Green,Blue}
  let c: Color = Color.Green;
  // 默认从0开始，也可改成从1开始
  enum Color {Red = 1, Green, Blue};
  // 或全部手动赋值
  enum Color {Red = 1, Green = 2, Blue = 4}
  let c: Color = Color.Green;
  // 通过枚举值得到名字
  let colorName: string = Color[2]; // 'Green'
  ````

* Any

  不确定是什么类型，系统通过编译阶段来检查。

  ````typescript
  let notSure: any = 4;
  notSure = "maybe a string instead";
  notSure = false; // okay, definitely a boolean
  // Object赋予任意值，但不能任意调用方法（除非真的有）
  notSure.ifItExists(); // okay, ifItExists might exist at runtime
  notSure.toFixed(); // okay, toFixed exists (but the compiler doesn't check)
  
  let prettySure: Object = 4;
  prettySure.toFixed(); // Error: Property 'toFixed' doesn't exist on type 'Object'.
  
  // 定义数组时，有不同类型的数据
  let list: any[] = [1,true,'free']
  ````

* Void

  没有类型

  ````js
  // 用于函数返回值
  function warnUser(): void {
      console.log('This is my warning message');
  }
  // 用于变量，只能赋予undefined和null
  let unusable: void = undefined;
  ````

* Null和Undefined

  是所有类型的子类型，即可以赋值给其他类型

  但指定了--strictNullChecks标记，就只能赋值给void和它们自己。

  ````typescript
  let u: undefined = undefined;
  let n: null = null;
  ````

* Never

  表不存在值的类型。如抛出异常或不会有返回值的函数

  是任何类型的子类型。

* Object

  `delete function create(o:object|):void;`

* 类型断言

  没有运行时的影响，只是在编译阶段起作用。

  * 尖括号形式

  ````js
  let someValue: any = "this is a string";
  let strLength: number = (<string>someValue).length;
  ````

  * as语法

  ````js
  let someValue: any = "this is a string";
  let strLength: number = (someValue as string).length;
  ````

## 变量声明

* let 

没有变量提升；

不能重复声明；

let会形成块作用域（暂时性死区）

````js
function f(x){
    let x = 100; // interferes with parameter declaration
}
// 在不同的块里声明
function f(condition, x){
    if(condition){
        let x = 100;
        return x;
    }
    return x;
}
````

````js
for(let i=0;i<10;i++){
    setTimeout(function(){console.log(i)},100*i);
    // 1 2 3 4 5 6 7 8 9 10
}
for(var i=0;i<10;i++){
    setTimeout(function(){console.log(i)},100*i);
    // 10 10 10 ....
}
// wrong
for(var i=0;i<10;i++){   
    setTimeout(function(){console.log(i)},100)(i)
}
````

* const 

同let的作用域规则，是对let的增强，能阻止对变量再次赋值；

* var

函数作用域：函数参数也使用函数作用域

for嵌套for 的var i = 0;会被覆盖

* 解构

````js
let {a:newName1,b:newName2} = o;
let {a,b}:{a:string,b:number} = o;

function defaultValue(wholeObject:{a:string,b?:number}){
    let {a, b=1001} = wholeObject;
}
````

* 函数声明

````js
type C = {a:string,b?number};
function f({a,b}:C):void{}
function f({a="",b=0}={}):void{}
function f({a,b=0}={a:""}):void{}
````

小心使用解构，层级较深的时候不使用

## 接口

对值所具有的结构进行类型检查。

````js
// pringLabel 有一个参数名为label类型为string的属性
function printLabel(labelledObj:{label:string}){}
printLabel({size: 10,label:"df"});
````

````typescript
interface LabeledValue{
    label: string; // 必选
    color?: string; // 可选属性
    readonly x: number; // 只读属性，不能被改变了
    [propName:string]: any; // 还有任意数量的属性
}
function printLabel(labelObj:LabeledValue){}
printLabel({size:10,label:'df'});

let a:number[]=[1,2,3,4];
let ro: ReadonlyArray<number> = a;
a = re;// error
// ro不能被改变了。
a = ro as number[]; // right
````

**函数类型**=参数列表+返回值

````typescript
interface SearchFunc {
    (source: string, subString: string):boolean;
}

let mySearch: SearchFunc;
mySearch = function(source: string, subString: string){
    let result = source.search(subString);
    return result > -1;
}
````

* 参数名可不与定义的名字相匹配

````js
mySearch = function(src: string,sub: string): boolean{
    //....
}
````

* 函数的参数 要求对应位置上的参数类型是兼容的。

````typescript
mySearch = function(src,sub){
	//....
}
````

**可索引的类型**

支持两种类型签名：字符串或数字；eg:a[10]或ageMap["daniel"]  

数字索引会自动转成字符串。例如10=>"10"

````typescript
interface StringArray {
    [index: number]: string;
}
let myArray: StringArray = ['Bob','Fred'];
let myStr: string = myArray[0];
````

只读索引

````typescript
interface ReadonlyStringArray {
    readonly [index: number]: string;
}
let myArray: ReadonlyStringArray = ["Alice","Bob"];
myArray[2] = "Mallory"; // error!!
````

````typescript
interface NumberDictionary {
  [index: string]: number;
  length: number;    // 可以，length是number类型
  name: string       // error,应该也是number类型
}
````

**类类型**

* 实现接口

即类的公共部分

````typescript
interface ClockInterface {
    currentTime: Date;
    setTime(d: Date);
}
class Clock implements ClockInterface {
    currentTime: Date;
    setTime(d: Date);
    constructor(h: number,m: number);
}

````

类静态部分与实例部分

当类实现implements另外一个接口时，只对其实例部分进行类型检查

* 继承接口

````typescript
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
````

**混合类型**

````typescript
interface Counter {
    (start: number): string; // 函数
	interval: number; // 属性
	reset(): void; // 
}
function getCounter(): Counter {
    let counter = <Counter> function(start:number){};
    counter.interval = 123;
    counter.reset = function(){};
    return counter;
}
let c = getCounter();
c(10);
c.reset();
c.interval = 5.0;
````

**接口继承类**

当接口继承一个类类型时，会继承成员（包括private和protected）但不包括实现

* not ending

## 类

传统的JavaScript程序使用函数和基于原型的继承来创建可重用的组件

面向对象方式:用的是基于类的继承并且对象是由类构建出来的

子类=派生类；基类=超类

**修饰符**

* public(default)

* private 不能在声明它的类的外部访问（子类不能访问了）

* protected 子类可通过实例方法访问

````typescript
class Person {
    protected name: string;
    protected constructor(theName: string){ this.name = theName; }
}
// Employee 能够继承Person
class Employee extends Person {
    private department: string;
    constructor(name: strubg, department: string){
        super(name);
        this.department = department;
    }
    public getElevatorPitch(){
        return `${this.name}`;
    }
}
let howard = new Employee("aa","bb");
let john = new Person("cc"); // error,'Person'的构造函数是被保护的
````

* readonly 必须在声明时或构造函数里被初始化

````typescript
class Octopus {
    readonly name: string;
    readonly numberOfLefgs: number = 8;
    constructor(theName: string){
        this.name = theName;
    }
}
let obj = new Octopus("aa");
aa.name ="dsaf"; // error: 
````

* 参数属性：即将声明和赋值合并至一处

````typescript
class Octopus {
    // 等价于readonly name: string;private department: string;
    constructor(readonly name: string,private department: string) {}
}
````

**存取器**

ts支持通过getters/setters截取对对象成员的访问，能有效的控制对对象成员的访问

````typescript
// 当密码不对时，会提示我们没有权限去修改员工
let passcode = "secret password";
class Employee {
    private _fullName: string;
    
    get fullName():string{
        return this._fullName;
    }
    
    set fullName(newName: string){
        if(password&&password=="secret password"){
            this.fullName = newName;
        } else {
            console.log("Error:Unauthorized update of employee");
        }
    }
}
````

**静态属性**

静态属性存在于类本身而不是实例上。类名.属性名

类的实例成员：仅当类被实例化的时候才会被初始化的属性。this.属性名



**抽象类——abstract**

作为基类使用，一般不会直接被实例化。

不同于接口，抽象类可包含成员的实现细节。

抽象类中的抽象方法不包含具体实现且必须在派生类中实现。

````typescript
abstract class Department {
    constructor(public name: string){}
    printName(): void {
        console.log('Department name: ' + this.name);
    }
    abstract printMeeting(): void; // 必须在派生类中实现
}

class AccountingDepartment extends Department {
    constructor(){
        super('name'); // 在派生类中的构造函数必须用super
    }
    pringMeeting(): void {
        console.log('pringMeeting');
    }
    generateReports(): void {
        console.log('generateReports')
    }
}

let department: Department; // 允许创建一个抽象类型的引用
department = new Department(); // error: 不能创建一个抽象类的实例
department = new AccountingDepartment(); // 允许对一个抽象子类进行实例化和赋值
department.printName();
department.printMeeting();
````

**高级技巧**

* 构造函数

````typescript
class Greeter {
    greeting: string;
    constructor(message: string){
        this.greeting = message;
    }
    greet(){
        return 'Hello,' + this.greeting;
    }
}
let greeter: Greeter;
greeter = new Greeter('world');
console.log(greeter.greet())

// 等同于
let Greeter = (function(){
    function Greeter(message){
        this.message = message;
    }
    Greeter.prototype.greet = function(){
        return 'Hello,' + this.greeting;
    }
    return Greeter;
})();
let greeter = new Greeter("world");
````

* 把类当做接口使用

类=类的实例类型+构造函数

````typescript
class Point {
    x: number;
    y: number;
}
interface Point3d extends Point {
    z: number;
}
let point3d: Point3d = {x:1,y:2,z:3};
````

## 函数

name function; anoymous function

* 函数类型=参数类型+返回值类型

````typescript
let myAdd: (baseValue: number, increment: number) => number = 
    function(x:number,y:number): number { return x+y };
````

只要参数类型是匹配的，则认为是有效的函数类型，不在乎传参数名是否正确。

返回值可为void

* 必要参数，可选参数，默认参数

````typescript
function fun(A: string,B?:string, C="name",){}
// A: 必传，B可传，当C为undefined或者没有传递该参数时为“name”
````

* 剩余参数

js用arguments来访问所有传入的参数。

````typescript
function buildName(A:string,...reset:string[]){
    return A + " " + reset.join(" ");
}
````

###  this

ts是js的超集。ts能通知错误地使用了this的地方。

* this和箭头函数
* this参数
* this参数在回调函数里

### 重载

js里函数根据传入不同的参数而返回不同类型的数据。

方法是为同一个函数提供多个函数类型定义来进行函数重载。

## 泛型

泛型函数：考虑函数的可重用性。

* 定义泛型函数

**类型变量**，是一种特殊的变量，只用于表示类型而不是值。

````typescript
function identity<T>(arg: T):T {
    return arg;
}
````

以上添加了类型变量T，采纳数类型与返回值类型都是相同的，这允许跟踪函数里使用的类型的信息。

* 使用泛型函数

````typescript
// 第一种： 传入所有的参数，
let output = identity<string>("myString");
// 第二种：用类型推论，即编译器会根据传入的参数自动帮助我们确定T的类型
let output = identity("myString"); // 普遍用法
````

* 使用泛型变量

````typescript
function loggingIdentity<T>(arg: T):T{
    console.log(arg.length); // error，
    return arg;
}
function loggingIdentity<T>(arg: T[]): T[] {
    return arg;
}
// 等价于
function loggingIdentity<T>(arg: Array<T>): Array<T>{
    return arg;
}
````

以上元素类型是T的数组。把泛型变量T当做类型的一部分来使用。

### 泛型类型

函数本身的类型。

````typescript
function identity<T>(arg: T): T {
    return arg;
}
let myIdentity: <T>(arg: T) => T = identity;
// 也可使用不同的泛型参数名。
let myIdentity2: <U>(arg: U) => U = identity;
// 也可使用带有签名的对象字面量来定义泛型函数--引入泛型接口
let myIndentity3: {<T>(arg: T): T} = identity;
````

* 泛型接口

````typescript
interface GenericIdentityFn {
    <T>(arg: T): T;
}
function identity<T>(arg: T): T {
    return arg;
}
let myIdentity: GenericIdentityFn = identity;
````

把泛型参数当做整个接口的一个参数，这样接口里的其他成员能知道这个参数的类型

````typescript
interface GenericIdentityFn<T>{
    (arg: T): T;
}
function identity<T>(arg: T): T {
    return arg;
}
let myIdentity: GenericIdentityFn<number> = identity;

````

* 泛型类

````typescript
class GenericNumber<T> {
    zeroValue: T;
    add: (x:T,y:T) => T;
}
let myGenericNumber = new GenericNumber<number>();
myGenericNumber.zeroValue = 0;
myGenericNumber.add = function(x,y){ return x+y; }
````

类=静态部分和实例部分。

泛型类型指的是实例部分的类型，类的静态属性不能使用这个泛型类型。

### 泛型约束

````typescript
// 定义一个接口来描述约束条件
interface Lenthwise {
    length: number;
}
function loggingIdentity<T extends Lengthwise>(arg: T): T {
    return arg;
}

loggingIdentity(3); // error: number doesn't have a .length property
loggingIdentity({length: 10, value: 3});
````

* 在泛型约束中使用类型参数

````typescript
function getProperty(obj: T, key: K) {
    return obj[key];
}
let x = {a: 1,b:2,c:3,d:4}
getProperty(x,"a");
getProperty(x,"m");
````

* 在泛型里使用类类型

在TypeScript使用泛型创建工厂函数时，需要引用构造函数的类类型。

````typescript
function create<T>(c:{new():T}):T{
    return new c();
}
````

使用原型属性并约束构造函数与类实例的关系

not ending......

````typescript
class Bee
````







## 枚举

枚举：可定义一些带名字的常量，可创建一组有区别的用例。

ts支持数字和基于字符串的枚举。

* 数字枚举-值自动增长。

默认值从0开始。

````typescript
enum Direction {
    Up = 1,
    Down,
    Left,
    Right
}
// 通过枚举的属性访问枚举成员
let message = Direction.Up;
````

* 字符串枚举--每个成员都必须用初始化

* 异构枚举（即混合枚举）-但不建议这么写

````typescript
enum BooleanLikeHeterogeneousEnum {
    No = 0,
    Yes = "YES",
}
````

* 计算的和常量成员
* 联合枚举与枚举成员的类型
* 运行时的枚举
* 反向映射
* 外部枚举

## 类型推论

在没有明确指出类型时，类型推论会帮助提供类型。

这种推断发生在初始化变量和成员，设置默认参数值和决定函数返回值时。

````typescript
let zoo: Animal[] = [new Rhino(), new Elephant(), new Snake()];
// 推断类型的结果为联合数组类型： (Rhino|Elephant|Snake)
````

* 上下文类型

## 类型兼容

类型兼容——即能否赋值

ts结构化类型系统的规则：如果x要兼容y，则y至少具有与x相同的属性

````typescript
interface Named {
    name: string
}
let x: Named;
let y = { name: 'Alice', location: 'Seattle' };
x = y; // right
// 检查x中的每个属性，是否在y中也能找到对应属性
````

* 如何判断两个函数是兼容的

参数，返回值

````typescript

````

### 枚举

不同枚举类型之间是不兼容的。

### 类

类有静态部分和实例部分。

比较两个类类型的对象时，只有实例的成员会被比较。静态成员和构造函数不在比较的范围内。

* 类的私有成员和受保护的成员

若目标类型有一个private或protected成员，则源类型也必须来自同一个类的。

这运行子类赋值给父类，但不能赋值给其它有同源类型的类。

## 泛型

````typescript
nterface Empty<T> {
}
let x: Empty<number>;
let y: Empty<string>;
x = y;  // OK, because y matches structure of x

// 
interface NotEmpty<T> {
    data: T;
}
let x: NotEmpty<number>;
let y: NotEmpty<string>;
x = y;  // Error, because x and y are not compatible
````



## 高级类型

## Symbols
Symbols 是不可改变且唯一的。
````typescript
const getClassNameSymbol = Symbol();
class C {
    [getClassNameSymbol](){
       return "C";
    }
}
let c = new C();
let className = c[getClassNameSymbol](); // "C"

let sym = Symbol();
let obj = {
    [sym]: "value"
};
console.log(obj[sym]); // "value"
````
* Symbol.hasInstance
 识别一个对象是否是其实例
* Symbol.isConcatSpreadable
 布尔值，表示当在一个对象上调用Array.prototype.concat时，这个对象的数组元素是否可展开。
* Symbol.iterator
 被for-of调用，返回对象的默认迭代器
* Symbol.match
* Symbol.replace
* Symbol.search
* Symbol.species
 函数值，为一个构造函数，用来创建派生对象
* Symbol.split
* Symbol.toPrimitive
 把对象转换为相应的原始值
* Symbol.toStringTag
 被内置的Object.prototype.toString调用。返回创建对象时默认的字符串描述
* Symbol.unscopables
 它自己拥有的属性会被with作用于排除在外

## 迭代器和生成器
for..of和for..in语句
都可迭代一个列表，但是用于迭代的值却不同。for...in迭代的是对象的建，for...of迭代的是值

## 模块

## 命名空间

## 命名空间和模块









serverless 云开发 ,端开发； 全栈（BFF）



* type ,interface区别
interface只能是object，class，function;type可以是其他的类型
