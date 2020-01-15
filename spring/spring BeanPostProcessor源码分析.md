##spring后置处理器只BeanPostProcessor源码分析

###后置处理器的概念

###后置处理器调用时机

####spring共有9处调用了后置处理器
- 第一处 InstantiationAwareBeanPostProcessor
AbstractAutowireCapableBeanFactory.createBean(beanName, mbd, args)
```text
Object bean = resolveBeforeInstantiation(beanName, mbdToUse);
if (bean != null) {
    return bean;
}

protected Object resolveBeforeInstantiation(String beanName, RootBeanDefinition mbd) {
    Object bean = null;
    //如果尚未被解析
    if (!Boolean.FALSE.equals(mbd.beforeInstantiationResolved)) {
        // Make sure bean class is actually resolved at this point.
        if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
            Class<?> targetType = determineTargetType(beanName, mbd);
            if (targetType != null) {
                //应用后置处理器
                bean = applyBeanPostProcessorsBeforeInstantiation(targetType, beanName);
                if (bean != null) {
                    bean = applyBeanPostProcessorsAfterInitialization(bean, beanName);
                }
            }
        }
        mbd.beforeInstantiationResolved = (bean != null);
    }
    return bean;
}

```

- 第二处 SmartInstantiationAwareBeanPostProcessor
AbstractAutowireCapableBeanFactory.createBeanInstance(beanName, mbd, args)
```text
Constructor<?>[] ctors = determineConstructorsFromBeanPostProcessors(beanClass, beanName);

protected Constructor<?>[] determineConstructorsFromBeanPostProcessors(@Nullable Class<?> beanClass, String beanName)
			throws BeansException {

    if (beanClass != null && hasInstantiationAwareBeanPostProcessors()) {
        for (BeanPostProcessor bp : getBeanPostProcessors()) {
            if (bp instanceof SmartInstantiationAwareBeanPostProcessor) {
                SmartInstantiationAwareBeanPostProcessor ibp = (SmartInstantiationAwareBeanPostProcessor) bp;
                Constructor<?>[] ctors = ibp.determineCandidateConstructors(beanClass, beanName);
                if (ctors != null) {
                    return ctors;
                }
            }
        }
    }
    return null;
}
```

- 第三处 MergedBeanDefinitionPostProcessor
AbstractAutowireCapableBeanFactory.createBeanInstance(beanName, mbd, args)
```text
synchronized (mbd.postProcessingLock) {
    if (!mbd.postProcessed) {
        try {
            //应用后置处理器
            //第三次调用后置处理器
            applyMergedBeanDefinitionPostProcessors(mbd, beanType, beanName);
        }
        catch (Throwable ex) {
            throw new BeanCreationException(mbd.getResourceDescription(), beanName,
                    "Post-processing of merged bean definition failed", ex);
        }
        mbd.postProcessed = true;
    }
}

protected void applyMergedBeanDefinitionPostProcessors(RootBeanDefinition mbd, Class<?> beanType, String beanName) {
    for (BeanPostProcessor bp : getBeanPostProcessors()) {
        if (bp instanceof MergedBeanDefinitionPostProcessor) {
            MergedBeanDefinitionPostProcessor bdp = (MergedBeanDefinitionPostProcessor) bp;
            bdp.postProcessMergedBeanDefinition(mbd, beanType, beanName);
        }
    }
}

```

- 第四处 SmartInstantiationAwareBeanPostProcessor
AbstractAutowireCapableBeanFactory.createBeanInstance(beanName, mbd, args)
```text
if (earlySingletonExposure) {
    ...
    //为避免后期循环依赖，可以在bean初始化完成前将创建实例的ObjectFactory加入工厂
    //实际上此时加入的是已经实例化但尚未初始化完成的bean
    //第四次调用后置处理器
    addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
}
```

- 第五处 InstantiationAwareBeanPostProcessor
AbstractAutowireCapableBeanFactory.populateBean(beanName, mbd, bw)
```text
if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
    for (BeanPostProcessor bp : getBeanPostProcessors()) {
        if (bp instanceof InstantiationAwareBeanPostProcessor) {
            InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
            if (!ibp.postProcessAfterInstantiation(bw.getWrappedInstance(), beanName)) {
                return;
            }
        }
    }
}
```

- 第六处 InstantiationAwareBeanPostProcessor
AbstractAutowireCapableBeanFactory.populateBean(beanName, mbd, bw)
```text
for (BeanPostProcessor bp : getBeanPostProcessors()) {
    if (bp instanceof InstantiationAwareBeanPostProcessor) {
        InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
        PropertyValues pvsToUse = ibp.postProcessProperties(pvs, bw.getWrappedInstance(), beanName);
        if (pvsToUse == null) {
            if (filteredPds == null) {
                filteredPds = filterPropertyDescriptorsForDependencyCheck(bw, mbd.allowCaching);
            }
            pvsToUse = ibp.postProcessPropertyValues(pvs, filteredPds, bw.getWrappedInstance(), beanName);
            if (pvsToUse == null) {
                return;
            }
        }
        pvs = pvsToUse;
    }
}
```

- 第七处 spring生命周期初始化方法前，容器中的后置处理器
AbstractAutowireCapableBeanFactory.initializeBean(beanName, bean, mbd)
```text
if (mbd == null || !mbd.isSynthetic()) {
    //应用后处理器
    //第七次调用后置处理器
    wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
}

public Object applyBeanPostProcessorsBeforeInitialization(Object existingBean, String beanName)
			throws BeansException {

    Object result = existingBean;
    for (BeanPostProcessor processor : getBeanPostProcessors()) {
        Object current = processor.postProcessBeforeInitialization(result, beanName);
        if (current == null) {
            return result;
        }
        result = current;
    }
    return result;
}

```

- 第八处 spring生命周期初始化方法后，容器中的后置处理器
AbstractAutowireCapableBeanFactory.initializeBean(beanName, bean, mbd)
```text
if (mbd == null || !mbd.isSynthetic()) {
    //后处理器应用
    //第八次调用后置处理器
    wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
}

public Object applyBeanPostProcessorsAfterInitialization(Object existingBean, String beanName)
			throws BeansException {

    Object result = existingBean;
    for (BeanPostProcessor processor : getBeanPostProcessors()) {
        Object current = processor.postProcessAfterInitialization(result, beanName);
        if (current == null) {
            return result;
        }
        result = current;
    }
    return result;
}

```

###spring中的后置处理器
1.InstantiationAwareBeanPostProcessor
```text
InstantiationAwareBeanPostProcessor该类主要有三个方法
1.postProcessBeforeInstantiation()在spring尚未实例化对象时，将类的Class信息及beanName信息暴露出来，给与开发者自定义处理的机
会，这里可以做如AOP的代理类创建功能等
2.postProcessAfterInstantiation()在对象已经实例化了但尚未属性赋值，spring给与开发者一个机会来自定义处理，只要有任何一个后
置处理器，返回false，那么populateBean属性注入的方法都会终止，即后面的属性将不会被应用
3.postProcessProperties()将要执行属性注入，但尚未属性注入之前，将所有属性及对象信息暴露出来，给与开发者一个自定义的机会
```
```text
#目标对象尚未实例化，可以在此创建代理对象，参考案例
default Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
    return null;
}

#对象已经调用构造方法或者工厂方法实例化后，但尚未完成属性注入前，这个方法可以允许给一个给定的对象进行自定义属性注入，在
#spring自定注入之前
#调用时间在AbstractAutowireCapableBeanFactory.populateBean属性注入前调用
default boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {
    return true;
}

#对象已经创建，但是尚未设置属性值，spring将所有属性及实例化对象都暴露出来，给与开发者一次自定义的机会
default PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName) throws BeansException {
    return null;
}

```
```text
@Component
public class OrderServiceInstantiationAwareBeanPostProcessor implements InstantiationAwareBeanPostProcessor {

    public Object postProcessBeforeInstantiation (Class<?> beanClass, String beanName) throws BeansException {
        if(beanClass == UserServiceimpl.class){
            System.out.println ("class:"+UserServiceimpl.class.getName ());
            try {
                Constructor<?> constructor = beanClass.getConstructor (null);
                constructor.setAccessible (true);
                Object instance = constructor.newInstance ();
                if(instance != null){
                    Class<?>[] interfaces = beanClass.getInterfaces ();
                    return Proxy.newProxyInstance (getClass ().getClassLoader (), interfaces, new UserServiceInvocationHandler(instance));
                }
            } catch (NoSuchMethodException e) {
                e.printStackTrace ();
            } catch (IllegalAccessException e) {
                e.printStackTrace ();
            } catch (InstantiationException e) {
                e.printStackTrace ();
            } catch (InvocationTargetException e) {
                e.printStackTrace ();
            }
        }
        return null;
    }

    public boolean postProcessAfterInstantiation (Object bean, String beanName) throws BeansException {
        System.out.println (beanName+"=\t"+bean+"\t"+bean.getClass ());

        return true;
    }

    public class UserServiceInvocationHandler implements InvocationHandler{
        private Object target;

        public UserServiceInvocationHandler (Object target) {
            this.target = target;
        }

        public Object invoke (Object proxy, Method method, Object[] args) throws Throwable {
            System.out.println ("进入代理方法前...");
            Object result = method.invoke (target, args);
            System.out.println ("进入代理方法后...");
            return result;
        }
    }
}
```

2.SmartInstantiationAwareBeanPostProcessor      
```
预测Bean的类型，返回第一个预测成功的Class类型，如果不能预测返回null
default Class<?> predictBeanType(Class<?> beanClass, String beanName) throws BeansException {
    return null;
}

选择合适的构造器，比如目标对象有多个构造器，在这里可以进行一些定制化，选择合适的构造器
default Constructor<?>[] determineCandidateConstructors(Class<?> beanClass, String beanName)
        throws BeansException {

    return null;
}

#获取提前暴露的bean的引用
default Object getEarlyBeanReference(Object bean, String beanName) throws BeansException {
    return bean;
}
```

3.MergedBeanDefinitionPostProcessor
```text
#合并bean的定义信息
postProcessMergedBeanDefinition(RootBeanDefinition beanDefinition, Class<?> beanType, String beanName);
```
4.BeanPostProcessor处理器
1.AutowiredAnnotationBeanPostProcessor处理@AutoWired和@Value注解
```text
public AutowiredAnnotationBeanPostProcessor() {
    this.autowiredAnnotationTypes.add(Autowired.class);
    this.autowiredAnnotationTypes.add(Value.class);
    try {
        this.autowiredAnnotationTypes.add((Class<? extends Annotation>)
                ClassUtils.forName("javax.inject.Inject", AutowiredAnnotationBeanPostProcessor.class.getClassLoader()));
        logger.trace("JSR-330 'javax.inject.Inject' annotation found and supported for autowiring");
    }
    catch (ClassNotFoundException ex) {
        // JSR-330 API not available - simply skip.
    }
}

public PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName) {
    InjectionMetadata metadata = findAutowiringMetadata(beanName, bean.getClass(), pvs);
    try {
        metadata.inject(bean, beanName, pvs);
    }
    catch (BeanCreationException ex) {
        throw ex;
    }
    catch (Throwable ex) {
        throw new BeanCreationException(beanName, "Injection of autowired dependencies failed", ex);
    }
    return pvs;
}

#属性注入
protected void inject(Object bean, @Nullable String beanName, @Nullable PropertyValues pvs) throws Throwable {
        Field field = (Field) this.member;
        Object value;
        if (this.cached) {
            value = resolvedCachedArgument(beanName, this.cachedFieldValue);
        }
        else {
            DependencyDescriptor desc = new DependencyDescriptor(field, this.required);
            desc.setContainingClass(bean.getClass());
            Set<String> autowiredBeanNames = new LinkedHashSet<>(1);
            Assert.state(beanFactory != null, "No BeanFactory available");
            TypeConverter typeConverter = beanFactory.getTypeConverter();
            try {
                value = beanFactory.resolveDependency(desc, beanName, autowiredBeanNames, typeConverter);
            }
            catch (BeansException ex) {
                throw new UnsatisfiedDependencyException(null, beanName, new InjectionPoint(field), ex);
            }
            synchronized (this) {
                if (!this.cached) {
                    if (value != null || this.required) {
                        this.cachedFieldValue = desc;
                        registerDependentBeans(beanName, autowiredBeanNames);
                        if (autowiredBeanNames.size() == 1) {
                            String autowiredBeanName = autowiredBeanNames.iterator().next();
                            if (beanFactory.containsBean(autowiredBeanName) &&
                                    beanFactory.isTypeMatch(autowiredBeanName, field.getType())) {
                                this.cachedFieldValue = new ShortcutDependencyDescriptor(
                                        desc, autowiredBeanName, field.getType());
                            }
                        }
                    }
                    else {
                        this.cachedFieldValue = null;
                    }
                    this.cached = true;
                }
            }
        }
        if (value != null) {
            ReflectionUtils.makeAccessible(field);
            field.set(bean, value);
        }
    }
}

#set方法注入
protected void inject(Object bean, @Nullable String beanName, @Nullable PropertyValues pvs) throws Throwable {
    if (checkPropertySkipping(pvs)) {
        return;
    }
    Method method = (Method) this.member;
    Object[] arguments;
    if (this.cached) {
        // Shortcut for avoiding synchronization...
        arguments = resolveCachedArguments(beanName);
    }
    else {
        int argumentCount = method.getParameterCount();
        arguments = new Object[argumentCount];
        DependencyDescriptor[] descriptors = new DependencyDescriptor[argumentCount];
        Set<String> autowiredBeans = new LinkedHashSet<>(argumentCount);
        Assert.state(beanFactory != null, "No BeanFactory available");
        TypeConverter typeConverter = beanFactory.getTypeConverter();
        for (int i = 0; i < arguments.length; i++) {
            MethodParameter methodParam = new MethodParameter(method, i);
            DependencyDescriptor currDesc = new DependencyDescriptor(methodParam, this.required);
            currDesc.setContainingClass(bean.getClass());
            descriptors[i] = currDesc;
            try {
                Object arg = beanFactory.resolveDependency(currDesc, beanName, autowiredBeans, typeConverter);
                if (arg == null && !this.required) {
                    arguments = null;
                    break;
                }
                arguments[i] = arg;
            }
            catch (BeansException ex) {
                throw new UnsatisfiedDependencyException(null, beanName, new InjectionPoint(methodParam), ex);
            }
        }
        synchronized (this) {
            if (!this.cached) {
                if (arguments != null) {
                    DependencyDescriptor[] cachedMethodArguments = Arrays.copyOf(descriptors, arguments.length);
                    registerDependentBeans(beanName, autowiredBeans);
                    if (autowiredBeans.size() == argumentCount) {
                        Iterator<String> it = autowiredBeans.iterator();
                        Class<?>[] paramTypes = method.getParameterTypes();
                        for (int i = 0; i < paramTypes.length; i++) {
                            String autowiredBeanName = it.next();
                            if (beanFactory.containsBean(autowiredBeanName) &&
                                    beanFactory.isTypeMatch(autowiredBeanName, paramTypes[i])) {
                                cachedMethodArguments[i] = new ShortcutDependencyDescriptor(
                                        descriptors[i], autowiredBeanName, paramTypes[i]);
                            }
                        }
                    }
                    this.cachedMethodArguments = cachedMethodArguments;
                }
                else {
                    this.cachedMethodArguments = null;
                }
                this.cached = true;
            }
        }
    }
    if (arguments != null) {
        try {
            ReflectionUtils.makeAccessible(method);
            method.invoke(bean, arguments);
        }
        catch (InvocationTargetException ex) {
            throw ex.getTargetException();
        }
    }
}
```
2.CommonAnnotationBeanPostProcessor处理Java注解如@Resource、@PostConstruct、@PreDestroy注解
```text
static {
    ...
    resourceAnnotationTypes.add(Resource.class);
    ...
}
	
public CommonAnnotationBeanPostProcessor() {
    setOrder(Ordered.LOWEST_PRECEDENCE - 3);
    setInitAnnotationType(PostConstruct.class);
    setDestroyAnnotationType(PreDestroy.class);
    ignoreResourceType("javax.xml.ws.WebServiceContext");
}

public PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName) {
    InjectionMetadata metadata = findResourceMetadata(beanName, bean.getClass(), pvs);
    try {
        metadata.inject(bean, beanName, pvs);
    }
    catch (Throwable ex) {
        throw new BeanCreationException(beanName, "Injection of resource dependencies failed", ex);
    }
    return pvs;
}

protected void inject(Object target, @Nullable String requestingBeanName, @Nullable PropertyValues pvs)
        throws Throwable {

    if (this.isField) {
        Field field = (Field) this.member;
        ReflectionUtils.makeAccessible(field);
        field.set(target, getResourceToInject(target, requestingBeanName));
    }
    else {
        if (checkPropertySkipping(pvs)) {
            return;
        }
        try {
            Method method = (Method) this.member;
            ReflectionUtils.makeAccessible(method);
            method.invoke(target, getResourceToInject(target, requestingBeanName));
        }
        catch (InvocationTargetException ex) {
            throw ex.getTargetException();
        }
    }
}

protected Object getResourceToInject(Object target, @Nullable String requestingBeanName) {
    return (this.lazyLookup ? buildLazyResourceProxy(this, requestingBeanName) :
            getResource(this, requestingBeanName));
}

protected Object getResource(LookupElement element, @Nullable String requestingBeanName)
			throws NoSuchBeanDefinitionException {

    if (StringUtils.hasLength(element.mappedName)) {
        return this.jndiFactory.getBean(element.mappedName, element.lookupType);
    }
    if (this.alwaysUseJndiLookup) {
        return this.jndiFactory.getBean(element.name, element.lookupType);
    }
    if (this.resourceFactory == null) {
        throw new NoSuchBeanDefinitionException(element.lookupType,
                "No resource factory configured - specify the 'resourceFactory' property");
    }
    return autowireResource(this.resourceFactory, element, requestingBeanName);
}

```
从这里也侧面解释了spring的三种注入方法，构造注入，set注入和Field注入
