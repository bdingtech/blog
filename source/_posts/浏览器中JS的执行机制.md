---
title: 浏览器中JS的执行机制
date: 2020-06-04 21:40:54
category: 
- 前端
tags:
- JavaScript
---

## 前言

刚入门的时候，在平时的业务代码实践中，经常会遇到各种稀奇古怪的问题，比如下面这样，结果是全部输出 10。（你也不用手动执行了，我把执行结果都贴下面了，够良心了，捂脸）

```javascript

var arr = []
for(var i = 0; i < 10; i++){
	arr.push(()=>{
    console.log(i)
  })
}
arr.map( temp => {
  temp()
})

```

![image-20200604215830483](https://img-1251598303.cos.ap-guangzhou.myqcloud.com/image-20200604215830483.png)

又比如我们在写嵌套函数或者回调的时候，恐怕脑袋里全都是问号，bar 中的this不应该指向showThis吗？为什么指向的是window？为什么和我想的不一样？？？

```javascript

var myObj = {
  name : "极客时间", 
  showThis: function(){
    console.log(this)
    function bar(){
      console.log(this)
    }
    bar()
  }
}
myObj.showThis()

```

![image-20200604215721592](https://img-1251598303.cos.ap-guangzhou.myqcloud.com/image-20200604215721592.png)

---

这一切的一切，根本原因都只有一个字

![28_8a1dd6cfa10b5fcb7370d5fac4feac2b](https://img-1251598303.cos.ap-guangzhou.myqcloud.com/28_8a1dd6cfa10b5fcb7370d5fac4feac2b.png)

是的，现实就是这么残酷，作为初学者的我们，最开始入门的是大学开的 C 语言课，碰上JavaScript这么骚气语言，想不被坑都难，所以啊，要想少被坑，只能好好学习这门语言的执行机制，正所谓```知其然，更知其所以然```，或者``` 知己知彼，方能百战不殆```，反正就是这么个意思，老老实实去学学更深层次的东西，不仅能少被坑，而且还能够提升自己的核心价值，加油，奥利给！

## 正文

接下来我们就一起来谈谈这些磨人的小妖精到底是个什么东东

- 变量提升
- 执行期上下文
  - 词法环境
  - 变量环境
- 调用栈
- 作用域
- 作用域链
- 闭包
- This

