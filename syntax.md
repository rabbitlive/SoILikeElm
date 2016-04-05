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

这里有一个小技巧，每个 Elm 程序员都应该知道的：

```elm
{--}
add x y = x + y
--}
```

仅仅在第一行添加或移除 } 就可以将代码注释或解开注释！

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
保持 JSON 或内容中有“引号”时，他很有用
"""
```

字面量的典型操作：

```elm
True && not (True || False)
(2 + 4) * (4 ^ 2 - 9)
"abc" ++ "def"
```

## 列表

这里的四种形式，都是相同的：

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

如果你有多个条件分支，把他们链起来即可：

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

每个模式是敏感的，意味着你必须群举所有的模式。
