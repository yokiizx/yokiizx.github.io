# 重学设计模式 -- 行为型


设计模式其实并不是多么高大尚的东西，都是码农届前辈积累下来的一些编程经验，在工作中可以说随处可见，作为前端开发者，学习这些"套路"很有必要，跟着 [desing-patterns](https://refactoringguru.cn/design-patterns) 再来回顾一下吧。

## 行为型

### 职责链

核心：使得多个对象都有机会处理请求，从而避免了请求发送者和接收者之间的耦合关系，将这些对象形成一条链，并沿着这条链传递请求，直到有一个对象能够处理该请求为止。

```TS
abstract class Handler {
  nextHandler: Handler;

  setNextHandler(handler: Handler): Handler {
    this.nextHandler = handler;
    return handler;
  }

  handleRequest(request: string): string {
    if (this.nextHandler) {
      return this.nextHandler.handleRequest(request);
    }
    return null;
  }
}

class ConcreteHandler1 extends Handler {
  handleRequest(request: string): string {
    if (request === 'type1') {
      return 'Type 1 handled';
    }
    return super.handleRequest(request);
  }
}

class ConcreteHandler2 extends Handler {
  handleRequest(request: string): string {
    if (request === 'type2') {
      return 'Type 2 handled';
    }
    return super.handleRequest(request);
  }
}

class ConcreteHandler3 extends Handler {
  handleRequest(request: string): string {
    if (request === 'type3') {
      return 'Type 3 handled';
    }
    return super.handleRequest(request);
  }
}

// usage
const handler1 = new ConcreteHandler1();
const handler2 = new ConcreteHandler2();
const handler3 = new ConcreteHandler3();

handler1.setNextHandler(handler2).setNextHandler(handler3);

console.log(handler1.handleRequest('type2')); // "Type 2 handled"
console.log(handler1.handleRequest('type4')); // "null"

```

该示例中，定义了一个抽象类 Handler，表示职责链中的处理者。每个具体的处理者类（ConcreteHandler1、ConcreteHandler2、ConcreteHandler3）都继承自 Handler，并实现了 handleRequest 方法。在处理请求时，每个处理者都会判断能否处理该请求，如果不能处理则将请求转发给下一个处理者。最后，通过设置每个处理者的下一个处理者，形成了一个职责链。

通过该示例可以看到，职责链模式可以方便地扩展处理者，从而增强系统的灵活性。同时，职责链模式也有可能导致请求被多个处理者处理，从而可能降低系统性能。因此，在使用该模式时需要根据具体情况进行权衡。

### 命令

核心：将请求封装为一个对象，从而使得可以将请求操作和请求对象解耦，并且可以很容易地添加新的请求操作。

可以使用命令模式来消除操作的调用者与接收者之间的耦合关系，适用于需要将请求排队、记录请求指令历史或者撤销请求的场景。

```TS
interface Command {
  execute(): void;
}

class TurnOnCommand implements Command {
  private receiver: Receiver;
  constructor(receiver: Receiver) {
    this.receiver = receiver;
  }

  execute(): void {
    console.log("执行开灯操作");
    this.receiver.turnOn();
  }
}

class TurnOffCommand implements Command {
  private receiver: Receiver;
  constructor(receiver: Receiver) {
    this.receiver = receiver;
  }

  execute(): void {
    console.log("执行关灯操作");
    this.receiver.turnOff();
  }
}

/**
 * 请求对象
 */
class Receiver {
  turnOn() {
    console.log("开灯");
  }

  turnOff() {
    console.log("关灯");
  }
}

class Invoker {
  private commands: Command[] = [];

  addCommand(command: Command): void {
    this.commands.push(command);
  }

  executeCommands(): void {
    this.commands.forEach(command => command.execute());
  }
}

// usage
const invoker = new Invoker();
const receiver = new Receiver();

const turnOnCommand = new TurnOnCommand(receiver);
const turnOffCommand = new TurnOffCommand(receiver);

invoker.addCommand(turnOnCommand);
invoker.addCommand(turnOffCommand);
invoker.executeCommands();

```

该示例中，定义了 Command 接口，其包含一个方法 execute()。实现了两个具体命令类 TurnOnCommand 和 TurnOffCommand，它们均实现了 Command 接口，并封装了对应的请求操作。定义了 Receiver 类，它代表了请求的接收者，包含了对应的请求操作具体实现。最后定义了 Invoker 类，它包含了命令队列的集合，并包含了执行命令的方法。

在使用命令模式时，可以解除命令的调用者和接收者之间的关系，通过命令对象进行传递和操作。通过 Invoker 类来管理命令集合和执行请求，从而提高了代码的灵活性和复用性。

### 中介者

核心：旨在降低对象之间的耦合度，通过通过一个中介者对象进行协调通信。

`MessageChannel` 就是一个典型的中介者模式。

```TS
var channel = new MessageChannel();
var port1 = channel.port1;
var port2 = channel.port2;
port1.onmessage = function(event) {
    console.log("port1收到来自port2的数据：" + event.data);
}
port2.onmessage = function(event) {
    console.log("port2收到来自port1的数据：" + event.data);
}

port1.postMessage("发送给port2");
port2.postMessage("发送给port1");
```

原理类似：

```TS
interface Mediator {
  send(message: string, colleague: Colleague): void;
}

abstract class Colleague {
  private mediator: Mediator;

  constructor(mediator: Mediator) {
    this.mediator = mediator;
  }

  public getMediator(): Mediator {
    return this.mediator;
  }

  public send(message: string): void {
    this.mediator.send(message, this);
  }

  public abstract receive(message: string): void;
}

class ConcreteColleague1 extends Colleague {
  constructor(mediator: Mediator) {
    super(mediator);
  }

  public receive(message: string): void {
    console.log(`[ConcreteColleague1] received message: ${message}`);
  }
}

class ConcreteColleague2 extends Colleague {
  constructor(mediator: Mediator) {
    super(mediator);
  }

  public receive(message: string): void {
    console.log(`[ConcreteColleague2] received message: ${message}`);
  }
}

class ConcreteMediator implements Mediator {
  private colleague1: ConcreteColleague1;
  private colleague2: ConcreteColleague2;

  public setColleague1(colleague: ConcreteColleague1) {
    this.colleague1 = colleague;
  }

  public setColleague2(colleague: ConcreteColleague2) {
    this.colleague2 = colleague;
  }

  public send(message: string, colleague: Colleague): void {
    if (colleague === this.colleague1) {
      this.colleague2.receive(message);
    } else {
      this.colleague1.receive(message);
    }
  }
}

// usage
const mediator: ConcreteMediator = new ConcreteMediator();
const colleague1: ConcreteColleague1 = new ConcreteColleague1(mediator);
const colleague2: ConcreteColleague2 = new ConcreteColleague2(mediator);

mediator.setColleague1(colleague1);
mediator.setColleague2(colleague2);

colleague1.send("Hello from ConcreteColleague1"); // ConcreteColleague2 receives message: Hello from ConcreteColleague1
colleague2.send("Hello from ConcreteColleague2"); // ConcreteColleague1 receives message: Hello from ConcreteColleague2
```

### 观察者

还用多说嘛？Vue 的响应式原理就是观察者模式。

一个对象（称为“ Subject” 或“ Observable”）维护一组订阅者（称为“ Observer” 或“ Subscriber”），并在发生某些重要事件时自动通知它们。

```TS
/**
 * 被观察对象
 */
interface Subject {
  subs: Observer[]; // 订阅者集合
  attach: (observer: Observer) => void;
  detach: (observer: Observer) => void;
}

class Sub implements Subject {
  subs: Observer[];
  constructor() {
    this.subs = [];
  }
  attach(observer: Observer) {
    this.subs.push(observer);
  }
  detach(observer: Observer) {
    const rmIndex = this.subs.findIndex((ob) => ob.name === observer.name) >>> 0;
    this.subs.splice(rmIndex, 1);
  }
  notify(message: string) {
    this.subs.forEach((ob) => ob.update(message));
  }
}

/**
 * 观察者
 */
interface Observer {
  name: string;
  update: (data: any) => void;
}

class Ob implements Observer {
  name: string;
  constructor(name: string) {
    this.name = name;
  }
  update(received) {
    console.log(`${this.name} got ${received}`);
  }
}

const subject = new Sub();
const ob1 = new Ob('hello');
const ob2 = new Ob('world');

subject.attach(ob1);
subject.attach(ob2);

subject.notify('ABC');

subject.detach(ob2);
subject.notify('123');

// hello got ABC
// world got ABC
// hello got 123
```

### 状态

核心：允许对象在内部状态改变时改变它的行为。这种模式可以看作是根据状态改变而改变实例化对象的行为。

举个例子比如电灯，一般也就开关两个状态，开关按一下开，再按一下关。但是现在高级点的还有弱光、强光等状态，以前一个 if-else 就能搞定，如果在原来的状态中继续补充扩大 if-else 会使得代码变得难以维护。

思考：按一下微光，按两下强光，按三下暖光，按四下关闭：这是一种状态的改变，且状态之间是相关联的：微->强->暖->闭。

```TS
interface Light {
  currentState: LightState;
  offLight: LightState;
  weakLight: LightState;
  strongLight: LightState;
  setState(lightState: LightState): void;
}

interface LightState {
  light: Light;
  setLightState?(): void;
}
class ConcreteLightState implements LightState {
  light: Light;
  constructor(light: Light) {
    this.light = light;
  }
}
class OffLight extends ConcreteLightState {
  setLightState(): void {
    console.log('weak');
    this.light.setState(this.light.weakLight);
  }
}
class WeakLight extends ConcreteLightState {
  setLightState(): void {
    console.log('strong');
    this.light.setState(this.light.strongLight);
  }
}
class StrongLight extends ConcreteLightState {
  setLightState(): void {
    console.log('off');
    this.light.setState(this.light.offLight);
  }
}

class ConcreteLight implements Light {
  offLight: LightState;
  weakLight: LightState;
  strongLight: LightState;
  currentState: LightState;
  constructor() {
    this.offLight = new OffLight(this);
    this.weakLight = new WeakLight(this);
    this.strongLight = new StrongLight(this);
    this.currentState = this.offLight;
  }
  setState(lightState: LightState) {
    this.currentState = lightState;
  }
}

const light = new ConcreteLight();
light.currentState.setLightState(); // weak
light.currentState.setLightState(); // strong
light.currentState.setLightState(); // off
light.currentState.setLightState(); // weak
```

> https://juejin.cn/post/6844903862013280269#heading-6

### 策略

核心：允许在运行时选择算法的行为。

```TS
// 黄/绿/红 之间没有联系
interface Signal {
  yellow: () => void;
  green: () => void;
  red: () => void;
}

function yellow() {
  console.log('yellow');
}
function green() {
  console.log('green');
}
function red() {
  console.log('red');
}

const ActionWithSignal: Signal = {
  yellow,
  green,
  red
};

ActionWithSignal.yellow(); // green
ActionWithSignal.green();  // yellow
ActionWithSignal.red();    // red
```

> 策略模式和状态模式都是能让 `if-else` 变得优雅方式，但是两者的内核完全不一样，状态模式是内部状态有联系，一个状态会对另一个状态产生影响，而策略模式则是平行的逻辑。

### 访问者

熟悉 babel 的应该都懂，babel 的插件机制中就利用了访问者模式。

核心：在不修改对象结构的情况下向其中添加新的操作。

该模式中有两类元素，一类是访问者，一类是被访问的元素。访问者可以访问被访问的元素，并对其执行某些操作，而被访问的元素并不需要知道是哪个具体的访问者来访问自己，也不需要知道访问者将对自己进行哪些操作。

```TS
interface Visitor {
  visitElementA(element: ElementA): void;
  visitElementB(element: ElementB): void;
}

class ConcreteVisitor implements Visitor {
  visitElementA(element: ElementA): void {
    console.log(element.operationA());
  }

  visitElementB(element: ElementB): void {
    console.log(element.operationB());
  }
}

interface VisitedElement {
  accept(visitor: Visitor): void;
}

class ElementA implements VisitedElement {
  public operationA(): string {
    return 'ElementA operation';
  }

  public accept(visitor: Visitor): void {
    visitor.visitElementA(this);
  }
}

class ElementB implements VisitedElement {
  public operationB(): string {
    return 'ElementB operation';
  }

  public accept(visitor: Visitor): void {
    visitor.visitElementB(this);
  }
}

const elements: VisitedElement[] = [new ElementA(), new ElementB()];
const visitor: Visitor = new ConcreteVisitor();

elements.forEach((element: VisitedElement) => {
  element.accept(visitor);
});
// ElementA operation
// ElementB operation
```

看上去也比较简单，

- 访问者 visitor：内部有方法以 `被访问者` 为入参，之后可以对被访问者进行操作；
- 被访问者 element：设定一个 `接受访问者实例` 的接口，这个接口负责把 `被访问者自身` 传递给访问者

think a little bit，再 Vue 的响应式原理中 wather 和 Dep 之间关联的操作，好像都是被访问者，互相把自身传递给对方。

---

> 这篇文章内容主要来自 chatgpt~，重点是要掌握住编程思想。  
> 行为模式还有迭代器、模板方法、备忘录模式，此处省略。

