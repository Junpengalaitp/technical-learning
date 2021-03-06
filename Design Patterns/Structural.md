# 适配器(Adapter)

## 例子
* Java多线程中的RunnableFuture接口，同时继承Runnable和Future接口。在调用线程池的submit(Callable)方法后转化为这个对象，使之可以通过线程池的execute(Runnable)方法执行。
# 桥梁(Bridge)

# 组合(Composite)
## 意图
* 可以使用它将对象组合成树状结构，并且能像使用独立对象一样使用它们
* 如何应用的核心模型能用树状结构表示，在应用中使用组合模式才有价值
## 使用场景
* 需要实现树状对象结构，可以使用组合模式
* 希望客户端代码以相同方式处理简单和复杂元素，可以使用该模式
## 实现方式
1. 确保应用的核心模型能够以树状结构表示。尝试将其分解为简单元素和容器，容器必须能够同时包含简单元素和其他容器。
2. 声明组件接口及其一系列方法，这些方法对简单和复杂元素都有意义。
3. 创建一个叶节点表示简单元素。可以有多个不同叶节点类。
4. 创建一个容器表示复杂元素。在该类中，创建一个数组成员变量来存储对于其子元素得到引用。该数组必须能够同时保存叶节点和容器。实现组件接口方法时，容器应该将大部分工作交给其子元素来完成。
5. 在容器中定义和添加删除子元素的方法
## 优缺点
* 优点
  * 利用多态和递归更方便地使用复杂树结构
  * 开闭原则，无需更改现有代码，就可以添加新元素，使其成为对象树的一部分
* 缺点
  * 对于功能差异较大的类，提供公共接口或许比较困难。有时需要过度一般化组件接口，使其变得令人难以理解

## 与其他模式的关系
* 和桥接模式、状态模式、策略模式和适配器模式的接口非常相似。它们都基于组合模式，即将工作委派给其他对象。
* 可以在创建复杂树是使用生成器模式，可使其构造步骤以递归的方式运行。
* 责任链模式通常和组合模式结合使用。这种情况下，叶组件接收到请求后，可以将请求递归传递到对象树底部
* 你可以使用迭代器模式来遍历组合树。
* 你可以使用访问者模式对整个组合树执行操作。
* 你可以使用享元模式实现组合树的共享叶节点以节省内存。
* 组合和装饰模式的结构图很相似， 因为两者都依赖递归组合来组织无限数量的对象。
* 装饰类似于组合， 但其只有一个子组件。 此外还有一个明显不同： 装饰为被封装对象添加了额外的职责， 组合仅对其子节点的结果进行了 “求和”。
  但是， 模式也可以相互合作： 你可以使用装饰来扩展组合树中特定对象的行为。
* 大量使用组合和装饰的设计通常可从对于原型模式的使用中获益。 你可以通过该模式来复制复杂结构， 而非从零开始重新构造。

# 装饰(Decorator)
## 意图
* 通过将对象放入包含行为的特殊封装对象中来为原对象绑定新的行为
* 当需要更改一个对象得到行为时，第一想法是扩展它所属的类，但是继承也有可能带来严重的问题
  * 继承是静态的。你无法在运行时更改已有对象得到行为，只能通过由不同子类创建的对象来替代当前整个对象。
  * 子类只能有一个父类。大部分编程语言不允许多继承。
* 其中一种方法是使用聚合或组合，而不是继承。一个对象包含指向另一个对象的引用，并将部分工作委派给引用对象。
  * 例子: DDD中的值对象、实体类和聚合
* 当封装器对象与它所包含的对象继承同一个接口时，就可称作装饰器

## 应用场景
* 无需修改代码的情况下即可使用对象，且希望在运行时为对象新增额外的行为
  * 装饰能将业务逻辑组织为层次结构，可为各层创建一个装饰，在运行时将各种不同逻辑组合成对象。因为这些对象都遵循通用接口，客户端代码能以相同的方式使用这些对象。
* 如果用继承来扩展对象行为的方案难以实现或者根本不可行，可以使用装饰器模式
  * 很多编程语言使用final关键字禁止对某个类的进一步扩展。

## 优缺点
* 优点
  * 无需创建新子类就可扩展对象的行为
  * 可以在运行时添加或删除对象的功能
  * 你可以用多个装饰对象来组合几种行为
  * 单一职责原则。可以将实现了许多不同行为的一个大类拆分成多个小类
* 缺点
  * 在封装器栈中删除特定封装器比较困难
  * 实现行为不受装饰栈顺序影响的装饰比较困难
  * 各层的初始化配置代码看上去可能会糟糕

## 例子
* Java线程池中BlockingQueue的实现PriorityBlockingQueue是个无界队列，即使设置了初始容量，当队列满时，会触发tryGrow()方法扩充容量。因此会导致线程池的拒绝策略无法触发，造成OOM。
  * 使用装饰器模式继承BlockingQueue接口, 将PriorityBlockingQueue的一个实例作为参数传入，除了offer()方法之外的实现直接调用该实例的对应方法。offer()方法会对比当前队列长度和容量，如果长度超过容量会返回false，使得拒绝策略可以正常触发。
  * PriorityBlockingQueue的size()方法只需要O(1)时间，因为内部使用了AtomicInteger来记录size。
* ThreadPoolExecutor实现类本身只接受runnable的类型，它可以使用submit(Callable)是其父类AbstractExecutorService实现的, 该方法会将Callable经过newTaskFor()方法转化成一个RunnableFuture类(就是一个接口同时继承了Runnable, Future接口)。再将RunnableFuture作为Runnable提交给ThreadPoolExecutor。
  * 因为newTaskFor()的转换，原本的Callable如果继承的其它的接口(比如说Comparable)类型会丢失，所以需要重写newTaskFor()方法以及继承FutureTask来创造一个ComparableFutureTask
  * 重写newTaskFor()会返回ComparableFutureTask对象。
  * ComparableFutureTask算是装饰者模式，将继承了Comparable的Callable作为变量传入，然后ComparableFutureTask的CompareTo()方法使用这个变量的compareTo()方法比较
# 门面(Facade)

# 享元(Flyweight)

# 代理(Proxy)

## 意图
* 能够提供对象的代替品或其占位符。代理控制着对于原对象的访问，并允许在将请求提交给对象前后进行统一处理。

## 问题
* 为什么要控制对于某个对象的访问呢？ 举个例子： 有这样一个消耗大量系统资源的巨型对象， 你只是偶尔需要使用它， 并非总是需要。
* 你可以实现延迟初始化： 在实际有需要时再创建该对象。 对象的所有客户端都要执行延迟初始代码。 不幸的是， 这很可能会带来很多重复代码。
* 在理想情况下， 我们希望将代码直接放入对象的类中， 但这并非总是能实现： 比如类可能是第三方封闭库的一部分。

## 解决方案
* 代理模式建议新建一个与原服务对象接口相同的代理类， 然后更新应用以将代理对象传递给所有原始对象客户端。 代理类接收到客户端请求后会创建实际的服务对象， 并将所有工作委派给它。

## 应用场景
* 延迟初始化 （虚拟代理）。 如果你有一个偶尔使用的重量级服务对象， 一直保持该对象运行会消耗系统资源时， 可使用代理模式。
  * 你无需在程序启动时就创建该对象， 可将对象的初始化延迟到真正有需要的时候。
* 访问控制 （保护代理）。 如果你只希望特定客户端使用服务对象， 这里的对象可以是操作系统中非常重要的部分， 而客户端则是各种已启动的程序 （包括恶意程序）， 此时可使用代理模式。
  * 代理可仅在客户端凭据满足要求时将请求传递给服务对象。

* 本地执行远程服务 （远程代理）。 适用于服务对象位于远程服务器上的情形。
* 在这种情形中， 代理通过网络传递客户端请求， 负责处理所有与网络相关的复杂细节。

* 记录日志请求 （日志记录代理）。 适用于当你需要保存对于服务对象的请求历史记录时。 代理可以在向服务传递请求前进行记录。
* 缓存请求结果 （缓存代理）。 适用于需要缓存客户请求结果并对缓存生命周期进行管理时， 特别是当返回结果的体积非常大时。
* 代理可对重复请求所需的相同结果进行缓存， 还可使用请求参数作为索引缓存的键值。
* 智能引用。 可在没有客户端使用某个重量级对象时立即销毁该对象。
* 代理会将所有获取了指向服务对象或其结果的客户端记录在案。 代理会时不时地遍历各个客户端， 检查它们是否仍在运行。 如果相应的客户端列表为空， 代理就会销毁该服务对象， 释放底层系统资源。
* 代理还可以记录客户端是否修改了服务对象。 其他客户端还可以复用未修改的对象。

### 实现方式
* 如果没有现成的服务接口， 你就需要创建一个接口来实现代理和服务对象的可交换性。 从服务类中抽取接口并非总是可行的， 因为你需要对服务的所有客户端进行修改， 让它们使用接口。 备选计划是将代理作为服务类的子类， 这样代理就能继承服务的所有接口了。

* 创建代理类， 其中必须包含一个存储指向服务的引用的成员变量。 通常情况下， 代理负责创建服务并对其整个生命周期进行管理。 在一些特殊情况下， 客户端会通过构造函数将服务传递给代理。

* 根据需求实现代理方法。 在大部分情况下， 代理在完成一些任务后应将工作委派给服务对象。

* 可以考虑新建一个构建方法来判断客户端可获取的是代理还是实际服务。 你可以在代理类中创建一个简单的静态方法， 也可以创建一个完整的工厂方法。

* 可以考虑为服务对象实现延迟初始化。


## 优缺点
* 优点
  * 你可以在客户端毫无察觉的情况下控制服务对象。
  * 如果客户端对服务对象的生命周期没有特殊要求， 你可以对生命周期进行管理。
  * 即使服务对象还未准备好或不存在， 代理也可以正常工作。
  * 开闭原则。 你可以在不对服务或客户端做出修改的情况下创建新代理。
* 缺点
  * 代码可能会变得复杂， 因为需要新建许多类。
  * 服务响应可能会延迟。

## 与其他模式的关系
* 适配器模式能为被封装对象提供不同的接口， 代理模式能为对象提供相同的接口， 装饰模式则能为对象提供加强的接口。

* 外观模式与代理的相似之处在于它们都缓存了一个复杂实体并自行对其进行初始化。 代理与其服务对象遵循同一接口， 使得自己和服务对象可以互换， 在这一点上它与外观不同。

* 装饰和代理有着相似的结构， 但是其意图却非常不同。 这两个模式的构建都基于组合原则， 也就是说一个对象应该将部分工作委派给另一个对象。 两者之间的不同之处在于代理通常自行管理其服务对象的生命周期， 而装饰的生成则总是由客户端进行控制。