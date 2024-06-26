# 重学设计模式 -- 创建型


设计模式其实并不是多么高大尚的东西，都是码农届前辈积累下来的一些编程经验，在工作中可以说随处可见，作为前端开发者，学习这些"套路"很有必要，跟着 [desing-patterns](https://refactoringguru.cn/design-patterns) 再来回顾一下吧。

## 创建型模式

首先先了解下常用的工厂模式，又分为 「工厂方法模式」 和 「抽象工厂模式」。

### 工厂方法模式

一句话：父类工厂提供创建对象的方法，由子类去决定实例化什么样的对象。

- 意义：可以在子类中重写工厂方法， 从而改变其创建「产品」的类型
- 注意：仅当「产品」具有共同的基类或者接口时， 子类才能返回不同类型的「产品」

> 工厂方法返回的对象被称作 “产品”

想象一下 Audi 的工厂，一开始只生产 A3，每台车出厂前都要做一次最高时速测试。随着发展，后面又要生产 A4，A5...省略不了的步骤是 -- 出厂前最高时速测试，很明显这是可以与 A3 共用测试车间，但同时又可以用不同的测试方案，这就可以利用起 「工厂方法」 模式了。

```TS
/* ---------- 生产类 -- 提供 「抽象工厂方法」 ---------- */
abstract class Produce {
  /**
   * 抽象工厂方法
   */
  abstract produceAudi(): Car;
  /**
   * 重点是:  1. 一些业务逻辑是依赖于工厂方法返回的产品
   *         2. 这些产品都有相同的业务逻辑
   */
  fastSpeed(): void {
    const car = this.produceAudi();
    car.run();
  }
}
/**
 * 实现具体工厂方法
 */
class ProduceA3 extends Produce {
  produceAudi() {
    return new A3();
  }
}
class ProduceA4 extends Produce {
  produceAudi() {
    return new A4();
  }
}

/* ---------- 产品类 -- 具体对象的创建和业务逻辑的重写 ---------- */
type Model = 'A3' | 'A4' | 'A5';
type Engine = 'EA888' | 'EA777' | 'EA666';

interface Car {
  model: Model;
  engine: Engine;
  run: () => void;
}

class A3 implements Car {
  model: Model = 'A3';
  engine: Engine = 'EA666';
  run(): void {
    console.log(
      `${this.model} runs with ${this.engine}， testing maximum speed..., wow is 200km/h`
    );
  }
}
class A4 implements Car {
  model: Model = 'A4';
  engine: Engine = 'EA888';
  run(): void {
    console.log(
      `${this.model} runs with ${this.engine}， testing maximum speed..., wow is 280km/h`
    );
  }
}

const A3ins = new ProduceA3(); // A3 runs with EA666， testing maximum speed..., wow is 200km/h
A3ins.fastSpeed();
const A4ins = new ProduceA4(); // A4 runs with EA888， testing maximum speed..., wow is 280km/h
A4ins.fastSpeed();
```

不用太在意具体的方法实现，抓住重点思想：

1. when use：**当相似业务逻辑依赖于不同类型的产品时，就可以使用「工厂方法模式」了**
2. how use：父类提供创建对象的工厂方法，子类可以重写覆盖

### 抽象工厂模式

核心：创建一系列相关的对象，而无需指定其具体类。

举个例子，一辆 Audi A3 由很底盘，方向盘，发动机，轮子等等一系列零件组成，需要在流水线上一步步安装，很好 A4,A5 也需要同样的流程，那么此时就可以使用抽象工厂了。

```TS
/**
 * 抽象工厂接口 返回不同 「抽象产品」 的方法
 */
interface AudiFactory {
  createEngine(): Engine;
  createWheel(): Wheel;
  // ...
}

class A3Factory implements AudiFactory {
  createEngine() {
    return new EA666();
  }
  createWheel(): Wheel {
    return new Michelin();
  }
}

/**
 * 『抽象零件』
 */
interface Engine {
  useEnginType(): string;
}
interface Wheel {
  useWheelType(): string;
}


class EA666 implements Engine {
  useEnginType(): string {
    return 'EA666';
  }
}
class Michelin implements Wheel {
  useWheelType(): string {
    return 'michelin';
  }
}
```

上方代码，当拓展生产 A4，A5 的时候就很容易了，不用对抽象工厂进行修改，直接拓展个新类即可，同时原来的生产线材料有些也可以共用，比如轮子。

掌握思想：主要针对的是 「系列」 相关的对象，或者经常换代升级的。

### 生成器模式

核心：分步骤创建复杂对象。

```TS
interface Builder {
  producePartA(): void;
  producePartB(): void;
  producePartC(): void;
  // ... more and more
}

class ConcreteBuilder implements Builder {
  private product: Product;
  constructor() {
    this.product = new Product();
  }
  producePartA(): void {
    this.product.parts.push('partA');
  }
  producePartB(): void {
    this.product.parts.push('partB');
  }
  producePartC(): void {
    this.product.parts.push('partC');
  }
  getProduct(): Product {
    return this.product;
  }
}

class Product {
  parts: string[] = [];
  listParts() {
    console.log(`Product parts: ${this.parts.join(', ')}\n`);
  }
}

const builder = new ConcreteBuilder();
builder.producePartA();
builder.producePartC();
builder.getProduct().listParts(); // Product parts: partA, partC
```

可以看出，生成器模式就是一堆零件摆在面前需要什么用什么，可以能帮助我们处理构造函数入参过多带来的混乱问题。

### 原型模式

JavaScript 天然具有原型链。TypeScript 的 Cloneable （克隆） 接口就是立即可用的原型模式。

```TS
class Prototype {
   public clone(): this {
        const clone = Object.create(this);
        clone.xx = xx
        /**
         * ...一系列对clone对象的增强, 最后把 clone 返回出去
         */
        return clone
   }
}
```

### 单例模式

核心：保证一个类只有一个实例。类似于 windows 的回收站，只能创建打开一个窗口。

这个前端应该很熟悉了，比如 dialog 弹窗同理。

```TS
class Singleton {
  private static instance: Singleton;
  public static getInstance(): Singleton {
    if (!Singleton.instance) {
      Singleton.instance = new Singleton();
    }
    return Singleton.instance;
  }
  // ...其他业务逻辑
}

// test
const a = Singleton.getInstance()
const b = Singleton.getInstance()
console.log(a === b ? 1 : 2); // 1
```

逻辑也比较简单，私有静态实例属性 `instance` 用来存储实例，`getInstance` 返回唯一的实例。

