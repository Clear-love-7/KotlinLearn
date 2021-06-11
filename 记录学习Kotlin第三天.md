## 类型别名

## 内联类 --- inline

业务逻辑需要围绕某种类型创建包装器, 然而由于额外的堆内存分配问题, 会引入运行时的性能开销

内联类必须含有唯一的一个属性在主构造函数中初始化。在运行时，将使用这个唯一属性来表示内联类的实例

然而，内联类的成员也有一些限制：

- 内联类不能含有 *init* 代码块
- 内联类不能含有幕后字段
- 因此，内联类只能含有简单的计算属性（不能含有延迟初始化/委托属性）
- 内联类允许去继承接口, 禁止内联类参与到类的继承关系结构中。这就意味着内联类不能继承其他的类而且必须是 *final*

### 表示方式

``` kotlin
interface I

inline class Foo(val i: Int) : I

fun asInline(f: Foo) {}
fun <T> asGeneric(x: T) {}
fun asInterface(i: I) {}
fun asNullable(i: Foo?) {}

fun <T> id(x: T): T = x

fun main() {
    val f = Foo(42) 
    
    asInline(f)    // 拆箱操作: 用作 Foo 本身
    asGeneric(f)   // 装箱操作: 用作泛型类型 T
    asInterface(f) // 装箱操作: 用作类型 I
    asNullable(f)  // 装箱操作: 用作不同于 Foo 的可空类型 Foo?
    
    // 在下面这里例子中，'f' 首先会被装箱（当它作为参数传递给 'id' 函数时）然后又被拆箱（当它从'id'函数中被返回时）
    // 最后， 'c' 中就包含了被拆箱后的内部表达(也就是 '42')， 和 'f' 一样
    val c = id(f)  
}
```

因为内联类既可以表示为基础类型有可以表示为包装器，[引用相等](https://www.kotlincn.net/docs/reference/equality.html#引用相等)对于内联类而言毫无意义，因此这也是被禁止的。

## 内联类与类型别名

初看起来，内联类似乎与[类型别名](https://www.kotlincn.net/docs/reference/type-aliases.html)非常相似。实际上，两者似乎都引入了一种新的类型，并且都在运行时表示为基础类型。

然而，关键的区别在于类型别名与其基础类型(以及具有相同基础类型的其他类型别名)是 *赋值兼容* 的，而内联类却不是这样。

换句话说，内联类引入了一个真实的新类型，与类型别名正好相反，类型别名仅仅是为现有的类型取了个新的替代名称(别名)：

``` kotlin
typealias NameTypeAlias = String
inline class NameInlineClass(val s: String)

fun acceptString(s: String) {}
fun acceptNameTypeAlias(n: NameTypeAlias) {}
fun acceptNameInlineClass(p: NameInlineClass) {}

fun main() {
    val nameAlias: NameTypeAlias = ""
    val nameInlineClass: NameInlineClass = NameInlineClass("")
    val string: String = ""

    acceptString(nameAlias) // 正确: 传递别名类型的实参替代函数中基础类型的形参
    acceptString(nameInlineClass) // 错误: 不能传递内联类的实参替代函数中基础类型的形参

    // And vice versa:
    acceptNameTypeAlias(string) // 正确: 传递基础类型的实参替代函数中别名类型的形参
    acceptNameInlineClass(string) // 错误: 不能传递基础类型的实参替代函数中内联类类型的形参
}
```

## 委托

### 委托模式

实现继承的一个很好的替代方式

``` kotlin
interface Base {
    fun print()
}

class BaseImpl(val x: Int) : Base {
    override fun print() { print(x) }
}

class Derived(b: Base) : Base by b

fun main() {
    val b = BaseImpl(10)
    Derived(b).print()
}
```

### 覆盖由委托实现的接口成员

覆盖符合预期: 编译器会使用 `override` 覆盖的实现而不是委托对象中的

以这种方式重写的成员不会在委托对象的成员中调用, 委托对象的成员只能访问其自身对接口成员实现: 

``` kotlin
interface Base {
    val message: String
    fun print()
}

class BaseImpl(val x: Int) : Base {
    override val message = "BaseImpl: x = $x"
    override fun print() { println(message) }
}

class Derived(b: Base) : Base by b {
    // 在 b 的 `print` 实现中不会访问到这个属性
    override val message = "Message of Derived"
}

fun main() {
    val b = BaseImpl(10)
    val derived = Derived(b)
    derived.print()
    println(derived.message)
}
```

## 委托属性

- 延迟属性: 其值只在首次访问时计算
- 可观察属性: 监听器会收到有关属性变更的通知
- 把多个属性储存在一个映射(map)中, 而不是每个存在单独的字段中

语法:  `val/var <属性名>: <类型> by <表达式>` 在*by*后面的表达式时该*委托*, 因为属性对应的 `get()`（与 `set()`）会被委托给它的 `getValue()` 与 `setValue()` 方法。 属性的委托不必实现任何的接口，但是需要提供一个 `getValue()` 函数（与 `setValue()`——对于 *var* 属性）

### 标准委托

#### 延迟属性 Lazy

[`lazy()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/lazy.html) 是接受一个 lambda 并返回一个 `Lazy <T>` 实例的函数，返回的实例可以作为实现延迟属性的委托： 第一次调用 `get()` 会执行已传递给 `lazy()` 的 lambda 表达式并记录结果， 后续调用 `get()` 只是返回记录的结果。

``` kotlin
val lazyValue: String by lazy {
    println("computed!")
    "Hello"
}

fun main() {
    println(lazyValue)
    println(lazyValue)
}
//输出
/*
computed!
Hello
Hello
*/

```

#### 可观察属性 Observable

[`Delegates.observable()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.properties/-delegates/observable.html) 接受两个参数：初始值与修改时处理程序（handler）。 每当我们给属性赋值时会调用该处理程序（在赋值*后*执行）。它有三个参数：被赋值的属性、旧值与新值：

``` kotlin
import kotlin.properties.Delegates

class User {
    var name: String by Delegates.observable("<no name>") {
        prop, old, new ->
        println("$old -> $new")
    }
}

fun main() {
    val user = User()
    user.name = "first"
    user.name = "second"
}
//输出
/*
<no name> -> first
first -> second
*/
```

如果你想截获赋值并“否决”它们，那么使用 [`vetoable()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.properties/-delegates/vetoable.html) 取代 `observable()`。 在属性被赋新值生效*之前*会调用传递给 `vetoable` 的处理程序。

### 委托给另一个属性

一个属性可以把它的 getter 与 setter 委托给另一个属性。这种委托对于顶层和类的属性（成员和扩展）都可用。该委托属性可以为：

- 顶层属性
- 同一个类的成员或扩展属性
- 另一个类的成员或扩展属性

将一个属性委托给另一个属性，应在委托名称中使用恰当的 `::` 限定符

当想要以一种向后兼容的方式重命名一个属性的时候：引入一个新的属性、 使用 `@Deprecated` 注解来注解旧的属性、并委托其实现

``` kotlin
class MyClass {
   var newName: Int = 0
   @Deprecated("Use 'newName' instead", ReplaceWith("newName"))
   var oldName: Int by this::newName
}

fun main() {
   val myClass = MyClass()
   // 通知：'oldName: Int' is deprecated.
   // Use 'newName' instead
   myClass.oldName = 42
   println(myClass.newName) // 42
   myClass.newName = 1
   println(myClass.oldName)
    //输出
    /*
    42 
    1
    */
}
```

### 将属性储存在映射中

``` kotlin
class User(val map: Map<String, Any?>) {
    val name: String by map
    val age: Int     by map
}

val user = User(mapOf(
    "name" to "John Doe",
    "age"  to 25
))

println(user.name) // Prints "John Doe"
println(user.age)  // Prints 25
```

### 局部委托属性

### 属性委托要求

对于一个`只读`属性(即*val*声明的), 委托必须提供一个操作符函数`getValue()`, 该函数具有以下参数:

- `thisRef` —— 必须与 *属性所有者* 类型（对于扩展属性——指被扩展的类型）相同或者是其超类型。
- `property` —— 必须是类型 `KProperty<*>` 或其超类型

`getValue()` 必须返回与属性相同的类型（或其子类型）

``` kotlin
class Resource

class Owner {
    val valResource: Resource by ResourceDelegate()
}

class ResourceDelegate {
    operator fun getValue(thisRef: Owner, property: KProperty<*>): Resource {
        return Resource()
    }
}
```

对于一个`可变`属性(即*var*声明), 委托必须额外提供一个操作符函数`setValue()`,该函数具有以下参数:

- `thisRef` —— 必须与 *属性所有者* 类型（对于扩展属性——指被扩展的类型）相同或者是其超类型。
- `property` —— 必须是类型 `KProperty<*>` 或其超类型。
- `value` — 必须与属性类型相同（或者是其超类型）。

``` kotlin
class Resource

class Owner {
    var varResource: Resource by ResourceDelegate()
}

class ResourceDelegate(private var resource: Resource = Resource()) {
    operator fun getValue(thisRef: Owner, property: KProperty<*>): Resource {
        return resource
    }
    operator fun setValue(thisRef: Owner, property: KProperty<*>, value: Any?) {
        if (value is Resource) {
            resource = value
        }
    }
}
```

## 函数

### 函数声明

### 函数用法

#### 参数

#### 默认参数

``` kotlin
fun read(
    b: Array<Byte>, 
    off: Int = 0, 
    len: Int = b.size,
) { /*……*/ }
```

覆盖方法总是使用与基类型方法相同的默认参数值。 当覆盖一个带有默认参数值的方法时，必须从签名中省略默认参数值：

``` kotlin
open class A {
    open fun foo(i: Int = 10) { /*……*/ }
}

class B : A() {
    override fun foo(i: Int) { /*……*/ }  // 不能有默认值
}
```

如果一个默认参数在一个无默认值的参数之前，那么该默认值只能通过使用[具名参数](https://www.kotlincn.net/docs/reference/functions.html#具名参数)调用该函数来使用：

如果在默认参数之后的最后一个参数是 [lambda 表达式](https://www.kotlincn.net/docs/reference/lambdas.html#lambda-表达式语法)，那么它既可以作为具名参数在括号内传入，也可以在[括号外](https://www.kotlincn.net/docs/reference/lambdas.html#passing-a-lambda-to-the-last-parameter)传入：

``` kotlin
fun foo(
    bar: Int = 0,
    baz: Int = 1,
    qux: () -> Unit,
) { /*……*/ }

foo(1) { println("hello") }     // 使用默认值 baz = 1
foo(qux = { println("hello") }) // 使用两个默认值 bar = 0 与 baz = 1
foo { println("hello") }        // 使用两个默认值 bar = 0 与 baz = 1
```

#### 具名参数

#### 返回Unit的参数

如果一个函数不返回任何有用的值，它的返回类型是 `Unit`。`Unit` 是一种只有一个值——`Unit` 的类型。这个值不需要显式返回：

#### 单表达式函数

当函数返回单个表达式时，可以省略花括号并且在 **=** 符号之后指定代码体即可：

当返回值类型可由编译器推断时，显式声明返回类型是[可选](https://www.kotlincn.net/docs/reference/functions.html#显式返回类型)的：

#### 显式返回类型

#### 可变数量的参数(Varargs)

当我们调用 `vararg`-函数时，我们可以一个接一个地传参，例如 `asList(1, 2, 3)`，或者，如果我们已经有一个数组并希望将其内容传给该函数，我们使用**伸展（spread）**操作符（在数组前面加 `*`）：

``` kotlin
fun <T> asList(vararg ts: T): List<T> {
    val result = ArrayList<T>()
    for (t in ts) // ts is an Array
        result.add(t)
    return result
}

val a = arrayOf(1, 2, 3)
val list = asList(-1, 0, *a, 4)
```

#### 中缀表示法

标有 *infix* 关键字的函数也可以使用中缀表示法（忽略该调用的点与圆括号）调用。中缀函数必须满足以下要求：

- 它们必须是成员函数或[扩展函数](https://www.kotlincn.net/docs/reference/extensions.html)；
- 它们必须只有一个参数；
- 其参数不得[接受可变数量的参数](https://www.kotlincn.net/docs/reference/functions.html#可变数量的参数varargs)且不能有[默认值](https://www.kotlincn.net/docs/reference/functions.html#默认参数)。

``` kotlin
infix fun Int.shl(x: Int): Int { …… }

// 用中缀表示法调用该函数
1 shl 2

// 等同于这样
1.shl(2)
```

### 函数作用域

#### 局部函数

一个函数在另一个函数内部

局部函数可以访问外部函数（即闭包）的局部变量

#### 成员函数

### 泛型函数

### 扩展函数

### 高阶函数和Lambda表达式

> ==>记录学习Kotlin第四天

### 尾递归函数



