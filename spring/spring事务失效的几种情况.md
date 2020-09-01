###spring事务失效的几种情况

- 没有被 Spring 管理
```text
// @Service
public class OrderServiceImpl implements OrderService {   
 @Transactional    
public void updateOrder(Order order) {       
 // update order；  
  }
}
```

- 方法不是public的以及其他不恰当的申明方式
该异常一般情况都会被编译器帮忙识别,spring官网有做说明

- 类中调用了本类的另一个方法
注意这里需要特殊处理一下即可

- 数据源没有配置事务管理器

- 不支持事务
传播行为为不支持事务

- 异常被吃了
自己内部捕获了异常

- 异常类型错误或格式配置错误
spring默认只会回滚RuntimeException

