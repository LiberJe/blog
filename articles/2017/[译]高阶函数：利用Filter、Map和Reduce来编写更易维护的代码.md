# [译]高阶函数：利用Filter、Map和Reduce来编写更易维护的代码

原文：[Higher Order Functions: Using Filter, Map and Reduce for More Maintainable Code](http://link.zhihu.com/?target=https%3A//medium.freecodecamp.org/higher-order-functions-in-javascript-d9101f9cf528)

作者：Guido Schmitz

译者：JeLewine

高阶函数可以帮助你增强你的JavaScript，让你的代码更具有声明性。简单来说，就是简单，简练，可读。

知道什么时候和怎样使用高阶函数是至关重要的。它们可以让你的代码更容易理解和具有更好的可维护性。它们也可以让你很轻松的进行函数间的组合。我们叫它复合函数，不过我不会在这里进行详细的介绍。在本文中，我将介绍JavaScript中三个最常用的高阶函数：`.filter()`，`.map()`，`.reduce`。

## Filter

想象一下你正在编写一段代码：有一个写满不同人信息的列表，不过你想要过滤出一个大于等于18岁人的列表。

我们的列表看起来就像下面这样：
```
const people = [
 { name: ‘John Doe’, age: 16 },
 { name: ‘Thomas Calls’, age: 19 },
 { name: ‘Liam Smith’, age: 20 },
 { name: ‘Jessy Pinkman’, age: 18 },
];
```
我们先来看看第一个高阶函数是如何筛选出大于等于18岁人的栗子。为了简洁，我将使用ES6标准中的箭头函数。这是一种非常简洁的定义函数的方式，可以让我们不必再写`function`和`return`，以及一些括号、大括号和分号。

```
const peopleAbove18 = (collection) => {
  const results = [];
 
  for (let i = 0; i < collection.length; i++) {
    const person = collection[i];
 
    if (person.age >= 18) {
      results.push(person);
    }
  }
  return results;
};
```
那现在如果我们想要筛选出18~20岁之间的人呢？
```
const peopleBetween18And20 = (collection) => {
  const results = [];
 
  for (let i = 0; i < collection.length; i++) {
    const person = collection[i];
 
    if (person.age >= 18 && person.age <= 20) {
      results.push(person);
    }
  }
  return results;
};
```
你可能已经意识到了这里有许多重复的代码。我们可以抽象出一个通用的解决方案。这两个函数有一些共同点。它们都在一个列表中进行迭代，并且在给定的条件下进行过滤。
>"高阶函数是一个将一个或多个函数作为参数的函数"——ClojureBridge

我们可以通过使用更具声明性的`.filter()`方法来改进我们之前的函数。
```
const peopleAbove18 = (collection) => {
  return collection
    .filter((person) => person.age >= 18);
}
```
太棒了!我们通过使用高阶函数减少了许多额外的代码。同时也让我们的代码更具可读性。我们不在乎如何过滤东西，我们只是希望它被过滤。这篇文章稍后会介绍组合函数。

## Map

让我们拿着刚刚的人员名单和一个其中喜欢喝咖啡的人员名单。
```
const coffeeLovers = [‘John Doe’, ‘Liam Smith’, ‘Jessy Pinkman’];
```
用命令式的实现方式就像下面这样：
```
const addCoffeeLoverValue = (collection) => {
  const results = [];
 
  for (let i = 0; i < collection.length; i++) {
    const person = collection[i];
    if (coffeeLovers.includes(person.name)) {
      person.coffeeLover = true;
    } else {
      person.coffeeLover = false;
    }
 
    results.push(person);
  }
 
  return results;
};
```
我们可以利用`.map()`来让代码更具有声明性：
```
const incrementAge = (collection) => {
  return collection.map((person) => {
    person.coffeeLover = coffeeLovers.includes(person.name);
 
    return person;
  });
};
```
再说一遍，`.map()`是一个高阶函数。它允许我们将一个函数作为参数传递。

## Reduce

我敢打赌，当你知道什么时候和怎样使用它的时候，你会喜欢上这个函数。`.reduce()`很酷，上面提到的的大部分函数都可以通过它来实现。

让我们先举一个简单的栗子。我们想计算所有人年龄的和。当然了，我们还是会首先看看如何用命令式的方式实现。它基本上就是通过循环来增加总年龄变量。
```
const sumAge = (collection) => {
  let num = 0;
 
  collection.forEach((person) => {
    num += person.age;
  });
 
  return num;
}
```
接下来是使用`.reduce()`的声明式方法：
```
const sumAge = (collection) => collection.reduce((sum, person) => {
 return sum + person.age;
}, 0);
```
我们甚至可以使用`.reduce()`来创建我们自己的`.map()`和`.filter()`。
```
const map = (collection, fn) => {
  return collection.reduce((acc, item) => {
    return acc.concat(fn(item));
  }, []);
}
const filter = (collection, fn) => {
  return collection.reduce((acc, item) => {
    if (fn(item)) {
      return acc.concat(item);
    }
 
    return acc;
  }, []);
}
```
一开始这一块儿可能比较难理解。不过，`.reduce()`做的基本上就是以一个集合和一个定义了初始值的变量开始。然后，遍历该集合并将值添加到变量中去。

## 组合map，filter和reduce

太好了，这些函数我们都有了。而且很重要的一点是，他们都存在于JavaScript的数组原型上。这意味着我们可以同时使用它们。这可以让我们轻松创建各种可复用的函数，减少编写某些功能所需要的代码量。

我们已经讨论过了如何利用`.filter()`来过滤出大于等于18岁的人；利用`.map()`来添加`coffeeLover`属性；通过`.reduce()`来计算所有人年龄的和。现在，我们写一点代码将这三个步骤合并起来。
```
const people = [
 { name: ‘John Doe’, age: 16 },
 { name: ‘Thomas Calls’, age: 19 },
 { name: ‘Liam Smith’, age: 20 },
 { name: ‘Jessy Pinkman’, age: 18 },
];
const coffeeLovers = [‘John Doe’, ‘Liam Smith’, ‘Jessy Pinkman’];
const ageAbove18 = (person) => person.age >= 18;
const addCoffeeLoverProperty = (person) => {
 person.coffeeLover = coffeeLovers.includes(person.name);
 
 return person;
}
const ageReducer = (sum, person) => {
 return sum + person.age;
}, 0);
const coffeeLoversAbove18 = people
 .filter(ageAbove18)
 .map(addCoffeeLoverProperty);
const totalAgeOfCoffeeLoversAbove18 = coffeeLoversAbove18
 .reduce(ageReducer);
const totalAge = people
 .reduce(ageReducer);
```
如果你用命令式方法的话，你最后会写一堆重复代码。

通过`.map()`，`.reduce()`和`.filter()`来创建函数的思维将会极大的提高你的代码质量。而且可以增加可读性。你根本不必在意函数内到底发生了什么，它非常容易理解。

***2017.08.28***