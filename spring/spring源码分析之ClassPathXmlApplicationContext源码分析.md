##spring源码分析之ClassPathXmlApplicationContext分析

```text
#构建容器
ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("spring-test.xml");
```
```text
public ClassPathXmlApplicationContext(
			String[] configLocations, boolean refresh, @Nullable ApplicationContext parent)
			throws BeansException {

    super(parent);
    //设置配置文件的路径
    setConfigLocations(configLocations);
    if (refresh) {
        //刷新容器
        refresh();
    }
}
```
refresh()刷新容器，也就是容器重启的过程，也是spring启动的重要环境，是spring环境真正启动的方法，即所谓的12个方法
```text
public void refresh() throws BeansException, IllegalStateException {
    synchronized (this.startupShutdownMonitor) {
        // Prepare this context for refreshing.
        //准备刷新的上下文环境
        prepareRefresh();

        // Tell the subclass to refresh the internal bean factory.
        //初始化容器-这一步实际上只是将bean的信息加载到了伪IOC容器，并没有对bean进行初始化
        ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

        // Prepare the bean factory for use in this context.
        //对BeanFactory 进行各种功能填充
        prepareBeanFactory(beanFactory);

        try {
            // Allows post-processing of the bean factory in context subclasses.
            //子类覆盖方法做额外的处理
            postProcessBeanFactory(beanFactory);

            // Invoke factory processors registered as beans in the context.
            //激活各种BeanFactory处理器
            invokeBeanFactoryPostProcessors(beanFactory);

            // Register bean processors that intercept bean creation.
            //后置处理器,注册拦截Bean创建的Bean处理器，这里只是注册，真正的调用是在getBean时候
            registerBeanPostProcessors(beanFactory);

            // Initialize message source for this context.
            //为上下文初始化Message源，即不同语言的消息体，国际化处理
            initMessageSource();

            // Initialize event multicaster for this context.
            //初始化应用消息广播器，并放入ApplicationEventMulticaster bean中
            initApplicationEventMulticaster();

            // Initialize other special beans in specific context subclasses.
            //留给子类来初始化其它的Bean
            onRefresh();

            // Check for listener beans and register them.
            //在所有注册的bean中查找Listener bean，注册到消息广播报中
            registerListeners();

            // Instantiate all remaining (non-lazy-init) singletons.
            //实例化所有非懒加载的bean
            finishBeanFactoryInitialization(beanFactory);

            // Last step: publish corresponding event.
            //完成刷新过程，通知生命周期处现器lifecycleProcessor刷新过程，同时发出
            //ContextRefreshEvent通知别人
            finishRefresh();
        }

        catch (BeansException ex) {
            ...
            // Destroy already created singletons to avoid dangling resources.
            destroyBeans();

            // Reset 'active' flag.
            cancelRefresh(ex);

            // Propagate exception to caller.
            throw ex;
        }

        finally {
            // Reset common introspection caches in Spring's core, since we
            // might not ever need metadata for singleton beans anymore...
            resetCommonCaches();
        }
    }
}
```
加载xml配置信息
```text
protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
    //初始化BeanFactory，并进行XML文件读取，并将得到的BeanFacotry记录在当前实体的属性中
    refreshBeanFactory();
    //返回当前实体的beanFactory属性
    return getBeanFactory();
}
```
```text
protected final void refreshBeanFactory() throws BeansException {
    if (hasBeanFactory()) {
        destroyBeans();
        closeBeanFactory();
    }
    try {
        //创建DefaultListableBeanFactory
        DefaultListableBeanFactory beanFactory = createBeanFactory();
        //为了序列化指定id，如果需要的话，让这个BeanFactory从id反序列化到BeanFactory对象
        beanFactory.setSerializationId(getId());
        //定制beanFactory，设置相关属性，包括是否允许覆盖同名称的不同定义的对象以及是否允许bean间的循环依赖
        customizeBeanFactory(beanFactory);
        //初始化DodumentReader ， 并进行XML文件读取及解析
        loadBeanDefinitions(beanFactory);
        synchronized (this.beanFactoryMonitor) {
            this.beanFactory = beanFactory;
        }
    }
    catch (IOException ex) {
        throw new ApplicationContextException("I/O error parsing bean definition source for " + getDisplayName(), ex);
    }
}
```
createBeanFactory()创建默认的工厂，也就我们常说的spring Bean工厂就是DefaultListableBeanFactory
```text
protected DefaultListableBeanFactory createBeanFactory() {
    return new DefaultListableBeanFactory(getInternalParentBeanFactory());
}
```
customizeBeanFactory对DefaultListableBeanFactory进行个性化定制，主要是针对是否运行BeanDefinition定义覆盖，即容器中已经存在
某一个bean的BeanDefinition信息，这个时候是否允许覆盖，设置是否允许循环引用即allowCircularReferences，spring默认是支持循环引
用的
```text
protected void customizeBeanFactory(DefaultListableBeanFactory beanFactory) {
    //如果属性allowBeanDefinitionOverriding不为空，设置给beanFactory对象相应属性，
    // 此屑性的含义， 是否允许覆盖同名称的不同定义的对象
    if (this.allowBeanDefinitionOverriding != null) {
        beanFactory.setAllowBeanDefinitionOverriding(this.allowBeanDefinitionOverriding);
    }
    //如果属性allowCircularReferences不为空，设置给beanFactory对象相应属性，
    //此属性的含义， 是否允许bean之间存在循环依赖
    if (this.allowCircularReferences != null) {
        beanFactory.setAllowCircularReferences(this.allowCircularReferences);
    }
}
```
loadBeanDefinitions加载BeanDefinition定义，后续的流程跟XmlBeanFactory一样，详细可以参考XmlBeanFactory源码分析
```text
protected void loadBeanDefinitions(DefaultListableBeanFactory beanFactory) throws BeansException, IOException {
    // Create a new XmlBeanDefinitionReader for the given BeanFactory.
    //为指定beanFactory创建XmlBeanDefinitionReader
    XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(beanFactory);

    // Configure the bean definition reader with this context's
    // resource loading environment.
    //对beanDefinitionReader进行环境变量设置
    beanDefinitionReader.setEnvironment(this.getEnvironment());
    beanDefinitionReader.setResourceLoader(this);
    beanDefinitionReader.setEntityResolver(new ResourceEntityResolver(this));

    // Allow a subclass to provide custom initialization of the reader,
    // then proceed with actually loading the bean definitions.
    //对BeanDefinitionReader进行设置，可以被盖
    initBeanDefinitionReader(beanDefinitionReader);
    //加载bean定义信息
    loadBeanDefinitions(beanDefinitionReader);
}
```
prepareRefresh()容器刷新前的准备工作，重点就是getEnvironment().validateRequiredProperties()方法
```text
protected void prepareRefresh() {
    // Switch to active.
    this.startupDate = System.currentTimeMillis();
    this.closed.set(false);
    this.active.set(true);

    if (logger.isDebugEnabled()) {
        if (logger.isTraceEnabled()) {
            logger.trace("Refreshing " + this);
        }
        else {
            logger.debug("Refreshing " + getDisplayName());
        }
    }

    // Initialize any placeholder property sources in the context environment.
    //留给子类覆盖
    //在应用启动之前替换一些属性占位符，这个方法再Spring的对象中一般都没有实现，应该是用来方便我们后期扩展使用的方法。
    initPropertySources();

    // Validate that all properties marked as required are resolvable:
    // see ConfigurablePropertyResolver#setRequiredProperties
    //验证需要的属性文件是否都应经放入环境中
    //Environment类的方法验证必须的属性是否正确
    getEnvironment().validateRequiredProperties();

    // Store pre-refresh ApplicationListeners...
    if (this.earlyApplicationListeners == null) {
        this.earlyApplicationListeners = new LinkedHashSet<>(this.applicationListeners);
    }
    else {
        // Reset local application listeners to pre-refresh state.
        this.applicationListeners.clear();
        this.applicationListeners.addAll(this.earlyApplicationListeners);
    }

    // Allow for the collection of early ApplicationEvents,
    // to be published once the multicaster is available...
    this.earlyApplicationEvents = new LinkedHashSet<>();
}
```
this.requiredProperties遍历属性，看是否存在，如果不存在则报错
```text
public void validateRequiredProperties() {
    MissingRequiredPropertiesException ex = new MissingRequiredPropertiesException();
    for (String key : this.requiredProperties) {
        if (this.getProperty(key) == null) {
            ex.addMissingRequiredProperty(key);
        }
    }
    if (!ex.getMissingRequiredProperties().isEmpty()) {
        throw ex;
    }
}
```
prepareBeanFactory()准备阶段，主要是给beanFactory设置当前context的类加载器、设置StandardBeanExpressionResolver表达式解析器、
增加ResourceEditorRegistrar属性解析器、注册ApplicationContextAwareProcessor后置处理器、设置几个常见的忽略自动装配的接口
```text
protected void prepareBeanFactory(ConfigurableListableBeanFactory beanFactory) {
    // Tell the internal bean factory to use the context's class loader etc.
    //／／设置beanFactory的classLoader为当前context的classLoader
    beanFactory.setBeanClassLoader(getClassLoader());
    //设置beanFactory的表达式语言处理器，Spring3增加了表达式语言的支持，默认可以使用#{bean.xxx}的形式来调用相关属性值。
    beanFactory.setBeanExpressionResolver(new StandardBeanExpressionResolver(beanFactory.getBeanClassLoader()));
    //为beanFactory增加了一个默认的propertyEditor ，这个主要是对bean的属性等设置管理的一个工具
    beanFactory.addPropertyEditorRegistrar(new ResourceEditorRegistrar(this, getEnvironment()));

    // Configure the bean factory with context callbacks.
    //注册BeanPostProcessor
    beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this));
    //设置了几个忽略自动装配的接口
    beanFactory.ignoreDependencyInterface(EnvironmentAware.class);
    beanFactory.ignoreDependencyInterface(EmbeddedValueResolverAware.class);
    beanFactory.ignoreDependencyInterface(ResourceLoaderAware.class);
    beanFactory.ignoreDependencyInterface(ApplicationEventPublisherAware.class);
    beanFactory.ignoreDependencyInterface(MessageSourceAware.class);
    beanFactory.ignoreDependencyInterface(ApplicationContextAware.class);

    // BeanFactory interface not registered as resolvable type in a plain factory.
    // MessageSource registered (and found for autowiring) as a bean.
    //设置了几个自动装配的特殊规则
    beanFactory.registerResolvableDependency(BeanFactory.class, beanFactory);
    beanFactory.registerResolvableDependency(ResourceLoader.class, this);
    beanFactory.registerResolvableDependency(ApplicationEventPublisher.class, this);
    beanFactory.registerResolvableDependency(ApplicationContext.class, this);

    // Register early post-processor for detecting inner beans as ApplicationListeners.
    //注册后置处理器
    beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(this));

    // Detect a LoadTimeWeaver and prepare for weaving, if found.
    //增加对AspectJ的支持
    if (beanFactory.containsBean(LOAD_TIME_WEAVER_BEAN_NAME)) {
        beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));
        // Set a temporary ClassLoader for type matching.
        beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));
    }

    // Register default environment beans.
    //添加默认的系统环境bean
    if (!beanFactory.containsLocalBean(ENVIRONMENT_BEAN_NAME)) {
        beanFactory.registerSingleton(ENVIRONMENT_BEAN_NAME, getEnvironment());
    }
    if (!beanFactory.containsLocalBean(SYSTEM_PROPERTIES_BEAN_NAME)) {
        beanFactory.registerSingleton(SYSTEM_PROPERTIES_BEAN_NAME, getEnvironment().getSystemProperties());
    }
    if (!beanFactory.containsLocalBean(SYSTEM_ENVIRONMENT_BEAN_NAME)) {
        beanFactory.registerSingleton(SYSTEM_ENVIRONMENT_BEAN_NAME, getEnvironment().getSystemEnvironment());
    }
}
```
ApplicationContextAwareProcessor后置处理器的功能，只要实现了ApplicationContextAware类，就会自动将ApplicationContext实例注
入到该bean中，其他的雷同
```text
private void invokeAwareInterfaces(Object bean) {
    if (bean instanceof EnvironmentAware) {
        ((EnvironmentAware) bean).setEnvironment(this.applicationContext.getEnvironment());
    }
    if (bean instanceof EmbeddedValueResolverAware) {
        ((EmbeddedValueResolverAware) bean).setEmbeddedValueResolver(this.embeddedValueResolver);
    }
    if (bean instanceof ResourceLoaderAware) {
        ((ResourceLoaderAware) bean).setResourceLoader(this.applicationContext);
    }
    if (bean instanceof ApplicationEventPublisherAware) {
        ((ApplicationEventPublisherAware) bean).setApplicationEventPublisher(this.applicationContext);
    }
    //如果bean实现了MessageSourceAware，自动注入applicationContext
    if (bean instanceof MessageSourceAware) {
        ((MessageSourceAware) bean).setMessageSource(this.applicationContext);
    }
    //如果bean实现了ApplicationContextAware，自动注入applicationContext
    if (bean instanceof ApplicationContextAware) {
        ((ApplicationContextAware) bean).setApplicationContext(this.applicationContext);
    }
}
```
ApplicationContextAwareProcessor示例
```text
@Component
public class Bird implements ApplicationContextAware {
    //得到管理它的Spring容器
    private ApplicationContext context;

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.context = applicationContext;
    }

    @Override
    public String toString() {
        return "Bird [context=" + context + "]";
    }
}
```
自动装配时忽略指定接口
```text
beanFactory.ignoreDependencyInterface(EnvironmentAware.class);
beanFactory.ignoreDependencyInterface(EmbeddedValueResolverAware.class);
beanFactory.ignoreDependencyInterface(ResourceLoaderAware.class);
beanFactory.ignoreDependencyInterface(ApplicationEventPublisherAware.class);
beanFactory.ignoreDependencyInterface(MessageSourceAware.class);
beanFactory.ignoreDependencyInterface(ApplicationContextAware.class);
```
```text
@Autowire
ResourceLoaderAware aware;
在注入的时候spring会忽略此方法指定的接口类。也就是指定的接口不会被注入进去。即上述的注入并不会生效
这个在后期也可以自己处理，假如某个类需要忽略注入，那么可以通过增加自定义的BeanFactoryPostProcessor重写方法中获取beanFactory
对象，然后将自定义需要忽略的类加入到接口忽略中去即可
```
注册registerResolvableDependency
```text
beanFactory.registerResolvableDependency(BeanFactory.class, beanFactory);
beanFactory.registerResolvableDependency(ResourceLoader.class, this);
beanFactory.registerResolvableDependency(ApplicationEventPublisher.class, this);
beanFactory.registerResolvableDependency(ApplicationContext.class, this);
```
ApplicationListenerDetector后置处理器主要作用就是检测哪些实现了ApplicationListener的bean，在他们创建初始化之后，加入到spring
上下文的多播器中，当bean销毁的时候，从上下文的多播器中移除掉
```text
public Object postProcessAfterInitialization(Object bean, String beanName) {
    if (bean instanceof ApplicationListener) {
        // potentially not detected as a listener by getBeanNamesForType retrieval
        Boolean flag = this.singletonNames.get(beanName);
        if (Boolean.TRUE.equals(flag)) {
            // singleton bean (top-level or inner): register on the fly
            //加入上下文多播器中
            this.applicationContext.addApplicationListener((ApplicationListener<?>) bean);
        }
        else if (Boolean.FALSE.equals(flag)) {
            if (logger.isWarnEnabled() && !this.applicationContext.containsBean(beanName)) {
                // inner bean with other scope - can't reliably process events
                logger.warn("Inner bean '" + beanName + "' implements ApplicationListener interface " +
                        "but is not reachable for event multicasting by its containing ApplicationContext " +
                        "because it does not have singleton scope. Only top-level listener beans are allowed " +
                        "to be of non-singleton scope.");
            }
            this.singletonNames.remove(beanName);
        }
    }
    return bean;
}

public void postProcessBeforeDestruction(Object bean, String beanName) {
    if (bean instanceof ApplicationListener) {
        try {
            //移除当前bean
            ApplicationEventMulticaster multicaster = this.applicationContext.getApplicationEventMulticaster();
            multicaster.removeApplicationListener((ApplicationListener<?>) bean);
            multicaster.removeApplicationListenerBean(beanName);
        }
        catch (IllegalStateException ex) {
            // ApplicationEventMulticaster not initialized yet - no need to remove a listener
        }
    }
}
```
LoadTimeWeaverAwareProcessor的主要功能，LoadTimeWeaver被Spring用来在将类加载到Java虚拟机(JVM)中时动态地转换类。若要开启
加载时织入，得在@Configuration类中增加@EnableLoadTimeWeaving
```text
public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
    if (bean instanceof LoadTimeWeaverAware) {
        LoadTimeWeaver ltw = this.loadTimeWeaver;
        if (ltw == null) {
            Assert.state(this.beanFactory != null,
                    "BeanFactory required if no LoadTimeWeaver explicitly specified");
            ltw = this.beanFactory.getBean(
                    ConfigurableApplicationContext.LOAD_TIME_WEAVER_BEAN_NAME, LoadTimeWeaver.class);
        }
        ((LoadTimeWeaverAware) bean).setLoadTimeWeaver(ltw);
    }
    return bean;
}
```
开启LoadTimeWeaverAwareProcessor功能示例
```text
@Configuration
@EnableLoadTimeWeaving
public class AppConfig {
}
```
```text
<beans>
    <context:load-time-weaver/>
</beans>
```
一旦配置为ApplicationContext。 该ApplicationContext中的任何bean都可以实现LoadTimeWeaverAware，从而接收对load-time weaver
实例的引用。 这特别适用于Spring的JPA支持，其中load-time weaving加载织入对JPA类转换非常必要。 参考
LocalContainerEntityManagerFactoryBean javadocs更多的细节。 有关AspectJ加载时编织的更多信息，请参见第7.8.4节"Spring框架中
使用AspectJ加载时编织"。

getBeanFactoryPostProcessors()手动注册的BeanFactoryPostProcessor，即调用addBeanFactoryPostProcessor()方法注册的
```text
protected void invokeBeanFactoryPostProcessors(ConfigurableListableBeanFactory beanFactory) {
    PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors(beanFactory, getBeanFactoryPostProcessors());

    // Detect a LoadTimeWeaver and prepare for weaving, if found in the meantime
    // (e.g. through an @Bean method registered by ConfigurationClassPostProcessor)
    if (beanFactory.getTempClassLoader() == null && beanFactory.containsBean(LOAD_TIME_WEAVER_BEAN_NAME)) {
        beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));
        beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));
    }
}
```
```text
public void addBeanFactoryPostProcessor(BeanFactoryPostProcessor postProcessor) {
    Assert.notNull(postProcessor, "BeanFactoryPostProcessor must not be null");
    this.beanFactoryPostProcessors.add(postProcessor);
}
```
关于BeanDefinitionRegistryPostProcessor和BeanFactoryPostProcessor参阅spring BeanFactory后置处理器章节的描述
注册spring BeanPostProcessor 关于BeanPostProcessor可以参阅spring后置处理器源码分析章节
```text
protected void registerBeanPostProcessors(ConfigurableListableBeanFactory beanFactory) {
    PostProcessorRegistrationDelegate.registerBeanPostProcessors(beanFactory, this);
}
```
spring国际化相关模块，spring可以根据语言环境来加载不同语言的配置文件实现夺过语言切换，最简单的例子就是中英文切换就是通过
它来实现的，详细见spring国际化模块章节
```text
protected void initMessageSource() {
    ConfigurableListableBeanFactory beanFactory = getBeanFactory();
    //如果在配置中已经配置了messageSource，那么将messageSource提取并记录在this.messageSource中
    if (beanFactory.containsLocalBean(MESSAGE_SOURCE_BEAN_NAME)) {
        this.messageSource = beanFactory.getBean(MESSAGE_SOURCE_BEAN_NAME, MessageSource.class);
        // Make MessageSource aware of parent MessageSource.
        if (this.parent != null && this.messageSource instanceof HierarchicalMessageSource) {
            HierarchicalMessageSource hms = (HierarchicalMessageSource) this.messageSource;
            if (hms.getParentMessageSource() == null) {
                // Only set parent context as parent MessageSource if no parent MessageSource
                // registered already.
                hms.setParentMessageSource(getInternalParentMessageSource());
            }
        }
        if (logger.isTraceEnabled()) {
            logger.trace("Using MessageSource [" + this.messageSource + "]");
        }
    }
    else {
        // Use empty MessageSource to be able to accept getMessage calls.
        //如果用户并没有定义配置文件，那么使用临时的DelegatingMessageSource以便于作为调用
        //getMessage方法的返回
        DelegatingMessageSource dms = new DelegatingMessageSource();
        dms.setParentMessageSource(getInternalParentMessageSource());
        this.messageSource = dms;
        beanFactory.registerSingleton(MESSAGE_SOURCE_BEAN_NAME, this.messageSource);
        if (logger.isTraceEnabled()) {
            logger.trace("No '" + MESSAGE_SOURCE_BEAN_NAME + "' bean, using [" + this.messageSource + "]");
        }
    }
}
```
spring事件广播，见spring的消息广播及事件机制模块章节
```text
protected void initApplicationEventMulticaster() {
    ConfigurableListableBeanFactory beanFactory = getBeanFactory();
    //如果用户自定义了事件广播器
    if (beanFactory.containsLocalBean(APPLICATION_EVENT_MULTICASTER_BEAN_NAME)) {
        this.applicationEventMulticaster =
                beanFactory.getBean(APPLICATION_EVENT_MULTICASTER_BEAN_NAME, ApplicationEventMulticaster.class);
        if (logger.isTraceEnabled()) {
            logger.trace("Using ApplicationEventMulticaster [" + this.applicationEventMulticaster + "]");
        }
    }
    else {
        //否则采用自定义的事件广播器
        this.applicationEventMulticaster = new SimpleApplicationEventMulticaster(beanFactory);
        beanFactory.registerSingleton(APPLICATION_EVENT_MULTICASTER_BEAN_NAME, this.applicationEventMulticaster);
        if (logger.isTraceEnabled()) {
            logger.trace("No '" + APPLICATION_EVENT_MULTICASTER_BEAN_NAME + "' bean, using " +
                    "[" + this.applicationEventMulticaster.getClass().getSimpleName() + "]");
        }
    }
}
```
向广播器中注册事件，详见spring消息广播及事件机制模块章节
```text
protected void registerListeners() {
    // Register statically specified listeners first.
    //配置硬编码方式注册的监听器处理
    for (ApplicationListener<?> listener : getApplicationListeners()) {
        getApplicationEventMulticaster().addApplicationListener(listener);
    }

    // Do not initialize FactoryBeans here: We need to leave all regular beans
    // uninitialized to let post-processors apply to them!
    //配置文件注册的监听器处理
    String[] listenerBeanNames = getBeanNamesForType(ApplicationListener.class, true, false);
    for (String listenerBeanName : listenerBeanNames) {
        getApplicationEventMulticaster().addApplicationListenerBean(listenerBeanName);
    }

    // Publish early application events now that we finally have a multicaster...
    //调用早期发布的事件
    Set<ApplicationEvent> earlyEventsToProcess = this.earlyApplicationEvents;
    this.earlyApplicationEvents = null;
    if (earlyEventsToProcess != null) {
        for (ApplicationEvent earlyEvent : earlyEventsToProcess) {
            getApplicationEventMulticaster().multicastEvent(earlyEvent);
        }
    }
}
```
实例化剩余的单例bean，到这一步才是正在开始出适合我们剩余的单例bean，为什么说剩余的呢，因为在此之前spring陆续初始化了部分
代码中可能用到的bean，详细见bean的实例化流程章节
```text
protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
    // Initialize conversion service for this context.
    if (beanFactory.containsBean(CONVERSION_SERVICE_BEAN_NAME) &&
            beanFactory.isTypeMatch(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class)) {
        beanFactory.setConversionService(
                beanFactory.getBean(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class));
    }

    // Register a default embedded value resolver if no bean post-processor
    // (such as a PropertyPlaceholderConfigurer bean) registered any before:
    // at this point, primarily for resolution in annotation attribute values.
    if (!beanFactory.hasEmbeddedValueResolver()) {
        beanFactory.addEmbeddedValueResolver(strVal -> getEnvironment().resolvePlaceholders(strVal));
    }

    // Initialize LoadTimeWeaverAware beans early to allow for registering their transformers early.
    String[] weaverAwareNames = beanFactory.getBeanNamesForType(LoadTimeWeaverAware.class, false, false);
    for (String weaverAwareName : weaverAwareNames) {
        getBean(weaverAwareName);
    }

    // Stop using the temporary ClassLoader for type matching.
    beanFactory.setTempClassLoader(null);

    // Allow for caching all bean definition metadata, not expecting further changes.
    //冻结所有的bean定义，说明注册的bean定义将不被修改或任何进一步的处理
    beanFactory.freezeConfiguration();

    // Instantiate all remaining (non-lazy-init) singletons.
    //初始化剩下的单实例（非惰性的）
    beanFactory.preInstantiateSingletons();
}
```
```text
public void preInstantiateSingletons() throws BeansException {
    if (logger.isTraceEnabled()) {
        logger.trace("Pre-instantiating singletons in " + this);
    }

    // Iterate over a copy to allow for init methods which in turn register new bean definitions.
    // While this may not be part of the regular factory bootstrap, it does otherwise work fine.
    List<String> beanNames = new ArrayList<>(this.beanDefinitionNames);

    // Trigger initialization of all non-lazy singleton beans...
    for (String beanName : beanNames) {
        RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
        //单例的且不是抽象类且不是懒加载的
        if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
            //FactoryBean处理
            if (isFactoryBean(beanName)) {
                Object bean = getBean(FACTORY_BEAN_PREFIX + beanName);
                if (bean instanceof FactoryBean) {
                    final FactoryBean<?> factory = (FactoryBean<?>) bean;
                    boolean isEagerInit;
                    if (System.getSecurityManager() != null && factory instanceof SmartFactoryBean) {
                        isEagerInit = AccessController.doPrivileged((PrivilegedAction<Boolean>)
                                        ((SmartFactoryBean<?>) factory)::isEagerInit,
                                getAccessControlContext());
                    }
                    else {
                        isEagerInit = (factory instanceof SmartFactoryBean &&
                                ((SmartFactoryBean<?>) factory).isEagerInit());
                    }
                    if (isEagerInit) {
                        getBean(beanName);
                    }
                }
            }
            else {
                //普通bean处理
                getBean(beanName);
            }
        }
    }
```
后续的流程请参考BeanFactory中的getBean的流程分析
容器刷新结束，完成整个容器的启动过程
```text
protected void finishRefresh() {
    // Clear context-level resource caches (such as ASM metadata from scanning).
    clearResourceCaches();

    // Initialize lifecycle processor for this context.
    //Lifecycle 接口,Lifecycle 中包含start/stop 方法，实现此接口后Spring 会
    //保证在启动的时候调用其start 方法开始生命周期，并在Spring 关闭的时候调用stop 方法来结束生
    //命周期，通常用来配置后台程序，在启动后一直运行（如对MQ 进行轮询等）。而ApplicationContext
    //的初始化最后正是保证了这一功能的实现。
    initLifecycleProcessor();

    // Propagate refresh to lifecycle processor first.
    getLifecycleProcessor().onRefresh();

    // Publish the final event.
    //发布ContextRefreshedEvent事件
    publishEvent(new ContextRefreshedEvent(this));

    // Participate in LiveBeansView MBean, if active.
    LiveBeansView.registerApplicationContext(this);
}
```
