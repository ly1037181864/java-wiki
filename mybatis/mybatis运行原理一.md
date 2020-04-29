##mybatis运行原理分析

###mybatis官网地址
http://mybatis.github.io/mybatis-3/
###mybatis git地址
https://github.com/mybatis/mybatis-3

当前比较流程的 mybatis-plus

####mybatis中使用到的设计模式
- 建造者模式
- 代理模式
- 单例模式
- 策略模式
- 工厂模式
- 装饰器模式
- 适配器模式

####mybatis配置文件解析
- mybatis-config.xml文件解析[configurationg标签解析]
1.配置文件解析过程中，mybatis-config.xml的configurationg标签对应我们的Configuration配置类，且全局唯一
2.配置文件解析主要是采用了建造者模式SqlSessionFactoryBuilder类，其创建SqlSessionFactory的对象实例便是采用建造者模式，因为
创建SqlSessionFactory类需要解析大量的配置文件，其过程繁杂冗长，采用建造者模式最好。
```text
public SqlSessionFactory build(Reader reader, String environment, Properties properties) {
    try {
      //解析配置文件得到Configuration类
      XMLConfigBuilder parser = new XMLConfigBuilder(reader, environment, properties);
      return build(parser.parse());
    } catch (Exception e) {
      throw ExceptionFactory.wrapException("Error building SqlSession.", e);
    } finally {
      ErrorContext.instance().reset();
      try {
        reader.close();
      } catch (IOException e) {
        // Intentionally ignore. Prefer previous error.
      }
    }
}
```
```text
public SqlSessionFactory build(Configuration config) {
    //将解析配置文件得到的配置类传入到DefaultSqlSessionFactory中并创建DefaultSqlSessionFactory实例
    return new DefaultSqlSessionFactory(config);
}
```

3.配置文件解析
```text
public Configuration parse() {
    if (parsed) {
      throw new BuilderException("Each XMLConfigBuilder can only be used once.");
    }
    parsed = true;
    //解析configuration标签
    parseConfiguration(parser.evalNode("/configuration"));
    return configuration;
  }
```
```text
private void parseConfiguration(XNode root) {//解析configuration节点下的所有子节点
    try {
      //issue #117 read properties first
      propertiesElement(root.evalNode("properties"));//解析properties标签
      Properties settings = settingsAsProperties(root.evalNode("settings"));//解析settings标签
      loadCustomVfs(settings);
      loadCustomLogImpl(settings);
      typeAliasesElement(root.evalNode("typeAliases"));//解析别名标签
      pluginElement(root.evalNode("plugins"));//解析插件标签
      objectFactoryElement(root.evalNode("objectFactory"));//解析工厂对象标签
      objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));//解析工厂包装对象标签
      reflectorFactoryElement(root.evalNode("reflectorFactory"));//解析反射工厂标签
      settingsElement(settings);//设置settings配置
      // read it after objectFactory and objectWrapperFactory issue #631
      environmentsElement(root.evalNode("environments"));//设置环境变量
      databaseIdProviderElement(root.evalNode("databaseIdProvider"));//设置数据库厂商标识
      typeHandlerElement(root.evalNode("typeHandlers"));//解析typeHandlers标签
      mapperElement(root.evalNode("mappers"));//解析mappers标签
    } catch (Exception e) {
      throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + e, e);
    }
}
```
```text
private void propertiesElement(XNode context) throws Exception {
    if (context != null) {
      Properties defaults = context.getChildrenAsProperties();
      String resource = context.getStringAttribute("resource");
      String url = context.getStringAttribute("url");
      if (resource != null && url != null) {
        throw new BuilderException("The properties element cannot specify both a URL and a resource based property file reference.  Please specify one or the other.");
      }
      //加载.properties文件
      if (resource != null) {
        defaults.putAll(Resources.getResourceAsProperties(resource));
      } else if (url != null) {
      //通过网络获取配置文件
        defaults.putAll(Resources.getUrlAsProperties(url));
      }
      Properties vars = configuration.getVariables();
      if (vars != null) {
        defaults.putAll(vars);
      }
      parser.setVariables(defaults);
      configuration.setVariables(defaults);
    }
}
```
