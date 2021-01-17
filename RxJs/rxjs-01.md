| 被观察 | 观察者 |
| ------ | ------ |
| 发布   | 订阅   |
| 主动   | 被动   |

## 网页的异步

1. 脚本加载
2. 播放器
3. 数据访问
4. 动画
5. DOM事件绑定，数据事件绑定

## Rx

| 生产者 | 消费者                   |
| ------ | ------------------------ |
| 拉取   | 被动，当请求时产生数据   |
| 推送   | 主动，按自己节奏产生数据 |

observable 就是推送系统 和 Promise 也是。

## Observable

1. 创建 Observable
2. 订阅
3. 执行
4. 清理

### 创建 observable

```js
const observable = Observable.create(observer => {
	observer.next(1)
	observer.error()
	observar.complete()
})
```

### 订阅 observable

```js
const subscribe = observable.subscribe({
  next: () =>,
  error: () => {},
   complete => {}
})
```

## Subject

像是 `EventEmitters`

每个subject都是Observable，可以提供Observable的方法。

每个Subject可以是观察者，来订阅源Observable。通过 `observable.multicast(subject)` 来订阅。

### 多播

## Operators

提供函数式编程风格的纯函数

> 操作符是函数，它基于当前的 Observable 创建一个新的 Observable。这是一个无副作用的操作：前面的 Observable 保持不变。

操作符本质上是一个纯函数 (pure function)，它接收一个 Observable 作为输入，并生成一个新的 Observable 作为输出。订阅输出 Observable 同样会订阅输入 Observable 。



