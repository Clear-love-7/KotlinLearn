## 扩展声明为成员

> 在一个类的内部你可以为另外一个类声明扩展
>
> 扩展声明所在的类的实例称为分发接收者, 扩展方法调用所在的接收者类型的实例称为扩展接收者

声明为成员的扩展可以声明为 `open` 并在子类中覆盖。这意味着这些函数的分发对于分发接收者类型是虚拟的，但对于扩展接收者类型是静态的。

``` kotlin
open class Base { }

class Derived : Base() { }

open class BaseCaller {
    open fun Base.printFunctionInfo() {
        println("Base extension function in BaseCaller")
    }

    open fun Derived.printFunctionInfo() {
        println("Derived extension function in BaseCaller")
    }

    fun call(b: Base) {
        b.printFunctionInfo()   // 调用扩展函数
    }
}

class DerivedCaller: BaseCaller() {
    override fun Base.printFunctionInfo() {
        println("Base extension function in DerivedCaller")
    }

    override fun Derived.printFunctionInfo() {
        println("Derived extension function in DerivedCaller")
    }
}

fun main() {
    BaseCaller().call(Base())   // “Base extension function in BaseCaller”
    DerivedCaller().call(Base())  // “Base extension function in DerivedCaller”——分发接收者虚拟解析
    DerivedCaller().call(Derived())  // “Base extension function in DerivedCaller”——扩展接收者静态解析
}
```

## 对象表达式与对象声明

有时候, 我们需要创建一个对某个类做了轻微改动的类的对象, 而不用为之显式声明新的子类.  Kotlin 用对象表达式和对象声明处理这种情况

### 对象表达式

创建继承自某个(某类型)类型的匿名类的对象

```kotlin
window.addMouseListener(object : MouseAdapter() {
    override fun mouseClicked(e: MouseEvent) { /*……*/ }

    override fun mouseEntered(e: MouseEvent) { /*……*/ }
})
```

如果超类型有一个构造函数, 则必须传递适当的构造函数参数. 多个超类型可以由跟在冒号后面的逗号分隔的列表指定

```kotlin
open class A(x: Int) {
    public open val y: Int = x
}

interface B { /*……*/ }

val ab: A = object : A(1), B {
    override val y = 15
}
```

任何时候, 如果我们只需要一个对象而已, 并不需要特殊超类型, 那么我们可以简单地写

```kotlin
fun foo() {
    val adHoc = object {
        var x: Int = 0
        var y: Int = 0
    }
    print(adHoc.x + adHoc.y)
}
```

### 对象声明

```kotlin
object DataProviderManager {
    fun registerDataProvider(provider: DataProvider) {
        // ……
    }

    val allDataProviders: Collection<DataProvider>
        get() = // ……
}

//这些对象可以有超类型
object DefaultListener : MouseAdapter() {
    override fun mouseClicked(e: MouseEvent) { …… }

    override fun mouseEntered(e: MouseEvent) { …… }
}
```

> 注意: 对象声明不能再局部作用域(即直接嵌套在函数内部), 但是它们可以嵌套到其他对象声明或非内部类中

## 伴生对象

类内部的对象可以用 *companion* 关键字标记

即使伴生对象的成员看起来像其他语言的静态成员，在运行时他们仍然是真实对象的实例成员

## 对象表达式和对象声明之间的语义差异

对象表达式和对象声明之间有一个重要的语义差别：

- 对象表达式是在使用他们的地方**立即**执行（及初始化）的；
- 对象声明是在第一次被访问到时**延迟**初始化的；
- 伴生对象的初始化是在相应的类被加载（解析）时，与 Java 静态初始化器的语义相匹配。
