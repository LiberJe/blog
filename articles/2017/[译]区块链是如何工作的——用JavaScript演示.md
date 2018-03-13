# [译]区块链是如何工作的——用JavaScript演示

原文：[How does blockchain really work? I built an app to show you.](https://medium.freecodecamp.org/how-does-blockchain-really-work-i-built-an-app-to-show-you-6b70cd4caf7d)

作者：[Sean Han](https://medium.freecodecamp.org/@seanseany?source=post_header_lockup)

译者：[JeLewine](http://weibo.com/jly199518)

根据维基百科，区块链是：
>一个用于维护不断增长的记录列表的分布式数据库，我们称之为区块链。

这听起来很棒，那它是如何工作的呢？

为了说明区块链，我们将会使用一个名为Blockchain CLI的开源命令行工具。

我同时也建立了一个[基于浏览器的版本](http://blockchaindemo.io/)

![](http://upload-images.jianshu.io/upload_images/1801991-a4bee1056c11a8dd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 安装命令行工具

在此之前请先安装[Node.js](https://nodejs.org/download/)

然后在你的命令行中运行以下指令：
```
npm install blockchain-cli -g
blockchain
```
你应该会看到`👋 Welcome to Blockchain CLI!`和一个`blockchain →`提示。这说明已经准备好了。

## 区块长什么样？

想要查看当前的区块链，你需要在命令提示行下输入`blockchain`或者`bc`。你应该会看到像下面的图片一样的一个区块。
![区块链上的一个区块](http://upload-images.jianshu.io/upload_images/1801991-51833bb52c24d450.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* Index：是哪一个区块（创世块的索引是0）？
* Hash：块是否有效？
* Previous Hash：前一个区块是否有效？
* Timestamp：什么时候添加的区块？
* Data：什么信息存储在区块上？
* Nonce：在找到有效区块之前，我们进行了多少次迭代？

#### 创世块

每一个区块链都是从`🏆 Genesis Block`开始的。正如你们将要在后面看到的，区块链上的每一个区块都依赖于前一个区块。所以，需要创世块来挖出我们的第一个区块。

## 当一个新的区块被开采时会发生什么？

![](http://upload-images.jianshu.io/upload_images/1801991-c4f77848b208bcc3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
让我们挖出我们的第一个区块。在命令行中输入`mine freeCodeCamp♥︎`。

区块链查看链上最新的区块来获取`index`和`previous hash`。在这个案例下创世块是最新的区块。

* Index：0+1=1
* Previous Hash：0000018035a828da0…
* Timestamp：区块被添加的时间
* Data：freeCodeCamp❤
* Hash：？？？
* Nonce：？？？

## Hash是如何计算的？

哈希值是唯一标识数据的固定长度的数值。

Hash是通过将`Index`、`Previous Hash`、`Timestamp`、`Data`和`Nonce`作为输入值来计算的。
```
CryptoJS.SHA256(index + previousHash + timestamp + data + nonce)
```
SHA256算法将会依据这些输入计算出一个唯一Hash值。同样的输入总是会返回同样的结果。

#### 你是否注意到区块Hash中的四个前导0？

四个前导0是一个有效Hash的最低要求。所需前导0的数量被称之为**难度**
```
function isValidHashDifficulty(hash, difficulty) {
  for (var i = 0, b = hash.length; i < b; i ++) {
      if (hash[i] !== '0') {
          break;
      }
  }
  return i >= difficulty;
}
```
这也被称为[工作证明系统](https://en.wikipedia.org/wiki/Proof-of-work_system)

## Nonce是什么？

Nonce是用来查找一个有效Hash的次数。
```
let nonce = 0;
let hash;
let input;
while(!isValidHashDifficulty(hash)) {     
  nonce = nonce + 1;
  input = index + previousHash + timestamp + data + nonce;
  hash = CryptoJS.SHA256(input)
}
```
Nonce迭代到直到Hash有效。在我们的案例中，一个有效的Hash至少要拥有4个前置0。查找与有效Hash对应的Nonce的过程就是挖矿。

随着**难度**的增加，可能的有效Hash数量就会减少。伴随着有效Hash的减少，我们需要更强的算力来查找有效Hash。

## 为什么这么重要？

这些机制非常重要，它们使区块链不可变。

如果我们有这么一个区块链“A->B->C”，而且有一个人想要改变区块A上的数据。那么会发生什么呢？

1. 区块A上的数据改变了。
2. 区块A的hash改变了，因为数据被用来计算hash。
3. 区块A失效了，因为它的hash不再有4个前导0。
4. 区块B的hash改变了，因为区块A的hash被用来计算区块B的hash。
5. 区块B失效了，因为它的hash不再有4个前导0。
6. 区块B的hash改变了，因为区块C的hash被用来计算区块B的hash。
7. 区块C失效了，因为它的hash不再有4个前导0。

**改变一个区块的唯一方法就是将这个区块重新挖一遍，接下来是所有的区块。由于总是有新的区块被添加，因此改变区块几乎是一件不可能的事。**

我希望这个教程能够对您有所帮助！

如果您想要查看网页版的演示，请出门右转[http://blockchaindemo.io](http://blockchaindemo.io/)

***2017.09.11***