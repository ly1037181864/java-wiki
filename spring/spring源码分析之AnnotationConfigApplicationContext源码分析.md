##spring源码分析之AnnotationConfigApplicationContext分析

```text
#BeanDefinition解析
private final AnnotatedBeanDefinitionReader reader;
#扫描器
private final ClassPathBeanDefinitionScanner scanner;
	
public AnnotationConfigApplicationContext() {
    this.reader = new AnnotatedBeanDefinitionReader(this);
    this.scanner = new ClassPathBeanDefinitionScanner(this);
}

public AnnotationConfigApplicationContext(DefaultListableBeanFactory beanFactory) {
    super(beanFactory);
    this.reader = new AnnotatedBeanDefinitionReader(this);
    this.scanner = new ClassPathBeanDefinitionScanner(this);
}
	
public AnnotationConfigApplicationContext(Class<?>... componentClasses) {
    this();
    #注册componentClasses类，该componentClasses类一般都是注解了@Configuration注解，如果是启动类
    #则还需要注@ComponentScan，spring会在invokeBeanFactoryPostProcessors调用后置处理器来加载
    register(componentClasses);
    refresh();
}

public AnnotationConfigApplicationContext(String... basePackages) {
    this();
    #直接扫描指定路径下的包，后续的后置处理器将不再重复加载bean的定义信息
    scan(basePackages);
    refresh();
}
```
```text
public void register(Class<?>... componentClasses) {
    Assert.notEmpty(componentClasses, "At least one component class must be specified");
    this.reader.register(componentClasses);
}
```
```text
public void register(Class<?>... componentClasses) {
    for (Class<?> componentClass : componentClasses) {
        #注册
        registerBean(componentClass);
    }
}
```
```text
public void scan(String... basePackages) {
    Assert.notEmpty(basePackages, "At least one base package must be specified");
    this.scanner.scan(basePackages);
}
```
```text
public int scan(String... basePackages) {
		int beanCountAtScanStart = this.registry.getBeanDefinitionCount();
       
        #扫描
		doScan(basePackages);

		// Register annotation config processors, if necessary.
		if (this.includeAnnotationConfig) {
			AnnotationConfigUtils.registerAnnotationConfigProcessors(this.registry);
		}

		return (this.registry.getBeanDefinitionCount() - beanCountAtScanStart);
	}
```
```text
protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
    //初始化BeanFactory
    refreshBeanFactory();
    //返回当前实体的beanFactory属性
    return getBeanFactory();
}
```
这里需要注意的是，当AnnotationConfigApplicationContext调用自己的无参构造器时，会首先初始化父类的无参构造器，这个时候会
给beanFactory给定默认的DefaultListableBeanFactory实现
```text
protected final void refreshBeanFactory() throws IllegalStateException {
    if (!this.refreshed.compareAndSet(false, true)) {
        throw new IllegalStateException(
                "GenericApplicationContext does not support multiple refresh attempts: just call 'refresh' once");
    }
    this.beanFactory.setSerializationId(getId());
}
```
```text
public AnnotationConfigApplicationContext() {
    #这里执行的时候，同时也会先初始化父类的无参构造器
    this.reader = new AnnotatedBeanDefinitionReader(this);
    this.scanner = new ClassPathBeanDefinitionScanner(this);
}
```
```text
public GenericApplicationContext() {
    #这个时候会指定默认的BeanFactory
    this.beanFactory = new DefaultListableBeanFactory();
}
```
其余的AnnotationConfigApplicationContext的逻辑都跟ClassPathXmlApplicationContext雷同，都会调用父类的refresh()中的逻辑处理，
不同在于一个是注解，一个是xml文件解析的形式

AnnotationConfigApplicationContext注解扫描的逻辑实现
```text
public AnnotationConfigApplicationContext() {
    this.reader = new AnnotatedBeanDefinitionReader(this);
    this.scanner = new ClassPathBeanDefinitionScanner(this);
}
```
```text
public AnnotatedBeanDefinitionReader(BeanDefinitionRegistry registry) {
    this(registry, getOrCreateEnvironment(registry));
}
```
```text
public AnnotatedBeanDefinitionReader(BeanDefinitionRegistry registry, Environment environment) {
    Assert.notNull(registry, "BeanDefinitionRegistry must not be null");
    Assert.notNull(environment, "Environment must not be null");
    this.registry = registry;
    this.conditionEvaluator = new ConditionEvaluator(registry, environment, null);
    //这里注入@Configuration、@AutoWired、Java注解、Jpa注解等
    AnnotationConfigUtils.registerAnnotationConfigProcessors(this.registry);
}
```
```text
public static Set<BeanDefinitionHolder> registerAnnotationConfigProcessors(
			BeanDefinitionRegistry registry, @Nullable Object source) {

    DefaultListableBeanFactory beanFactory = unwrapDefaultListableBeanFactory(registry);
    if (beanFactory != null) {
        if (!(beanFactory.getDependencyComparator() instanceof AnnotationAwareOrderComparator)) {
            beanFactory.setDependencyComparator(AnnotationAwareOrderComparator.INSTANCE);
        }
        if (!(beanFactory.getAutowireCandidateResolver() instanceof ContextAnnotationAutowireCandidateResolver)) {
            beanFactory.setAutowireCandidateResolver(new ContextAnnotationAutowireCandidateResolver());
        }
    }

    Set<BeanDefinitionHolder> beanDefs = new LinkedHashSet<>(8);

    //注入ConfigurationClassPostProcessor扫描@Configuration注解处理
    if (!registry.containsBeanDefinition(CONFIGURATION_ANNOTATION_PROCESSOR_BEAN_NAME)) {
        RootBeanDefinition def = new RootBeanDefinition(ConfigurationClassPostProcessor.class);
        def.setSource(source);
        beanDefs.add(registerPostProcessor(registry, def, CONFIGURATION_ANNOTATION_PROCESSOR_BEAN_NAME));
    }

    //注入AutowiredAnnotationBeanPostProcessor扫描@AutoWired等注解处理
    if (!registry.containsBeanDefinition(AUTOWIRED_ANNOTATION_PROCESSOR_BEAN_NAME)) {
        RootBeanDefinition def = new RootBeanDefinition(AutowiredAnnotationBeanPostProcessor.class);
        def.setSource(source);
        beanDefs.add(registerPostProcessor(registry, def, AUTOWIRED_ANNOTATION_PROCESSOR_BEAN_NAME));
    }

    // Check for JSR-250 support, and if present add the CommonAnnotationBeanPostProcessor.
    //注入CommonAnnotationBeanPostProcessor扫描Java注解处理
    if (jsr250Present && !registry.containsBeanDefinition(COMMON_ANNOTATION_PROCESSOR_BEAN_NAME)) {
        RootBeanDefinition def = new RootBeanDefinition(CommonAnnotationBeanPostProcessor.class);
        def.setSource(source);
        beanDefs.add(registerPostProcessor(registry, def, COMMON_ANNOTATION_PROCESSOR_BEAN_NAME));
    }

    // Check for JPA support, and if present add the PersistenceAnnotationBeanPostProcessor.
    //注入PersistenceAnnotationBeanPostProcessor处理jpa相关的注解扫描处理
    if (jpaPresent && !registry.containsBeanDefinition(PERSISTENCE_ANNOTATION_PROCESSOR_BEAN_NAME)) {
        RootBeanDefinition def = new RootBeanDefinition();
        try {
            def.setBeanClass(ClassUtils.forName(PERSISTENCE_ANNOTATION_PROCESSOR_CLASS_NAME,
                    AnnotationConfigUtils.class.getClassLoader()));
        }
        catch (ClassNotFoundException ex) {
            throw new IllegalStateException(
                    "Cannot load optional framework class: " + PERSISTENCE_ANNOTATION_PROCESSOR_CLASS_NAME, ex);
        }
        def.setSource(source);
        beanDefs.add(registerPostProcessor(registry, def, PERSISTENCE_ANNOTATION_PROCESSOR_BEAN_NAME));
    }

    //时间相关的EventListenerMethodProcessor处理器扫描事件相关的注解
    if (!registry.containsBeanDefinition(EVENT_LISTENER_PROCESSOR_BEAN_NAME)) {
        RootBeanDefinition def = new RootBeanDefinition(EventListenerMethodProcessor.class);
        def.setSource(source);
        beanDefs.add(registerPostProcessor(registry, def, EVENT_LISTENER_PROCESSOR_BEAN_NAME));
    }

    if (!registry.containsBeanDefinition(EVENT_LISTENER_FACTORY_BEAN_NAME)) {
        RootBeanDefinition def = new RootBeanDefinition(DefaultEventListenerFactory.class);
        def.setSource(source);
        beanDefs.add(registerPostProcessor(registry, def, EVENT_LISTENER_FACTORY_BEAN_NAME));
    }

    return beanDefs;
}
```
这里特别要指出的是ConfigurationClassPostProcessor处理器，他实现了BeanDefinitionRegistryPostProcessor接口，这点一定要注意，
因为在调用BeanFactoryPostProcessor时，会最先调用BeanDefinitionRegistryPostProcessor接口的逻辑，而这个又是在AnnotationConfigApplicationContext
构造器中手动注入到spring容器中去的，所以ConfigurationClassPostProcessor处理器会最先执行，在这里去完成各种注解的扫描等，完成
BeanDefiniton信息的加载，同时AnnotationConfigUtils.registerAnnotationConfigProcessors(this.registry);也注入了向@AutoWired
和@Resource等Java注解的处理器，用来在后续的调用流程中完成这些注解的功能逻辑。基于这些AnnotationConfigApplicationContext容
器才能真正实现零xml配置完成spring的启动功能。
```text
public class ConfigurationClassPostProcessor implements BeanDefinitionRegistryPostProcessor,
		PriorityOrdered, ResourceLoaderAware, BeanClassLoaderAware, EnvironmentAware {}
```
