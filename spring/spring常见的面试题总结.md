##spring常见的面试题总结

###spring实例化和初始化的区别
实例化：从BeanDefinition->Java对象的过程叫实例化，这个时候spring完成了对bean进行反射创建Java对象的过程,但是spring并没有完成
bean的生命周期内所有的流程，如代理创建、属性注入、回调函数等等的处理
初始化：BeanDefinition已经完成了整个spring的生命周内的所有函数，对spring而言，用户可以直接使用该bean做任何操作

###spring后置处理器

###9个后置处理器
spring共有9处调用了后置处理器

###spring循环依赖问题
1.什么是循环依赖问题
2.spring是如何解决循环依赖问题的
3.如果关闭spring默认的循环依赖
见spring循环依赖章节解释

###spring事务

###spring bean的生命周期

###spring bean的生命周期回调函数

###三级缓存
```text
Map<String, Object> singletonObjects              一级缓存，单例缓存池
Map<String, ObjectFactory<?>> singletonFactories  二级缓存池，ObjectFactory缓存
Map<String, Object> earlySingletonObjects         三级缓存

```
singletonObjects：当bean实例化后，bean会被缓存到该map中去，以便后续的调用获取
earlySingletonObjects：bean已实例化，但尚未完成初始化，即尚未加入到单例缓存池中，且三级缓存已经被清除
singletonFactories:缓存ObjectFactory对象，用于调用getObject()方法，返回bean实例

###BeanFactory和ApplicationContext有什么区别

###spring bean的作用域

###spring自动装配
@Autowired和@Resource主动装配
```text
protected BeanDefinitionParserDelegate createDelegate(
			XmlReaderContext readerContext, Element root, @Nullable BeanDefinitionParserDelegate parentDelegate) {

    BeanDefinitionParserDelegate delegate = new BeanDefinitionParserDelegate(readerContext);
    //初始化默认的属性值
    delegate.initDefaults(root, parentDelegate);
    return delegate;
}
```
```text
protected void populateDefaults(DocumentDefaultsDefinition defaults, @Nullable DocumentDefaultsDefinition parentDefaults, Element root) {

    //autowire自动注入设置默认值
    String autowire = root.getAttribute(DEFAULT_AUTOWIRE_ATTRIBUTE);
    if (isDefaultValue(autowire)) {
        // Potentially inherited from outer <beans> sections, otherwise falling back to 'no'.
        autowire = (parentDefaults != null ? parentDefaults.getAutowire() : AUTOWIRE_NO_VALUE);
    }
    defaults.setAutowire(autowire);

    //设置autowire-candidate
    if (root.hasAttribute(DEFAULT_AUTOWIRE_CANDIDATES_ATTRIBUTE)) {
        defaults.setAutowireCandidates(root.getAttribute(DEFAULT_AUTOWIRE_CANDIDATES_ATTRIBUTE));
    }
    else if (parentDefaults != null) {
        defaults.setAutowireCandidates(parentDefaults.getAutowireCandidates());
    }
}
```
```text
public AbstractBeanDefinition parseBeanDefinitionElement(
			Element ele, String beanName, @Nullable BeanDefinition containingBean) {

    ...

    //硬编码解析默认bean的各种属性
    parseBeanDefinitionAttributes(ele, beanName, containingBean, bd);
    ...
}
```
```text
public AbstractBeanDefinition parseBeanDefinitionAttributes(Element ele, String beanName,
			@Nullable BeanDefinition containingBean, AbstractBeanDefinition bd) {

    ...
    //autowite解析
    String autowire = ele.getAttribute(AUTOWIRE_ATTRIBUTE);
    //设置自动注入类型
    bd.setAutowireMode(getAutowireMode(autowire));

    ...
    return bd;
}
```
```text
public int getAutowireMode(String attrValue) {
    String attr = attrValue;
    if (isDefaultValue(attr)) {
        attr = this.defaults.getAutowire();
    }
    int autowire = AbstractBeanDefinition.AUTOWIRE_NO;
    if (AUTOWIRE_BY_NAME_VALUE.equals(attr)) {
        autowire = AbstractBeanDefinition.AUTOWIRE_BY_NAME;
    }
    else if (AUTOWIRE_BY_TYPE_VALUE.equals(attr)) {
        autowire = AbstractBeanDefinition.AUTOWIRE_BY_TYPE;
    }
    else if (AUTOWIRE_CONSTRUCTOR_VALUE.equals(attr)) {
        autowire = AbstractBeanDefinition.AUTOWIRE_CONSTRUCTOR;
    }
    else if (AUTOWIRE_AUTODETECT_VALUE.equals(attr)) {
        autowire = AbstractBeanDefinition.AUTOWIRE_AUTODETECT;
    }
    // Else leave default value.
    return autowire;
}
```
正常情况下spring自动装配默认(xml的形式，即低版本的spring)是不自动装配的，即默认情况下resolvedAutowireMode为0不自动装配，
后续的spring版本中逐渐支持注解的方式来实现自动装配。
spring自动装配一般在xml中用的比较多，即在注解@Autowired等出现前用的比较多
```text
<bean class="com.viewscenes.netsupervisor.entity.Role">
    <property name="id" value="1001"></property>
    <property name="name" value="管理员"></property>
</bean>

<bean id="user" class="com.viewscenes.netsupervisor.entity.User" autowire="byType"></bean>
```
spring自动装配的类型
- 构造注入
```text
public class User{
    private Role role;

    public User(Role role) {
        this.role = role;
    }
}

<bean id="user" class="com.viewscenes.netsupervisor.entity.User" autowire="constructor"></bean>

```
- byType
```text
<bean class="com.viewscenes.netsupervisor.entity.Role">
    <property name="id" value="1001"></property>
    <property name="name" value="管理员"></property>
</bean>

<bean id="user" class="com.viewscenes.netsupervisor.entity.User" autowire="byType"></bean>
```
- byName
```text
<bean id="myRole" class="com.viewscenes.netsupervisor.entity.Role">
    <property name="id" value="1001"></property>
    <property name="name" value="管理员"></property>
</bean>

<bean id="user" class="com.viewscenes.netsupervisor.entity.User" autowire="byName"></bean>
```
- autodetect混合模式(已废弃)
在这种模式下spring会首先使用constructor进行自动装配，如果失败再尝试使用byType
```text
int resolvedAutowireMode = mbd.getResolvedAutowireMode();
if (resolvedAutowireMode == AUTOWIRE_BY_NAME || resolvedAutowireMode == AUTOWIRE_BY_TYPE) {
    MutablePropertyValues newPvs = new MutablePropertyValues(pvs);
    // Add property values based on autowire by name if applicable.
    //name注入
    if (resolvedAutowireMode == AUTOWIRE_BY_NAME) {
        autowireByName(beanName, mbd, bw, newPvs);
    }
    // Add property values based on autowire by type if applicable.
    //类型注入
    if (resolvedAutowireMode == AUTOWIRE_BY_TYPE) {
        autowireByType(beanName, mbd, bw, newPvs);
    }
    pvs = newPvs;
}
```
- 默认情况下spring不自动装配

在后续的spring版本中逐渐采用注解的方式实现自动装配
- @Autowired
- @Resource
- @Inject

###spring属性注入的方式
- 构造注入
- set注入
- field注入

###spring中@Autowired和@Resource注解的区别
- 相同点       
1.@Autowired和@Resource都用来什么注入属性bean的，都可以用来注入属性bean      
2.两者注入的时机都是在bean初始化属性注入的时候调用        
3.都是通过BeanPostProcessor处理的      
- 不同点       
1.@Autowired是由AutowiredAnnotationBeanPostProcessor处理器处理的        
2.@Resource则是由CommonAnnotationBeanPostProcessor处理器处理的       

