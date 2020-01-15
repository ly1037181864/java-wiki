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