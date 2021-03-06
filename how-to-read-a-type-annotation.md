翻自 [How to Read a Type Annotation](https://github.com/elm-guides/elm-for-js/blob/master/How%20to%20Read%20a%20Type%20Annotation.md)

# 如何读懂一个类型注释

如果你来自一些JavaScript或Ruby之类的动态类型语言，或是c家族中像Java一样的静态类型语言，那么， Elm 的类型注释看上去有一点奇怪。然而，你一旦知道如何读他，将比像`int strcmp(const char *s1, const char *s2)`之类的写法更具意义。虽然对于编译器而言并没有什么暖用（因为编译器有类型推导可以自行计算出来）但是对于你，程序员来说，真真是极好的……

弄明白类型注释一个很重要的原因是，当你开始阅读标准库的文档时，会发现每个函数都有类型注释。类型注释会告诉你这个函数需要多少个参数，他们的类型是什么，或者说要传递什么样的函数，他的返回类型是什么。这些信息已经全都在类型注释当中了。

一旦搞懂了他们，写起来是非常容易的。虽然写不写都是可以的，但还是极力推荐把他加上。类型注释可以帮助你思考函数应该做些什么，并可以帮助编译器验证代码。（你知道么？一个过时的注释比彻底没有注释更坑爹，那么好的，类型注释永远不会过时。）此外，如果你想要发布一个第三方的包，你也会需要类型注释。

## 定义

第一件事实是要弄明白`:`的意思，他表示“类型是什么”。

```elm
answer : Int
answer = 42
```

上面的例子可以这么读：“answer 的类型是 Int；answer 等于 42”。

常见的基本类型包括`Int`，`Float`，`Bool`和`String`。元组可以有一对类型，如`(Int, Bool)`。也可以扩展到任意多个元素，如`(Int, Float, Int)`就是一个三元组，第一个元素是`Int`类型，第二是`Float`，第三个是`Int`。

```elm
myTuple : (String, Int, Bool)
myTuple = ("the answer", 42, True)
```

不知道你有没有注意到，类型的首字母是大写的。（或在括号中）

有一种很特殊的类型，他只有一种值。类型和值都读做“unit”，写为`()`。在我们已经知道类型有一个值的时候，Unit 经常被用作占位符。

## 函数

`->`用于将函数中参数和返回值的类型分隔开。这明显这是“到”的意思，例如，`String.length : String -> Int`可表示为，“String.length 的类型是 String 到 Int”。只需要像一个句子一样从左往右读就好了。哦，顺便一提，`String.length`表示`length`函数在`String`模块中。无论何时，大写单词后面跟个点，就表示他是个模块，而并不是一个类型。

有趣的是当出现多个箭头时，如`update: Action -> Model -> Model`。这个函数取 Action 和一个 Model 作为参数（按照这个顺序），并且返回一个 Model。或者说“update 有一个 Action 到 Model 到 Model 的类型”。

类型注释真正想要告诉你的其实关于_偏应用_的东东：你可以只给函数部分参数，这样就会得到一个函数作为结果。得到新函数的类型注释就是原函数类型注释覆盖掉左侧之后的部分。

```elm
example : Model -> Model
example = update someAction
```

其实在类型注释中隐藏有一对括号，我们也可以把他写为：`update : Action -> (Model -> Model)`。

不需要太去在意偏应用或是柯里化之类的概念，太多新东西了。只要知道最后一个箭头做为函数的返回值，其他的作为参数就可以了。

## 高阶函数

就像JavaScript那样，函数也可以作为一个函数的参数传递。（我们已经见到过柯里化是如何让他们返回函数的。）

让我们看下专业定制版`List.map`函数，他取一个函数并且将他应用于列表的每个Float，然后返回一个新的Int列表作为结果。

```elm
specialMap : (Float -> Int) -> List Float -> List Int
```

第一个参数需要一个函数，他取一个 Float 作为参数并且返回一个 Int。当读到这个注释时，你可能会对“Float 到 Int”感到一脸懵逼，暂停一下。这里，先来理解括号的重要性。他不同于`Int -> Float -> List Int -> List Float`，这个取的是两个数字和一个列表，但并不是一个函数。

我们知道`round : Float -> Int`后，可以这么写：

```elm
roundMap : List Float -> List Int
roundMap = specialMap round
```

即便`roundMap`不带有任何参数，他的类型很明显就是 specialMap 调用 round 后返回函数的类型，这要感谢柯里化。我们同样可以写为`roundMap xs = specialMap round xs`；这只是风格问题。

## 类型变量

如果你看过 List 库，会发现那其实并不是[List.map](http://package.elm-lang.org/packages/elm-lang/core/latest/List#map)的定义。相反，他有着小写的类型名字，他们就是类型变量：

```elm
List.map : (a -> b) -> List a -> List b
```

他意味着函数将作用于任何类型的 a 和 b，通常在知道参数的具体类型时，才会去固定这些变量的值。所以我们也可以写为`(Float -> Int)`和一个`List Float`，或是`(String -> Action)`和一个`List String`等等。（比起JavaScript，使用类型变量更像是在做数学运算。）

按照惯例，类型变量从字母a开始，你可以用其他的小写字母。当有一个以上的类型变量时，偶尔也会用其他字母或词来帮助理解。比如`Dict k v`可以看出表示的是键和值。一个类型可以有任意多的类型变量，但超过两个的很罕见。

类型变量可以让我们编写泛型代码，如列表和其他容器，被用来容纳任意类型的值。每个特定的容器只能容纳一个类型，但是你可以自由选择。然后可以用`List.map`遍历列表并应用一个函数，你无需知道列表中放着的是什么。仅仅每个元素上应用的函数才需要知道这些元素的类型。

`List a`表示一个任意类型的列表，那如果只有`List`那会是什么？他被称为一个类型构造器，但更好的回答是，他什么都不是。他并不能独自存在。最好把他想象成是列表的基类，当有了类型变量时，才会被替换成一个真正的类型。

## 记录

记录很像JavaScript中的 object，但他是在编译期提前定义好的。像JavaScript一样，他们使用大括号。但不同的是，记录的键和值之间用等号连接；而冒号用于定义记录的类型。下面是一个简单的记录：

```elm
point : { x : Float, y : Float }
point = { x = 3.2, y = 2.5 }
```

大多数情况下，你需要知道记录的类型。但有时在写函数时只用到记录的某些字段，而忽略其他字段。

```elm
planarDistance : { a | x : Float, y : Float } -> { b | x : Float, y : Float } -> Float
planarDistance p1 p2 =
  let dx = p2.x - p1.x
      dy = p2.y - p1.y
  in
      sqrt (dx ^ 2 + dy ^ 2)
```

`{a |`表示一个基本记录，继承了`a`的类型。然后依次列出了要继承的字段和他们的类型。最简单的`a`可以是一个空记录，即没有额外的字段。我们用不同的类型变量`b`，来做第二个变量，这两个记录不必含有相同的类型。举个栗子：

```elm
point3D = { x = 1.0, y = 6.3, z = -0.9}
dist = planarDistance point point3D
```

## 类型约束

Elm 有三种特殊的类型变量，表明该值几种不同类型的值，但并不是任意类型。

一个`number`表示`Int`或`Float`。number 类型支持基本数学运算（但除了除法，除法运算每个类型都要单独处理）。

一个`comparable`可以是一个数字，字符，字符串，递归列表或是可比较的元组。牛掰的是，comparable 可以用`(>)`来做比较。Elm中的字典和有序集合被实现为二叉搜索树，所以键或元素必须是可比较的。

一个`appendable`可以是一个字符串，文本，或是一个列表（包含任意类型）。两个相同类型的 appendable 可以用`(++)`来拼接。

想要使用这些类型，只需要在注释中使用他们的名称就好，不需要指定类型或类型变量。

如果这些类型中的某个在类型注释中多次出现，所有的东西必须被解释为同一类型。你也可以在末端加一些东西让他们不同，就像`appendable2`之类的。例如，如果你在Elm REPL键入`(4, 2)`，他会被推导为`(number, number')`。一撇表示第二个 number 不必和第一个相同。

## Find me

```elm
import QQGroup exposing ( 537981557 )
```
