## AOP

#### 概念

把与核心业务逻辑无关的代码全部抽取出来，放置到某个地方集中管理，然后在运行时,动态地将代码切入到类的指定方法、指定位置上.用于切入到指定类指定方法的代码片段叫做切面,而切入到哪些类中的哪些方法叫做切入点.

AOP主要一般应用于签名验签、参数校验、日志记录、事务控制、权限控制、性能统计、异常处理等。

#### JDK动态代理

如果目标类实现了接口， Spring AOP会使用JDK动态代理。因为 JDK 动态代理生成的代理对象需要继承 `Proxy` 这个类，在 Java 中类只能是单继承关系，无法再继承一个代理类，所以只能基于接口代理。JDK动态代理步骤：

1. 创建被代理类及接口（JDK代理是接口代理）
2. 创建Handle类实现 InvocationHandler接口 ，重写invoke方法
3. 通过Proxy的newProxyInstance（）方法获取代理类对象
4. 通过代理类对象调用被代理类的方法

#### CGLIB动态代理

如果目标没有实现接口， Spring AOP会使用CGLIB动态代理

#### JDK与CGLIB的区别

两者都是在 JVM 运行时期新创建一个 Class 对象，实例化一个代理对象，对目标类（或接口）进行代理。JDK 动态代理只能基于接口进行代理，生成的代理类实现了这些接口；而 CGLIB 动态代理则是基于类进行代理的，生成的代理类继承目标类，但是不能代理被 final 修饰的类，也不能重写 final 或者 private 修饰的方法。

CGLIB 动态代理比 JDK 动态代理复杂许多，性能也相对比较差。

## IOC

#### 概念

控制反转，指由Spring容器管理bean的整个生命周期，通过反射实现对对象的控制，降低类之间的耦合度

#### IOC加载过程

1. BeanDefinitionReader读取配置类,解析成BeanDefinition
2. BeanDefinitionScanner扫描第一步读取到的配置类，确定需要注册的bean
3. BeanDefinitionRegistry将BeanDefinition注册到BeanDefinitionMap
4. ApplicationContext将BeanDefinitionMap传递给BeanFactory

#### Bean的生命周期

1. 实例化. bean实例化的时候从BeanDefinition中得到Bean的名字, 然后通过反射机制, 将Bean实例化. 实例化以后, 这是还只是个壳子, 里面什么都没有.
2. 属性赋值. 经过初始化以后, bean的壳子就有了, bean里面有哪些属性呢? 在这一步填充，包括依赖注入
3. 初始化. 初始化的时候, 会调用initMethod()初始化方法, destory()初始化结束方法这个时候, 类就被构造好了.
4. 构造好了的类, 会被放到IoC的一个Map中. Map的key是beanName, value是bean实例. 这个Map是一个单例池, 也就是我们说的一级缓存
5.  我们就可以通过getBean("user"), 从单例池中获取类名是user的类了.

#### BeanFactory和FactoryBean的区别

- BeanFactory是个Factory，也就是IOC容器或对象工厂，负责生产和管理Bean.
- FactoryBean是个Bean。在Spring中，所有的Bean都是由BeanFactory(也就是IOC容器)来进行管理的。但对FactoryBean而言，这个Bean不是简单的Bean，而是一个能生产或者修饰对象生成的工厂Bean,**即一个Bean A如果实现了FactoryBean接口，那么A就变成了一个工厂，根据A的名称获取到的实际上是工厂调用`getObject()`返回的对象**，而不是A本身，如果要获取工厂A自身的实例，那么需要在名称前面加上'`&`'符号

#### Spring解决循环依赖

**三级缓存**

![img](https://img-blog.csdnimg.cn/20200715095307401.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTIwOTgwMjE=,size_16,color_FFFFFF,t_70)

假设A依赖B， B依赖A

1. 实例化 A，此时 A 还未完成属性填充和初始化方法（@PostConstruct）的执行，A 只是一个半成品。
2. 为 A 创建一个 Bean 工厂，并放入到  singletonFactories 中。
3. 发现 A 需要注入 B 对象，但是一级、二级、三级缓存均为发现对象 B。
4. 实例化 B，此时 B 还未完成属性填充和初始化方法（@PostConstruct）的执行，B 只是一个半成品。
5. 为 B 创建一个 Bean 工厂，并放入到  singletonFactories 中。
6. 发现 B 需要注入 A 对象，此时在一级、二级未发现对象 A，但是在三级缓存中发现了对象 A，从三级缓存中得到对象 A，并将对象 A 放入二级缓存中，同时删除三级缓存中的对象 A。（注意，此时的 A 还是一个半成品，并没有完成属性填充和执行初始化方法）
7. 将对象 A 注入到对象 B 中。
8. 对象 B 完成属性填充，执行初始化方法，并放入到一级缓存中，同时删除三级缓存中的对象 B。（此时对象 B 已经是一个成品）
9. 对象 A 得到对象 B，将对象 B 注入到对象 A 中。（对象 A 得到的是一个完整的对象 B）
10. 对象 A 完成属性填充，执行初始化方法，并放入到一级缓存中，同时删除二级缓存中的对象 A。

- 三级缓存不仅可以暴露A对象早期bean还可以暴露A对象代理bean，如果A对象存在aop代理，则依赖的应该是A的代理对象，而不是原始的A对象。如果没有循环依赖，A对象是在初始化后生成代理A对象的，如果有循环依赖，则需要在填充属性前就生成代理A对象并暴露出去，所以在B对象从第三级缓存中拿取A早期对象时，如果A对象需要被代理(存在beanPostProcessors -> SmartInstantiationAwareBeanPostProcessor)，则提前生成代理对象。

## 事务

#### @Transactional的常用配置参数

- propagation,事务的传播行为，默认值为REQUIRED
  1. **PROPAGATION_REQUIRED:**如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新事务
  2. **PROPAGATION_REQUIRES_NEW**:创建一个新的事务，如果当前存在事务，则把当前事务挂起
  3. **PROPAGATION_NESTED**
  4. **PROPAGATION_MANDATORY**
- isolation,事务的隔离级别，默认值为DEFAULT
  1. ISOLATION_DEFAULT:使用与后端数据库默认的隔离级别
  2. ISOLATION_READ_UNCOMMITTED: 读未提交
  3. ISOLATION_READ_COMMITTED:读已提交
  4. ISOLATION_REPEATABLE_READ:可重复读
  5. ISOLATION_SERIALIZABLE: 可序列化

- timeout,事务超时时间，默认值为-1（不会超时）

  所谓事务超时，就是指一个事务所允许执行的最长时间，如果超过该时间限制但事务还没有完成，则自动回滚事务。在 `TransactionDefinition` 中以 int 的值来表示超时时间，其单位是秒，默认值为-1

- readOnly,指定事务是否为只读事务，默认为false

  如果你一次执行单条查询语句，则没有必要启用事务支持，数据库默认支持 SQL 执行期间的读一致性；

  如果你一次执行多条查询语句，例如统计查询，报表查询，在这种场景下，多条查询 SQL 必须保证整体的读一致性，否则，在前条 SQL 查询之后，后条 SQL 查询之前，数据被其他用户改变，则该次整体的统计查询将会出现读数据不一致的状态，此时，应该启用事务支持

- rollbackFor,用于指定能够触发事务回滚的异常类型，并且可以指定多个异常类型

  这些规则定义了哪些异常会导致事务回滚而哪些不会。默认情况下，事务只有遇到运行期异常（`RuntimeException` 的子类）时才会回滚，`Error` 也会导致事务回滚，但是，在遇到检查型（Checked）异常时不会回滚。