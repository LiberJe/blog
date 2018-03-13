# [译]用JavaScript写一个区块链

原文：[Writing a tiny blockchain in JavaScript](https://www.savjee.be/2017/07/Writing-tiny-blockchain-in-JavaScript/)

作者：Savjee.be

译者：JeLewine

几乎每个人都听说过像比特币和以太币这样的加密货币，但是只有极少数人懂得隐藏在它们背后的技术。在这篇博客中，我将会用JavaScript来创建一个简单的区块链来演示它们的内部究竟是如何工作的。我将会称之为SavjeeCoin！

# Part1:实现一个基本的区块链

## 区块链

区块链是由一个个任何人都可以访问的区块构成的公共数据库。这好像没什么特别的，不过它们有一个有趣的属性：它们是不可变的。一旦一个区块被添加到区块链中，除非让剩余的其余区块失效，否则它是不会再被改变的。

这就是为什么加密货币是基于区块链的原因。你肯定不希望人们在交易完成后再变更交易！

## 创造一个区块

区块链是由许许多多的区块链接在一起的（这听上去好像没毛病..）。链上的区块通过某种方式允许我们检测到是否有人操纵了之前的任何区块。

那么我们如何确保数据的完整性呢？每个区块都包含一个基于其内容计算出来的hash。同时也包含了前一个区块的hash。

下面是一个区块类用JavaScript写出来大致的样子：

```
const SHA256 = require("crypto-js/sha256");
class Block {
  constructor(index, timestamp, data, previousHash = '') {
    this.index = index;
    this.previousHash = previousHash;
    this.timestamp = timestamp;
    this.data = data;
    this.hash = this.calculateHash();
  }

  calculateHash() {
    return SHA256(this.index + this.previousHash + this.timestamp + JSON.stringify(this.data)).toString();
  }
}
```

因为JavaScript中并不支持sha256所以我引入了[crypto-js](https://github.com/brix/crypto-js)库。然后我定义了一个构造函数来初始化我区块的属性。每一个区块上都被赋予了`index`属性来告知我们这个区块在整个链上的位置。我们同时也生成了一个时间戳，以及需要在区块里存储的一些数据。最后是前一个区块的hash。

## 创造一个链

现在我们可以在Blockchain类中将区块链接起来了！下面是用JavaScript实现的代码：

```
class Blockchain{
  constructor() {
    this.chain = [this.createGenesisBlock()];
  }

  createGenesisBlock() {
    return new Block(0, "01/01/2017", "Genesis block", "0");
  }

  getLatestBlock() {
    return this.chain[this.chain.length - 1];
  }

  addBlock(newBlock) {
    newBlock.previousHash = this.getLatestBlock().hash;
    newBlock.hash = newBlock.calculateHash();
    this.chain.push(newBlock);
  }

  isChainValid() {
    for (let i = 1; i < this.chain.length; i++){
      const currentBlock = this.chain[i];
      const previousBlock = this.chain[i - 1];

      if (currentBlock.hash !== currentBlock.calculateHash()) {
        return false;
      }

      if (currentBlock.previousHash !== previousBlock.hash) {
        return false;
      }
    }
    return true;
  }
}
```

在构造函数里，我通过创建一个包含创世块的数组来初始化整个链。第一个区块是特殊的，因为它不能指向前一个区块。我还添加了下面两个方法：

* `getLatestBlock()`返回我们区块链上最新的区块。
* `addBlock()`负责将新的区块添加到我们的链上。为此，我们将前一个区块的hash添加到我们新的区块中。这样我们就可以保持整个链的完整性。因为只要我们变更了最新区块的内容，我们就需要重新计算它的hash。当计算完成后，我将把这个区块推进链里（一个数组）。

最后，我创建一个`isChainValid()`来确保没有人篡改过区块链。它会遍历所有的区块来检查每个区块的hash是否正确。它会通过比较`previousHash`来检查每个区块是否指向正确的上一个区块。如果一切都没有问题它会返回`true`否则会返回`false`。

## 使用区块链

我们的区块链类已经写完啦，可以真正的开始使用它了！

```
let savjeeCoin = new Blockchain();
savjeeCoin.addBlock(new Block(1, "20/07/2017", { amount: 4 }));
savjeeCoin.addBlock(new Block(2, "20/07/2017", { amount: 8 }));
```

在这里我仅仅是创建了一个区块链的实例，并且命名它为SavjeeCoin！之后我在链上添加了一些区块。区块里可以包含任何你想要放的数据，不过在上面的代码里，我选择添加了一个带有`amount`属性的对象。

## 试着操作吧！

在介绍里我曾说过区块链是不可变的。一旦添加，区块就不可能再变更了。让我们试一下！

```
// 检查是否有效(将会返回true)
console.log('Blockchain valid? ' + savjeeCoin.isChainValid());

// 现在尝试操作变更数据
savjeeCoin.chain[1].data = { amount: 100 };

// 再次检查是否有效 (将会返回false)
console.log("Blockchain valid? " + savjeeCoin.isChainValid());
```

我会在一开始通过运行`isChainValid()`来验证整个链的完整性。我们操作过任何区块，所以它会返回true。

之后我将链上的第一个（索引为1）区块的数据进行了变更。之后我再次检查整个链的完整性，发现它返回了false。我们的整个链不再有效了。

## 结论

这个小栗子还远未达到完成的程度。它还没有实现POW（工作量证明机制）或P2P网络来与其它矿工来进行交流。

但他确实证明了区块链的工作原理。许多人认为原理会非常复杂，但这篇文章证明了区块链的基本概念是非常容易理解和实现的。

# Part2:实现POW(proof-of-work:工作量证明)

在part1中我们用JavaScript创建了一个简单的区块链来演示区块链的工作原理。不过这个实现并不完整，很多人发现依旧可以篡改该系统。没错！我们的区块链需要另一种机制来抵御攻击。那么让我们来看看我们该如何做到这一点！

## 问题

现在我们可以很快的创造区块然后非常迅速的将它们添加进我们的区块链中。不过这导致了三个问题：

* 第一：人们可以快速创建区块然后在我们的链里塞满垃圾。大量的区块会导致我们区块链过载并让其无法使用。
* 第二：因为创建一个有效的区块太容易了，人们可以篡改链中的某一个区块，然后重新计算所有区块的hash。即使它们已经篡改了区块，他们仍然可以以有效的区块来作为结束。
* 第三：你可以通过结合上述两个破绽来有效控制区块链。区块链由p2p网络驱动，其中节点会将区块添加到可用的最长链中。所以你可以篡改区块，然后计算所有其他的区块，最后添加多任意你想要添加的区块。你最后会得到一个最长的链，所有的其它节点都会接受它然后往上添加自己的区块。

显然我们需要一个方案来解决这些问题：POW。

## 什么是POW

POW是在第一个区块链被创造之前就已经存在的一种机制。这是一项简单的技术，通过一定数量的计算来防止滥用。工作量是防止垃圾填充和篡改的关键。如果它需要大量的算力，那么填充垃圾就不再值得。

比特币通过要求hash以特定0的数目来实现POW。这也被称之为**难度**

不过等一下！一个区块的hash怎么可以改变呢？在比特币的场景下，一个区块包含有各种金融交易信息。我们肯定不希望为了获取正确的hash而混淆了那些数据。

为了解决这个问题，区块链添加了一个`nonce`值。Nonce是用来查找一个有效Hash的次数。而且，因为无法预测hash函数的输出，因此在获得满足难度条件的hash之前，只能大量组合尝试。寻找到一个有效的hash（创建一个新的区块）在圈内称之为挖矿。

在比特币的场景下，POW确保每10分钟只能添加一个区块。你可以想象垃圾填充者需要多大的算力来创造一个新区块，他们很难欺骗网络，更不要说篡改整个链。

## 实现POW

我们该如何实现呢？我们先来修改我们区块类并在其构造函数中添加Nonce变量。我会初始化它并将其值设置为0。

```
constructor(index, timestamp, data, previousHash = '') {
  this.index = index;
  this.previousHash = previousHash;
  this.timestamp = timestamp;
  this.data = data;
  this.hash = this.calculateHash();
  this.nonce = 0;
}
```

我们还需要一个新的方法来增加Nonce，直到我们获得一个有效hash。强调一下，这是由难度决定的。所以我们会收到作为参数的难度。

```
mineBlock(difficulty) {
    while (this.hash.substring(0, difficulty) !== Array(difficulty + 1).join("0")) {
        this.nonce++;
        this.hash = this.calculateHash();
    }
    console.log("BLOCK MINED: " + this.hash);
}
```

最后，我们还需要更改一下`calculateHash()`函数。因为目前他还没有使用Nonce来计算hash。

```
calculateHash() {
  return SHA256(this.index +
    this.previousHash +
    this.timestamp +
    JSON.stringify(this.data) +
    this.nonce
  ).toString();
}
```

将它们结合在一起，你会得到如下所示的区块类：

```
class Block {
  constructor(index, timestamp, data, previousHash = '') {
    this.index = index;
    this.previousHash = previousHash;
    this.timestamp = timestamp;
    this.data = data;
    this.hash = this.calculateHash();
    this.nonce = 0;
  }

  calculateHash() {
    return SHA256(this.index + this.previousHash + this.timestamp + JSON.stringify(this.data) + this.nonce).toString();
  }

  mineBlock(difficulty) {
    while (this.hash.substring(0, difficulty) !== Array(difficulty + 1).join("0")) {
      this.nonce++;
      this.hash = this.calculateHash();
    }
    console.log("BLOCK MINED: " + this.hash);
  }
}
```

## 修改区块链

现在，我们的区块已经拥有Nonce并且可以被开采了，我们还需要确保我们的区块链支持这种新的行为。让我们先在区块链中添加一个新的属性来跟踪整条链的难度。我会将它设置为2（这意味着区块的hash必须以2个0开头）。

```
constructor() {
  this.chain = [this.createGenesisBlock()];
  this.difficulty = 2;
}
```

现在剩下要做的就是改变`addBlock()`方法，以便在将其添加到链中之前确保实际挖到该区块。下面我们将难度传给区块。

```
addBlock(newBlock) {
  newBlock.previousHash = this.getLatestBlock().hash;
  newBlock.mineBlock(this.difficulty);
  this.chain.push(newBlock);
}
```

大功告成！我们的区块链现在拥有了POW来抵御攻击了。

## 测试

现在让我们来测试一下我们的区块链，看看在POW下添加一个新区块会有什么效果。我将会使用之前的代码。我们将创建一个新的区块链实例然后往里添加2个区块。

```
let savjeeCoin = new Blockchain();

console.log('Mining block 1');
savjeeCoin.addBlock(new Block(1, "20/07/2017", { amount: 4 }));

console.log('Mining block 2');
savjeeCoin.addBlock(new Block(2, "20/07/2017", { amount: 8 }));
```

如果你运行了上面的代码，你会发现添加新区块依旧非常快。这是因为目前的难度只有2（或者你的电脑性能非常好）。

如果你创建了一个难度为5的区块链实例，你会发现你的电脑会花费大概十秒钟来挖矿。随着难度的提升，你的防御攻击的保护程度越高。

## 放弃

就像之前说的：这绝不是一个完整的区块链。它仍然缺少很多功能（像P2P网路）。这只是为了说明区块链的工作原理。

并且：由于单线程的原因，用JavaScript来挖矿并不快。

# Part3 交易与挖矿奖励

