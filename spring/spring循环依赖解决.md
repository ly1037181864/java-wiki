##spring解决循环依赖的问题

####singleton类型
- spring默认支持单例的循环引用的
```text
public abstract class AbstractAutowireCapableBeanFactory extends AbstractBeanFactory
		implements AutowireCapableBeanFactory {
		
    /** Whether to automatically try to resolve circular references between beans. */
    private boolean allowCircularReferences = true;
}
``` 
从这里可以看出spring是默认支持循环引用的，但对构造注入的循环引用spring是不支持的，因为构造注入循环引用无法解决对象之间的
创建问题。

1.当创建某个对象时,spring会首先尝试从单例缓存池中获取，由于是首次创建，所以单例池中肯定不存在该对象
```text
Object sharedInstance = getSingleton(beanName);
```
```text
public Object getSingleton(String beanName) {
    //参数true设置标识允许早期依赖
    return getSingleton(beanName, true);
}
```
```text
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
    #此时单例池中没有
    Object singletonObject = this.singletonObjects.get(beanName);
    if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
        ...
    }
    return singletonObject;
}
```
2.spring开始创建该对象
```text
getSingleton(beanName, () -> {
    try {
        //创建bean的实例
        return createBean(beanName, mbd, args);
    }
    ...
});
```
3.在spring创建对象时，会将该对象的beanName放入到singletonsCurrentlyInCreation Set集合中去，这一步很重要，在后续的循环
引用中会用到
```text
protected void beforeSingletonCreation(String beanName) {
    #beanName加入到singletonsCurrentlyInCreation集合中去
    if (!this.inCreationCheckExclusions.contains(beanName) && !this.singletonsCurrentlyInCreation.add(beanName)) {
        throw new BeanCurrentlyInCreationException(beanName);
    }
}
```
4.然后继续创建对象，在创建完对象实例后，spring会根据条件将暴露单例bean的ObjectFactory对象加入三级缓存中去
```text
//创建bean的实例
instanceWrapper = createBeanInstance(beanName, mbd, args);
```
```text
boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
				isSingletonCurrentlyInCreation(beanName));
//如果需要提前曝光
if (earlySingletonExposure) {
    if (logger.isTraceEnabled()) {
        logger.trace("Eagerly caching bean '" + beanName +
                "' to allow for resolving potential circular references");
    }
    //为避免后期循环依赖，可以在bean初始化完成前将创建实例的ObjectFactory加入工厂
    //实际上此时加入的是已经实例化但尚未初始化完成的bean
    //第四次调用后置处理器
    addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
}
```
```text
protected void addSingletonFactory(String beanName, ObjectFactory<?> singletonFactory) {
    Assert.notNull(singletonFactory, "Singleton factory must not be null");
    synchronized (this.singletonObjects) {
        if (!this.singletonObjects.containsKey(beanName)) {
            this.singletonFactories.put(beanName, singletonFactory);
            this.earlySingletonObjects.remove(beanName);
            this.registeredSingletons.add(beanName);
        }
    }
}
```
上述判断中spring主要判断当前bean是否是单例的，且系统是否开启循环依赖，且当前bean是否正在创建中，正常来说bean默认就是单例
的，且该bean已经在步骤3中已经加入到了singletonsCurrentlyInCreation集合中去，因此都满足，而spring默认又是支持循环引用的，
所以earlySingletonExposure提前暴露此时肯定是true，于是spring将提前暴露该bean的ObjectFactory存放到了spring的三级缓存中
singletonFactories中,此时spring创建的bean是不完整的bean，其循环引用的属性尚未赋值。

5.spring继续执行populateBean()方法对其属性进行赋值，即再次重复执行getBean操作，当实例化该属性Bean的时候，同样需要对其循环
依赖bean执行属性注入populateBean()方法，即再次执行getBean逻辑，二次获取当前Bean。

6.当二次对当前bean执行getBean()操作时，此时会再次执行getSingleton()方法,再次判断if (singletonObject == null && 
isSingletonCurrentlyInCreation(beanName))条件，singletonObject == null成立，因为当前bean尚未执行完populateBean()，即尚未
加入到单例缓存池中，isSingletonCurrentlyInCreation(beanName)也成立，因为早在步骤3中当前beanName已经加入到正在创建bean的集
合中，因此程序会继续从三级缓存中获取ObjectFactory对象，并调用getObject()方法，来获取已经暴露的Bean实例
```text
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
		Object singletonObject = this.singletonObjects.get(beanName);
		if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
			synchronized (this.singletonObjects) {
				//二级缓存，提前暴露的单例池
				singletonObject = this.earlySingletonObjects.get(beanName);
				if (singletonObject == null && allowEarlyReference) {
					//当某些方法需要提前初始化的时候则会调用方法将对应的addsingletonFactory方法，将对应的ObjectFactory
					// 初始化策略存储在singletonFactories
					//三级缓存singletonFactories缓存
					ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
					if (singletonFactory != null) {
						//调用addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));中的getEarlyBeanReference方法
						//暴露给SmartInstantiationAwareBeanPostProcessor后置处理器进行处理
						singletonObject = singletonFactory.getObject();
						this.earlySingletonObjects.put(beanName, singletonObject);
						this.singletonFactories.remove(beanName);
					}
				}
			}
		}
		return singletonObject;
	}
```
7.当获取到Bean实例时，spring会将其放入二级缓存中去，并从三级缓存将ObjectFactory对象移除，思考为什么spring这么设计，为什么
需要三级缓存。         
初步设想，spring此时仍然尚未完成populateBean()方法，即当前bean并没有加入到单例缓存池中，而三级缓存中的ObjectFactory对象此
时已经被移除，如果其他对象在当前bean仍然还未执行完populateBean()方法将当前bean加入到单例缓存池前，需要依赖当前bean，那么
可以从二级缓存中获取。

8.当获得bean的实例后，会依次完成属性注入，并回到最初的getSingleton(beanName,singletonFactory)方法中去，继续完成后续的逻辑
即执行addSingleton()方法，将该单例bean放入到单例缓存池中，并依次移除二级三级缓存，到此单例的循环引用问题解决
```text
protected void addSingleton(String beanName, Object singletonObject) {
    synchronized (this.singletonObjects) {
        this.singletonObjects.put(beanName, singletonObject);
        this.singletonFactories.remove(beanName);
        this.earlySingletonObjects.remove(beanName);
        this.registeredSingletons.add(beanName);
    }
}
```
![avatar](../imags/spring/spring-04.png) 


####prototype类型
- 该类型的直接报错，spring不支持该类型的循环引用

####spring关闭循环引用
方法1
```text
AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
//关闭循环引用
context.setAllowCircularReferences (false);
context.register (AppConfig.class);
context.refresh ();
```
方法2
```text
@Component
public class MyBeanDefinitionRegistryPostProcessor implements BeanDefinitionRegistryPostProcessor {

    public static boolean flag = false;
    
    #自定义后置处理器设置，正常逻辑不能这样处理，会多次被调用
    public void postProcessBeanFactory (ConfigurableListableBeanFactory configurableListableBeanFactory) throws BeansException {
        System.out.println ("=============我自定义的BeanDefinitionRegistryPostProcessor.postProcessBeanFactory============");
        if(!flag){
            AbstractAutowireCapableBeanFactory factory = (AbstractAutowireCapableBeanFactory)configurableListableBeanFactory;
            factory.setAllowCircularReferences (false);
            flag = true;
        }
    }
}
```