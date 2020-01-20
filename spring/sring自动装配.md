##spring自动装配

- spring自动装配的类型
```text
public interface AutowireCapableBeanFactory{

    //无需自动装配
    int AUTOWIRE_NO = 0;

    //按名称自动装配bean属性
    int AUTOWIRE_BY_NAME = 1;

    //按类型自动装配bean属性
    int AUTOWIRE_BY_TYPE = 2;

    //按构造器自动装配
    int AUTOWIRE_CONSTRUCTOR = 3;

    //过时方法，Spring3.0之后不再支持
    @Deprecated
    int AUTOWIRE_AUTODETECT = 4;
}
```

- 基于xml配置的自动装配
1.byName 
根据属性名自动装配对应的bean
```text
public class User{
    private Role myRole;#根据属性名myRole，查找容器中对应的bean
}
public class Role {
    private String id;  
    private String name;
}
```
```text
<bean id="myRole" class="com.viewscenes.netsupervisor.entity.Role">
    <property name="id" value="1001"></property>
    <property name="name" value="管理员"></property>
</bean>

<bean id="user" class="com.viewscenes.netsupervisor.entity.User" autowire="byName"></bean>
```

2.byType
根据属性类型自动装配对应的bean
```text
public class User{
    private Role myRole;#根据属性类型，查找容器中对应的bean
}
public class Role {
    private String id;  
    private String name;
}
```
```text
<bean class="com.viewscenes.netsupervisor.entity.Role">
    <property name="id" value="1001"></property>
    <property name="name" value="管理员"></property>
</bean>

<bean id="user" class="com.viewscenes.netsupervisor.entity.User" autowire="byType"></bean>
```

3.constructor
把与Bean的构造器入参具有相同类型的其他Bean自动装配到Bean构造器的对应入参中，这种与构造注入又有关系
```text
public class User{
    private Role role;

    public User(Role role) {
        this.role = role;
    }
}
```
```text
<bean id="user" class="com.viewscenes.netsupervisor.entity.User" autowire="constructor"></bean>
```

4.autodetect
它首先会尝试使用constructor进行自动装配，如果失败再尝试使用byType。不过，它在Spring3.0之后已经被标记为@Deprecated。            

- 基于注解的自动装配
从Spring2.5开始，开始支持使用注解来自动装配Bean的属性。它允许更细粒度的自动装配，我们可以选择性的标注某一个属性来对其应用
自动装配。

Spring支持几种不同的应用于自动装配的注解。
1.Spring自带的@Autowired注解。
2.JSR-330的@Inject注解。
3.JSR-250的@Resource注解。

1.使用@Autowired注解
```text
@Autowired
UserService userService;
```
注意事项
a.强制性
默认情况下，它具有强制契约特性，其所标注的属性必须是可装配的。如果没有Bean可以装配到Autowired所标注的属性或参数中，那么你
会看到NoSuchBeanDefinitionException的异常信息。
```text
public Object doResolveDependency(DependencyDescriptor descriptor, String beanName,
            Set<String> autowiredBeanNames, TypeConverter typeConverter) throws BeansException {
    
    //查找Bean
    Map<String, Object> matchingBeans = findAutowireCandidates(beanName, type, descriptor);
    //如果拿到的Bean集合为空，且isRequired，就抛出异常。
    if (matchingBeans.isEmpty()) {
        if (descriptor.isRequired()) {
            raiseNoSuchBeanDefinitionException(type, "", descriptor);
        }
        return null;
    }
}
```
看到上面的源码，我们可以得到这一信息，Bean集合为空不要紧，关键isRequired条件不能成立，那么，如果我们不确定属性是否可以装
配，可以这样来使用Autowired。
```text
@Autowired(required=false)
UserService userService;
```
b.装配策略
我记得曾经有个面试题是这样问的：Autowired是按照什么策略来自动装配的呢？
关于这个问题，不能一概而论，你不能简单的说按照类型或者按照名称。但可以确定的一点的是，它默认是按照类型来自动装配的，即byType。
(a).默认按照类型装配
```text
protected Map<String, Object> findAutowireCandidates(
        String beanName, Class<?> requiredType, DependencyDescriptor descriptor) {
    
    //获取给定类型的所有bean名称，里面实际循环所有的beanName，获取它的实例
    //再通过isTypeMatch方法来确定
    String[] candidateNames = BeanFactoryUtils.beanNamesForTypeIncludingAncestors(
            this, requiredType, true, descriptor.isEager());
            
    Map<String, Object> result = new LinkedHashMap<String, Object>(candidateNames.length);
    
    //根据返回的beanName，获取其实例返回
    for (String candidateName : candidateNames) {
        if (!isSelfReference(beanName, candidateName) && isAutowireCandidate(candidateName, descriptor)) {
            result.put(candidateName, getBean(candidateName));
        }
    }
    return result;
}
```
(b).按照名称装配
可以看到它返回的是一个列表，那么就表明，按照类型匹配可能会查询到多个实例。到底应该装配哪个实例呢？我看有的文章里说，可以
加注解以此规避。
比如@qulifier、@Primary等，实际还有个简单的办法。比如，按照UserService接口类型来装配它的实现类。UserService接口有多个实现
类，分为UserServiceImpl、UserServiceImpl2。那么我们在注入的时候，就可以把属性名称定义为Bean实现类的名称。[这种方法需要考
虑下文分析的优先级的问题，spring针对多个同类型属性注入的情况，先是根据注解@Primary解析一次，找到即返回，否则解析@Priority
注解的，找到即返回，如果还没有则，继续根据属性的名称来配，找到即返回，否则报错，也就是说这种考虑其实是牺牲了性能，如果IOC
容器中真的存在同类型的多个的情况，应该指明注入的具体是那个bean，或者借用@Primary或@Priority，以提升spring性能]

```text
@Autowired
UserService UserServiceImpl2;
```
这样的话，Spring会按照byName来进行装配。首先，如果查到类型的多个实例，Spring已经做了判断。

```text
public Object doResolveDependency(DependencyDescriptor descriptor, String beanName,
            Set<String> autowiredBeanNames, TypeConverter typeConverter) throws BeansException {
            
    //按照类型查找Bean实例
    Map<String, Object> matchingBeans = findAutowireCandidates(beanName, type, descriptor);
    //如果Bean集合为空，且isRequired成立就抛出异常
    if (matchingBeans.isEmpty()) {
        if (descriptor.isRequired()) {
            raiseNoSuchBeanDefinitionException(type, "", descriptor);
        }
        return null;
    }
    //如果查找的Bean实例大于1个
    if (matchingBeans.size() > 1) {
        //找到最合适的那个，如果没有合适的。。也抛出异常
        String primaryBeanName = determineAutowireCandidate(matchingBeans, descriptor);
        if (primaryBeanName == null) {
            throw new NoUniqueBeanDefinitionException(type, matchingBeans.keySet());
        }
        if (autowiredBeanNames != null) {
            autowiredBeanNames.add(primaryBeanName);
        }
        return matchingBeans.get(primaryBeanName);
    }   
}
```
可以看出，如果查到多个实例，determineAutowireCandidate方法就是关键。它来确定一个合适的Bean返回。其中一部分就是按照Bean的名称来匹配。
```text
protected String determineAutowireCandidate(Map<String, Object> candidateBeans, 
                DependencyDescriptor descriptor) {
    //循环拿到的Bean集合
    for (Map.Entry<String, Object> entry : candidateBeans.entrySet()) {
        String candidateBeanName = entry.getKey();
        Object beanInstance = entry.getValue();
        //通过matchesBeanName方法来确定bean集合中的名称是否与属性的名称相同
        if (matchesBeanName(candidateBeanName, descriptor.getDependencyName())) {
            return candidateBeanName;
        }
    }
    return null;
}
```
最后我们回到问题上，得到的答案就是：@Autowired默认使用byType来装配属性，如果匹配到类型的多个实例，再通过byName来确定Bean。

c.主和优先级
上面我们已经看到了，通过byType可能会找到多个实例的Bean。然后再通过byName来确定一个合适的Bean，如果通过名称也确定不了呢？
还是determineAutowireCandidate这个方法，它还有两种方式来确定。
```text
protected String determineAutowireCandidate(Map<String, Object> candidateBeans, 
                DependencyDescriptor descriptor) {
    Class<?> requiredType = descriptor.getDependencyType();
    //通过@Primary注解来标识Bean
    String primaryCandidate = determinePrimaryCandidate(candidateBeans, requiredType);
    if (primaryCandidate != null) {
        return primaryCandidate;
    }
    //通过@Priority(value = 0)注解来标识Bean value为优先级大小
    String priorityCandidate = determineHighestPriorityCandidate(candidateBeans, requiredType);
    if (priorityCandidate != null) {
        return priorityCandidate;
    }
    return null;
}
```
(a).Primary
它的作用是看Bean上是否包含@Primary注解，如果包含就返回。当然了，你不能把多个Bean都设置为@Primary，不然你会得到
NoUniqueBeanDefinitionException这个异常。
```text
protected String determinePrimaryCandidate(Map<String, Object> candidateBeans, Class<?> requiredType) {
    String primaryBeanName = null;
    for (Map.Entry<String, Object> entry : candidateBeans.entrySet()) {
        String candidateBeanName = entry.getKey();
        Object beanInstance = entry.getValue();
        if (isPrimary(candidateBeanName, beanInstance)) {
            if (primaryBeanName != null) {
                boolean candidateLocal = containsBeanDefinition(candidateBeanName);
                boolean primaryLocal = containsBeanDefinition(primaryBeanName);
                if (candidateLocal && primaryLocal) {
                    throw new NoUniqueBeanDefinitionException(requiredType, candidateBeans.size(),
                            "more than one 'primary' bean found among candidates: " + candidateBeans.keySet());
                }
                else if (candidateLocal) {
                    primaryBeanName = candidateBeanName;
                }
            }
            else {
                primaryBeanName = candidateBeanName;
            }
        }
    }
    return primaryBeanName;
}
```

(b).Priority
你也可以在Bean上配置@Priority注解，它有个int类型的属性value，可以配置优先级大小。数字越小的，就被优先匹配。同样的，你也
不能把多个Bean的优先级配置成相同大小的数值，否则NoUniqueBeanDefinitionException异常照样出来找你。
```text
protected String determineHighestPriorityCandidate(Map<String, Object> candidateBeans, 
                                    Class<?> requiredType) {
    String highestPriorityBeanName = null;
    Integer highestPriority = null;
    for (Map.Entry<String, Object> entry : candidateBeans.entrySet()) {
        String candidateBeanName = entry.getKey();
        Object beanInstance = entry.getValue();
        Integer candidatePriority = getPriority(beanInstance);
        if (candidatePriority != null) {
            if (highestPriorityBeanName != null) {
                //如果优先级大小相同
                if (candidatePriority.equals(highestPriority)) {
                    throw new NoUniqueBeanDefinitionException(requiredType, candidateBeans.size(),
                        "Multiple beans found with the same priority ('" + highestPriority + "') " +
                            "among candidates: " + candidateBeans.keySet());
                }
                else if (candidatePriority < highestPriority) {
                    highestPriorityBeanName = candidateBeanName;
                    highestPriority = candidatePriority;
                }
            }
            else {
                highestPriorityBeanName = candidateBeanName;
                highestPriority = candidatePriority;
            }
        }
    }
    return highestPriorityBeanName;
}
```
最后，有一点需要注意。Priority的包在javax.annotation.Priority;，如果想使用它还要引入一个坐标。
```text
<dependency>
    <groupId>javax.annotation</groupId>
    <artifactId>javax.annotation-api</artifactId>
    <version>1.2</version>
</dependency>
```

d.装配策略总结
spring优先根据类型查找，如果找到多个，则依次按照解析@Primary、解析@Priority、匹配名称尝试获取，如果获取到了则直接返回，否
则继续尝试获取，如果都无法获取到则报错
```text
//当查找到的装配类大于1时，根据名称继续查找
if (matchingBeans.size() > 1) {
    autowiredBeanName = determineAutowireCandidate(matchingBeans, descriptor);
    if (autowiredBeanName == null) {
        if (isRequired(descriptor) || !indicatesMultipleBeans(type)) {
            return descriptor.resolveNotUnique(descriptor.getResolvableType(), matchingBeans);
        }
        else {
            // In case of an optional Collection/Map, silently ignore a non-unique case:
            // possibly it was meant to be an empty collection of multiple regular beans
            // (before 4.3 in particular when we didn't even look for collection beans).
            return null;
        }
    }
    instanceCandidate = matchingBeans.get(autowiredBeanName);
}
```
```text
protected String determineAutowireCandidate(Map<String, Object> candidates, DependencyDescriptor descriptor) {
    Class<?> requiredType = descriptor.getDependencyType();
    //@Primary注解解析
    String primaryCandidate = determinePrimaryCandidate(candidates, requiredType);
    if (primaryCandidate != null) {
        return primaryCandidate;
    }
    //@Priority注解解析
    String priorityCandidate = determineHighestPriorityCandidate(candidates, requiredType);
    if (priorityCandidate != null) {
        return priorityCandidate;
    }
    // Fallback
    //根据属性名称来匹配
    for (Map.Entry<String, Object> entry : candidates.entrySet()) {
        String candidateName = entry.getKey();
        Object beanInstance = entry.getValue();
        if ((beanInstance != null && this.resolvableDependencies.containsValue(beanInstance)) ||
                matchesBeanName(candidateName, descriptor.getDependencyName())) {
            return candidateName;
        }
    }
    return null;
}
```

e.@Autowired和@Resource装配的异同
- 相同点       
1.@Autowired和@Resource都用来什么注入属性bean的，都可以用来注入属性bean      
2.两者注入的时机都是在bean初始化属性注入的时候调用        
3.都是通过BeanPostProcessor处理的      
- 不同点       
1.@Autowired是由AutowiredAnnotationBeanPostProcessor处理器处理的        
2.@Resource则是由CommonAnnotationBeanPostProcessor处理器处理的     

- 自动装配的说明
a.显示依赖和自动装配的优先级
显式依赖项property和constructor-arg设置始终会覆盖自动装配
显示依赖:需要手动配置property属性或配置constructor-arg构造参数，来指定依赖的具体属性bean
```text
<bean id="beanOne" class="com.xiaoantimes.cn.ExampleBean" >
    <property name="manager" ref="manager" />
</bean>

<bean id="manager" class="com.xiaoantimes.cn.ManagerBean" />
```
自动装配：申明自动装配类型即可，spring会根据装配策略自动为bean注入依赖bean，而不再需要手动配置property属性或配置
constructor-arg构造参数
```text
<bean id="myRole" class="com.viewscenes.netsupervisor.entity.Role">
    <property name="id" value="1001"></property>
    <property name="name" value="管理员"></property>
</bean>

<bean id="user" class="com.viewscenes.netsupervisor.entity.User" autowire="byName"></bean>
```
b.autowire-candidate
从自动装配中排除bean，将</bean>元素的autowire-candidate属性设置false，该bean将不再自动装配

c.default-autowire-candidates
根据策略排除指定bean的自动装配功能，顶级<beans/>元素在其default-autowire-candidates属性内接受一个或多个模式，例如，要将
自动装配候选状态限制为名称以Repository结尾的任何bean，请提供值*Repository，要提供多种模式，请在以逗号分隔的列表中定义它们。
```text
#以Repository结尾的bean将不再支持自动装配功能
<beans default-autowire-candidates="*Repository"></beans>
```


b.自动装配的局限
1.自动装配不能用在基本类型、string、Classes如基本类型数组等
2.自动装配不如手动装配精确
3.自动装配要求bean具有唯一性
详细参考spring官网说明

- 总结
本章节重点阐述了Spring中的自动装配的几种策略，又通过源码分析了Autowired注解的使用方式。
在Spring3.0之后，有效的自动装配策略分为byType、byName、constructor三种方式。注解Autowired默认使用byType来自动装配，如果存
在类型的多个实例就尝试使用byName匹配，如果通过byName也确定不了，可以通过Primary和Priority注解来确定。



案例1，正常
```text
@Configuration
@ComponentScan("com.xiaoantimes.cn")
//@Component
public class AppConfig {

    @Bean
    public User user(School school){
        return new User(school);
    }

    @Bean
    public School school(){
        return new School();
    }

}
```
案例2，正常，按照类型找
```text
@Configuration
@ComponentScan("com.xiaoantimes.cn")
//@Component
public class AppConfig {

    #属性school到底转配的是school还是school2取决于usr()方法里的形参的名称，如果容器中只有一个，类型和名称都一样，但有多个
    #的时候，会根据形参名称来自动装配
    @Bean
    public User user(School school){
        return new User(school);
    }

    @Bean
    public School school2(){
        return new School();
    }
    
}
```
案例3，编译通不过
```text
@Configuration
@ComponentScan("com.xiaoantimes.cn")
//@Component
public class AppConfig {

    @Bean
    public User user(School school){#编译报错
        return new User(school);
    }

    @Bean
    public School school2(){
        return new School();
    }

    @Bean
    public School school3(){
        return new School();
    }

}
```


参考文档：https://www.jianshu.com/p/2f1c9fad1d2d
