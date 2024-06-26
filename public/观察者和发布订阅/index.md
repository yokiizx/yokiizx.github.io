# 观察者和发布订阅


在前端领域，这两个设计模式，有着很多的应用，必须熟练掌握。

### 观察者模式

在 `promsie A+` 实现时，我们初始化了两个数组 -- `onFulfilledCb` 和 `onRejectedCb`，它们用来收集依赖，当 promise 注册了多个 then 且因为 promise 是可能进行时，就把 `onFulfilled` 函数加入 `onFulfilledCb` 中，当得到 `resolve` 的通知后，再去通知执行。

这个过程其实就是一个简单的观察者模式。这里被观察者是: promise，观察者是: 两个队列里的回调函数，promise 的 resolve 通知回调执行。 两者的关系是通过 **被观察者** 建立起来的，被观察者有**添加**，**删除**和**通知**观察者的功能。

```js
// 简单实现一下
class Subject {
  constructor() {
    this.subs = []
  }
  addObserver(observer) {
    this.subs.push(observer)
  }
  removeObserver(observer) {
    const rmIndex = this.subs.findIndex((ob) => ob.id === observer.id ) >>> 0
    this.subs.splice(rmIndex,1)
  }
  notify(message) {
    this.subs.forEach(ob => ob.notified(message))
  }
}

class Observer {
  constructor(id, name, subject) {
    this.id = id
    this.name = name
     // 除了让被观察者主动添加进, 观察者创建时也可以直接添加到观察者列表
    if(subject) subject.addObserver(this)
  }
  notified(msg) {
    console.log('copy that: ', msg)
  }
}

// test
const subject = new Subject()

const ob1 = new Observer(001, '001')
const ob2 = new Observer(007, '007', subject)
subject.addObserver(ob1)

subject.notify('world')
// 007  copy that:  world
// 001  copy that:  world
```

### 发布-订阅模式

用过 vue 的朋友应该都知道在 vue 中有一种跨组件通信的方式叫 `EventBus`，这就是一个典型的 `发布-订阅模式`。

发布订阅模式将发布者与订阅者通过消息中心/事件池来联系起来。

```js
class EventCenter {
  constructor() {
    this.eventMap = {}
  }
  on(eventType, callback) {
    if (typeof callback !== 'function') {
      throw new Error('请设置正确的函数作为回调')
    }
    if (!this.eventMap[eventType]) {
      this.eventMap[eventType] = []
    }
    this.eventMap[eventType].push(callback)
  }
  emit(eventType, data) {
    if (!this.eventMap[eventType]) return
    this.eventMap[eventType].forEach(callback => callback(data))
  }
  off(eventType, callback) {
    if (!this.eventMap[eventType]) return
    if (!callback.name) return
    const cbIndex = this.eventMap[eventType].findIndex(cb => cb.name === callback.name) >>> 0
    this.eventMap[eventType].splice(cbIndex, 1)
  }
}

// test
const EC = new EventCenter()
const log = data => console.log(`demo `, data)

EC.on('demo', log)
EC.emit('demo', 'hello world')

EC.off('demo', log)
EC.emit('demo', 'hello world')
```

> 可以看到，订阅者在订阅事件的时候，只关注事件本身，而不关心谁会发布这个事件；发布者在发布事件的时候，只关注事件本身，而不关心谁订阅了这个事件。

### 总结

- 在观察者模式中，观察者是知道发布者的，发布者一直保持对观察者进行记录。然而，在发布订阅模式中，发布者和订阅者互相不知道对方的存在。它们通过调度中心进行通信。
- 在发布订阅模式中，组件是松耦合的，正好和观察者模式相反。
- 观察者模式大多数时候是同步的，比如当事件触发，发布者就会去调用观察者的方法。而发布-订阅模式大多数时候是异步的（使用消息队列）。

