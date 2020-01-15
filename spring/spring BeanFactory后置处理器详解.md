##spring BeanFactory相关的后置处理器详解

####BeanDefinitionRegistryPostProcessor后置处理器详解
1.从源码的角度来看目前实现了BeanDefinitionRegistryPostProcessor接口中最重要的就是ConfigurationClassPostProcessor后置处理器了
该类的主要功能便是在执行invokeBeanFactoryPostProcessors方法的时候，执行ConfigurationClassPostProcessor后置处理器，将所有的
注解bean扫描到容器中去，即@Component、@PropertySources、@ComponentScans、@ImportResource、@Bean

2.我们引入spring通常有通过xml的方式，或者注解的方式，而这两种方式在加载bean的定义信息时又会有一些细微的差别       
2.1如果我们采用xml的方式引入构建spring，那么当我们在执行到obtainFreshBeanFactory方法时，如果没有申明开启注解扫描的，spring
只会将我们在xml中定义的bean的信息加入的spring Map中，但是只要是开启注解扫描的，spring也会在这个时候将注解的bean加载到map中去的
下面的例子，实际上不光是user的定义信息会扫描到容器中去，AppConfig的bean的信息也会扫描到容器中去
```text
#开启注解扫描
<context:component-scan base-package="com.xiaoantimes.cn"/>

<bean id="user" class="com.xiaoantimes.cn.entity.User">
    <property name="name" value="liuyou"/>
    <property name="age" value="21"/>
</bean>
```
```text
@Component
public class AppConfig {
}
```
当spring在解析xml的时候，由于首先解析时beans标签，它会递归再次调用doRegisterBeanDefinitions()方法，再次解析beans的子元素
```text
private void parseDefaultElement(Element ele, BeanDefinitionParserDelegate delegate) {
    //对import 标签的处理
    if (delegate.nodeNameEquals(ele, IMPORT_ELEMENT)) {
        importBeanDefinitionResource(ele);
    }
    //对alias标签的处理
    else if (delegate.nodeNameEquals(ele, ALIAS_ELEMENT)) {
        processAliasRegistration(ele);
    }
    //对bean标签的处理，同时将bean的信息注册到伪IOC容器中去
    else if (delegate.nodeNameEquals(ele, BEAN_ELEMENT)) {
        processBeanDefinition(ele, delegate);
    }
    //对beans标签的处理
    else if (delegate.nodeNameEquals(ele, NESTED_BEANS_ELEMENT)) {
        // recurse
        //递归解析
        doRegisterBeanDefinitions(ele);
    }
}
```
由于root元素此时是context:component-scan，不再是默认的元素标签，因此会调用delegate.parseCustomElement(root);找到METE-INFO
下面的ContextNamespaceHandler解析器去解析context元素标签
```text
protected void parseBeanDefinitions(Element root, BeanDefinitionParserDelegate delegate) {
    //判断元素是否是默认名称空间如beans等标签
    if (delegate.isDefaultNamespace(root)) {
        ...
    }
    else {
        //用户自定义的解析器如事务标签解析，aop标签等
        //这里处理的就是用户自定义的元素处理，如JdbcNamespaceHandler，AopNamespaceHandler等等
        delegate.parseCustomElement(root);
    }
}
```
```text
public class ContextNamespaceHandler extends NamespaceHandlerSupport {

	@Override
	public void init() {
		registerBeanDefinitionParser("property-placeholder", new PropertyPlaceholderBeanDefinitionParser());
		registerBeanDefinitionParser("property-override", new PropertyOverrideBeanDefinitionParser());
		registerBeanDefinitionParser("annotation-config", new AnnotationConfigBeanDefinitionParser());
		#处理包扫描的处理器
		registerBeanDefinitionParser("component-scan", new ComponentScanBeanDefinitionParser());
		registerBeanDefinitionParser("load-time-weaver", new LoadTimeWeaverBeanDefinitionParser());
		registerBeanDefinitionParser("spring-configured", new SpringConfiguredBeanDefinitionParser());
		registerBeanDefinitionParser("mbean-export", new MBeanExportBeanDefinitionParser());
		registerBeanDefinitionParser("mbean-server", new MBeanServerBeanDefinitionParser());
	}

}
```
在ComponentScanBeanDefinitionParser中已经对指定包路径下的类进行了扫描，因此使用xml的方式引入spring，最终基本上类的BeanDefinition
加载都在obtainFreshBeanFactory()方法中完成
```text
public BeanDefinition parse(Element element, ParserContext parserContext) {
    String basePackage = element.getAttribute(BASE_PACKAGE_ATTRIBUTE);
    basePackage = parserContext.getReaderContext().getEnvironment().resolvePlaceholders(basePackage);
    String[] basePackages = StringUtils.tokenizeToStringArray(basePackage,
            ConfigurableApplicationContext.CONFIG_LOCATION_DELIMITERS);

    // Actually scan for bean definitions and register them.
    ClassPathBeanDefinitionScanner scanner = configureScanner(parserContext, element);
    #扫描所有bean的定义
    Set<BeanDefinitionHolder> beanDefinitions = scanner.doScan(basePackages);
    registerComponents(parserContext.getReaderContext(), beanDefinitions, element);

    return null;
}
```
2.2如果我们是采用注解的方式，这种方式又会有所不同，如果我们指定了扫描包，如下所述的示例1，那么spring会从容器创建的时候就去
扫描指定路径下的所有的bean的定义信息，但是我们只是通过示例2的这种方式，那么spring在构建容器的时候，只会将AppConfig.class
注入到spring Map中去，其他的所有的bean的信息都需要调用invokeBeanFactoryPostProcessors方法时由spring后置处理器去加载
```text
示例1
public class AppTest{
    AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext("xxx.xxx.xxx"); #xxx.xxx.xxx指定扫描包路径
}

示例2
@Configuration
@ComponentScans("xxx.xxx.xxx")
public class AppConfig {
}

public class AppTest{
    #只会注入AppConfig.class类，剩下的类会通过@ComponentScans在BeanDefinitionRegistryPostProcessor加载的时候处理
    AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
}
```
实际上注解版的容器创建，在scan(basePackages)的方法中就已经在各种注解的bean加入到容器中去了，不再会在refresh()方法中重复再
加载bean的定义信息
```text
public AnnotationConfigApplicationContext(String... basePackages) {
    this();
    scan(basePackages);  #spring在此处就已经将bean的定义扫描到了map中去了，后续的流程中都不会再重复加载
    refresh();
}
```
3.BeanDefinitionRegistryPostProcessor给与了程序员一个机会，在spring加载beanDefinition且尚未完全加载完成的时候，可以干预
BeanDefinition的加载过程，来实现个性化的功能扩展，需要注意的上述2中的细节点，但是spring同样允许我们去修改示例1情况的已经在
obtainFreshBeanFactory方法加载过的bean的定义信息
```text
public interface BeanDefinitionRegistryPostProcessor extends BeanFactoryPostProcessor {

	void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException;
}
```
4.向容器中添加自定义的BeanDefinitionRegistryPostProcessor时需要注意的是，spring默认的ConfigurationClassPostProcessor的后
置处理的优先级最高,ConfigurationClassPostProcessor返回的是Integer.MAX_VALUE，换句话说，当用户去干预spring的beanDefinition
加载时，其实spring的ConfigurationClassPostProcessor处理器已经工作了，之后才是去执行用户自定义的后置处理器
```text
public int getOrder() {
    return Ordered.LOWEST_PRECEDENCE;  // within PriorityOrdered
}
```
```text
public interface Ordered {

	int HIGHEST_PRECEDENCE = Integer.MIN_VALUE;

	int LOWEST_PRECEDENCE = Integer.MAX_VALUE;
	
	int getOrder();

}
```
案例3
```text
public class AppConfigTest {

    public static void main(String[] args){
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
//        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
//        context.addBeanFactoryPostProcessor (new MyBeanDefinitionRegistryPostProcessor ());
//        context.register (AppConfig.class);
//        context.refresh ();
        //AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext("com.xiaoantimes.cn");
        //ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext ("spring-test.xml");
    }
}
```
```text
@Component
public class MyBeanDefinitionRegistryPostProcessor implements BeanDefinitionRegistryPostProcessor {

    public void postProcessBeanDefinitionRegistry (BeanDefinitionRegistry beanDefinitionRegistry) throws BeansException {
        for (String beanDefinitionName : beanDefinitionRegistry.getBeanDefinitionNames ()) {
            System.out.println ("beanDefinitionName:"+beanDefinitionName);
        }
        System.out.println ("=============我自定义的BeanDefinitionRegistryPostProcessor.postProcessBeanDefinitionRegistry============");
    }

    public void postProcessBeanFactory (ConfigurableListableBeanFactory configurableListableBeanFactory) throws BeansException {
        System.out.println ("=============我自定义的BeanDefinitionRegistryPostProcessor.postProcessBeanFactory============");
    }

}
```
输出结果
```text
beanDefinitionName:org.springframework.context.annotation.internalConfigurationAnnotationProcessor
beanDefinitionName:org.springframework.context.annotation.internalAutowiredAnnotationProcessor
beanDefinitionName:org.springframework.context.annotation.internalCommonAnnotationProcessor
beanDefinitionName:org.springframework.context.event.internalEventListenerProcessor
beanDefinitionName:org.springframework.context.event.internalEventListenerFactory
beanDefinitionName:appConfig
beanDefinitionName:user
beanDefinitionName:myBeanDefinitionRegistryPostProcessor
=============我自定义的BeanDefinitionRegistryPostProcessor.postProcessBeanDefinitionRegistry============
=============我自定义的BeanDefinitionRegistryPostProcessor.postProcessBeanFactory============
```
案例4
```text
public class AppConfigTest {

    public static void main(String[] args){
//        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
        #手动注册BeanDefinitionRegistryPostProcessor
        context.addBeanFactoryPostProcessor (new MyBeanDefinitionRegistryPostProcessor ());
        context.register (AppConfig.class);
        context.refresh ();
        //AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext("com.xiaoantimes.cn");
        //ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext ("spring-test.xml");
    }
}
```
```text
public class MyBeanDefinitionRegistryPostProcessor implements BeanDefinitionRegistryPostProcessor {

    public void postProcessBeanDefinitionRegistry (BeanDefinitionRegistry beanDefinitionRegistry) throws BeansException {
        System.out.println ("=============我自定义的BeanDefinitionRegistryPostProcessor.postProcessBeanDefinitionRegistry============");
        for (String beanDefinitionName : beanDefinitionRegistry.getBeanDefinitionNames ()) {
            System.out.println ("beanDefinitionName:"+beanDefinitionName);
        }
        System.out.println ("=============我自定义的BeanDefinitionRegistryPostProcessor.postProcessBeanDefinitionRegistry============");
    }

    public void postProcessBeanFactory (ConfigurableListableBeanFactory configurableListableBeanFactory) throws BeansException {
        System.out.println ("=============我自定义的BeanDefinitionRegistryPostProcessor.postProcessBeanFactory============");
    }

}
```
```text
@Component
public class MyBeanDefinitionRegistryPostProcessor2 implements BeanDefinitionRegistryPostProcessor {

    public void postProcessBeanDefinitionRegistry (BeanDefinitionRegistry beanDefinitionRegistry) throws BeansException {
        System.out.println ("=============我自定义的MyBeanDefinitionRegistryPostProcessor2.postProcessBeanDefinitionRegistry============");
        for (String beanDefinitionName : beanDefinitionRegistry.getBeanDefinitionNames ()) {
            System.out.println ("beanDefinitionName:"+beanDefinitionName);
        }
        System.out.println ("=============我自定义的MyBeanDefinitionRegistryPostProcessor2.postProcessBeanDefinitionRegistry============");
    }

    public void postProcessBeanFactory (ConfigurableListableBeanFactory configurableListableBeanFactory) throws BeansException {
        System.out.println ("=============我自定义的MyBeanDefinitionRegistryPostProcessor2.postProcessBeanFactory============");
    }

}
```
输出结果
```text
=============我自定义的BeanDefinitionRegistryPostProcessor.postProcessBeanDefinitionRegistry============
beanDefinitionName:org.springframework.context.annotation.internalConfigurationAnnotationProcessor
beanDefinitionName:org.springframework.context.annotation.internalAutowiredAnnotationProcessor
beanDefinitionName:org.springframework.context.annotation.internalCommonAnnotationProcessor
beanDefinitionName:org.springframework.context.event.internalEventListenerProcessor
beanDefinitionName:org.springframework.context.event.internalEventListenerFactory
#注意点，手动注册无法获取到容器中ConfigurationClassPostProcessor自动扫描的BeanDefinition信息
beanDefinitionName:appConfig
=============我自定义的BeanDefinitionRegistryPostProcessor.postProcessBeanDefinitionRegistry============

=============我自定义的MyBeanDefinitionRegistryPostProcessor2.postProcessBeanDefinitionRegistry============
beanDefinitionName:org.springframework.context.annotation.internalConfigurationAnnotationProcessor
beanDefinitionName:org.springframework.context.annotation.internalAutowiredAnnotationProcessor
beanDefinitionName:org.springframework.context.annotation.internalCommonAnnotationProcessor
beanDefinitionName:org.springframework.context.event.internalEventListenerProcessor
beanDefinitionName:org.springframework.context.event.internalEventListenerFactory
#注意点，容器注入则可以获取
beanDefinitionName:appConfig
beanDefinitionName:user
beanDefinitionName:myBeanDefinitionRegistryPostProcessor2
=============我自定义的MyBeanDefinitionRegistryPostProcessor2.postProcessBeanDefinitionRegistry============

=============我自定义的BeanDefinitionRegistryPostProcessor.postProcessBeanFactory============
=============我自定义的MyBeanDefinitionRegistryPostProcessor2.postProcessBeanFactory============
```
从上述两处的差异我们可以看出，手动注入的BeanDefinitionRegistryPostProcessor要优先于向容器注册的后置处理器执行，其次是如果
我们非要让自定义的后置处理器优先于spring的ConfigurationClassPostProcessor执行，那么可以采用手动注册后置处理的方式，spring
会先执行手动注册的后置处理器，然后才是执行容器中注册的后置处理器，不过手动注册会导致我们无法获取到ConfigurationClassPostProcessor
扫描到的bean的定义信息，只能获取到spring初始化的几个类和AppConfig类，这点需要注意




####BeanFactoryPostProcessor后置处理器详解
BeanFactoryPostProcessor给与了程序员一个机会，在spring已经将所有的beanDefinition加载到spring的缓存中，但尚未实例化所有的bean
时，可以允许程序员对所有的beanDefinition定义信息进行修改等扩展功能操作，在实现BeanFactoryPostProcessor的时候，需要重写
postProcessBeanFactory方法，而在这个方法中，spring已经在整个bean工厂作为参数传给了实现类，因此我们可以就这个方法中做一些
自定义的扩展功能，实现个性化的需求
```text
public interface BeanFactoryPostProcessor {

	void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException;
}
```

####BeanDefinitionRegistryPostProcessor和BeanFactoryPostProcessor的区别
1.BeanDefinitionRegistryPostProcessor是BeanFactoryPostProcessor的子类，并有其特殊的方法postProcessBeanDefinitionRegistry
2.两者调用的时机不同，BeanDefinitionRegistryPostProcessor要优先于BeanFactoryPostProcessor调用，且BeanDefinitionRegistryPostProcessor
在spring尚未加载完所有的beanDefinition定义信息，且bean尚未实例化的时候，而BeanFactoryPostProcessor的调用是在所有的beanDefinition
定义信息已经全部加载完成，且所有的bean尚未实例化的时候，从spring的设计角度来说，两个后置处理给了我们程序员干预spring beanDefinition
定义信息的加载过程以及所有beanDefinition信息加载完成后的扩展功能的能力，总而言之BeanDefinitionRegistryPostProcessor是spring
尚未完成BeanDefinition的定义信息加载，而BeanFactoryPostProcessor是在spring已经完成了对所有bean的BeanDefinition信息的加载
但是两者都存在bean尚未实例化之前被加载的。不过还有一处需要注意BeanDefinitionRegistryPostProcessor在调用的时候，需要注意注
解版中的示例1的情况说明，如果注解版中，在构建spring容器时就已经指定了包扫描路径，那么此时，后置处理器虽然会调用，但不会再
重复加载bean的定义信息。
