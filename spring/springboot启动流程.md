##SpringBoot启动流程

```text
public static ConfigurableApplicationContext run(Class<?> primarySource, String... args) {
    return run(new Class<?>[] { primarySource }, args);
}
```
```text
public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
    this.resourceLoader = resourceLoader;
    Assert.notNull(primarySources, "PrimarySources must not be null");
    this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
    this.webApplicationType = WebApplicationType.deduceFromClasspath();
    //3、扫描当前路径下META-INF/spring.factories文件的，加载ApplicationContextInitializer接口实例
    setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));
    //4、扫描当前路径下META-INF/spring.factories文件的，加载ApplicationListener接口实例
    setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
    this.mainApplicationClass = deduceMainApplicationClass();
}
```

利用SPI机制扫描 META-INF/spring.factories 这个文件，并且加载 ApplicationContextInitializer、ApplicationListener 接口实例。
1、ApplicationContextInitializer 这个类当springboot上下文Context初始化完成后会调用
2、ApplicationListener 当springboot启动时事件change后都会触发

```text
public ConfigurableApplicationContext run(String... args) {
    <!--1、这个是一个计时器，没什么好说的-->
    StopWatch stopWatch = new StopWatch();
    stopWatch.start();
    ConfigurableApplicationContext context = null;
    Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
    <!--2、这个也不是重点，就是设置了一些环境变量-->
    configureHeadlessProperty();
    <!--3、获取事件监听器SpringApplicationRunListener类型，并且执行starting()方法-->
    SpringApplicationRunListeners listeners = getRunListeners(args);
    listeners.starting();
    try {
        <!--4、把参数args封装成DefaultApplicationArguments，这个了解一下就知道-->
        ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
        <!--5、这个很重要准备环境了，并且把环境跟spring上下文绑定好，并且执行environmentPrepared()方法-->
        ConfigurableEnvironment environment = prepareEnvironment(listeners, applicationArguments);
        <!--6、判断一些环境的值，并设置一些环境的值-->
        configureIgnoreBeanInfo(environment);
        <!--7、打印banner-->
        Banner printedBanner = printBanner(environment);
        <!--8、创建上下文，根据项目类型创建上下文-->
        context = createApplicationContext();
        <!--8、创建上下文，根据项目类型创建上下文-->
        exceptionReporters = getSpringFactoriesInstances(SpringBootExceptionReporter.class,
                new Class[] { ConfigurableApplicationContext.class }, context);
        <!--10、准备上下文，执行完成后调用contextPrepared()方法,contextLoaded()方法--
        prepareContext(context, environment, listeners, applicationArguments, printedBanner);
        <!--11、这个是spring启动的代码了，这里就回去里面就回去扫描并且初始化单实列bean了-->
        //这个refreshContext()加载了bean，还启动了内置web容器，需要细细的去看看
        refreshContext(context);
        <!--12、啥事情都没有做-->
        afterRefresh(context, applicationArguments);
        stopWatch.stop();
        if (this.logStartupInfo) {
            new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), stopWatch);
        }
        listeners.started(context);
        callRunners(context, applicationArguments);
    }
    catch (Throwable ex) {
        handleRunFailure(context, ex, exceptionReporters, listeners);
        throw new IllegalStateException(ex);
    }

    try {
        listeners.running(context);
    }
    catch (Throwable ex) {
        handleRunFailure(context, ex, exceptionReporters, null);
        throw new IllegalStateException(ex);
    }
    return context;
}
```