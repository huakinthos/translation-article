## transducing

比起我们已经在这本书中提到的，转导需要更先进的技术。在第九章列表操作中它扩展了更多的概念。

我不想严格的称这个话题为“Functional-Light”，但是更多的是从中得到益处。我发布这个附录是因为你现在可能非常想跳过这个讨论以及再次返回你感到非常舒适的--确保你非常熟练!--这本书的主要概念。

讲真的，即使多次讲述转导以及写了这个章节，我还在尝试完全理解这个技术，所以不要感到难过，如果这个这个技术打击到你了。收藏这个章节以及当你准备好了再回来看。

转导意味着以reduction的方式转变函数，和适配器模式相识。

我知道这可能听起来有点像杂乱的词汇，这个词汇更让人混淆 而不是澄清。但是让我们看看它是有多么强大。事实上，我认为这是一个最好的阐述，当你理解了函数式编程的概念之后，你可以干什么。

就像这本书的其他部分一样，我的方法首先是解释*why*，然后是*how*，最后是总结性的，重复的*what*。这和通常的教学相反，但是我认为你可以以这种方法更深层次的学习。

## 首先是，Why

让我们首先通过扩展第三章我们讲述的场景，测试单词去看看他们是否足够长或足够短。

在第三章，我们使用了这些判断方法取测试单个词汇。再在第九章，我们学习了如何重复像测试 filter() 这样的列表操作。

这可能并不明显，但是这种分隔相近列表操作的模式有点非我们所想的特征。当你只处理一个有少量数值的数组时，所有事都是好的。但是当这个数组有很多的值时，每个单独处理列表 `filter( .. )` 会降低一些我们所需求的速度。

当我们数组是同步/懒惰（像Observables）时，以及当以响应事件的方式多次处理值，这种类似的性能问题就上升了。在这种场景中，在一个时间段只有单个值传入到事件流之中，所以以两个单独的 `filter` 函数调用来处理这些离散的值不是个非常好的解决方法。

但是不明显的是，每个 filter 方法都产出单独的 observable。将一个不是observable的值注入到另外一个的开销确实会增加。在这个例子中这是确实会发生的，对于几千或上百万的被加工的值，者并不常见；甚至一些小的开销会飞速上升。

其他缺点也是显而易见的，特别是当我们需要对多个列表重复一系列相同的操作。

重复了，不是吗？

如果我们组合两个判断函数到一个函数中，是否会变得更好呢？

但是这并不是FP的方式。

在第九章我们讨论了融合 -- 组合相近的 mapping 函数。

不幸的是，组合相近的函数并不像组合相近的mapping函数一样轻松。为了理解为什么，想想看判断函数的形状 -- 一种描述输入和输出的签名的学术方式。它接收单个值进入，以及返回单个 true or false。

如果尝试 `isShortEnough(isLongEnough(str))`, 它将不会以适合的方式工作。`isLongEnough( .. )` 将返回 true/false，而不是 `isShortEnough` 所期待的单个值。

一个类似的令人诅丧的事实存在，当你尝试组合两个相近的reducer函数。这个reducer的形状是一个接收两个值作为输入，以及返回单个组合值的函数。作为单个值的一个reducer的输出是不适合作为输入给另外接收两个输入的reducer的。

还有就是，reduce( .. ) 接收可选参数 `initialValue` 。有时它可以被忽略，但有时需要被传入。它还有可能是一个复杂组合，因为一个 reduction 可能需要一个初始值以及别的reduction可能需要不同初始值。如果我们要以一种组合的reducer来进行一次reduce调用，那么我们该怎么做？

思考下面这个链式调用：

你可以想象一个包括这些所有步骤 -- map, filter, filter, filter -- 的组合吗？这些函数的形状各不相同，所以他们不能直接组合在一起。我们需要稍加改变他们的形状来将他们适配在一起。

希望通过这些简单的观察已经表述了为什么这些简单的融合式风格的组合不能胜任这个任务。我们需要更加强大的工具，`transducing` 转导就是这种工具。

## 接下来是，how

让我们说说如何推导出 mapper, predicates 以及 reducer 的组合。

### 像 Reduce 一样表示 Map/Filter

我们需要表现的第一个技巧就是像调用 reduce 一样调用 filter 和 map。回想第九章所做的：创建接受两个参数的函数，再返回单个函数，期间为和map或filter一样的操作。

这是一种极大的提升。我们现在有四个相近的 reducer 调用函数，而不是三个不同形状的不同方法。我们还没有 `compose( .. )` 这四个reducer，因为他们需要接受两个参数而不是一个。

我们使用带副作用的 `list.push( .. )` 来改变值，而不是创建一个完全新的数组去连接它们。让我们退一步以及稍微正规化一些：

稍后我们再讨论是否需要创建了一个新的数组去连接他们。

### 参数化 Reducers

filter reducers 除了使用不同的判断函数， 他们都是相同的。让我们参数化这些函数，来获得单个定义任意filter-reducer的实用方法：

让我们对mapper做同样的参数化，来获得产出任意map-reducer的实用方法：

我们的链式调用可以变得 point-free 风格。

### 提取公用逻辑

仔细看先前的 `mapReducer( .. )` 和 `filterReducer` 函数，你是否注意到了可以分享的公共功能？

没错，就是在返回数组这部分: 

让我们为这个公用逻辑定义一个函数，就叫 `listCombine`。

使用 `listCombine` 重新定义产出filter-reducer的方法。减少重复代码。

### 参数化 combination

我们的简单的 `listCombine` 方法只有组合两个值的一个用途。参数化这个方法让 `filterReucer` 更加全面。

这些产出函数的方法接受两个参数而不是一个参数，对于组合来说非常不方便，所以使用 `curry( .. )`这个方法：

这看起来有点冗长而且看起来不是那么有用。

但是这对获得推到的下一步至关重要。请记住，我们的最终目的是能够 `compose` 这些 reducers。

### 组合 Curried

传入了第一个函数作为参数，但是第二个参数`listcombine` 没有传入。

思考下这三个的中间函数的形状。每个函数都预期一个 combination 函数，然后产出一个 reducer。

我们将给个 curried 函数产出的reducer作为下一个 curried函数的第二个参数 `listCombine`。

```js
var composition = compose(
    curriedMapReducer( strUppercase ),
    curriedFilterReducer( isLongEnough ),
    curriedFilterReducer( isShortEnough )
);

var upperLongAndShortEnoughReducer = composition( listCombine );

words.reduce( upperLongAndShortEnoughReducer, [] );
```

让我们想象一下这个函数的数据流：

1. `listCombine( .. )` 作为 combination 函数流进，生成`isShortEnough` 的 filter-reducer
2. reducer 函数作为 combination 函数流进，生成 filter-reducer 
3. 最后，reducer 函数作为 combination 函数流进，生成 map-reducer 

在先前的代码段中，`compostion` 是一个组合好的函数，它预期一个 combination 函数让它变成 reducer；它有一个特殊的名字：transducer。提供 combination 函数来让transducer 产出组合好的reducer。

**注意：**我们在前两段片段中可以观察到 `compose( .. )`的执行顺序可能回引起我们的困惑。回想最开始的链式例子，我们以(map => filter => filter)这种方式；这些操作确实是按这种顺序执行的。但是在第四章，我们学习到 compose 是典型的翻转这些列表顺序来执行这些函数的。所以我们在这里为什么不需要翻转顺序来获得相同的结果呢？从每个reducer的`combinerFn`的抽象翻转了在引擎下有效应用的操作顺序。所以与我们的直觉相反，当你组合transducer，你确实想要以渴望的执行顺序来排列他们。

## 最后，What

让我们把注意力转回来，不跳过这些思维障碍来推导他是如何工作的，在我们的应用中使用`transducing`

```js
var transduceMap =
    curry( function mapReducer(mapperFn,combinerFn){
        return function reducer(list,v){
            return combinerFn( list, mapperFn( v ) );
        };
    } );

var transduceFilter =
    curry( function filterReducer(predicateFn,combinerFn){
        return function reducer(list,v){
            if (predicateFn( v )) return combinerFn( list, v );
            return list;
        };
    } );
```

然后我们是这样使用的：

```js
var transducer = compose(
    transduceMap( strUppercase ),
    transduceFilter( isLongEnough ),
    transduceFilter( isShortEnough )
);
```

transducer 仍然需要一个 combination 函数来生产transduce-reducer 函数，这个函数将会在reduce中被使用。

但是为了更声明式的表达这些transducing步骤，我们需要定义一个函数，来帮助我们不必重复这些步骤。

```js
function transduce(transducer,combinerFn,initialValue,list) {
    var reducer = transducer( combinerFn );
    return list.reduce( reducer, initialValue );
}
```

现在可以这样执行：

```js
var transducer = compose(
    transduceMap( strUppercase ),
    transduceFilter( isLongEnough ),
    transduceFilter( isShortEnough )
);

transduce( transducer, listCombine, [], words );
// ["WRITTEN","SOMETHING"]

transduce( transducer, strConcat, "", words );
// WRITTENSOMETHING
```

## 总结

transduce 意味着以reduce进行变换。再详细点说，transducer 就是一个能组合的 reducer。

我们用 `transducing` 去组合相近的map，filter和reduce。我们首先通过将map，filter表达为reduce，然后抽出公用方法来创造易于组合的一元函数来完成transducer。

`transducing`主要 是提升了性能，这使得当我们使用 observable 时优化比较明显。

在更宽广的方面来说，`transducing`是我们如何更加声明式的表达那些函数组合，而这些函数不能直接组合。如果你能够像这本书里的其他技术一样合理的使用，那么显而易见的能够带来代码可读性的提升！