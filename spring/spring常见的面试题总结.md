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

###spring事务

###spring bean的生命周期

###spring bean的生命周期回调函数

###三级缓存

###BeanFactory和ApplicationContext有什么区别

###spring bean的作用域

###spring自动装配

