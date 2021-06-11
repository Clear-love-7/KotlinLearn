## 高阶函数与 lambda 表达式

### 高阶函数

> 将函数用作参数或返回值的函数

``` kotlin
val items = listOf(1, 2, 3, 4, 5)

    // Lambdas 表达式是花括号括起来的代码块。
    items.fold(0, { 
        // 如果一个 lambda 表达式有参数，前面是参数，后跟“->”
        acc: Int, i: Int -> 
        print("acc = $acc, i = $i, ") 
        val result = acc + i
        println("result = $result")
        // lambda 表达式中的最后一个表达式是返回值：
        result
    })

    // lambda 表达式的参数类型是可选的，如果能够推断出来的话：
    val joinedToString = items.fold("Elements:", { acc, i -> acc + " " + i })

    // 函数引用也可以用于高阶函数调用：
    val product = items.fold(1, Int::times)

  	println("joinedToString = $joinedToString")
    println("product = $product")

//输出
/*
acc = 0, i = 1, result = 1
acc = 1, i = 2, result = 3
acc = 3, i = 3, result = 6
acc = 6, i = 4, result = 10
acc = 10, i = 5, result = 15
joinedToString = Elements: 1 2 3 4 5
product = 120
*/
```

### 函数类型

- 所有函数类型都有一个圆括号括起来的参数类型列表以及一个返回类型：`(A, B) -> C` 表示接受类型分别为 `A` 与 `B` 两个参数并返回一个 `C` 类型值的函数类型。 参数类型列表可以为空，如 `() -> A`。[`Unit` 返回类型](https://www.kotlincn.net/docs/reference/functions.html#返回-unit-的函数)不可省略。
- 函数类型可以有一个额外的*接收者*类型，它在表示法中的点之前指定： 类型 `A.(B) -> C` 表示可以在 `A` 的接收者对象上以一个 `B` 类型参数来调用并返回一个 `C` 类型值的函数。 [带有接收者的函数字面值](https://www.kotlincn.net/docs/reference/lambdas.html#带有接收者的函数字面值)通常与这些类型一起使用。
- [挂起函数](https://www.kotlincn.net/docs/reference/coroutines.html#挂起函数)属于特殊种类的函数类型，它的表示法中有一个 *suspend* 修饰符 ，例如 `suspend () -> Unit` 或者 `suspend A.(B) -> C`。

### 函数类型实例化

- 使用函数字面值的代码块，采用以下形式之一：

  - [lambda 表达式](https://www.kotlincn.net/docs/reference/lambdas.html#lambda-表达式与匿名函数): `{ a, b -> a + b }`,
  - [匿名函数](https://www.kotlincn.net/docs/reference/lambdas.html#匿名函数): `fun(s: String): Int { return s.toIntOrNull() ?: 0 }`

  [带有接收者的函数字面值](https://www.kotlincn.net/docs/reference/lambdas.html#带有接收者的函数字面值)可用作带有接收者的函数类型的值。

  

- 使用已有声明的可调用引用：

  - 顶层、局部、成员、扩展[函数](https://www.kotlincn.net/docs/reference/reflection.html#函数引用)：`::isOdd`、 `String::toInt`，
  - 顶层、成员、扩展[属性](https://www.kotlincn.net/docs/reference/reflection.html#属性引用)：`List<Int>::size`，
  - [构造函数](https://www.kotlincn.net/docs/reference/reflection.html#构造函数引用)：`::Regex`

  这包括指向特定实例成员的[绑定的可调用引用](https://www.kotlincn.net/docs/reference/reflection.html#绑定的函数与属性引用自-11-起)：`foo::toString`。

  

  带与不带接收者的函数类型非字面值可以互换,其中接收者可以替代第一个参数,反之亦然。例如，`(A, B) -> C` 类型的值可以传给或赋值给期待 `A.(B) -> C` 的地方，反之亦然：

### 函数类型实例调用

函数类型的值可以通过其 [`invoke(……)` 操作符](https://www.kotlincn.net/docs/reference/operator-overloading.html#invoke)调用：`f.invoke(x)` 或者直接 `f(x)`。

如果该值具有接收者类型，那么应该将接收者对象作为第一个参数传递。 调用带有接收者的函数类型值的另一个方式是在其前面加上接收者对象， 就好比该值是一个[扩展函数](https://www.kotlincn.net/docs/reference/extensions.html)：`1.foo(2)`，

## Lambda表达式与匿名函数

### 传递末尾的 lambda 表达式

如果函数的最后一个参数是函数，那么作为相应参数传入的 lambda 表达式可以放在圆括号之外：

```kotlin
val product = items.fold(1) { acc, e -> acc * e }
```

这种语法也称为*拖尾 lambda 表达式*。

如果该 lambda 表达式是调用时唯一的参数，那么圆括号可以完全省略：

```kotlin
run { println("...") }
```

### `it`：单个参数的隐式名称

一个 lambda 表达式只有一个参数是很常见的。

如果编译器自己可以识别出签名，也可以不用声明唯一的参数并忽略 `->`。 该参数会隐式声明为 `it`：

```kotlin
ints.filter { it > 0 } // 这个字面值是“(it: Int) -> Boolean”类型的
```

### 从 lambda 表达式中返回一个值

``` kotlin
ints.filter {
    val shouldFilter = it > 0 
    shouldFilter
}

ints.filter {
    val shouldFilter = it > 0 
    return@filter shouldFilter
}
```

这一约定连同[在圆括号外传递 lambda 表达式](https://www.kotlincn.net/docs/reference/lambdas.html#passing-a-lambda-to-the-last-parameter)一起支持 [LINQ-风格](https://docs.microsoft.com/en-us/previous-versions/dotnet/articles/bb308959(v=msdn.10)) 的代码：

```kotlin
strings.filter { it.length == 5 }.sortedBy { it }.map { it.toUpperCase() }
```

### 匿名函数

匿名函数参数总是在括号内传递。 允许将函数留在圆括号外的简写语法仅适用于 lambda 表达式

Lambda表达式与匿名函数之间的另一个区别是[非局部返回](https://www.kotlincn.net/docs/reference/inline-functions.html#非局部返回)的行为。一个不带标签的 *return* 语句总是在用 *fun* 关键字声明的函数中返回。这意味着 lambda 表达式中的 *return* 将从包含它的函数返回，而匿名函数中的 *return* 将从匿名函数自身返回。

## 内联函数

`inline` 修饰符标记 `lock()` 函数：

```kotlin
inline fun <T> lock(lock: Lock, body: () -> T): T { …… }
```

