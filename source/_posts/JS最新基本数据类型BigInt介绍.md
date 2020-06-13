---
title: JS最新基本数据类型BigInt介绍
date: 2020-06-12 18:07:21
tags: 
- 
---

**为了保证的可读性，本文采用意译而非直译。**

`BigInt`数据类型的目的是比`Number`数据类型支持的范围更大的整数值。在对大整数执行数学运算时，以任意精度表示整数的能力尤为重要。使用`BigInt`，整数溢出将不再是问题。

此外，可以安全地使用更加准确时间戳，大整数ID等，而无需使用变通方法。 `BigInt`目前是第3阶段提案， 一旦添加到规范中，它就是JS 第二个数字数据类型，也将是 JS 第8种基本数据类型：

- Boolean
- Null
- Undefined
- Number
- BigInt
- String
- Symbol
- Object

在本文中，咱们将详细介绍`BigInt`，看看它如何解决使用`Number`类型的限制。

## 问题

对于学过其他语言的程序员来说，JS中缺少显式整数类型常常令人困惑。许多编程语言支持多种数字类型，如浮点型、双精度型、整数型和双精度型，但JS却不是这样。在JS中，按照[IEEE 754-2008](https://en.wikipedia.org/wiki/IEEE_754-2008_revision)标准的定义，所有数字都以[双精度64位浮点](http://en.wikipedia.org/wiki/Double_precision_floating-point_format)格式表示。

在此标准下，无法精确表示的非常大的整数将自动四舍五入。确切地说，JS 中的`Number`类型只能安全地表示`-9007199254740991 (-(2^53-1))` 和`9007199254740991(2^53-1)`之间的整数，任何超出此范围的整数值都可能失去精度。

```
console.log(9999999999999999);    // → 10000000000000000
```

该整数大于JS Number 类型所能表示的最大整数，因此，它被四舍五入的。意外四舍五入会损害程序的可靠性和安全性。这是另一个例子：

```
// 注意最后一位的数字
9007199254740992 === 9007199254740993;    // → true
```

JS 提供`Number.MAX_SAFE_INTEGER`常量来表示 最大安全整数，`Number.MIN_SAFE_INTEGER`常量表示最小安全整数：

```
const minInt = Number.MIN_SAFE_INTEGER;

console.log(minInt);         // → -9007199254740991

console.log(minInt - 5);     // → -9007199254740996

// notice how this outputs the same value as above
console.log(minInt - 4);     // → -9007199254740996
```

## 解决方案

为了解决这些限制，一些JS开发人员使用字符串类型表示大整数。 例如，[Twitter API](https://developer.twitter.com/en/docs/basics/twitter-ids) 在使用 JSON 进行响应时会向对象添加字符串版本的 ID。 此外，还开发了许多库，例如 [bignumber.js](https://github.com/MikeMcl/bignumber.js/)，以便更容易地处理大整数。

使用BigInt，应用程序不再需要变通方法或库来安全地表示`Number.MAX_SAFE_INTEGER`和`Number.Min_SAFE_INTEGER`之外的整数。 现在可以在标准JS中执行对大整数的算术运算，而不会有精度损失的风险。

要创建`BigInt`，只需在整数的末尾追加n即可。比较:

```
console.log(9007199254740995n);    // → 9007199254740995n
console.log(9007199254740995);     // → 9007199254740996
```

或者，可以调用`BigInt()`构造函数

```
BigInt("9007199254740995");    // → 9007199254740995n
```

`BigInt`文字也可以用二进制、八进制或十六进制表示

```
// binary
console.log(0b100000000000000000000000000000000000000000000000000011n);
// → 9007199254740995n

// hex
console.log(0x20000000000003n);
// → 9007199254740995n

// octal
console.log(0o400000000000000003n);
// → 9007199254740995n

// note that legacy octal syntax is not supported
console.log(0400000000000000003n);
// → SyntaxError
```

请记住，不能使用严格相等运算符将`BigInt`与常规数字进行比较，因为它们的类型不同：

```
console.log(10n === 10);    // → false

console.log(typeof 10n);    // → bigint
console.log(typeof 10);     // → number
```

相反，可以使用等号运算符，它在处理操作数之前执行隐式类型转换

```
console.log(10n == 10);    // → true
```

除一元加号(`+`)运算符外，所有算术运算符都可用于`BigInt`

```
10n + 20n;    // → 30n
10n - 20n;    // → -10n
+10n;         // → TypeError: Cannot convert a BigInt value to a number
-10n;         // → -10n
10n * 20n;    // → 200n
20n / 10n;    // → 2n
23n % 10n;    // → 3n
10n ** 3n;    // → 1000n

const x = 10n;
++x;          // → 11n
--x;          // → 9n
```

不支持一元加号（`+`）运算符的原因是某些程序可能依赖于`+`始终生成`Number`的不变量，或者抛出异常。 更改`+`的行为也会破坏`asm.js`代码。

当然，与`BigInt`操作数一起使用时，算术运算符应该返回`BigInt`值。因此，除法(`/`)运算符的结果会自动向下舍入到最接近的整数。例如:

```
25 / 10;      // → 2.5
25n / 10n;    // → 2n
```

## 隐式类型转换

因为隐式类型转换可能丢失信息，所以不允许在`bigint`和 `Number` 之间进行混合操作。当混合使用大整数和浮点数时，结果值可能无法由`BigInt`或`Number`精确表示。思考下面的例子：

```
(9007199254740992n + 1n) + 0.5
```

这个表达式的结果超出了`BigInt`和`Number`的范围。小数部分的`Number`不能精确地转换为`BigInt`。大于`2^53`的`BigInt`不能准确地转换为数字。

由于这个限制，不可能对混合使用`Number`和`BigInt`操作数执行算术操作。还不能将`BigInt`传递给Web api和内置的 JS 函数，这些函数需要一个 `Number` 类型的数字。尝试这样做会报`TypeError`错误

```
10 + 10n;    // → TypeError
Math.max(2n, 4n, 6n);    // → TypeError
```

**请注意**，关系运算符不遵循此规则，如下例所示：

```
10n > 5;    // → true
```

如果希望使用`BigInt`和`Number`执行算术计算，首先需要确定应该在哪个类型中执行该操作。为此，只需通过调用`Number()`或`BigInt()`来转换操作数：

```
BigInt(10) + 10n;    // → 20n
// or
10 + Number(10n);    // → 20
```

当 `Boolean` 类型与`BigInt` 类型相遇时，`BigInt`的处理方式与`Number`类似，换句话说，只要不是`0n`，`BigInt`就被视为`truthy`的值：

```
if (5n) {
    // 这里代码块将被执行
}

if (0n) {
    // 这里代码块不会执行
}
```

排序`BigInts`和`Numbers`数组时，不会发生隐式类型转换：

```
const arr = [3n, 4, 2, 1n, 0, -1n];

arr.sort();    // → [-1n, 0, 1n, 2, 3n, 4]
```

位操作符如`|、&、<<、>>`和`^`对`Bigint`的操作方式与`Number`类似。下面是一些例子

```
90 | 115;      // → 123
90n | 115n;    // → 123n
90n | 115;     // → TypeError
```

## BigInt构造函数

与其他基本类型一样，可以使用构造函数创建`BigInt`。传递给`BigInt()`的参数将自动转换为`BigInt`:

```
BigInt("10");    // → 10n
BigInt(10);      // → 10n
BigInt(true);    // → 1n
```

无法转换的数据类型和值会引发异常:

```
BigInt(10.2);     // → RangeError
BigInt(null);     // → TypeError
BigInt("abc");    // → SyntaxError
```

可以直接对使用构造函数创建的`BigInt`执行算术操作

```
BigInt(10) * 10n;    // → 100n
```

使用严格相等运算符的操作数时，使用构造函数创建的`Bigint`与常规`Bigint`的处理方式类似

```
BigInt(true) === 1n;    // → true
```

## 库函数

在撰写本文时，`Chrome +67` 和`Opera +54`完全支持`BigInt`数据类型。不幸的是，`Edge`和`Safari`还没有实现它。`Firefox`默认不支持BigInt，但是可以在`about:config`中将`javascript.options.bigint` 设置为`true`来开启它，最新支持的情况可在“[Can I use](https://caniuse.com/#search=bigint)”上查看。

不幸的是，转换`BigInt`是一个极其复杂的过程，这会导致严重的运行时性能损失。直接polyfill `BigInt`也是不可能的，因为该提议改变了几个现有操作符的行为。目前，更好的选择是使用[JSBI](https://github.com/GoogleChromeLabs/jsbi)库，它是`BigInt`提案的纯JS实现。

这个库提供了一个与原生`BigInt`行为完全相同的API。下面是如何使用JSBI：

```
import JSBI from './jsbi.mjs';

const b1 = JSBI.BigInt(Number.MAX_SAFE_INTEGER);
const b2 = JSBI.BigInt('10');

const result = JSBI.add(b1, b2);

console.log(String(result));    // → '9007199254741001'
```

使用`JSBI`的一个优点是，一旦浏览器支持，就不需要重写代码。 相反，可以使用`babel`插件自动将JSBI代码编译为原生 `BigInt`代码。

## 总结

`BigInt`是一种新的数据类型，用于当整数值大于`Number`数据类型支持的范围时。这种数据类型允许我们安全地对大整数执行算术操作，表示高分辨率的时间戳，使用大整数id，等等，而不需要使用库。

重要的是要记住，不能使用`Number`和`BigInt`操作数的混合执行算术运算，需要通过显式转换其中的一种类型。 此外，出于兼容性原因，不允许在`BigInt`上使用一元加号（`+`）运算符。