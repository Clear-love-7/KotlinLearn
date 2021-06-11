# 通配符  =  ?

> < ? **extends** String>

- 只接受String及其子类, 生产者-使用处型变

> < ? **super** String> 

- 只接受String及其父类, 消费者

### 只能从中读取的对象为生产者,只能写入的对象为消费者

# 声明处型变 -Kotlin

> **out** 修饰符-型变注解
>
> 出现的位置是只读属性的类型或者函数的**返回值**类型
>
> 作为生产者的角色,请求向外部输出

```kotlin
interface Source<out T> {
    fun nextT(): T
}

fun demo(strs: Source<String>) {
    val objects: Source<Any> = strs
}

```

```kotlin
// Collections.kt
// E被声明为out
public interface List<out E> : Collection<E> {

    override val size: Int
    override fun isEmpty(): Boolean

    // E作为函数形参类型，而且还加上了@UnsafeVariance注解，下面会解释
    override fun contains(element: @UnsafeVariance E): Boolean

    // E作为函数返回值的类型Iterator泛型的类型实参
    override fun iterator(): Iterator<E>

    // E作为函数形参的类型Collection泛型的类型实参，而且还加上了@UnsafeVariance注解，下面会解释
    override fun containsAll(elements: Collection<@UnsafeVariance E>): Boolean

    // E作为函数返回值的类型
    public operator fun get(index: Int): E

    // E作为函数形参类型，而且还加上了@UnsafeVariance注解，下面会解释
    public fun indexOf(element: @UnsafeVariance E): Int

    // E作为函数形参类型，而且还加上了@UnsafeVariance注解，下面会解释
    public fun lastIndexOf(element: @UnsafeVariance E): Int

    // E作为函数返回值的类型ListIterator泛型的类型实参
    public fun listIterator(): ListIterator<E>

    // E作为函数返回值的类型ListIterator泛型的类型实参
    public fun listIterator(index: Int): ListIterator<E>

    // E作为函数返回值的类型List泛型的类型实参
    public fun subList(fromIndex: Int, toIndex: Int): List<E>

}
```

**@UnsafeVariance**注解告诉编译器这个地方是**合法**、**安全**，让其通过编译

> **in** 修饰符-型变注释

```kotlin
interface Comparable<in T> {
    operator fun compareTo(other: T): Int
}

fun demo(x: Comparable<Number>) {
    x.compareTo(1.0)// 1.0 拥有类型 Double，它是 Number 的子类型
    // 因此，我们可以将 x 赋给类型为 Comparable <Double> 的变量
    val y: Comparable<Double> = x
}
```

# 不型变

> 可以出现在任何位置

```kotlin
// Collections.kt
public interface MutableList<E> : List<E>, MutableCollection<E> {

    // E作为函数形参类型
    override fun add(element: E): Boolean

    // E作为函数形参类型
    override fun remove(element: E): Boolean

    // E作为函数形参的类型Collection泛型的类型实参
    override fun addAll(elements: Collection<E>): Boolean

    // E作为函数形参的类型Collection泛型的类型实参
    public fun addAll(index: Int, elements: Collection<E>): Boolean

    // E作为函数形参的类型Collection泛型的类型实参
    override fun removeAll(elements: Collection<E>): Boolean

    // E作为函数形参的类型Collection泛型的类型实参
    override fun retainAll(elements: Collection<E>): Boolean
    
    override fun clear(): Unit

    // E作为函数形参类型
    public operator fun set(index: Int, element: E): E

    // E作为函数形参类型
    public fun add(index: Int, element: E): Unit

    // E作为函数返回值的类型
    public fun removeAt(index: Int): E

    // E作为函数返回值的类型MutableListIterator泛型的类型实参
    override fun listIterator(): MutableListIterator<E>

    // E作为函数返回值的类型MutableListIterator泛型的类型实参
    override fun listIterator(index: Int): MutableListIterator<E>

    // E作为函数返回值的类型MutableList泛型的类型实参
    override fun subList(fromIndex: Int, toIndex: Int): MutableList<E>

}
```

# 星投影

有时你想说，你对类型参数一无所知，但仍然希望以安全的方式使用它。 这里的安全方式是定义泛型类型的这种投影，该泛型类型的每个具体实例化将是该投影的子类型。

Kotlin 为此提供了所谓的**星投影**语法：

- 对于 `Foo <out T : TUpper>`，其中 `T` 是一个具有上界 `TUpper` 的协变类型参数，`Foo <*>` 等价于 `Foo <out TUpper>`。 这意味着当 `T` 未知时，你可以安全地从 `Foo <*>` *读取* `TUpper` 的值。
- 对于 `Foo <in T>`，其中 `T` 是一个逆变类型参数，`Foo <*>` 等价于 `Foo <in Nothing>`。 这意味着当 `T` 未知时，没有什么可以以安全的方式*写入* `Foo <*>`。
- 对于 `Foo <T : TUpper>`，其中 `T` 是一个具有上界 `TUpper` 的不型变类型参数，`Foo<*>` 对于读取值时等价于 `Foo<out TUpper>` 而对于写值时等价于 `Foo<in Nothing>`。

# 扩展

> 你可以为一个你不能修改的、来自第三方库中的类编写一个新的函数。 这个新增的函数就像那个原始类本来就有的函数一样，可以用普通的方法调用。 这种机制称为 *扩展函数* 。此外，也有 *扩展属性* ， 允许你为一个已经存在的类添加新的属性

## 扩展函数

> 声明一个扩展函数，我们需要用一个 *接收者类型* 也就是被扩展的类型来作为他的前缀

```kotlin
fun MutableList<Int>.swap(index1: Int, index2: Int) {
    val tmp = this[index1] //"this"对应该列表
    this[index1] = this[index2]
    this[index2] = tmp
}

val list = mutableListOf(1,2,3)
list.swap(0,2)

//泛化
fun <T> MutableList<T>.swap(index1: Int, index2: Int) {
    val tmp = this[index1] //"this"对应该列表
    this[index1] = this[index2]
    this[index2] = tmp
}
```

## 扩展是静态解析的

> 调用的扩展函数是由函数调用所在的表达式的类型来决定的， 而不是由表达式运行时求值结果决定的

```kotlin
open class Shape

class Rectangle: Shape()

fun Shape.getName() = "Shape"

fun Rectangle.getName() = "Rectangle"

fun printClassName(s: Shape) {
    println(s.getName())
}    

printClassName(Rectangle())
//输出"Shape"
```

>如果一个类定义有一个成员函数与一个扩展函数，而这两个函数又有相同的接收者类型、 相同的名字，并且都适用给定的参数，这种情况**总是取成员函数**

```kotlin
class Example {
    fun printFunctionType() { 
        println("Class method") 
    }
}

fun Example.printFunctionType() { 
    println("Extension function") 
}

fun Example.printFunctionType(i: Int) { 
    println("Extension function") 
}

Example().printFunctionType()
//输出"Class method"
//扩展函数重载同样名字但不同签名成员函数也完全可以
Example().printFunctionType(1)
//输出"Extension function"
```

## 可空接收者

```kotlin
fun Any?.toString(): String {
    if (this == null) return "null"
    // 空检测之后，“this”会自动转换为非空类型，所以下面的 toString()
    // 解析为 Any 类的成员函数
    return toString()
}
```

## 扩展属性

```kotlin
val <T> List<T>.lastIndex: Int
    get() = size - 1
```

> 注意：由于扩展没有实际的将成员插入类中，因此对扩展属性来说[幕后字段](https://www.kotlincn.net/docs/reference/properties.html#幕后字段)是无效的。这就是为什么**扩展属性不能有初始化器**。他们的行为只能由显式提供的 getters/setters 定义

## 伴生对象的扩展

```kotlin
class MyClass {
    companion object { }  // 将被称为 "Companion"
}

fun MyClass.Companion.printCompanion() { println("companion") }

fun main() {
    MyClass.printCompanion()
}
```

