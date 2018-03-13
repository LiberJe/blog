# [译]ES6的工厂函数

原文：[JavaScript Factory Functions with ES6+](https://medium.com/javascript-scene/javascript-factory-functions-with-es6-4d224591a8b1)

作者：Eric Elliott

译者：JeLewine


工厂函数是一个最后返回值是对象的函数，但它既不是类，也不是构造函数。在JavaScript中，任何函数都可以返回一个对象。但当函数没有使用new关键字时，那它便是一个工厂函数。

由于工厂函数提供了让我们轻松创建对象实例的能力，而且还不需要深入学习类和`new`关键字的复杂性。因此工厂函数在JavaScript中是非常具有吸引力的

JavaScript提供了非常方便的对象字面量语法。就像下面这样：
```
const user = {
  userName: 'echo',
  avatar: 'echo.png'
};
```
这很像是JSON，`:`左边是属性名，右边是属性值。你可以轻松的使用点符号来访问属性：
```
console.log(user.userName); // "echo"
```
你也可以通过方括号语法来访问属性：
```
const key = 'avatar';
console.log( user[key] ); // "echo.png"
```
如果在作用域内还有变量和你想要创建的对象属性名相同，那你也可以直接使用这一变量来创建对象字面量：
```
const userName = 'echo';
const avatar = 'echo.png';
const user = {
  userName,
  avatar
};
console.log(user);
// { "avatar": "echo.png",   "userName": "echo" }
```
对象字面量有着简单明了的函数语法。我们可以给对象添加一个`.setUserName()`方法：
```
const userName = 'echo';
const avatar = 'echo.png';
const user = {
  userName,
  avatar,
  setUserName (userName) {
    this.userName = userName;
    return this;
  }
};
console.log(user.setUserName('Foo').userName); // "Foo"
```
在这个方法中，`this`指向调用这个方法的对象。想要在一个对象中调用方法，只需要使用对象的点语法或者利用方括号语法就行了。像是`game.play()`将会在`game`中调用`play()`。不过使用点语法来进行调用是有前提的，就是这个方法必须是这个对象的属性。不过你也可以利用`.call()`，`.apply()`，`.bind()`来将一个方法应用在任意对象上。

在这个例子中，`user.setUserName('Foo')`是在`user`对象上调用`.setUserName()`，所以`this === user`。在`.setUserName()`方法中，通过`this`绑定，我们修改了`user`对象上的`.userName`属性。同时为了方便链式调用，它返回了相同的一个对象实例。

## 字面量语法针对单一对象，工厂函数更适合多对象创建
如果你需要创建许多对象，我觉得你会十分想把对象字面量创建与工厂函数结合起来。

只要你想，你可以用工厂函数创建任意多的`user`对象。举个栗子，如果你正在开发一个聊天app，你就可以创建一个对象来代表当前的用户。你同时还可以创建很多其它对象来代表其他那些已经登陆或者在聊天的用户，以方便来展示他们的名字和头像。

让我们把我们的`user`对象用`createUser()`工厂函数造出来：
```
const createUser = ({ userName, avatar }) => ({
  userName,
  avatar,
  setUserName (userName) {
    this.userName = userName;
    return this;
  }
});
console.log(createUser({ userName: 'echo', avatar: 'echo.png' }));
/*
{
  "avatar": "echo.png",
  "userName": "echo",
  "setUserName": [Function setUserName]
}
*/
```
## 返回对象
箭头函数（`=>`）具有隐式返回的特性。如果某个函数体只有单个表达式，你就可以忽略`return`关键字：`() => foo`是一个不需要参数，而且最后会返回字符串`'foo'`的的函数。

不过需要注意的是，当你想要返回一个对象字面量的时候，如果你使用了大括号，JavaScript会默认你想要创建一个函数体。像是`{ broken: true }`。如果你想要通过隐式返回来返回一个字面量对象，那你就需要在你的字面量对象外面包裹一层小括号来消除这种歧义：
```
const noop = () => { foo: 'bar' };
console.log(noop()); // undefined
const createFoo = () => ({ foo: 'bar' });
console.log(createFoo()); // { foo: "bar" }
```
在第一个栗子中，`foo:`会被JavaScript理解成一个标签，而`bar`会被理解成一个没有被赋值的表达式。这个函数会返回`undefined`。

而在`createFoo()`的栗子中，圆括号强制让大括号里的内容被解释为一个需要被计算的表达式，而不是一个函数体。

## 解构
需要特别注意一下函数的声明：
```
const createUser = ({ userName, avatar }) => ({
```
在这一行中，大括号（`{`,`}`）代表了对象的解构。这个函数接受一个参数（一个对象），但是从这个单一对象中又解构出了两个形参，`userName`和`avatar`。这些参数都可以被当作函数体作用域内的变量使用。你同样也可以解构一些数组：
```
const swap = ([first, second]) => [second, first];
console.log( swap([1, 2]) ); // [2, 1]
```
你也可以使用拓展运算符（`...varName`）来获取数组（或者参数列表）中的其它值，然后将这些数组元素回传成单个元素：
```
const rotate = ([first, ...rest]) => [...rest, first];
console.log( rotate([1, 2, 3]) ); // [2, 3, 1]
```
## 计算属性值
在前面我们曾通过方括号语法来动态的访问对象的属性：
```
const key = 'avatar';
console.log( user[key] ); // "echo.png"
```
我们也可以将计算到的属性值指定给某些对象：
```
const arrToObj = ([key, value]) => ({ [key]: value });
console.log( arrToObj([ 'foo', 'bar' ]) ); // { "foo": "bar" }
```
在这个栗子里，`arrToObj`将一个包含键值对（也叫元组）的数组转换成了一个对象。因为我们不知道键的名称，所以我们需要通过计算属性名来在对象中设置键值对。为此，我们借用了计算属性中方括号语法的思路来重建我们的对象字面量。
```
{ [key]: value }
```
在语句解析完成后，我们就得到了我们最终的对象：
```
{ "foo": "bar" }
```
## 默认参数
JavaScript函数支持使用默认值，这带来了许多好处：
1. 开发者可以通过合适的默认值来省略参数。
2. 默认值提供了期望输入，提高了函数自身的可读性。
3. IDE和静态检测可以通过默认值来推测参数的类型。举个栗子，默认值为`1`表明参数可能是Number类型的。

使用默认参数，就好像是给我们的`createUser`工厂函数提供了一份期待接口文档。如果user没有提供任何信息，函数就会自动设为默认值`Anonymous`。
```
const createUser = ({
  userName = 'Anonymous',
  avatar = 'anon.png'
} = {}) => ({
  userName,
  avatar
});
console.log(
  // { userName: "echo", avatar: 'anon.png' }
  createUser({ userName: 'echo' }),
  // { userName: "Anonymous", avatar: 'anon.png' }
  createUser()
);
```
函数定义的最后一部分看起来有点意思：
```
} = {}) => ({
```
在传参结束之前的最后的这部分`={}`用于表示：如果没有传入任何参数，那么将使用一个空对象作为默认值传入函数体。当你尝试从空对象中解构对象时，将会自动使用属性的默认值。因为这就是默认值干的事：用预设的值（空对象）来替换`undefined`。

如果没有`={}`这部分，`createUser()`就会报错。因为你无法访问`undefined`的属性值。

#### 类型判断
在我还在写这篇文章的时候，JavaScript 还没有任何原生的类型注释。但是近几年涌现了一批工具填补这一空白，包括JSDoc（由于出现了更好的选择，所以它现在呈现出了下降趋势），Facebook的Flow，还有Microsoft的TypeScript。我个人比较喜欢rtype，因为我觉得它在函数式编程方面比TypeScript拥有更好的可读性。

至少到我发文的时候，类型注释好像还没有一个明确的赢家。没有一个获得了JavaScript规范的支持，而且每一个选项似乎都有着较为明显的缺点。

类型判断是基于我们使用的变量上下文来判断类型的过程。在JavaScript中，类型注释是一个非常好的选择。

如果你可以在标准的JavaScript函数签名中提供足够多的线索，那么你就可以获得类型注释的绝大部分好处，而不用担心什么代价和风险。

即使你打算使用类似typescript或flow这样的工具，也应该尽可能的带上类型注释。这样可以减少一些情况下发生的强类型判断。比方说，原生JavaScript是不支持定义接口的。但是使用typescript和flow都可以方便的定义接口。

Tern.js是一个流行的JavaScript类型判断工具，它在很多代码编辑器或IDE上都有插件。

微软的VS Code并不需要Tern，因为它已经把TypeScript的类型判断功能加到了普通的JavaScript代码中去了。

当你在JavaScript函数中指定了默认参数值后，很多类型判断工具就已经可以在IDE中给予你提示，来帮助正确的使用API了。

没有默认值，各种IDE（更多时候，甚至连我们自己）没有足够的信息来判断函数预期的参数类型。

![没有默认值，userName的类型是未知的](http://upload-images.jianshu.io/upload_images/1801991-6cab638bd1b19941.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

通过默认值，IDE就可以显示`userName`预计的输入是一个字符串。

![有了默认值，userName的类型是string](http://upload-images.jianshu.io/upload_images/1801991-9ec1c2cdcbbb3053.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

将函数参数限制为固定类型（这会使通用函数和高阶函数更加受限）并不总是合理的。但在它合理的时候，使用默认参数通常就是最佳的方式。即便你已经在使用TypeScript或Flow做类型判断了。

## Mixin结构的工厂函数
工厂函数擅长利用封装好的API来创建对象。通常来说，这已经足够了。但是不久你就会发现，你总是需要将许多相似的功能构筑到不同类型的对象中去。你会想要把这些功能抽象为mixin函数，来进行重用。

这正是mixin函数将要大显身手的地方。我们来创建一个`withConstructor`mixin函数，把`.constructor`属性添加到所有对象实例当中去。
`with-constructor.js`
```
const withConstructor = constructor => o => {
  const proto = Object.assign({},
    Object.getPrototypeOf(o),
    { constructor }
  );
  return Object.assign(Object.create(proto), o);
};
```
现在你可以在其它mixin函数中使用它了
```
import withConstructor from `./with-constructor';
const pipe = (...fns) => x => fns.reduce((y, f) => f(y), x);
// or `import pipe from 'lodash/fp/flow';`
// Set up some functional mixins
const withFlying = o => {
  let isFlying = false;
  return {
    ...o,
    fly () {
      isFlying = true;
      return this;
    },
    land () {
      isFlying = false;
      return this;
    },
    isFlying: () => isFlying
  }
};
const withBattery = ({ capacity }) => o => {
  let percentCharged = 100;
  return {
    ...o,
    draw (percent) {
      const remaining = percentCharged - percent;
      percentCharged = remaining > 0 ? remaining : 0;
      return this;
    },
    getCharge: () => percentCharged,
    get capacity () {
      return capacity
    }
  };
};
const createDrone = ({ capacity = '3000mAh' }) => pipe(
  withFlying,
  withBattery({ capacity }),
  withConstructor(createDrone)
)({});
const myDrone = createDrone({ capacity: '5500mAh' });
console.log(`
  can fly:  ${ myDrone.fly().isFlying() === true }
  can land: ${ myDrone.land().isFlying() === false }
  battery capacity: ${ myDrone.capacity }
  battery status: ${ myDrone.draw(50).getCharge() }%
  battery drained: ${ myDrone.draw(75).getCharge() }%
`);
console.log(`
  constructor linked: ${ myDrone.constructor === createDrone }
`);
```

如你所见，可复用的`withConstructor()`mixin非常轻松的和其它mixin一起放进pipeline中。`withBattery()`可以被用在其它各种类型的对象上：机器人、电动滑板或是充电宝等等。`withFlying()`也可以被用在飞行模型、火箭和热气球身上。

对象组合更像是一种思维方式，而不是一种简单的编程技巧。你可以用各种各样的方式来实现它。而函数组合正是从头开始构建这种思维方式的最简单的方法。工厂函数是最简单的能够将实现细节封装成对外友好API的方法。

## 结论

ES6提供了非常方便的语法来处理对象创建和工厂函数。在大多数情况下，这已经可以满足你绝大多数的需求。不过还有一种更像Java的方式：利用`class`关键字。

在JavaScript中，类比工厂模式更加的冗余和受限。而且如果要涉及重构的话，这更像是一块儿雷区。不过类也被当前的一些像是React和Angular的主流前端框架所接受。还有一些其它的罕见情况也让类的存在更加有意义。

> “有时候，最优雅的实现仅仅需要一个函数。不是类，不是方法，也不是框架。仅仅是一个函数。” ~ John Carmack

从最简单的实现开始，根据需要去改变成更加复杂的实现方式。当涉及到对象时，整个变化过程看起来就像这样：

```
Pure function -> factory -> functional mixin -> class
```

***2017.08.14***