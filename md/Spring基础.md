# Spring基础

## 什么是Spring

### 1.为了解决企业级开发的复杂性，简化java开发

### 2.为企业级开发提供丰富的功能，这些功能的底层都依赖两个核心特性，也就是依赖注入和面向切面

## spring有哪些组成模块

### 如图：

### 1.spring core

- 提供了框架的基本组成部分，包括控制反转(IOC)和依赖注入(DI)

### 2.spring beans

- 提供了BeanFactory，是工厂模式的一个经典实现，Spring将管理对象称为Bean

### 3.spring context

- 构建于core封装包基础上的context封装包，提供框架式的对象访问方法

### 4.spring jdbc

- 提供jdbc的抽象层，消除了繁琐的jdbc编码和数据库厂商特有的错误代码解析，用于简化jdbc

### 5.spring aop

- 提供了面向切面编程的实现，让你可以自定义拦截器、切点等

### 6.spring web

- 提供针对web开发的集成特性，例如文件上传，利用servlet listeners进行ioc容器初始化和针对web的applicationContext

### 7.spring test

- 主要为测试提供支持，支持使用junit和testNG对spring组件进行单元测试和集成测试

## spring框架用到的设计模式

### 工厂模式

- BeanFactory就是简单工厂模式的体现，用来创建对象的实例

### 单例模式

- Bean默认为单例模式

### 代理模式

- spring的aop功能用到了JDK的动态代理和CGLIB字节码生成技术

### 模板方法

- 用来解决代码重复的问题。比如RestTemplate、JmsTemplate、JpaTemplate

### 观察者模式

- 定义对象键一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都会得到通知被制动的更新，如Spring中listener的实现-ApplicationListener

## spring context应用上下文切换

### 基本的spring模块，提供spring框架的基本功能，BeanFactory是任何spring为基础的应用的核心。spring框架简历在此模块之上，它使spring成为一个容器

### Bean工厂是工厂模式的一个实现，提供了控制反转功能，用来把应用的配置和依赖从应用代码中分离。最常用的就是XmlBeanFactory，他根据Xml文件中定义加载beans。该容器从XML文件读取配置元数据并用它去创建一个完全配置的系统或应用

## spring框架中有那些不同类型的事件

### 1.上下文更新事件(ContextRefreshedEvent)

- 在调用ConfigurableApplicationContext接口中的refresh()方法时触发

### 2.上下文开始事件(ContextStartedEvent)

- 当容器调用ConfigurableApplicationContext的start()方法时触发

### 3.上下文停止事件(ContextStopedEvent)

- 当容器调用ConfigurableApplicationContext的stop()方法时触发

### 4.上下文关闭事件(ContextClosedEvent)

- 当ApplicationContext被关闭时触发该事件。容器被关闭时，其管理所有的单例bean都会被销毁

### 5.请求处理事件(RequestHandledEvent)

- 在web应用中，当一个http请求(request)结束时触发该事件。如果一个bean实现了ApplicationListener接口，当一个ApplictionEvent被发布后，bean会被自动通知

### 提示：有关容器各种事件动作，我们完全可以使用实现ApplicationListener接口实现做处理

## Spring控制反转(IOC)

### 1.简介：
控制反转，把传统意义上的直接操控调用对象的调用权交给容器，通过容器来实现对象组件的装配和管理。所谓控制反转就是对对象控制权转移，从程序代码本身转移到外部容器

- 优点

	- 1.管理对象创建和依赖关系维护
	- 2.解耦，由容器去维护具体的对象

### 2.IOC的机制

- IOC的实现机制就是工厂模式加反射机制

	- 示例

	  interface Fruit {
	     public abstract void eat();
	   }
	  
	  class Apple implements Fruit {
	      public void eat(){
	          System.out.println("Apple");
	      }
	  }
	  
	  class Orange implements Fruit {
	      public void eat(){
	          System.out.println("Orange");
	      }
	  }
	  
	  class Factory {
	      public static Fruit getInstance(String ClassName) {
	          Fruit f=null;
	          try {
	              f=(Fruit)Class.forName(ClassName).newInstance();
	          } catch (Exception e) {
	              e.printStackTrace();
	          }
	          return f;
	      }
	  }
	  
	  class Client {
	      public static void main(String[] a) {
	          Fruit f=Factory.getInstance("io.github.dunwu.spring.Apple");
	          if(f!=null){
	              f.eat();
	          }
	      }
	  }
	  
	  

### 3.支持的功能

- 1.依赖注入
- 2.依赖检查
- 3.自动装配
- 4.支持集合
- 5.支持初始化方法和销毁方法
- 6.支持回调某些方法

### 问题

- 1.BeanFactory和ApplicationContext的区别？

	- 最主要的就是ApplicationContext是BeanFactory的子接口

		- 详细区别扩展

			- 1.BeanFactory是Spring最底层的接口，包含了各种bean的定义，读取bean配置文件，管理bean的加载、实例化，控制bean的生命周期，维护bean之间的依赖关系

ApplicationContext是BeanFactory的派生接口，提供了BeanFactory所具有的功能之外，还提供了其他功能：
①继承MessageSource，支持国际化
②统一的资源访问方式
③提供在监听器中注册bean的事件
④同时加载多个配置文件
⑤载入多个（继承关系）上下文，使得每一个上下文独有专注于一个特定的层次，比如应用的web层
			- 2.BeanFactory采用的是延迟加载的方式来注入bean(调用getBean方法)。

ApplicationContext容器在容器启动时一次性创建完所有bean。
			- 3.BeanFactory通常以编程的方式创建，ApplicationContext还能以声明的方式创建，如使用ContextLoader
			- 4.BeanFactory和ApplicationContext都支持BeanPostProcessor、BeanFactoryPostProcessor的使用，BeanFactory需要手动注册，ApplicationContext是自动注册

- 2.BeanFactory和ApplicationContext关系详解

	- BeanFactory

		- 低级容器，其就是维护一个HashMap，key是bean的name，value是bean的实例。

	- ApplicationContext

		- 高级容器，比BeanFactory多了很多功能，例如资源获取、支持多种消息。已经不能简单的称为Bean工厂了，而是"应用上下文"。接口定义的refresh()方法用于刷新整个容器，即重新加载刷新所有bean，是阅读spring源码的入口。

	- 小结：
低级容器只需要加载配置文件解析成BeanDefinition放入map容器中。getbean的时候从map找到对应的BeanDefinition所属的map里拿出Class对象进行实例化，有依赖关系将递归调用getBean方法完成依赖注入。

ApplicationContext包含了低级容器的所有功能支持不同信息源头，支持 BeanFactory 工具类，支持层级容器，支持访问文件资源，支持事件发布通知，支持接口回调等等。

- 3.ApplicationContext的实现有哪些？

	- 1.AnnotationConfigApplicationContext

		- 通过注解加载beans的定义

	- 2.ClassPathXmlApplicationContext

		- 从XML文件中加载Beans的定义

- 4.什么是Spring的依赖注入？

	- 组件之间的依赖关系由容器在应用系统运行期来决定，由容器动态地将某种依赖关系的目标对象实例注入到应用系统中的各个关联组件中。组件不做定位查询，指提供普通的Java方法让容器去决定依赖关系。

- 5.依赖注入的基本原则是什么？

	- 应用组件不应该负责查找资源或者其他依赖的协作对象。配置对象的工作应该由IOC容器负责，“查找资源”的逻辑应该从应用组件的代码中抽取出来，交由给IOC容器负责。容器全权负责组件的装配。

- 6.依赖注入由哪几种方式？

	- 1.属性注入
	- 2.构造器注入
	- 3.setter方法注入

- 7.构造器注入和set方法注入的区别？

	- 

- 8.Spring支持的Bean作用域有哪些？

	- 1.singleton

		- bean在每个spring ioc容器中只有一个实例

	- 2.prototype

		- 多实例，每次注入都是新的对象

			- 频繁创建销毁，影响性能

	- 3.request

		- 每次http请求都会创建一个新的bean，该作用域仅基于web的Spring ApplicationContext情形下有效

	- 4.session

		- 在一个HTTP Session中，一个bean定义对应的实例。该作用域仅在基于web的Spring ApplicationContext情形下有效

	- 5.global-session

		- 在一个全局的HTTP Session中，一个bean定义对应一个实例。该作用域仅在基于web的Spring ApplicationContext情形下有效

- 9.Spring框架中的单例bean是线程安全的吗？

	- 不是线程安全的，Spring bean默认是单例模式，Spring框架没有对单例bean进行多线程的封装处理。

- 10.Spring如何处理线程并发问题？

	- 1.把bean的作用域变为prototype
	- 2.使用ThreadLocal

- 11.什么是循环依赖？

	- 1.A的构造方法中依赖了B的实例对象，同时B的构造方法中依赖了A的实例对象

		- 这种循环依赖Spring也没办法搞定

	- 2.A的构造方法中依赖了B的实例对象，同时B的某个field或者setter需要A的实例对象，以及反之
	- 3.A的某个field或者setter依赖了B的实例对象，同时B的某个field或者setter依赖了A的实例对象，以及反之

- 12.Spring怎么解决循环依赖？(重点，必考)

	- 首先，我们需要知道创建对象的步骤

		- 1.createBeanInstance， 实例化，实际上就是调用对应的构造方法构造对象
		- 2.populateBean，填充属性
		- 3.initializeBean，调用spring xml中指定的init方法，或者AfterPropertiesSet方法

	- 会发生循环依赖的步骤集中在第一步和第二步，对于单例对象来说，在Spring的整个容器的生命周期内，有且只存在一个对象，很容易想到这个对象应该存在Cache中，Spring大量运用了Cache的手段，在循环依赖问题的解决过程中甚至使用了“三级缓存”。“三级缓存”主要是指：

	  /** Cache of singleton objects: bean name --> bean instance */
	  private final Map<String, Object> singletonObjects = new ConcurrentHashMap<String, Object>(256);
	  /** Cache of singleton factories: bean name --> ObjectFactory */
	  private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<String, ObjectFactory<?>>(16);
	  /** Cache of early singleton objects: bean name --> bean instance */
	  private final Map<String, Object> earlySingletonObjects = new HashMap<String, Object>(16);
	  

		- singletonObjects指单例对象的cache（第一级缓存）
		- singletonFactories指单例对象工厂的cache（第三级缓存）
		- earlySingletonObjects指提前曝光的单例对象的cache（第二级缓存）
		- Spring就用这三级缓存巧妙的解决了循环依赖问题

	- Bean创建的过程中，Spring会先从缓存中获取，这个缓存就是以及缓存(singletonObjects)，主要调用的方法getSingleton，分析getSingleton的过程，Spring首先从一级缓存(singletonObjects)中尝试获取，如果获取不到并且对象在创建中，则尝试从二级缓存(earlySingletonObject)中获取，如果还是获取不到，就从三级缓存中(singletonFactory.getObject())获取。
	- 例子：在populateBean()给属性赋值阶段里面Spring会解析属性，并且赋值，当发现，A对象里面依赖了B，此时会走getBean方法，如果这个时候B发现有对A的依赖可以直接从缓存中去拿。因为我们在A创建(createBeanInstance)的时候已经放到了缓存里边。此时B创建完，A也创建完了，一共执行了4次。至此Bean创建完成，最后将创建好的Bean放入到单例缓存容器中

## Spring框架中bean的生命周期

### Spring创建bean流程：

### 1.Bean实例化

### 2.将bean的引用注入到bean对应的属性中

### 3.假如实现了BeanNameAware接口，spring将bean的id传递给setBeanName()方法

### 4.假如实现了BeanFactoryAware接口，spring将setBeanFactory()方法，将BeanFactory容器的实例传入。

### 5.假如实现了ApplicationContextAware接口，spring将bean的上下文引用(ApplicationContext)传入进来

### 6.bean实现了BeanPostProcessor接口，spring将调用他们的post-ProcessBeforeInitialization()方法

### 7.bean实现了InitializiingBean接口，spring调用他们的afterProperties()方法，或者bean配置了init-method声明了初始化方法，该方法也会被调用

- 初始化方法注解：@PostConstruct

### 8.bean实现了BeanPostProcesser接口，spring将调用post-ProcessAfterInitalization()方法

### 9.如果ApplicationContext(应用上下文)被销毁，spring将会调用它的destroy()接口方法。或者bean配置的destory-method声明的销毁方法

- 销毁方法注解：@PreDestory

## 问题

### 1.什么是bean装配？

- bean装配是指在Spring容器中把bean组装到一起

### 2.使用@Autowired注解自动装配的过程是怎样的？

- 在启动Spring IOC时，容器自动装载了一个AutowiredAnnotationBeanPostProcessor后置处理器，当容器扫描到@Autowired、@Resource或@Inject时，就会在IOC容器自动查找需要的bean，并装配给该对象的属性。

	- @Autowired如何查询对应类型的bean？

		- 1.查询结果正好是一个，就将该bean装配给@Aotowired指定的属性
		- 2.如果查询的结果不止一个，那么@Autowired会根据名称来查找

			- 可以配合@Qualifier找到对应名字的bean

		- 3.如果上述查找的结果为空，那么会抛出异常。解决办法：使用required=false

### 3.注入容器注解有哪些？

- 1.Component
- 2.Controller
- 3.Service
- 4.Repository

### 4.自动装配的注解有哪些？

- 1.Autowried

	- 可用于构造函数、成员变量、Setter方法

- 2.Resource

	- JSR330 JDK自带的

		- 按照名称注入，找不到名称匹配的按照bean类型装配

## Spring DAO

### JdbcTemplate

- JdbcTemplate提供了很多便利的方法，例如把数据库数据装换成基本数据类型和对象，执行写好的或可调用的数据库操作语句，提供自定义数据错误处理

### 问题

- Spring支持的事物管理类型，Spring事务实现方式有哪些？

	- 1.编程式事务管理

		- 通过编程的方式管理事务，提供极大的灵活性，但是难维护

	- 2.声明式事务管理

		- (目前主流的方式)事务代码和事务管理分离，只需要注解和XML配置来管理事务

- Spring事务的实现方式和实现原理？

	- Spring事务的本质其实就是数据库对事务的支持，没有数据库事务的支持，Spring无法提供事务功能。真正的数据库层的事务提交和回滚是通过binlog或者redo log实现的

- Spring的事务传播行为有哪些？

	- 1.required：如果当前没有事务，就创建一个新事务，如果当前存在事务，就加入事务(最常用的设置)
	- 2.supports：支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就以非事务运行
	- 2
	- 4.requires_new：创建新的事务，无论当前存不存在事务，都创建新的事务。
	- 5.not_supported：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起
	- 6.never：以非事务方式执行，如果当前存在，则抛出异常
	- 7.nested：如果当前存在事务，则嵌套事务内执行。如果当前没有事务，则按required属性执行

- Spring的事务隔离级别有那些？

	- 1.default：用底层数据库的设置隔离级别，数据库设置的什么用什么
	- 2.read_uncommitted：未提交读，最低隔离级别、事务未提交前，就可被其他事务读取
	- 3.read_committed：提交读，一个事务提交后才能被其他事务读取到(会照成幻读、不可重复读)
	- 4.repeatable_read：可重复读，保证多次读取同一个数据时，其值和事务开始时候的内容是一致，禁止读取到别的事务未提交的数据(会造成幻读)，MySQL的默认设置级别
	- 5.isolation_serializeble：序列化，代价最高最可靠的隔离级别，该隔离级别能防止脏读、不可重复读、幻读。

- 什么是脏读？

	- 表示一个事务能够读取到另一个事务中还未提交的数据。比如，每个事务尝试插入记录A，此时该事务还未提交，然后另一个事务尝试读取到记录A

- 什么是不可重复读？

	- 是指在一个事务内，多次读取同一数据

		- 强调修改

- 什么是幻读？

	- 一个事务多次查询返回的结果不一样

		- 强调对数据的新增

## Spring AOP

### 简介：AOP(Aspect-Oriented Programming)，一般称为面向切面编程，作为面向对象的补充，用于将那些与业务无关的，但却对多个对象产生影响的公共行为和逻辑，抽取并封装为一个可重用的模块。减少系统中的重复代码，减低模块间的耦合度，同时提高系统的可维护性。可用于权限认证、日志、事务处理等。

### 问题

- 1.Spring AOP和AspectJ AOP有什么区别？

	- 1.AspectJ AOP是静态代理增强。所谓静态代理，就是AOP框架会在编译阶段生成AOP代理类，因此也称为编译时增强，它会在编译阶段将AspectJ(切面)织入到JAVA字节码中，运行的时候就是AOP增强之后的AOP对象
	- 2.Spring AOP采用的是动态代理。动态代理就是AOP框架不会去修改字节码，而是每次运行时在内存中临时方法生成一个AOP对象，这个AOP对象包含目标对象的所有方法，并且在特定的切点做了增强处理，并回调原对象

### Spring AOP动态代理有两种方式，JDK动态代理和CGLIB动态代理方式

### 问题

- 1.JDK动态代理和CGLIB动态代理有什么区别？

	- 1.JDK动态代理只提供接口的代理，不支持类代理。核心InvocationHandler和Proxy类，InvocationHandler通过invoke()方法反射执行目标类中的代码，动态的将横切逻辑和业务编织在一起，接着Proxy利用InvocationHandler动态创建一个符合某一接口的实例，生成目标类代理对象。

		- Proxy创建一个代理类，继承InvocationHandler类，代理类通过InvocationHandler的invoke执行目标类的方法

			- 扩展

			  InvocationHandler 的 invoke(Object proxy,Method method,Object[] args)：proxy是最终生成的代理实例; method 是被代理目标实例的某个具体方法; args 是被代理目标实例某个方法的具体入参, 在方法反射调用时使用。
			  

				- 需要注意，类必须实现接口

					- 原因：proxy使用的方法必须要有接口

					  public static Object newProxyInstance(ClassLoader loader,
					                                        Class<?>[] interfaces,
					                                        InvocationHandler h)
					      throws IllegalArgumentException
					  
					  

	- 2.如果代理类没有没有实现接口，Spring AOP会选择使用CBLIB来动态代理目标类。CGLIB（Code Generation Library）是一个代码生成的类库，可以在运行时动态生成指定类的一个子类对象，并覆盖其中指定的方法并添加增强代码，从而实现AOP。CGLIB是通过继承的方式实现动态代理，如果一个类被设置为final就无法使用CBLIB做动态代理

- 2.如何理解Spring中的代理对象？

	- Advice + Target Object = Proxy

### Spring AOP名词

- 1.切面(Aspect)
- 2.连接点(Join point)
- 3.通知(Advice)
- 4.切入点(Pointcut)
- 5.目标对象(Target Object)
- 6.织入(Weaving)

	- 1.编译期：切面在目标类编译时被织入。AspectJ的织入编译器是以这种方式织入切面的
	- 2.类加载期：切面在目标类加载JVM时被织入。需要特殊的类加载器，它可以在目标类被引入应用之前增强该目标类的字节码。AspectJ5的加载时织入就支持以这种方式织入切面
	- 3.运行期：切面在运行的某一个时刻被织入。一般情况下，在织入切面时AOP容器会为目标对象动态地创建一个代理对象。SpringAOP就是以这种方式织入切面

- 7.引入(Introduction)

### Spring通知类型有哪些？

- 1.前置通知(Before)

	- 方法调用之前执行

- 2.后置通知(After)

	- 方法调用之后执行

- 3.返回通知(After-returning)

	- 目标方法成功执行之后执行

- 4.异常通知(After-throwing)

	- 目标方法抛出异常之后执行

- 5.环绕通知(Around)

	- 方法调用前后执行

### Spring AOP的注解

- 1.@Before

	- 前置

- 2.@After

	- 后置

- 3.@AfterReturning

	- 返回之后

- 4.@AfterThrowing

	- 抛出异常之后

- 5.@Around

	- 环绕

- 6.@PointCut

	- 切入点

		- JoinPoint：作为函数的参数传入切面方法，可以得到目标方法的相关信息

- 7.@Aspect

	- 指定切面类

- 8.@EnableAspectJAutoProxy

	- 开启基于注解的AOP模式

