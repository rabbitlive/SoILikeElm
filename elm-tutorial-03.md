# Elm 架构

这一章内容包括：

* 添加模型
* update函数
* actions
* 添加一个mailbox
* start app 库
* 使用带副作用的 start app

## 模型 Model

现在，我们的状态使用`foldp`保持，下面我们用**Elm架构**模式来重构我们的程序。

现在先来重构一下之前的程序：

```elm
module Main (..) where

import Html exposing (Html)
import Mouse


type alias Model =
  { count : Int
  }


initialModel : Model
initialModel =
  { count = 0
  }


view : Model -> Html
view model =
  Html.text (toString model.count)


modelSignal : Signal.Signal Model
modelSignal =
  Signal.foldp (\_ state -> { state | count = state.count + 1 }) initialModel Mouse.clicks


main : Signal.Signal Html
main =
  Signal.map view modelSignal
```

[https://github.com/sporto/elm-tutorial-assets/blob/master/code/C030ElmArch/Model.elm](https://github.com/sporto/elm-tutorial-assets/blob/master/code/C030ElmArch/Model.elm)

让我们来看看有什么新东西。

### 模型 Model

```elm
type alias Model =
  { count : Int }
```

这里我们引入一个叫做`Model`的`type alias`。这个类型别名定义一个记录结构。一个`Model`记录在我们的程序中需要有一个`count`属性，他是一个整数。

### 初始模型

```elm
initialModel : Model
initialModel =
  { count = 0 }
```

这个函数返回初始的模型。这里他返回的`Model`记录的`count`是0。

### 视图 view

```elm
view : Model -> Html
view model =
  Html.text (toString model.count)
```

我们的视图现在取一个`Model`记录并将`model.count`表示的数量显示出来。

### model 信号

```elm
modelSignal : Signal.Signal Model
modelSignal =
  Signal.foldp (\_ state -> { state | count = state.count + 1 }) initialModel Mouse.clicks
```

现在的`modelSignal`用`Signal.Signal Model`取代了`Signal.Signal Int`。所以这个信号可以发射`Model`了。

`foldp`中的**积累函数**取前一模型并返回一个新的。用下面的方法来更新前模型的`update`属性：

```elm
{ state | count = state.count + 1 }
```

在 Elm 中所有的数据结构是不可变的。上面语法展示了如何更新一个属性并返回新的不可变记录。
