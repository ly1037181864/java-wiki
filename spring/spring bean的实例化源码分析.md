###bean实例化源码分析

spring实例化bean的流程
![avatar](../imags/spring/spring-03.png) 
注意后面的属性注入，如果是自动注入则在BeanPostProcessor中完成，其他的属性才会走applyPropertyValues逻辑，设置属性时会根据
是set注入还是Field注入来选择不同的属性注入策略