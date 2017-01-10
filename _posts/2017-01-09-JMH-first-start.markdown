---
layout: post
mathjax: true
title:  "JMH-Java微基准测试工具"
date:   2017-01-09 18:12:03 +0800
categories: jekyll update
header-img: "img/post-bg-rwd.jpg"
tags:
    - tutorial
    - jmh
    - jdk
---




JMH 是*Java Microbenchmark Harness* 的简称，其是一个工具用来实现java的微基准测试，这个工具是由开发java虚拟机的相关人员进行设计开发的，所以这群小伙伴知道他们做的是什么样的一款工具。

​	首先我们说一下为什么java的微基准测试很难，首先一个很大的应用程序，针对很其很小的一部分制定正确的性能测试标准，本身就很难。因为当我们独立的针对一个组件进行一定指标的性能测试的时候，有好多的基于硬件或者jvm的最优化调整，会影响我们的测试结果。而这些最优化调整可能在这个组件作为大型应用程序的一部分时，不会执行。因此不好的基准测试会使我们的性能测试结果不准。写一个好的微基准测试最典型的就是阻止真实的生产环境当中不可能执行的最优化，这也是JMH可以帮助我们的地方。下面我通过具体案例讲解一下JMH的使用技巧。

​	首先我们应该在我们的工程当中引入JMH的依赖，例如maven工程项目

```ruby
    <dependency>
        <groupId>org.openjdk.jmh</groupId>
        <artifactId>jmh-core</artifactId>
        <version>${jmh.version}</version>
    </dependency>
    <dependency>
        <groupId>org.openjdk.jmh</groupId>
        <artifactId>jmh-generator-annprocess</artifactId>
        <version>${jmh.version}</version>
        <scope>provided</scope>
    </dependency>
```


下面书写我们第一个benchmark例子，

```ruby
import org.openjdk.jmh.annotations.Benchmark;

public class MyBenchmark {

    @Benchmark
    public void testMethod() {
        // This is a demo/sample template for building your JMH benchmarks. Edit as needed.
        // Put your benchmark code here.
    }

}
```



这里的@Benchmark 注解表明testMethod是一个测试方法，这里有点类似Junit里面的@test 注解，下面我们添加一部分代码进行测试。

```ruby
import org.openjdk.jmh.annotations.Benchmark;

public class MyBenchmark {

    @Benchmark
    public void testMethod() {
        // This is a demo/sample template for building your JMH benchmarks. Edit as needed.
        // Put your benchmark code here.

        int a = 1;
        int b = 2;
        int sum = a + b;
    }

}
```

注意，这里可出的其实是一个关于benchmark的badcase ，因为jvm会分析到sum变量从未使用，因此可能会抛弃加法运算。事实上整个testMethod都会作为deadcode 被jvm丢弃。现在我们这里假想testMethod就是一个很好的benchmark方法实现，下面我会通过一步步的完善该方法，同时阐述如何使用JMH。

### Benchmark Modes

​	开始下一步之前，我们解释一下 JMH Benchmark Modes，JMH可以使用不同的模式运行我们的benchmark，而Benchmark Modes告诉JMH我们测量的是那些指标。框架本生默认提供的模式有以下

| Benchmark Modes  |                 caption                  |
| ---------------- | :--------------------------------------: |
| Throughput       |         测量每秒的操作数目，即每秒你的方法可以执行的次数         |
| Average Time     |           测量benchmark方法执行的平均耗时           |
| Sample Time      |      测试benchmark方法执行多长时间，包含最大值和最小值       |
| Single Shot Time | 测试benchmark方法一次执行多长时间，这是一个很好的测试其在冷启动的情况下（不会进行JVM预热优化） |
| All              |                 包含以上所有指标                 |

默认的测量指标是吞吐量，你可以通过将JMH 的 `BenchmarkMode` 注解放在你的benchmark方法上来指定你的测量指标，下面是一个指定`BenchmarkMode`的例子

```ruby
import org.openjdk.jmh.annotations.Benchmark;
import org.openjdk.jmh.annotations.BenchmarkMode;
import org.openjdk.jmh.annotations.Mode;

public class MyBenchmark {

    @Benchmark @BenchmarkMode(Mode.Throughput)
    public void testMethod() {
        // This is a demo/sample template for building your JMH benchmarks. Edit as needed.
        // Put your benchmark code here.

        int a = 1;
        int b = 2;
        int sum = a + b;
    }

}
```

注意Benchmark方法上面的注解`@BenchmarkMode(Mode.Throughput)` ,这个注解指示出该方法的测量指标，Mode类包含我们常用的几个测量指标.

###Benchmark Time Units

​	下面我们介绍一下如何指定JMH各个指标的输出时间单位，当然也是通过注解的方式。

```ruby
import org.openjdk.jmh.annotations.Benchmark;
import org.openjdk.jmh.annotations.BenchmarkMode;
import org.openjdk.jmh.annotations.Mode;
import org.openjdk.jmh.annotations.OutputTimeUnit;

import java.util.concurrent.TimeUnit;

public class MyBenchmark {

    @Benchmark @BenchmarkMode(Mode.Throughput) @OutputTimeUnit(TimeUnit.MINUTES)
    public void testMethod() {
        // This is a demo/sample template for building your JMH benchmarks. Edit as needed.
        // Put your benchmark code here.

        int a = 1;
        int b = 2;
        int sum = a + b;
    }

}
```

这个例子里，时间单位指定的是分钟，这意味着你想输出的的展示时间单位是分钟（每分钟的操作）

`TimeUnit` 包含以下单位常量

- NANOSECONDS    纳秒  $10^9​$ 秒

- MICROSECONDS   微妙  $10^6​$ 秒

- MILLISECONDS     毫秒   $10^3$ 秒 

- SECONDS               秒

- MINUTES                分钟

- HOURS                   小时

- DAYS                       天

  ​                         

### Benchmark State

有时你想针对你想初始化一部分数据变量作为benchmark测试方法的输入参数，但是又不想把这部分初始化过程作为测量指标的一部分，这一类变量我们称作状态变量，状态变量我们可以定义在状态类中，状态类可以被实例化作为一个参数传递给benchmark 方法，下面我们通过一个例子说明一下

```ruby
import org.openjdk.jmh.annotations.*;

import java.util.concurrent.TimeUnit;


public class MyBenchmark {

    @State(Scope.Thread)
    public static class MyState {
        public int a = 1;
        public int b = 2;
        public int sum ;
    }


    @Benchmark @BenchmarkMode(Mode.Throughput) @OutputTimeUnit(TimeUnit.MINUTES)
    public void testMethod(MyState state) {
        state.sum = state.a + state.b;
    }

}
```

这个例子当中我们提供了一个内部静态类`MyState`  ,其上方标有一个JMH注解 `@State`  , 这告诉JMH这是一个状态类，可以实例化用做测试方法的输入。注意测试方法已经修改为使用MyState作为输入。

#### State Scope

状态对象可以被测试方法多次调用时作为输入参数重复使用，JMH 提供了一个 `Scope`  状态对象可以重复使用的范围，状态对象通过使用`@State`  指定参数对象的使用范围，上面例子的使用范围是 `Scope.Thread` ,这表明每个线程都会实例化一次状态对象作为参数，Scope类包含以下几种常用的使用范围

| Scope     | caption                        |
| --------- | ------------------------------ |
| Thread    | 每个线程运行测试方法时，都会创建一个他自己的状态对象     |
| Group     | 每个线程组运行测试方法时，都会创建一个自己组内公用的状态对象 |
| Benchmark | 所有线程运行测试方法时，使用同一个通用的公共状态对象     |

​	

### State Class Requirements

一个 JMH 的状态类必须满足一下规则：

- 首先状态类必须是public
- 如果状态类是内部类，他必须被申明成静态类（e.g.  public static class ）
- 状态类必须拥有一个无参构造构造函数



当一个符合以上规则时，你可以通过一个注解 `@State`  指定该类作为测试方法可以使用的参数，默认为Scope指定为Scope.Thread ，通过注解指定后，JMH便可以识别该类了



### State Object @Setup and @TearDown

我们可以通过在状态类的相关方法上指定`@Setup`  和 `@TearDown`  指定状态的初始化和销毁操作，`@Setup` 表示该方法在作为参数传递前调用初始化状态，`@TearDown` 表示该方法在JMH执行测试方法（Benchmark方法）后，清理状态对象的相关状态或销毁。

 Setup和TearDown方法的执行时间不会包含在测量指标时间之内。

下面是一个例子介绍如何使用`@Setup`  和 `@TearDown`  注解

```ruby
import org.openjdk.jmh.annotations.*;

import java.util.concurrent.TimeUnit;


public class MyBenchmark {

    @State(Scope.Thread)
    public static class MyState {

        @Setup(Level.Trial)
        public void doSetup() {
            sum = 0;
            System.out.println("Do Setup");
        }

        @TearDown(Level.Trial)
        public void doTearDown() {
            System.out.println("Do TearDown");
        }

        public int a = 1;
        public int b = 2;
        public int sum ;
    }

    @Benchmark @BenchmarkMode(Mode.Throughput) @OutputTimeUnit(TimeUnit.MINUTES)
    public void testMethod(MyState state) {
        state.sum = state.a + state.b;
    }
```

注意内部类两个方法被注解为`@Setup`  和 `@TearDown`  ，当然案例中展示了两个方法，你可以添加更多的`@Setup`  和 `@TearDown` 在更多的方法上，如果你需要的话。注意上面注解当中拥有一个参数，这个参数告诉 JMH 如何调用该方法。该方法包含以下几种类型供使用

| Call Type        | caption                                  |
| ---------------- | ---------------------------------------- |
| Level.Trial      | 该方法被 一个完整的fork (包含预热和benchmark迭代)只调用一次 。 |
| Level.Iteration  | 该方法 每个 `benchmark迭代` 调用一次。               |
| Level.Invocation | 每个`benchmark方法` 一但调用都会执行一次。              |

如果还是很困惑关于参数，你可以在`@Setup` 和 `@TearDown`  方法内添加System.out.print 来观察具体参数 输出的时间，来理解不同参数的意义。

### Writing Good Benchmarks

现在我们已经理解如何写一个基于 JMH 的 Benchmark 例子，是时候讨论如何写一个好的Benchmark 例子了。作为上面最初提到的Benchmark例子其实是包含一系列你很容易掉进的陷阱当中。我下面的一些章节会讨论一些常见的陷阱。

一个通用的陷阱就是有些代码在正式的线上生产环境有可能不会进行的JVM动态调优操作，有可能在我们单独进行组件化测试时 JVM 会进行动态优化。这些优化使我们的测试结果看起来比真实的性能快好多，下面我会讨论一部分动态优化。

#### Loop Optimizations

临时的把Benchmark Code 放进一个循环内，从而使没次调用Benchmark method都执行好多次 Benchmark Code  ，而jvm 非常擅长最优化循环代码，因此你可能得到和你期望的不一样的测试结果。尽可能的不要在你的测试代码当中添加循环，除非循环本身就是测试代码的一部分。

#### Dead Code Elimination

另外一个在进行Benchmark method测试的时候，需要避免的一个jvm调优是死亡代码抛弃。当jvm监测到计算结果从未使用，jvm就会抛弃该死亡代码。下面是一个例子

```ruby
import org.openjdk.jmh.annotations.Benchmark;

public class MyBenchmark {

    @Benchmark
    public void testMethod() {
        int a = 1;
        int b = 2;
        int sum = a + b;
    }

}
```

jvm 侦测 a+b计算然后 赋值给sum 这段代码，由于sum从未没使用，因此jvm认为`int sum = a + b `  为死亡代码，因此该段代码会被抛弃。该Benchmark 方法没有任何返回结果，执行代码被抛弃，因此测试结果是错误不准确的，基准测试不会测试两个变量赋值，然后求和的性能，也就是最后基准测试其实什么也没有进行测试。

##### Avoiding Dead Code Elimination

避免死亡代码丢弃最好办法就是你必须使你的代码看起来不像死亡代码，有以下两种方法

- 你的Benchmark 方法必须返回结果
- 将Benchmark 方法的计算结果交由 JMH 提供的 `black hole`  持有

下面是以上两种不同方案的例子，首先我们看必须返回结果的

```ruby
import org.openjdk.jmh.annotations.Benchmark;

public class MyBenchmark {

    @Benchmark
    public int testMethod() {
        int a = 1;
        int b = 2;
        int sum = a + b;

        return sum;
    }

}
```

注意 测试方法添加了int类型返回值sum，这使jvm不能认为求和代码是死亡代码从而被抛弃，因为返回结果可能被调用者使用。JMH框架本身会通过一个tricking 使jvm看起来返回结果被使用。如果你的计算结果产生多个值，而不返回，代码也会被认为是死亡代码，从而被丢弃，你可以将其包装成一个对象包含两个参数 ，返回单个对象。

第二个方案是将计算结果值传递给 JMH 提供的 `black hole`  持有，下面我们给出一个例子

```ruby
import org.openjdk.jmh.annotations.Benchmark;
import org.openjdk.jmh.infra.Blackhole;

public class MyBenchmark {

    @Benchmark
   public void testMethod(Blackhole blackhole) {
        int a = 1;
        int b = 2;
        int sum = a + b;
        blackhole.consume(sum);
    }
}
```

我们看到测试方法当中通过传递给测试方法`blackhole` ,然后调用其consumer方法进行持有，如果产生多个值，可以调用多次consumer方法进行持有。

#### Constant Folding

常量叠算（常量折叠），这里考虑到场景还是认为叠算更适合，这也是jvm常用的一种优化，当jvm侦测到我们的某个计算逻辑是基于常亮进行的，那么计算结果经常会返回相同的结果，jvm也许会用计算结果替换整个计算逻辑。下面我们看一个例子

```ruby
import org.openjdk.jmh.annotations.Benchmark;

public class MyBenchmark {

    @Benchmark
    public int testMethod() {
        int a = 1;
        int b = 2;
        int sum = a + b;

        return sum;
    }

}
```



jvm 也许会侦测到sum 是由两个常量 1 和 2 进行计算得到（由变量a,b持有），因此以上代码可能被替换成下面这样

```ruby
import org.openjdk.jmh.annotations.Benchmark;

public class MyBenchmark {

    @Benchmark
    public int testMethod() {
        int sum = 3;

        return sum;
    }

}
```

甚至直接返回3，jvm 会在所有调用testMethod的地方inline替换成常量 3，甚至都不需要调用testMethod方法。

避免这种常量替换的最好办法是使用Benchmark状态对象作为参数，这会使jvm无法识别测试方法时基于常量计算返回结果。如下面案例

```ruby
import org.openjdk.jmh.annotations.*;

public class MyBenchmark {

    @State(Scope.Thread)
    public static class MyState {
        public int a = 1;
        public int b = 2;
    }


    @Benchmark 
    public int testMethod(MyState state) {
        int sum = state.a + state.b;
        return sum;
    }
}
```

另外记住如果你的测试方法传递多个值，可以使用JMH 提供的`blackhole`  如下面案例

```ruby
 @Benchmark
    public void testMethod(MyState state, Blackhole blackhole) {
        int sum1 = state.a + state.b;
        int sum2 = state.a + state.a + state.b + state.b;

        blackhole.consume(sum1);
        blackhole.consume(sum2);
    }
```



JMH的精度可以精确到微秒级。可以看出 JMH 主要使用在当你已经找出了热点函数，而需要对热点函数进行进一步的优化时，就可以使用 JMH 对优化的效果进行定量的分析。

比较典型的使用场景还有：

- 想定量地知道某个函数需要执行多长时间，以及执行时间和输入 n 的相关性
- 一个函数有两种不同实现（例如实现 A 使用了 FixedThreadPool，实现 B 使用了 ForkJoinPool），不知道哪种实现性能更好
- 测试具体方法执行的吞吐量，平均时间、最大执行时间，最小执行时间等等指标，



下面我们给出一个完整的可以运行的例子，但是首先你必须构建一个maven工程项目，引入本文开始所说的jmh依赖。下面是jmh使用例子

```ruby
@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.MICROSECONDS)
@State(Scope.Thread)
public class FirstBenchmark {

    @Benchmark
    public int sleepAWhile() {
        try {
            Thread.sleep(500);
        } catch (InterruptedException e) {
            // ignore
        }
        return 0;
    }

    public static void main(String[] args) throws RunnerException {
        Options opt = new OptionsBuilder()
                .include(FirstBenchmark.class.getSimpleName())
                .forks(1)
                .warmupIterations(5)
                .measurementIterations(5)
                .build();

        new Runner(opt).run();
    }
}
```

我们知道Thread.sleep的唤醒时间并不是很精确的，我们这里主要测试一下相关指标，这里时间单位我们选择微秒，我们直接运行main方法

```ruby
# JMH 1.14.1 (released 112 days ago)
# VM version: JDK 1.8.0_112, VM 25.112-b15
# VM invoker: C:\Program Files\Java\jdk1.8.0_112\jre\bin\java.exe
# VM options: -Didea.launcher.port=7533 -Didea.launcher.bin.path=D:\Program Files\JetBrains\IntelliJ IDEA 2016.3\bin -Dfile.encoding=UTF-8
# Warmup: 5 iterations, 1 s each
# Measurement: 5 iterations, 1 s each
# Timeout: 10 min per iteration
# Threads: 1 thread, will synchronize iterations
# Benchmark mode: Average time, time/op
# Benchmark: com.jd.si.netty.jmh.FirstBenchmark.sleepAWhile

# Run progress: 33.32% complete, ETA 00:01:01
# Fork: 1 of 2
# Warmup Iteration   1: 500315.032 us/op
# Warmup Iteration   2: 500048.467 us/op
# Warmup Iteration   3: 499869.711 us/op
# Warmup Iteration   4: 500410.753 us/op
# Warmup Iteration   5: 499802.428 us/op
Iteration   1: 499801.858 us/op
Iteration   2: 500665.630 us/op
Iteration   3: 503949.159 us/op
Iteration   4: 499847.758 us/op
Iteration   5: 500019.102 us/op

# Run progress: 49.98% complete, ETA 00:00:45
# Fork: 2 of 2
# Warmup Iteration   1: 500000.142 us/op
# Warmup Iteration   2: 499981.469 us/op
# Warmup Iteration   3: 499874.842 us/op
# Warmup Iteration   4: 499813.974 us/op
# Warmup Iteration   5: 499986.600 us/op
Iteration   1: 499849.754 us/op
Iteration   2: 499957.663 us/op
Iteration   3: 499827.944 us/op
Iteration   4: 499902.782 us/op
Iteration   5: 499875.840 us/op


Result "sleepAWhile":
  500369.749 ±(99.9%) 1939.622 us/op [Average]
  (min, avg, max) = (499801.858, 500369.749, 503949.159), stdev = 1282.940
  CI (99.9%): [498430.127, 502309.371] (assumes normal distribution)

```

通过测试结果我们可以看到，sleep的唤醒时间，早测试机上基本上都在499ms左右