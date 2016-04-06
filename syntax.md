from [Elm Syntax](http://elm-lang.org/docs/syntax)

# Elm 语法

这是一份简单的语法指南，他包括:

* 注释
* 字面量
* 列表
* 条件
* 类型
* 记录
* 函数
* 中缀操作符
* let 表达式
* 应用函数
* 模块
* 类型注解
* 类型别名
* JS FFI

## 注释

```elm
-- 单行注释

{- 多行注释
  {- 并且能被嵌套 -}
-}
```

这里有一个每个 Elm 程序员都应该知道的小技巧：

```elm
{--}
add x y = x + y
--}
```

仅仅在第一行添加或移除`}`就可以将代码注释或解开注释！

## 字面量

```elm
-- Boolean
True : Bool
False : Bool

42 : number -- 依赖于使用 Int 或 Float 
3.14 : Float

'a' : Char
"abc" : String

-- 多行字符串
"""
在保持 JSON 格式或内容中有“引号”时，他很有用
"""
```

典型的字面量操作：

```elm
True && not (True || False)
(2 + 4) * (4 ^ 2 - 9)
"abc" ++ "def"
```

## 列表

这里有四种形式，他们都是相同的：

```elm
[1..4]
[1, 2, 3, 4]
1 :: [2, 3, 4]
1 :: 2 :: 3 :: 4 :: []
```

## 条件

```elm
if powerLevel > 9000 then "OVER 9000!!!" else "meh"
```

如果你有多个条件分支，可以把他们链起来：

```elm
if key == 40 then
  n + 1

else if key == 38 then
  n - 1
  
else
  n
```

也可以有基于代数数据结构和字面量的条件形式。

```elm
case maybe of
  Just xs -> xs
  Nothing -> []
  
case xs of
  hd::tl -> Just (td, tl)
  [] -> Nothing
  
case n of
  0 -> 1
  1 -> 1
  _ -> fib (n - 1) + fib (n - 2)
```

在这里的所有模式格式对不齐是会受到惩罚的呦，么么哒~

## 类型

```elm
type List = Empty | Node Int List
```

不明白这是什么意思？[读这里](http://elm-lang.org/guide/model-the-problem)

## 记录

更多关于 Elm 记录系统的说明，可以查看[this overview](http://elm-lang.org/docs/records)，[initial announcement](http://elm-lang.org/blog/announce/0.7)或是[this academic paper](http://research.microsoft.com/pubs/65409/scopedlabels.pdf)。

```elm
point =                            -- 创建一个记录
  { x = 3, y = 4 }
  
point.x                            -- 访问一个字段

map .x [ point, { x = 0, y = 0 } ] -- 字段访问函数

{ point | x = 6 }                  -- 更新一个字段

{ point |                          -- 更新一堆字段
    x = point.x + 1
	y = point.y + 1
}

dist { x, y } =                    -- 字段的模式匹配
  sqrt (x ^ 2 + y ^ 2)
  
type alias Location =              -- 记录的类型别名
  { line   : Int
  , column : Int
  }
```

## 函数

```elm
square n = 
  n ^ 2
  
hypotenuse a b =
  sqrt (square a + square b)
  
distance (a, b) (x, y) =
  hypotenuse (a - x) (b - y)
```

匿名函数：

```elm
square = 
  \n -> n ^ 2
  
squares =
  List.map (\n -> n ^ 2) [1..100]
```

## 中缀操作符

你也可以创建自己的中缀操作符。中缀操作符的优先级从0到9，9是最高级。默认的优先级就是9并且是左结合的。你可以自己设置，但是不能重写内建的操作符。

```elm
(?) : Maybe a -> a -> a
(?) maybe default =
  Maybe.withDefault default maybe
  
infixr 9 ?
```

使用 (<|) 和 (|>) 可以减少括号的使用次数。他们是函数应用的别名。

```elm
f <| x = fx
x |> f = fx

dot =
  scale 2 (move (20, 20) (filled blue (circle 10)))
  
dot' =
  circle 10
    |> filled blue
	|> move (20, 20)
	|> scale 2		
```

注：这玩意来自 F#,灵感来源于 Unix 的管道。

与此相关的，(<<) 和 (>>) 是函数组合。

## Let 表达式

Let 表达式用于给变量赋值，有点像JS中的var。

```elm
let
  x = 3 * 8
  y = 4 ^ 2
in
  x + y
```

也可以在 let 表达式里定义函数，还能使用解构。

```elm
let
  (x, y) = (3, 4)
  
  hypotenuse a b =
    sqrt (a ^ 2 + b ^ 2)
	
in
  hypotenuse x y
```

Let 表达式同样要求缩进对齐。

## 应用函数

```elm
-- 将两个 list 拼起来的别名
append xs ys = xs ++ ys
xs = [1, 2, 3]
ys = [4, 5, 6]

-- 下面所有的表达式都相同
a1 = append xs ys
a2 = (++) xs ys

b1 = xs `append` ys
b2 = xs ++ ys

c1 = (append xs) ys
c2 = ((++) xs) ys
```

基本的算数中缀运算符会自动计算他们的类型。

```elm
23 + 19  : number
2.0 + 1  : Float

6 * 7    : number
10 * 4.2 : Float

100 // 2 : Int
1 / 2    : Float
```

这里有个特殊的函数用于创建元组：

```elm
(,) 1 2             == (1, 2)
(,,,) 1 True 'a' [] == (1, True, 'a', [])
```

你可以使用多个逗号作为你需要的。

## 模块

```elm
module MyModule where

-- 限定导入
import List                         -- List.map, List.foldl
import List as L                    -- L.map, L.foldl

-- 开放导入
import List exposing (..)           -- map, foldl, concat, ...
import List exposing (map, foldl)   -- map, foldl

import Maybe exposing (Maybe)       -- Maybe
import Maybe exposing (Maybe(..))   -- Maybe, Just, Nothing
import Maybe exposing (Maybe(Just)) -- Maybe, Just
```

限定导入是优先的。模块名称必须匹配他的文件名，所以，模块`Parser.Utils`需要对应的文件是`Parser/Utils.elm`。

## 类型注解

```elm
answer : Int
answer =
  42
  
factorial : Int -> Int
factorial n = 
  List.product [1..n]
  
distance : { x : Float, y : Float } -> Float
distance { x, y } = 
  sqrt (x ^ 2 + y ^ 2)
```

## 类型别名

```elm
type alias Name = String
type alias Age = Int

info : (Name, Age)
info =
  ("Steve", 28)
  
type alias Point = { x : Float, y : Float }  

origin : Point
origin = 
  { x = 0, y = 0 }
```

## JS FFI

```elm
-- 传入值
port userID : String
port prices : Signal Float

-- 导出值
port time : Signal Float
port time = 
  every second
```

在JS端，你可以像下面这样使用这些 ports：

```elm
var example = Elm.worker(Elm.Example, {
  userID: "abc123",
  prices: 11
});

example.ports.prices.send(42);
example.ports.prices.send(13);

example.ports.time.subscribe(callback);
example.ports.time.unsubscribe(callback);

example.ports.increment(41) === 42;
```

更多的例子可以在[这里](https://github.com/evancz/elm-html-and-js)和[这里](https://gist.github.com/evancz/8521339)找到。

Elm 有一些内建的 port，自动完成一些必要的任务：

* `title` 设置页面的标题，忽略空字串
* `log` 将消息打印到开发者控制台
* `redirect` 重定向到其他页面，忽略空字串

处于试验中的 port：

* `favicon` 设置页面的图标
* `stdout` 打印输出在 node.js 和浏览器的 console
* `stderr` 打印错误在 node.js 和浏览器的 console
